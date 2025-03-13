Chapter 4: Processing Inputs and Outputs (25-30 pages)
Accepting user messages, returning JSON responses.
Working with structs and handling data.
Story: First user feedback — and the first signs of trouble.

Javi llegó a la oficina con una energía renovada, el eco de su API RESTful todavía resonando en su cabeza. Había construido un servidor que podía manejar mensajes, pero ahora venía el verdadero desafío: conectar esa API con ChatGPT para que los usuarios pudieran enviar mensajes y recibir respuestas inteligentes. Mientras se servía su café matutino, Olivia lo saludó desde su escritorio. "¿Listo para hacerla hablar de verdad?". "Más que listo", respondió Javi, abriendo api.go en VS Code. Era hora de procesar entradas de usuarios y devolver salidas en JSON, usando todo lo que había aprendido sobre structs y datos.

Javi sabía que necesitaba dos cosas: aceptar un mensaje del usuario en el cuerpo de una solicitud POST y enviar ese mensaje a ChatGPT para obtener una respuesta. Recordó su éxito del Capítulo 2, donde había hecho una solicitud POST básica a ChatGPT, y decidió integrarlo en su API. Primero, ajustó la estructura Message para que reflejara mejor lo que quería: un mensaje de entrada y una respuesta. Escribió:

```go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
)

// Definimos una estructura para manejar mensajes de entrada y salida
type Message struct {
    ID       int    `json:"id"`       // ID único del mensaje
    Input    string `json:"input"`    // Mensaje enviado por el usuario
    Response string `json:"response"` // Respuesta generada por ChatGPT
}

// Simulamos una lista de mensajes en memoria
var messages = []Message{
    {ID: 1, Input: "Hola", Response: "¡Hola! ¿En qué te ayudo?"},
}
```

"Esto es como un formulario con espacio para la respuesta", pensó Javi, satisfecho con cómo los structs organizaban todo.

Luego, Javi reescribió el handler POST para aceptar un mensaje del usuario y enviarlo a ChatGPT. Usaría net/http y el código del Capítulo 2 como base:

```go
// Handler para procesar mensajes y conectar con ChatGPT
func createMessage(w http.ResponseWriter, r *http.Request) {
    // Creamos una variable para el mensaje entrante
    var newMessage Message
    // Decodificamos el cuerpo JSON de la solicitud
    err := json.NewDecoder(r.Body).Decode(&newMessage)
    if err != nil {
        http.Error(w, "Error al procesar el mensaje", http.StatusBadRequest)
        return
    }

    // API key de ChatGPT (mejor en variables de entorno en producción)
    apiKey := "sk-xxxxxxxxxxxxxxxxxxxxxxxx"
    url := "https://api.openai.com/v1/chat/completions"

    // Creamos el payload JSON para ChatGPT con el input del usuario
    payloadStr := fmt.Sprintf(`{
        "model": "gpt-3.5-turbo",
        "messages": [{"role": "user", "content": "%s"}]
    }`, newMessage.Input)
    payload := strings.NewReader(payloadStr)

    // Creamos una solicitud POST para ChatGPT
    client := &http.Client{}
    req, err := http.NewRequest("POST", url, payload)
    if err != nil {
        http.Error(w, "Error creando la solicitud a ChatGPT", http.StatusInternalServerError)
        return
    }
    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Authorization", "Bearer "+apiKey)

    // Enviamos la solicitud y obtenemos la respuesta
    resp, err := client.Do(req)
    if err != nil {
        http.Error(w, "Error conectando con ChatGPT", http.StatusInternalServerError)
        return
    }
    defer resp.Body.Close()

    // Leemos la respuesta de ChatGPT
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        http.Error(w, "Error leyendo la respuesta de ChatGPT", http.StatusInternalServerError)
        return
    }

    // Parseamos el JSON de ChatGPT para extraer el mensaje
    var chatResponse struct {
        Choices []struct {
            Message struct {
                Content string `json:"content"`
            } `json:"message"`
        } `json:"choices"`
    }
    json.Unmarshal(body, &chatResponse)
    newMessage.Response = chatResponse.Choices[0].Message.Content
    newMessage.ID = len(messages) + 1
    messages = append(messages, newMessage)

    // Enviamos la respuesta al usuario como JSON
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(newMessage)
}
```

