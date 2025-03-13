# Chapter 6: Testing and Debugging (20-25 pages)
## Writing tests with testing and httptest.
## Debugging tricky issues.
### Story: Javi finds a critical bug just before a demo.

Javi llegó a la oficina con una sensación de calma engañosa. Su API estaba robusta gracias al logging y los reintentos, pero algo en su interior le decía que no debía confiarse demasiado. Mientras se servía su café matutino, Olivia lo saludó desde su escritorio. "Emma quiere una demo mañana", dijo con un tono que no admitía excusas. Javi casi deja caer la taza. "¿Mañana? Necesito probar esto a fondo primero". Abrió VS Code con un nudo en el estómago, decidido a escribir pruebas y cazar cualquier problema antes de que la demo lo pusiera en evidencia.

Javi sabía que confiar solo en curl no era suficiente. Había oído hablar del paquete testing de Go y de httptest para simular solicitudes HTTP, así que decidió usarlos. "Si voy a mostrar esto, tiene que ser a prueba de balas", pensó. Creó un nuevo archivo api_test.go junto a api.go y empezó a investigar cómo funcionaban las pruebas en Go. Olivia, que lo vio tecleando con furia, se acercó. "¿Tests unitarios?". "Sí, y rápido", respondió Javi. Ella sonrió. "Vas por buen camino".

Javi comenzó con una prueba simple para el endpoint GET:

```go
package main

import (
    "net/http"
    "net/http/httptest"
    "testing"
)

// Prueba para el endpoint GET /messages
func TestGetMessages(t *testing.T) {
    // Creamos un request simulado para GET /messages
    req, err := http.NewRequest("GET", "/messages", nil)
    if err != nil {
        t.Fatal("Error creando request:", err)
    }

    // Creamos un ResponseRecorder para capturar la respuesta
    rr := httptest.NewRecorder()
    // Llamamos al handler directamente con el request simulado
    handler := http.HandlerFunc(getMessages)
    handler.ServeHTTP(rr, req)

    // Verificamos que el código de estado sea 200 OK
    if status := rr.Code; status != http.StatusOK {
        t.Errorf("Código de estado incorrecto: obtuvo %v, esperaba %v", status, http.StatusOK)
    }

    // Verificamos que el Content-Type sea application/json
    if contentType := rr.Header().Get("Content-Type"); contentType != "application/json" {
        t.Errorf("Content-Type incorrecto: obtuvo %v, esperaba %v", contentType, "application/json")
    }

    // Verificamos que el cuerpo contenga los mensajes iniciales
    expected := `[{"id":1,"input":"Hola","response":"¡Hola! ¿En qué te ayudo?"}]`
    if rr.Body.String() != expected {
        t.Errorf("Respuesta incorrecta: obtuvo %v, esperaba %v", rr.Body.String(), expected)
    }
}
```
Corrió go test y la terminal dijo: PASS. "¡Funciona!", pensó Javi, sintiendo un pequeño subidón. Pero sabía que el POST sería más complicado.

Javi escribió una prueba para createMessage, simulando una solicitud a ChatGPT:

```go
// Prueba para el endpoint POST /messages
func TestCreateMessage(t *testing.T) {
    // Creamos un cuerpo JSON simulado para el request
    payload := strings.NewReader(`{"input":"¿Qué hora es?"}`)
    req, err := http.NewRequest("POST", "/messages", payload)
    if err != nil {
        t.Fatal("Error creando request:", err)
    }
    req.Header.Set("Content-Type", "application/json")

    // Simulamos la respuesta con un ResponseRecorder
    rr := httptest.NewRecorder()
    handler := http.HandlerFunc(createMessage)
    handler.ServeHTTP(rr, req)

    // Verificamos que el código de estado sea 200 OK
    if status := rr.Code; status != http.StatusOK {
        t.Errorf("Código de estado incorrecto: obtuvo %v, esperaba %v", status, http.StatusOK)
    }

    // Verificamos que el cuerpo contenga un ID y una respuesta
    var msg Message
    err = json.NewDecoder(rr.Body).Decode(&msg)
    if err != nil || msg.ID == 0 || msg.Response == "" {
        t.Errorf("Respuesta inválida: obtuvo %v", rr.Body.String())
    }
}
```
Ejecutó go test otra vez, pero esta vez falló: dial tcp: i/o timeout. "Claro, está intentando conectar con ChatGPT de verdad", gruñó Javi. Necesitaba simular esa parte.

El reloj marcaba las 3 de la tarde cuando Javi golpeó la mesa con frustración. "¡Esto no funciona en las pruebas!". Olivia se acercó, calmada pero curiosa. "¿Qué pasa?". "Las pruebas fallan porque llaman al API real", explicó Javi. Olivia asintió. "Necesitás un mock. Vamos a depurarlo". Juntos, refactorizaron createMessage para que aceptara una función de cliente como dependencia:

```go
// Cliente HTTP como interfaz para facilitar mocking
type chatClient func(req *http.Request) (*http.Response, error)

// Handler con cliente inyectado
func createMessageWithClient(w http.ResponseWriter, r *http.Request, client chatClient) {
    var newMessage Message
    err := json.NewDecoder(r.Body).Decode(&newMessage)
    if err != nil {
        http.Error(w, "Error al procesar el mensaje: "+err.Error(), http.StatusBadRequest)
        return
    }

    payloadStr := fmt.Sprintf(`{"model": "gpt-3.5-turbo", "messages": [{"role": "user", "content": "%s"}]}`, newMessage.Input)
    payload := strings.NewReader(payloadStr)
    req, err := http.NewRequest("POST", "https://api.openai.com/v1/chat/completions", payload)
    if err != nil {
        http.Error(w, "Error creando solicitud: "+err.Error(), http.StatusInternalServerError)
        return
    }
    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Authorization", "Bearer sk-xxxxxxxxxxxxxxxxxxxxxxxx")

    resp, err := client(req)
    if err != nil {
        http.Error(w, "Error conectando con ChatGPT: "+err.Error(), http.StatusGatewayTimeout)
        return
    }
    defer resp.Body.Close()
    // ... (resto del manejo de respuesta igual)
}
```
Luego, en la prueba, usaron un mock:

```go
func TestCreateMessageWithMock(t *testing.T) {
    payload := strings.NewReader(`{"input":"¿Qué hora es?"}`)
    req, err := http.NewRequest("POST", "/messages", payload)
    if err != nil {
        t.Fatal("Error creando request:", err)
    }
    req.Header.Set("Content-Type", "application/json")

    // Mock que simula una respuesta de ChatGPT
    mockClient := func(req *http.Request) (*http.Response, error) {
        return &http.Response{
            StatusCode: http.StatusOK,
            Body:       ioutil.NopCloser(strings.NewReader(`{"choices":[{"message":{"content":"Son las 3 PM"}}]}`)),
        }, nil
    }

    rr := httptest.NewRecorder()
    handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        createMessageWithClient(w, r, mockClient)
    })
    handler.ServeHTTP(rr, req)

    if status := rr.Code; status != http.StatusOK {
        t.Errorf("Código de estado incorrecto: obtuvo %v, esperaba %v", status, http.StatusOK)
    }
}
```
Esta vez, go test pasó. "¡Genial, Olivia!", exclamó Javi.

```go
// Validación antes de procesar
if strings.TrimSpace(newMessage.Input) == "" || len(newMessage.Input) > 1000 {
    http.Error(w, "Mensaje inválido o demasiado largo", http.StatusBadRequest)
    return
}
```

Corrieron la prueba otra vez y pasó. "Salvado por un pelo", suspiró Javi.