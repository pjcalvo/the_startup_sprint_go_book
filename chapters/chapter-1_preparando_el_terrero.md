# Chapter 1: Preparing the Ground (20-25 pages)
## Setting up Go (latest version) on macOS.
### First steps: "Hello, World!" in Go.
### Story: Javi’s nerves and excitement starting the project.


Javi miraba la pantalla, el mensaje de Slack de Emma seguía brillando. Su corazón latía más rápido de lo que quería admitir.

> *Hey Javi, necesitamos una herramienta interna que use el API de ChatGPT para ayudar al equipo de soporte a generar mejores respuestas. Quiero que construyas el backend en Go. Necesitamos un prototipo funcional en dos semanas.*

Dos semanas. Las manos de Javi comenzaron a sudar. Sabía un poco de Go, pero no lo suficiente para construir un API completo. Aun así, esta era su oportunidad — una muy grande.

Respiró profundo y respondió:
> *Voy con todo.*

Más tarde, tomando un café, Javi aprovechó para hablar con Olivia. “¿Por qué Go? Estoy emocionado, pero… ¿por qué no Python o Node?”

Olivia sonrió. “Porque Go es rápido. Es eficiente. Y es perfecto para APIs. Además, Emma quiere que nuestra stack sea simple y escalable. Ya verás, cuando le agarres la onda, te va a encantar.”

“Fácil para ti decirlo,” murmuró Javi. Pero la curiosidad ya estaba sembrada.

Esa noche, Javi se arremangó la camisa. Lo primero era poner Go en marcha en su Mac.

Abrió el Terminal y escribió:
```
brew install go
```
(Si no tienes Homebrew, instálalo desde [https://brew.sh/](https://brew.sh/).)

Una vez instalado, verificó la instalación:
```
go version
```
Vio algo como esto: `go version go1.21.0 darwin/amd64` (o la versión más reciente).

Después preparó el espacio de trabajo de Go, ya que a Go le gusta una estructura específica:
```
mkdir -p ~/go/src/github.com/tuusuario/nombreproyecto
cd ~/go/src/github.com/tuusuario/nombreproyecto
```

Para hacerlo más cómodo, configuró algunas variables de entorno añadiendo esto a su `~/.zshrc` o `~/.bash_profile`:
```
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```
Luego aplicó los cambios:
```
source ~/.zshrc  # o source ~/.bash_profile
```

“Ahora sí,” murmuró Javi. Era momento de escribir su primer programa en Go. Creó un archivo llamado `main.go`:
```go
// Declaramos el paquete principal
package main

// Importamos el paquete fmt para imprimir en consola
import "fmt"

// La función main es el punto de entrada del programa
func main() {
    // Imprimimos un mensaje en consola
    fmt.Println("¡Hola, Go!")
}
```

Para ejecutarlo, escribió:
```
go run main.go
```

Cuando vio el mensaje `¡Hola, Go!`, supo que había dado el primer paso.

Javi se recostó en su silla, mirando la respuesta de su "Hola, mundo" todavía en la terminal. El café se había enfriado, pero su mente estaba a mil. Había instalado Go y hecho que funcionara, pero conectar con ChatGPT iba a requerir más que un simple fmt.Println. "Necesito entender cómo funciona esto de verdad", pensó, garabateando en un post-it: HTTP, structs, algo de magia. Olivia pasó por su escritorio con su mochila al hombro. "¿Ya te rendiste o vas por más?", bromeó. "Voy por más", respondió Javi, con una sonrisa decidida. Sabía que antes de lanzarse al API, tenía que dominar los básicos de Go, y eso era exactamente lo que haría mañana.


