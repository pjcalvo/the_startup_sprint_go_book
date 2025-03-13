# Chapter 5: Enhancing the Experience (25-30 pages)
## Adding middleware (like logging) and better error handling.
### Story: Olivia steps in to help Javi through a tough debugging session.

Javi llegó a la oficina con una mezcla de orgullo y preocupación. Su API ya hablaba con ChatGPT y devolvía respuestas en JSON, pero el feedback de Pablo —"tarda demasiado" y esos errores 500— seguía rondándole la cabeza. Se sirvió un café, el tercero de la máquina como ritual, y abrió api.go en VS Code. "Necesito que sea más rápida y que no se rompa", pensó, decidido a mejorar la experiencia. Olivia, que ya estaba tecleando al otro lado de la sala, lo miró por encima de sus gafas. "¿Seguís peleando con esa API?". "Sí, pero hoy la hago indestructible", respondió Javi con una sonrisa forzada.

Antes de lanzarse al código, Javi repasó lo que sabía. La API funcionaba, pero era frágil: timeouts básicos no eran suficientes, y los errores de ChatGPT podían tumbarla. Recordó un comentario de Olivia sobre middleware para logging y pensó que era el momento de implementarlo. "Si puedo ver qué pasa en cada solicitud, voy a entender por qué falla", murmuró. Decidió empezar por ahí, añadiendo un middleware simple para registrar cada request.

Javi creó una función de middleware para envolver sus handlers:

```go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
    "time"
)

// Middleware para registrar cada solicitud
func loggingMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        // Capturamos el tiempo de inicio
        start := time.Now()
        // Llamamos al handler original
        next(w, r)
        // Calculamos la duración y registramos la solicitud
        duration := time.Since(start)
        fmt.Printf("Solicitud: %s %s - Duración: %v\n", r.Method, r.URL.Path, duration)
    }
}
```
Luego ajustó el main para usarlo:

```go
func main() {
    // Creamos un handler con el middleware de logging
    handler := loggingMiddleware(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        switch r.Method {
        case http.MethodGet:
            getMessages(w, r)
        case http.MethodPost:
            createMessage(w, r)
        default:
            http.Error(w, "Método no permitido", http.StatusMethodNotAllowed)
        }
    }))

    // Registramos la ruta con el handler envuelto
    http.Handle("/messages", handler)
    fmt.Println("Servidor corriendo en http://localhost:8080")
    http.ListenAndServe(":8080", nil)
}
```

Corrió go run api.go y probó con curl. La terminal mostró: Solicitud: POST /messages - Duración: 1.234s. "¡Ahora sé cuánto tarda!", pensó Javi, emocionado.

El logging era un buen comienzo, pero los errores 500 seguían siendo un problema. Javi decidió mejorar el handler createMessage para manejar fallos de ChatGPT con más gracia:

```go
// Handler para procesar mensajes con mejor manejo de errores
func createMessage(w http.ResponseWriter, r *http.Request) {
    var newMessage Message
    err := json.NewDecoder(r.Body).Decode(&newMessage)
    if err != nil {
        http.Error(w, "Error al procesar el mensaje: "+err.Error(), http.StatusBadRequest)
        return
    }

    apiKey := "sk-xxxxxxxxxxxxxxxxxxxxxxxx"
    url := "https://api.openai.com/v1/chat/completions"
    payloadStr := fmt.Sprintf(`{"model": "gpt-3.5-turbo", "messages": [{"role": "user", "content": "%s"}]}`, newMessage.Input)
    payload := strings.NewReader(payloadStr)

    client := &http.Client{Timeout: 10 * time.Second}
    req, err := http.NewRequest("POST", url, payload)
    if err != nil {
        http.Error(w, "Error creando solicitud a ChatGPT: "+err.Error(), http.StatusInternalServerError)
        return
    }
    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Authorization", "Bearer "+apiKey)

    resp, err := client.Do(req)
    if err != nil {
        http.Error(w, "Error conectando con ChatGPT: "+err.Error(), http.StatusGatewayTimeout)
        return
    }
    defer resp.Body.Close()

    // Verificamos el código de estado de ChatGPT
    if resp.StatusCode != http.StatusOK {
        body, _ := ioutil.ReadAll(resp.Body)
        http.Error(w, fmt.Sprintf("ChatGPT falló: %s", string(body)), resp.StatusCode)
        return
    }

    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        http.Error(w, "Error leyendo respuesta de ChatGPT: "+err.Error(), http.StatusInternalServerError)
        return
    }

    var chatResponse struct {
        Choices []struct {
            Message struct {
                Content string `json:"content"`
            } `json:"message"`
        } `json:"choices"`
    }
    err = json.Unmarshal(body, &chatResponse)
    if err != nil || len(chatResponse.Choices) == 0 {
        http.Error(w, "Respuesta inválida de ChatGPT: "+string(body), http.StatusInternalServerError)
        return
    }

    newMessage.Response = chatResponse.Choices[0].Message.Content
    newMessage.ID = len(messages) + 1
    messages = append(messages, newMessage)

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(newMessage)
}
```

