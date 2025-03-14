# Capítulo 1.5: Conceptos Básicos de Go

Javi llegó a la oficina con un cuaderno lleno de garabatos y una curiosidad que no lo dejaba dormir. La noche anterior había estado hojeando un tutorial de Go en línea, y palabras como "estructuras" y "punteros" seguían dando vueltas en su cabeza, aunque no terminaba de entenderlas. "¿Qué tan difícil puede ser?", pensó, abriendo su Mac mientras el aroma del café llenaba la sala. Olivia, que ya estaba aporreando su teclado, levantó la vista. "¿Qué, ahora vas a volverte gurú de Go?". "Paso a paso", respondió Javi con una sonrisa, abriendo VS Code y creando un archivo nuevo: basics.go. Antes de conectar con ChatGPT, quería descifrar esos conceptos básicos que todos en el equipo parecían dominar.

Javi comenzó con las variables, algo que parecía simple pero que en Go tenía su propio estilo. Recordó que no necesitaba declarar tipos explícitamente todo el tiempo gracias a la inferencia de tipos. Tecleó su primer ejemplo:

```go
package main

import "fmt"

func main() {
    // Declaramos una variable con inferencia de tipo (Go adivina que es string)
    nombre := "Javi"
    // Declaramos una variable con tipo explícito
    edad int = 28
    fmt.Println("Me llamo", nombre, "y tengo", edad, "años")
}
```

Ejecutó `go run basics.go` y la terminal mostró: `Me llamo Javi y tengo 28 años`. 

"Fácil", pensó. Le gustaba que Go fuera directo —nada de complicaciones innecesarias—. Pero también notó que si intentaba usar una variable sin inicializar, Go se quejaba. Era estricto, y eso le dio confianza: menos errores tontos más adelante.

Luego pasó a las estructuras, o struct, que había visto mencionadas en el tutorial como una forma de agrupar datos. "Esto podría servir para los mensajes de ChatGPT", murmuró, imaginando cómo organizaría cosas más tarde. Escribió un ejemplo:


```go
package main

import "fmt"

// Definimos una estructura para representar un mensaje
type Mensaje struct {
    Texto string  // Campo para el contenido del mensaje
    ID    int     // Campo para un identificador único
}

func main() {
    // Creamos una instancia de la estructura con valores iniciales
    msg := Mensaje{Texto: "Hola, Go!", ID: 1}
    // Accedemos a los campos con el operador punto
    fmt.Println("Mensaje:", msg.Texto, "ID:", msg.ID)

    // Creamos otra instancia sin valores (campos tendrán valores por defecto: "" y 0)
    var msg2 Mensaje
    fmt.Println("Mensaje vacío:", msg2.Texto, "ID:", msg2.ID)
}
```

Corrió el código y vio: `Mensaje: Hola, Go! ID: 1` y `Mensaje vacío:  ID: 0`. "Esto es como una caja donde guardo cosas", pensó Javi. Olivia, que pasaba con un café, comentó: "Los structs son tus amigos para JSON después". Javi anotó eso mentalmente —sonaba importante.

Los punteros fueron el siguiente misterio. El tutorial los había descrito como algo que "apuntaba" a datos, pero Javi no lo tenía claro aún. Decidió probarlo:

```go
package main

import "fmt"

func main() {
    // Declaramos una variable normal
    numero := 42
    // Creamos un puntero que apunta a la dirección de memoria de "numero"
    puntero := &numero
    // Usamos * para acceder al valor en esa dirección
    fmt.Println("Valor original:", numero)
    fmt.Println("Dirección del puntero:", puntero)
    fmt.Println("Valor vía puntero:", *puntero)

    // Cambiamos el valor a través del puntero
    *puntero = 100
    fmt.Println("Nuevo valor de numero:", numero)
}
```

La salida fue: `Valor original: 42`, una dirección como `0x...`, `Valor vía puntero: 42`, y `Nuevo valor de numero: 100`. "Ok, esto es poderoso", pensó Javi. Cambiar algo a través de un puntero le abrió los ojos: podía manipular datos sin copiarlos todo el tiempo.

Javi siguió con los paquetes, porque había visto package main y import "fmt" en todos lados y quería entenderlos mejor. Cada archivo en Go pertenecía a un paquete, y main era especial porque era donde empezaba el programa. Creó un archivo saludos.go en una carpeta utils:

```go
// Archivo: utils/saludos.go
package saludos

import "fmt"

// Función pública (empieza con mayúscula) para saludar
func Saludar(nombre string) {
    fmt.Println("¡Hola,", nombre, "desde el paquete saludos!")
}
```

Luego ajustó basics.go:

```go
package main

import (
    "fmt"
    "./utils" // Importamos el paquete local "saludos"
)

func main() {
    // Llamamos a la función del paquete saludos
    saludos.Saludar("Javi")
    fmt.Println("Esto es el programa principal")
}
```

Ejecutó `go run basics.go` y vio: `¡Hola, Javi desde el paquete saludos!` y Esto es el programa principal. "¡Funciona como un módulo!", pensó, emocionado por separar su código en piezas reutilizables.

El último tema fue los módulos, algo que Javi había notado al instalar Go pero no entendía del todo. En la terminal, creó un módulo con `go mod init mi-proyecto`, generando un archivo go.mod:

```text
module mi-proyecto
go 1.22
```

Ajustó `basics.go` para usarlo:

```go
package main

import (
    "fmt"
    "mi-proyecto/utils" // Importamos usando el nombre del módulo
)

func main() {
    // Llamamos a la función del paquete saludos
    saludos.Saludar("Javi")
    fmt.Println("¡Esto es un módulo en acción!")
}
```

Corrió `go run .` y funcionó igual. "Esto es como el jefe de mi código", pensó Javi. Los módulos organizaban todo y le permitían usar paquetes externos después, como los de ChatGPT. Olivia se acercó. "¿Ya sos experto en Go?". "No todavía", dijo Javi, "pero ya no me da miedo".

Javi cerró su Mac al final del día, satisfecho. Había aprendido a declarar variables, crear estructuras, usar punteros, organizar paquetes y manejar módulos —todo con código que podía tocar y entender. Cada ejemplo había sido un pequeño triunfo, y ahora se sentía listo para el siguiente paso: conectar con ChatGPT. "Mañana voy por ese API", se prometió, garabateando una estrella en su cuaderno. Olivia le dio un pulgar arriba desde la puerta. "Vas bien, Javi. No te pierdas". Él sonrió. Con estas bases, el camino hacia su API empezaba a verse más claro.