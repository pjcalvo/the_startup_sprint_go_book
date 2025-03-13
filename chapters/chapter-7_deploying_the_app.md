# Using Docker to package and deploy the API.
## Deploying to a simple cloud platform.
### Story: The final sprint — and a make-or-break presentation.

Javi llegó a la oficina con los nervios a flor de piel. La demo del día anterior había sido un éxito —Emma quedó impresionada con las pruebas—, pero ahora subía la apuesta: "Quiero esto en producción para el viernes". Era miércoles, y Javi sintió el peso del plazo como una mochila llena de ladrillos. Mientras se servía su café, Olivia lo miró desde su escritorio. "¿Listo para el mundo real?". "Si no me mata antes", respondió Javi, abriendo VS Code. Necesitaba empaquetar su API con Docker y subirla a la nube, todo en un sprint final que decidiría si brillaba o se estrellaba.

Javi había oído de Docker en el taller de Go, pero nunca lo había usado. "Es como poner tu API en una caja portátil", le había dicho Olivia una vez. Decidió empezar por ahí. Instaló Docker en su Mac, creó un archivo Dockerfile en el directorio de su proyecto, y escribió:

```dockerfile
Copy
# Usamos una imagen base oficial de Go
FROM golang:1.22

# Establecemos el directorio de trabajo dentro del contenedor
WORKDIR /app

# Copiamos los archivos del proyecto al contenedor
COPY . .

# Compilamos la API dentro del contenedor
RUN go build -o api

# Exponemos el puerto 8080 para que la API sea accesible
EXPOSE 8080

# Ejecutamos la API cuando el contenedor arranca
CMD ["./api"]

```
Guardó el archivo y construyó la imagen con:

```bash
docker build -t chatgpt-api .

```
Luego la corrió localmente:

```bash
docker run -p 8080:8080 chatgpt-api
```

Abrió otra terminal, probó con curl `http://localhost:8080/messages`, y sonrió al ver la respuesta JSON. "¡Funciona en una caja!", pensó.

Antes de seguir, Javi quería asegurarse de que todo estuviera bien. Recordó que la API key estaba hardcoded —un error que Olivia le había advertido— y decidió pasarla como variable de entorno. Modificó main.go:

```go
// Obtenemos la API key desde una variable de entorno
apiKey := os.Getenv("CHATGPT_API_KEY")
if apiKey == "" {
    log.Fatal("Falta la variable de entorno CHATGPT_API_KEY")
}
```

Actualizó el Dockerfile para usar un archivo .env (que no subiría al control de versiones):

```dockerfile
# Usamos una imagen base oficial de Go
FROM golang:1.22
WORKDIR /app
COPY . .
RUN go build -o api
EXPOSE 8080
# Cargamos variables de entorno desde un archivo .env
ENV CHATGPT_API_KEY=$CHATGPT_API_KEY
CMD ["./api"]
```

Reconstruyó y corrió el contenedor con:

```bash
docker run -p 8080:8080 --env-file .env chatgpt-api
```

Con un `.env` que decía `CHATGPT_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxx`, la API funcionó perfectamente. "Más seguro y portátil", pensó Javi.

Con el contenedor listo, Javi eligió una plataforma sencilla: Fly.io. Era ligera, gratis para empezar, y Olivia la había recomendado. Instaló el CLI de Fly con `brew install flyctl`, se autenticó, y creó un archivo `fly.toml`:

```toml
app = "chatgpt-api"
kill_signal = "SIGINT"
kill_timeout = 5

[env]
  PORT = "8080"

[http_service]
  internal_port = 8080
  force_https = true
```

Subió su imagen a Docker Hub con:

```bash
docker tag chatgpt-api javi/chatgpt-api
docker push javi/chatgpt-api
```

Luego desplegó:

```bash
fly launch --image javi/chatgpt-api --env CHATGPT_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxx
```

Tras unos minutos, Fly le dio una URL: https://chatgpt-api.fly.dev. Javi probó con curl https://chatgpt-api.fly.dev/messages y vio la respuesta JSON en la nube. "¡Está viva!", exclamó.

El día avanzaba, y la presión subía. Emma entró con noticias: "Los clientes quieren ver la API integrada con ChatGPT en vivo". Javi ajustó el Dockerfile para incluir la API key como variable de entorno (mejor que hardcoded):

```Dockerfile
# Añadimos la variable de entorno en runtime
ENV CHATGPT_API_KEY=""
CMD ["./api"]
```

Actualizó el despliegue con flyctl secrets set CHATGPT_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxx y redeployó con flyctl deploy. Probó un POST:

```bash
curl -X POST https://mi-api.fly.dev/messages \
-H "Content-Type: application/json" \
-d '{"input":"¿Qué es el universo?"}'
```

La respuesta fue: `{"id":2,"input":"¿Qué es el universo?","response":"El universo es todo lo que existe..."}`. "Funciona", suspiró Javi, aliviado.

Llegó el gran día. Javi y Emma estaban frente a los clientes en la sala de reuniones. Con el proyector encendido, Javi abrió http://mi-api.fly.dev y mostró el endpoint GET. Luego, en vivo, envió un mensaje a ChatGPT vía POST. La respuesta apareció instantáneamente en pantalla. Los clientes aplaudieron. "Esto es justo lo que necesitamos", dijo uno. Emma le dio una mirada de aprobación. Pero entonces, un cliente preguntó: "¿Qué pasa si ChatGPT falla?". Javi, gracias a los reintentos del Capítulo 5, respondió: "Tenemos reintentos automáticos". Hizo una demo forzando un fallo, y la API resistió. "Salvado otra vez", pensó.

Javi cerró su Mac esa noche, agotado pero triunfante. Docker había empaquetado su API, y Fly.io la había llevado al mundo sin complicaciones extras como Docker Hub. La presentación había sido un éxito, y el equipo tenía un nuevo cliente. Olivia le dio un choque de puños. "Lo lograste, Javi. Ahora a descansar". Él sonrió, garabateando "producción: check" en su cuaderno. Para profundizar en despliegues, planeó leer más sobre Docker en producción y Fly.io avanzado.

