# Chapter 8: Restructuring the Project (15-20 pages)
## Organizing code into packages and adding in-memory persistence.
### Story: Javi realizes his prototype needs a professional touch.

Javi llegó a la oficina con una mezcla de orgullo y un leve cosquilleo de incomodidad. La API estaba en producción, la presentación había sido un éxito, pero algo no le cuadraba. Mientras se servía su café matutino —el tercero de la máquina, su ritual infalible—, Olivia lo saludó desde su escritorio. "¿Todo bien, genio?". "Sí, pero mi código parece un experimento de laboratorio", confesó Javi, abriendo VS Code. Había funcionado hasta ahora, pero esa variable global messages y el archivo único gritaban "prototipo". Era hora de darle una estructura profesional y un almacenamiento decente.

Javi sabía que los proyectos reales en Go usaban paquetes para separar responsabilidades. "Si esto va a crecer, necesito orden", pensó. Inicializó un módulo con go mod init mi-api y reestructuró su código. Creó una carpeta models para las definiciones de datos, storage para la persistencia, handlers para la lógica HTTP, y cmd/api para el punto de entrada. Primero, definió el modelo en models/message.go:

```go
package models

// Message representa un mensaje con entrada y respuesta
type Message struct {
    ID       int    `json:"id"`
    Input    string `json:"input"`
    Response string `json:"response"`
}
```

"Esto es como la base de mi edificio", pensó Javi, sintiendo que las piezas empezaban a encajar.

La lista global messages había sido útil, pero no era segura ni escalable. Javi decidió simular una base de datos en memoria con un mapa protegido por un sync.RWMutex para manejar accesos concurrentes. Creó storage/memory.go:

```go
package storage

import (
    "sync"
    "mi-api/models"
)

// MemoryStore es un almacén en memoria con concurrencia segura
type MemoryStore struct {
    messages map[int]models.Message
    mu       sync.RWMutex // Mutex para proteger el acceso al mapa
    nextID   int
}

// NewMemoryStore crea un nuevo almacén con datos iniciales
func NewMemoryStore() *MemoryStore {
    return &MemoryStore{
        messages: map[int]models.Message{1: {ID: 1, Input: "Hola", Response: "¡Hola! ¿En qué te ayudo?"}},
        nextID:   2,
    }
}

// AddMessage añade un mensaje y devuelve su ID
func (s *MemoryStore) AddMessage(msg models.Message) int {
    s.mu.Lock() // Bloqueamos para escritura
    defer s.mu.Unlock()
    msg.ID = s.nextID
    s.messages[msg.ID] = msg
    s.nextID++
    return msg.ID
}

// GetMessages devuelve todos los mensajes
func (s *MemoryStore) GetMessages() []models.Message {
    s.mu.RLock() // Bloqueamos solo para lectura
    defer s.mu.RUnlock()
    var result []models.Message
    for _, msg := range s.messages {
        result = append(result, msg)
    }
    return result
}
```

Javi probó añadir un mensaje manualmente y vio que funcionaba. "Esto es persistencia de verdad", pensó, aunque sabía que una base de datos real vendría después.

Con el modelo y el almacenamiento listos, Javi trasladó la lógica HTTP a handlers/messages.go:

```go
package handlers

import (
    "encoding/json"
    "net/http"
    "mi-api/models"
    "mi-api/storage"
)

// MessageHandler maneja las solicitudes HTTP
type MessageHandler struct {
    store *storage.MemoryStore
}

// NewMessageHandler crea un nuevo handler con el almacén
func NewMessageHandler(store *storage.MemoryStore) *MessageHandler {
    return &MessageHandler{store: store}
}

// GetMessages devuelve todos los mensajes (GET)
func (h *MessageHandler) GetMessages(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(h.store.GetMessages())
}

// CreateMessage crea un nuevo mensaje (POST)
func (h *MessageHandler) CreateMessage(w http.ResponseWriter, r *http.Request) {
    var newMessage models.Message
    err := json.NewDecoder(r.Body).Decode(&newMessage)
    if err != nil {
        http.Error(w, "Error al procesar el mensaje: "+err.Error(), http.StatusBadRequest)
        return
    }
    // Aquí iría la llamada a ChatGPT, pero por ahora simulamos
    newMessage.Response = "Respuesta simulada"
    h.store.AddMessage(newMessage)

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(newMessage)
}
```

Javi simplificó el POST por ahora, dejando la integración con ChatGPT intacta pero fuera del ejemplo para enfocarse en la estructura.

Finalmente, Javi actualizó cmd/api/main.go para conectar los paquetes:

```go
package main

import (
    "fmt"
    "net/http"
    "mi-api/handlers"
    "mi-api/storage"
)

func main() {
    store := storage.NewMemoryStore()
    handler := handlers.NewMessageHandler(store)

    http.HandleFunc("/messages", func(w http.ResponseWriter, r *http.Request) {
        switch r.Method {
        case http.MethodGet:
            handler.GetMessages(w, r)
        case http.MethodPost:
            handler.CreateMessage(w, r)
        default:
            http.Error(w, "Método no permitido", http.StatusMethodNotAllowed)
        }
    })

    fmt.Println("Servidor corriendo en http://localhost:8080")
    http.ListenAndServe(":8080", nil)
}
```

Corrió go run `cmd/api/main.go` y probó con curl. La API respondió como antes, pero ahora estaba organizada. "¡Esto sí parece software serio!", exclamó.

Javi se recostó en su silla, mirando la nueva estructura en VS Code. Hace unas semanas, todo era un solo archivo desordenado; ahora tenía paquetes, un almacén en memoria, y una base sólida. Recordó su primer "Hola, mundo" y cómo había tropezado hasta llegar aquí. Olivia se acercó. "¿Contento con tu remodelación?". "Sí", dijo Javi, "pero siento que puede ser más rápida". Ella sonrió. "Eso suena a concurrencia. ¿Listo para el próximo nivel?". Javi asintió, garabateando "paquetes: check" en su cuaderno.

Con la API reestructurada, Javi sabía que había dado un gran paso. "Esto es como pasar de una maqueta a un edificio", pensó. El almacén en memoria era un comienzo, pero ya imaginaba bases de datos reales y quizás algo más rápido con Go. Cerró su Mac, dejando el cuaderno abierto. Su viaje había sido un caos ordenado, y ahora estaba listo para explorar lo que venía: la magia de la concurrencia.