Javi probó con un mensaje raro y vio errores más claros en lugar de un 500 genérico. "Esto ya no se rompe tan fácil", pensó.

Esa tarde, la API empezó a fallar otra vez. Pablo envió un mensaje: "Me dio un 504 en tres intentos seguidos". Javi revisó los logs: Solicitud: POST /messages - Duración: 10.001s. "Se está pasando del timeout", gruñó. Olivia, oyendo su frustración, dejó su teclado y se acercó. "Vamos a depurarlo juntos". Sentados frente a la pantalla, vieron que ChatGPT tardaba demasiado en responder a preguntas largas. "El timeout de 10 segundos no basta", dijo Olivia. "Y tu cliente no está manejando bien las desconexiones".

Olivia sugirió un enfoque más robusto:


```go
// Handler con reintentos y timeout ajustado
func createMessage(w http.ResponseWriter, r *http.Request) {
    var newMessage Message
    err := json.NewDecoder(r.Body).Decode(&newMessage)
    if err != nil {
        http.Error(w, "Error al procesar el mensaje: "+err.Error(), http.StatusBadRequest)
        return
    }

    apiKey := "sk-xxxxxxxxxxxxxxxxxxxxxxxx"
    url := "https://api.openai.com/v1/chat/completions"
    payloadStr := fmt.Sprintf(`{"model": "gpt-3.5-turbo", "messages": [{"role": "user", "content": "%s"}]}`, newMessage.Input)
    payload := strings.NewReader(payloadStr)

    // Creamos un cliente con un timeout más largo
    client := &http.Client{Timeout: 15 * time.Second}
    req, err := http.NewRequest("POST", url, payload)
    if err != nil {
        http.Error(w, "Error creando solicitud a ChatGPT: "+err.Error(), http.StatusInternalServerError)
        return
    }
    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Authorization", "Bearer "+apiKey)

    // Intentamos la solicitud hasta 3 veces
    maxRetries := 3
    for i := 0; i < maxRetries; i++ {
        resp, err := client.Do(req)
        if err == nil && resp.StatusCode == http.StatusOK {
            defer resp.Body.Close()
            body, err := ioutil.ReadAll(resp.Body)
            if err != nil {
                http.Error(w, "Error leyendo respuesta de ChatGPT: "+err.Error(), http.StatusInternalServerError)
                return
            }

            var chatResponse struct {
                Choices []struct {
                    Message struct {
                        Content string `json:"content"`
                    } `json:"message"`
                } `json:"choices"`
            }
            err = json.Unmarshal(body, &chatResponse)
            if err == nil && len(chatResponse.Choices) > 0 {
                newMessage.Response = chatResponse.Choices[0].Message.Content
                newMessage.ID = len(messages) + 1
                messages = append(messages, newMessage)
                w.Header().Set("Content-Type", "application/json")
                json.NewEncoder(w).Encode(newMessage)
                return
            }
        }
        // Si falla, esperamos un poco antes de reintentar
        if i < maxRetries-1 {
            time.Sleep(2 * time.Second)
        }
    }
    http.Error(w, "No se pudo conectar con ChatGPT tras varios intentos", http.StatusServiceUnavailable)
}
```

Javi probó con una pregunta larga y la API resistió, reintentando hasta que ChatGPT respondió. "¡Esto sí que es resiliencia!", exclamó.

Con el nuevo código corriendo, Javi y Olivia probaron varias solicitudes. Los logs mostraban tiempos más estables, y los errores 504 desaparecieron. "Los reintentos salvan el día", dijo Olivia, dándole una palmada en el hombro. "Pero no abuses —ChatGPT no es perfecto". Javi asintió, agradecido por la sesión. Había pasado de una API tambaleante a una que podía soportar fallos, y el logging le daba pistas claras de lo que pasaba. Pablo volvió a probar y envió un "¡Funciona genial ahora!" que hizo sonreír a Javi.

Al cerrar su Mac, Javi sintió que había dado un gran salto. El middleware de logging le mostraba el pulso de su API, y los reintentos la hacían más fuerte. Pero la sesión con Olivia le enseñó algo más: no estaba solo en esto. "Mañana la pruebo a fondo", pensó, garabateando "testeo" en su cuaderno. Olivia le lanzó un "Buen trabajo" desde la puerta. Con la API más robusta que nunca, Javi sabía que estaba listo para el siguiente desafío.