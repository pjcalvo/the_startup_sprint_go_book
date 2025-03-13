# Chapter 3: Designing the API (25-30 pages)
## Structuring a RESTful API in Go.
### Using net/http, creating routes.
### Story: Emma raises the stakes — they need a working prototype fast.

Javi se sentó frente a su Mac con una determinación que casi podía tocarse. Después de hacer que ChatGPT respondiera con un "Hola, ¿qué tal?" el día anterior, sabía que era hora de dar el siguiente paso: construir una API propia, algo sólido y RESTful. Mientras el café matutino calentaba sus manos, pensó en lo lejos que había llegado desde su primer "Hola, mundo". Olivia pasó por su escritorio con su taza en la mano. "Mantén la stack simple y escalable, Javi", le había dicho una vez, y esas palabras se le habían quedado grabadas. Abrió VS Code, creó un archivo api.go, y se lanzó a la aventura.

Antes de escribir código, Javi quiso aclarar qué demonios era eso de "RESTful". Recordó una charla con Olivia: "Una API RESTful sigue los principios de REST —Representational State Transfer—. Es stateless, usa métodos HTTP como GET, POST, PUT y DELETE, y organiza los recursos con URLs claras". Javi lo imaginó como un restaurante: cada endpoint era una mesa con un menú específico. Haces un pedido (request), te traen el plato (response), y lo que pediste antes no cambia lo siguiente —todo independiente. "Simple, pero elegante", pensó. Decidió empezar con una API para manejar mensajes, algo que pudiera crecer después.

Javi comenzó definiendo una estructura para los mensajes, usando lo que había aprendido sobre structs. Tecleó:

```go
import (
    "encoding/json" // Para manejar JSON en las respuestas
    "fmt"          // Para imprimir mensajes en la consola
    "net/http"     // Para crear el servidor y manejar solicitudes HTTP
)

// Definimos una estructura para un mensaje
type Message struct {
    ID      int    `json:"id"`      // ID único del mensaje, mapeado a JSON
    Content string `json:"content"` // Contenido del mensaje, mapeado a JSON
}

// Simulamos una base de datos en memoria con una lista inicial
var messages = []Message{
    {ID: 1, Content: "¡Hola, mundo!"}, // Mensaje inicial para probar
}
```

"Esto es como mi caja de herramientas", pensó Javi, recordando el Capítulo 1.5. La etiqueta json:"..." le sonaba de algo que Olivia había mencionado —sería clave para conectar con ChatGPT más adelante.

Párrafo 4:

Luego escribió los handlers para manejar las solicitudes. Primero, uno para GET:

```go
// Handler para obtener todos los mensajes (GET)
func getMessages(w http.ResponseWriter, r *http.Request) {
    // Establecemos el encabezado para que la respuesta sea JSON
    w.Header().Set("Content-Type", "application/json")
    // Codificamos la lista de mensajes como JSON y la enviamos
    json.NewEncoder(w).Encode(messages)
}
```
Y otro para POST:

```go
// Handler para crear un nuevo mensaje (POST)
func createMessage(w http.ResponseWriter, r *http.Request) {
    // Creamos una variable para almacenar el nuevo mensaje
    var newMessage Message
    // Decodificamos el cuerpo JSON de la solicitud en la estructura
    err := json.NewDecoder(r.Body).Decode(&newMessage)
    if err != nil {
        // Si hay error en el JSON, devolvemos un 400 Bad Request
        http.Error(w, "Error al procesar el mensaje", http.StatusBadRequest)
        return
    }
    // Asignamos un ID basado en el tamaño actual de la lista
    newMessage.ID = len(messages) + 1
    // Añadimos el nuevo mensaje a la lista
    messages = append(messages, newMessage)
    // Establecemos el encabezado para la respuesta
    w.Header().Set("Content-Type", "application/json")
    // Enviamos el nuevo mensaje como JSON
    json.NewEncoder(w).Encode(newMessage)
}
```
Javi sonrió. "Esto empieza a parecer una API de verdad".

Párrafo 5:

Finalmente, unió todo en la función main:

```go
// La función main inicia el servidor
func main() {
    // Registramos un manejador para la ruta "/messages"
    http.HandleFunc("/messages", func(w http.ResponseWriter, r *http.Request) {
        // Según el método HTTP, llamamos al handler correspondiente
        switch r.Method {
        case http.MethodGet:
            getMessages(w, r)
        case http.MethodPost:
            createMessage(w, r)
        default:
            // Si el método no es GET o POST, devolvemos un 405
            http.Error(w, "Método no permitido", http.StatusMethodNotAllowed)
        }
    })

    // Mostramos un mensaje para saber que el servidor está listo
    fmt.Println("Servidor corriendo en http://localhost:8080")
    // Iniciamos el servidor en el puerto 8080
    http.ListenAndServe(":8080", nil)
}
```
Ejecutó go run api.go y la terminal respondió: Servidor corriendo en http://localhost:8080. "¡Está vivo!", pensó Javi, con una emoción que apenas cabía en su pecho.

Párrafo 6:

Era hora de probarlo. Abrió otra terminal y escribió:

```bash
curl http://localhost:8080/messages
La respuesta fue: [{"id":1,"content":"¡Hola, mundo!"}].
```
 
"Perfecto", murmuró. Luego probó el POST:

```bash
curl -X POST http://localhost:8080/messages \
-H "Content-Type: application/json" \
-d '{"content":"¡Este es un nuevo mensaje!"}'
Y vio: {"id":2,"content":"¡Este es un nuevo mensaje!"}. ```

"¡Eso es una API RESTful!", exclamó Javi, casi saltando de su silla. Olivia, que pasaba por ahí, se detuvo. "¿Ya tenés el servidor levantado?". "Sí, y escucha y habla", dijo él, mostrando la terminal. "Nada mal", respondió ella con un guiño.

Javi cerró su laptop al final del día, satisfecho. Había construido una API RESTful desde cero —una que podía recibir y devolver mensajes—. Pero sabía que esto era solo la base. "Mañana le enseño a conversar con ChatGPT", pensó, recordando su éxito del Capítulo 2. Olivia se acercó antes de irse. "Levantaste la estructura. Ahora hacela inteligente". Javi asintió. "Eso viene después". Con el servidor corriendo en su mente, sintió que estaba a un paso de algo grande.