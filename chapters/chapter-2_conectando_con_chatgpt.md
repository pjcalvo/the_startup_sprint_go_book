# Chapter 2: Connecting with ChatGPT (20-25 pages)
## Making a basic API call to ChatGPT from Go.
### Introduction to HTTP requests and OpenAI API keys.
### Story: Javi celebrates his first success — but things quickly get harder.

Javi llegó temprano a la oficina, decidido a conquistar el siguiente reto: conectar su proyecto en Go con el API de ChatGPT. Mientras se servía un café, el sonido del teclado de Olivia ya llenaba el ambiente.

“¿Listo para la siguiente fase?” preguntó ella con una sonrisa.

“Listo es poco,” respondió Javi, aunque una parte de él seguía nerviosa.

En su estación, abrió su proyecto y comenzó a investigar cómo hacer una request HTTP en Go. Olivia se acercó y le dio un consejo: “Empieza simple. Haz una request básica y asegúrate de obtener una respuesta. Luego construyes desde ahí.”

Primero, necesitaba instalar una librería para manejar HTTP requests. Aunque la librería estándar de Go era suficiente para lo básico, decidió usar el paquete `net/http`.

En su archivo `main.go`, empezó a escribir:

```go
// Declaramos el paquete principal
package main

// Importamos librerías necesarias
import (
    "fmt"
    "net/http"
    "io/ioutil"
)

// La función main inicia el programa
func main() {
    // Definimos la URL del API de ChatGPT (ejemplo)
    url := "https://api.openai.com/v1/chat/completions"

    // Creamos una nueva request HTTP GET
    response, err := http.Get(url)
    if err != nil {
        // En caso de error, mostramos el mensaje y salimos
        fmt.Println("Error haciendo la request:", err)
        return
    }
    defer response.Body.Close()

    // Leemos la respuesta del cuerpo
    body, err := ioutil.ReadAll(response.Body)
    if err != nil {
        fmt.Println("Error leyendo la respuesta:", err)
        return
    }

    // Mostramos la respuesta en formato string
    fmt.Println(string(body))
}
```

Con los dedos cruzados, Javi ejecutó el programa:
```
go run main.go
```

Por supuesto, recibió un mensaje de error — el API requería una API key y una request POST con información específica.

“¿Te ayudó a calentar?” preguntó Olivia entre risas.

Javi se rió. “Al menos sé que la conexión está intentando hacerse. Ahora a hacerlo bien.”

El próximo paso sería enviar una request POST correctamente estructurada, con encabezados adecuados y el payload correcto. Pero por hoy, Javi sintió que había avanzado — y eso era suficiente para motivarlo a seguir.

