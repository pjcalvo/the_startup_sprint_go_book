# Chapter 9: Mastering Concurrency (15-20 pages)
## Exploring concurrency with goroutines and channels.
### Story: Javi optimizes his API and looks to the horizon.

Javi llegó a la oficina con una energía renovada, el café matutino —el tercero de la máquina— calentándole las manos. La API estaba bien estructurada gracias a su reorganización, pero una conversación con Olivia seguía resonando: "Si querés que sea más rápida, la concurrencia es el camino". Abrió VS Code, listo para sumergirse en las goroutines y canales de Go. "Esto es como darle turbo a mi proyecto", pensó, imaginando múltiples solicitudes a ChatGPT corriendo al mismo tiempo. Olivia lo miró desde su escritorio. "¿Listo para el próximo nivel?". "Absolutamente", respondió Javi, con una sonrisa decidida.

Antes de tocar el código, Javi quiso entender qué hacía especial a la concurrencia en Go. No era solo hacer cosas al mismo tiempo (paralelismo), sino gestionar tareas de forma eficiente, incluso en un solo núcleo. Las goroutines eran hilos ligeros —miles podían correr sin agotar la memoria—, y los canales permitían comunicación segura entre ellas. "Es como un equipo de mensajeros coordinados", pensó, recordando cómo otros lenguajes complicaban esto con threads pesados. Go lo simplificaba con su lema: "No compartas memoria para comunicarte; comunica para compartir memoria". Para más, revisó Effective Go.

Javi decidió optimizar el handler CreateMessage para usar goroutines y procesar la solicitud a ChatGPT en paralelo. Actualizó handlers/messages.go:

```go
package handlers

import (
    "encoding/json"
    "fmt"
    "net/http"
    "strings"
    "sync"
    "mi-api/models"
    "mi-api/storage"
)

// MessageHandler maneja las solicitudes HTTP
type MessageHandler struct {
    store *storage.MemoryStore
}

// NewMessageHandler crea un nuevo handler
func NewMessageHandler(store *storage.MemoryStore) *MessageHandler {
    return &MessageHandler{store: store}
}

// CreateMessage procesa mensajes con goroutines
func (h *MessageHandler) CreateMessage(w http.ResponseWriter, r *http.Request) {
    var newMessage models.Message
    err := json.NewDecoder(r.Body).Decode(&newMessage)
    if err != nil {
        http.Error(w, "Error al procesar el mensaje: "+err.Error(), http.StatusBadRequest)
        return
    }

    // Creamos un WaitGroup para esperar a la goroutine
    var wg sync.WaitGroup
    // Canal para recibir la respuesta de ChatGPT
    responseChan := make(chan string, 1)

    wg.Add(1)
    // Lanzamos una goroutine para la solicitud a ChatGPT
    go func(input string) {
        defer wg.Done()
        apiKey := "sk-xxxxxxxxxxxxxxxxxxxxxxxx" // Mejor en env vars
        url := "https://api.openai.com/v1/chat/completions"
        payloadStr := fmt.Sprintf(`{"model": "gpt-3.5-turbo", "messages": [{"role": "user", "content": "%s"}]}`, input)
        payload := strings.NewReader(payloadStr)

        req, _ := http.NewRequest("POST", url, payload)
        req.Header.Set("Content-Type", "application/json")
        req.Header.Set("Authorization", "Bearer "+apiKey)

        client := &http.Client{Timeout: 15 * time.Second}
        resp, err := client.Do(req)
        if err != nil {
            responseChan <- "Error al conectar con ChatGPT"
            return
        }
        defer resp.Body.Close()

        body, _ := ioutil.ReadAll(resp.Body)
        var chatResponse struct {
            Choices []struct {
                Message struct {
                    Content string `json:"content"`
                } `json:"message"`
            } `json:"choices"`
        }
        json.Unmarshal(body, &chatResponse)
        if len(chatResponse.Choices) > 0 {
            responseChan <- chatResponse.Choices[0].Message.Content
        } else {
            responseChan <- "Sin respuesta válida"
        }
    }(newMessage.Input)

    // Esperamos la respuesta y la almacenamos
    wg.Wait()
    newMessage.Response = <-responseChan
    h.store.AddMessage(newMessage)

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(newMessage)
}
```