Javi actualizó el main para usar este handler (dejando el GET como estaba) y ejecutó go run api.go.

Con el servidor corriendo, Javi probó su API:

```bash
curl -X POST http://localhost:8080/messages \
-H "Content-Type: application/json" \
-d '{"input":"¿Qué día es hoy?"}'
La respuesta fue: {"id":2,"input":"¿Qué día es hoy?","response":"Hoy es 13 de marzo de 2025."}. "¡Funciona!", exclamó Javi, casi derramando su café. ChatGPT estaba respondiendo a través de su API, y el JSON salía perfecto. Olivia se acercó, curiosa. "Probá con algo más raro", sugirió. Javi envió:
```

```bash
curl -X POST http://localhost:8080/messages \
-H "Content-Type: application/json" \
-d '{"input":"¿Por qué los pájaros cantan?"}'
Y obtuvo: {"id":3,"input":"¿Por qué los pájaros cantan?","response":"Los pájaros cantan para comunicarse, atraer parejas y marcar territorio."}. "Esto es magia", pensó Javi.
```

El primer feedback llegó esa tarde. Emma entró con un colega del equipo de soporte, Pablo, que había probado la API. "Funciona bien", dijo Pablo, "pero a veces tarda demasiado en responder, y una vez me dio un error 500". Javi frunció el ceño. "¿Error 500? Eso es un problema del servidor". Revisó los logs improvisados que había añadido con fmt.Println y vio que una solicitud había fallado porque ChatGPT devolvió un JSON mal formado. "El parsing está frágil", murmuró. Olivia, que escuchaba, comentó: "Necesitás manejar mejor los errores y tal vez un timeout". Javi asintió, sintiendo el primer pinchazo de realidad: su API no era perfecta.

Javi decidió robustecer el handler. Añadió un chequeo básico para el JSON de ChatGPT y un timeout:

```go
// Handler mejorado para procesar mensajes
func createMessage(w http.ResponseWriter, r *http.Request) {
    var newMessage Message
    err := json.NewDecoder(r.Body).Decode(&newMessage)
    if err != nil {
        http.Error(w, "Error al procesar el mensaje", http.StatusBadRequest)
        return
    }

    apiKey := "sk-xxxxxxxxxxxxxxxxxxxxxxxx"
    url := "https://api.openai.com/v1/chat/completions"
    payloadStr := fmt.Sprintf(`{
        "model": "gpt-3.5-turbo",
        "messages": [{"role": "user", "content": "%s"}]
    }`, newMessage.Input)
    payload := strings.NewReader(payloadStr)

    // Creamos un cliente con un timeout de 10 segundos
    client := &http.Client{Timeout: 10 * time.Second}
    req, err := http.NewRequest("POST", url, payload)
    if err != nil {
        http.Error(w, "Error creando la solicitud a ChatGPT", http.StatusInternalServerError)
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

    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        http.Error(w, "Error leyendo la respuesta de ChatGPT", http.StatusInternalServerError)
        return
    }

    // Parseamos el JSON con un chequeo básico
    var chatResponse struct {
        Choices []struct {
            Message struct {
                Content string `json:"content"`
            } `json:"message"`
        } `json:"choices"`
    }
    err = json.Unmarshal(body, &chatResponse)
    if err != nil || len(chatResponse.Choices) == 0 {
        http.Error(w, "Respuesta inválida de ChatGPT", http.StatusInternalServerError)
        return
    }
    newMessage.Response = chatResponse.Choices[0].Message.Content
    newMessage.ID = len(messages) + 1
    messages = append(messages, newMessage)

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(newMessage)
}
```

Probó de nuevo y la API resistió mejor, pero sabía que aún había trabajo por hacer.

Al final del día, Javi reflexionó sobre su progreso. Había conectado su API con ChatGPT, aceptaba mensajes de usuarios y devolvía respuestas en JSON, pero el feedback de Pablo le recordaba que no todo era color de rosa. "Tarda demasiado a veces", resonaba en su cabeza. Olivia se acercó antes de irse. "Los timeouts ayudan, pero pensá en caching o algo para que sea más rápido". Javi garabateó eso en su cuaderno. Su API estaba viva, pero los primeros signos de trouble le decían que necesitaba mejorarla. "Mañana la hago más fuerte", pensó, cerrando su Mac con una mezcla de orgullo y desafío.