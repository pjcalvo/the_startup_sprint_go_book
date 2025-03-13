Este libro es una guía práctica para llevarte de cero a una API funcional usando Go, un lenguaje que ha capturado la atención de desarrolladores en todo el mundo. Go (o Golang) fue creado en 2009 por Google para resolver problemas reales: es simple, rápido y diseñado para la era moderna de la computación distribuida. Aquí explorarás cómo construir una API que se conecta con ChatGPT, desde los fundamentos hasta el despliegue, todo mientras descubres por qué Go se ha convertido en una herramienta tan poderosa.

### ¿Por qué Go se ha vuelto tan popular?

Go ha ganado terreno por su combinación única de características. Es un lenguaje compilado que produce binarios rápidos, rivalizando con C o C++, pero con una sintaxis limpia que recuerda a Python. Su manejo de concurrencia con goroutines y canales lo hace ideal para aplicaciones que necesitan escalar —piensa en servidores, microservicios o sistemas en la nube—. Empresas como Uber, Docker y Kubernetes lo adoptaron por su eficiencia y su biblioteca estándar robusta, que incluye desde net/http para servidores hasta testing para pruebas. Go es popular porque entrega resultados sin complicaciones, y este libro te mostrará cómo aprovechar eso.

### Herramientas que usaremos

A lo largo de estas páginas, usarás un conjunto de herramientas que reflejan flujos reales de desarrollo. Empezaremos con el propio Go y su CLI (go run, go test) para construir y probar código. Conectarás con el API de ChatGPT usando net/http, estructurarás tu proyecto con paquetes, y probarás con httptest. Para persistencia, experimentarás con mapas en memoria y sync para concurrencia segura. Docker empaquetará tu API, y Fly.io —una plataforma que simplifica despliegues en la nube. Cada herramienta fue elegida por su practicidad y su presencia en el mundo real.

### Aprender haciendo y lo que importa

Este libro no es solo sobre código; es sobre aprender haciendo. Cada capítulo te empuja a ensuciarte las manos, desde instalar Go hasta optimizar con goroutines, porque la mejor forma de entender un lenguaje es construir algo con él. Go no solo te da herramientas técnicas, sino una mentalidad: resolver problemas con claridad y eficiencia. Ya seas un principiante o un desarrollador experimentado, mi esperanza es que termines con una API funcional y la confianza para crear más. Así que toma tu café, abre tu editor, y empecemos este viaje juntos.