Javi probó con curl y notó que la goroutine liberaba el hilo principal, haciendo la API más ágil. "Esto es solo el comienzo", pensó.

¿Por qué detenerse en una? Javi imaginó procesar varios mensajes a la vez. Creó un endpoint /batch para aceptar múltiples entradas:

```go
// BatchCreateMessages procesa múltiples mensajes en paralelo
func (h *MessageHandler) BatchCreateMessages(w http.ResponseWriter, r *http.Request) {
    var inputs []struct {
        Input string `json:"input"`
    }
    err := json.NewDecoder(r.Body).Decode(&inputs)
    if err != nil {
        http.Error(w, "Error al procesar los mensajes: "+err.Error(), http.StatusBadRequest)
        return
    }

    // Creamos un WaitGroup y un canal para las respuestas
    var wg sync.WaitGroup
    responseChan := make(chan models.Message, len(inputs))
    results := make([]models.Message, len(inputs))

    // Lanzamos una goroutine por cada mensaje
    for i, item := range inputs {
        wg.Add(1)
        go func(idx int, input string) {
            defer wg.Done()
            apiKey := "sk-xxxxxxxxxxxxxxxxxxxxxxxx"
            url := "https://api.openai.com/v1/chat/completions"
            payloadStr := fmt.Sprintf(`{"model": "gpt-3.5-turbo", "messages": [{"role": "user", "content": "%s"}]}`, input)
            payload := strings.NewReader(payloadStr)

            req, _ := http.NewRequest("POST", url, payload)
            req.Header.Set("Content-Type", "application/json")
            req.Header.Set("Authorization", "Bearer "+apiKey)

            client := &http.Client{Timeout: 15 * time.Second}
            resp, err := client.Do(req)
            if err != nil {
                responseChan <- models.Message{Input: input, Response: "Error"}
                return
            }
            defer resp.Body.Close()

            body, _ := ioutil.ReadAll(resp.Body)
            var chatResponse struct {
                Choices []struct {
                    Message struct {
                        Content string `json:"content"`
                    } `json:"message"`
                } `json:"choices"`
            }
            json.Unmarshal(body, &chatResponse)
            responseChan <- models.Message{Input: input, Response: chatResponse.Choices[0].Message.Content}
        }(i, item.Input)
    }

    // Recolectamos las respuestas en orden
    go func() {
        wg.Wait()
        close(responseChan)
    }()
    for msg := range responseChan {
        id := h.store.AddMessage(msg)
        results[id-1] = h.store.GetMessages()[id-1]
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(results)
}
```

Actualizó `main.go` para incluir la nueva ruta:

```go
http.HandleFunc("/batch", func(w http.ResponseWriter, r *http.Request) {
    if r.Method == http.MethodPost {
        handler.BatchCreateMessages(w, r)
    } else {
        http.Error(w, "Método no permitido", http.StatusMethodNotAllowed)
    }
})
```

Probó con `curl -X POST -d '[{"input":"Hola"},{"input":"Adiós"}]'` y vio respuestas paralelas. "¡Esto es potencia!", exclamó.

Javi se recostó en su silla, mirando la terminal con las respuestas múltiples. Había pasado de un solo archivo a una API estructurada, y ahora dominaba la concurrencia. Recordó su primer "Hola, mundo" y cómo cada paso —con Olivia a su lado— lo había traído aquí. "Go hace que lo complicado parezca fácil", pensó, garabateando "concurrencia: check" en su cuaderno. Emma pasó y asintió. "Esto está volando, Javi". Él sonrió, sabiendo que había desbloqueado algo grande.

Mirando al horizonte

Olivia se acercó antes de irse. "¿Qué más hay en ese cuaderno?". "Caching, bases de datos, más goroutines", dijo Javi. La concurrencia le había mostrado el alma de Go: eficiencia sin caos. Cerró su Mac, dejando el cuaderno abierto en una página nueva. Su viaje había sido una escalada, y el horizonte prometía más aventuras con Go y ChatGPT.