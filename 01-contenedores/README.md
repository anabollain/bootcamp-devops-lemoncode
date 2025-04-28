# Laboratorio Contenedores Docker

## Ejercicio 1

### Enunciado

Dockeriza la aplicación dentro de [lemoncode-challenge](./), la cual está compuesta de 3 partes:

- Un **front-end** con Node.js. *Nota Ana: Aplicación que renderiza las páginas del lado del servidor, NO las renderiza en el navegador.*
- Un **backend** en .NET (`dotnet-stack`) o en Node.js (`node-stack`) que utiliza un MongoDB para almacenar la información.
- El MongoDB donde se almacena la información en una **base de datos**.

Nota: como has podido comprobar, el directorio `lemoncode-challenge` tiene dos carpetas: `dotnet-stack` y `node-stack`. En ambos casos el frontend es el mismo, sólo cambia el backend. Usa el stack que prefieras.

*Nota Ana: realizar ejercicio con comandos con el cliente de Docker.*

#### Requisitos del ejercicio

1. Los tres componentes deben estar en una red llamada `lemoncode-challenge`.
2. El backend debe comunicarse con el mongodb a través de esta URL `mongodb://some-mongo:27017`.
3. El front-end debe comunicarse con la api a través de `http://topics-api:5000/api/topics`.
4. El front-end debe estar mapeado con el host para ser accesible a través del puerto 8080.
5. El MongoDB debe almacenar la información que va generando en un volumen, mapeado a la ruta `/data/db`.
6. Este debe de tener una base de datos llamada `TopicstoreDb` con una colección llamada `Topics` (*Nota Ana: esta colección ya está generada dentro del directorio routes del proyecto backend.*). La colección `Topics` debe tener esta estructura:

```json
{
  "_id": { "$oid" : "5fa2ca6abe7a379ec4234883" },
  "topicName" : "Contenedores"
}
```

¡Añade varios registros!

__Tip para backend__: Antes de intentar contenerizar y llevar a cabo todos los pasos del ejercicio, se recomienda intentar ejecutar la aplicación sin hacer cambios en ella (*Nota Ana: para poder ejecutar este paso necesitamos tener dotnet o node instalado; o levantar un contenedor dev temporal.*). En este caso, lo único que es posible que “no tengamos a mano” es el MongoDB. Por lo que empieza por crear este en Docker, usa un cliente como el que vimos en el primer día de clase (MongoDB Compass) para añadir datos que pueda devolver la API.

![Mongo compass](./images/mongodbcompass.png)

Nota: es más fácil si abres Visual Studio Code desde la carpeta `backend` para hacer las pruebas y las modificaciones que si te abres desde la raíz del repo. Para ejecutar este código solo debes lanzar `dotnet run` si usas el stack de .NET, o `npm install && npm start` si usas el stack de Node.js.

__Tip para frontend__: Para ejecutar el frontend abre esta carpeta en VS Code y ejecuta primero `npm install`. Una vez instaladas las dependencias ya puedes ejecutarla con `npm start`. Debería de abrirse un navegador con lo siguiente:

![Topics](./images/topics.png)

*Notas Ana: esquema visual generado en el tutorial:*

![Esquema](./images/schema.PNG)

### Resolución

#### Verificar que la aplicación funciona correctamente

##### 1. MongoDB

1. Lanzar contenedor:

```bash
docker run --name mongodb -d -p 27017:27017 mongo
```

2. Crear registro prueba:

- A través de la shell:

```bash
docker exec -it mongodb mongosh
```

```bash
use TopicstoreDb
db.Topics.insertOne({Name: "My first topic"})
```

- Con Curl:

```bash
curl.exe -X POST http://localhost:5000/api/topics -H "Content-Type: application/json" -d '{\"Name\": \"New topic\"}'
curl.exe -X PUT http://localhost:5000/api/topics/680a02f3a9b1c0c9bc3cbe56 -H "Content-Type: application/json" -d '{\"Name\": \"Updated topic\"}'
```

- Con Mongo Compass

https://www.mongodb.com/try/download/compass

Nueva conexión: mongodb://localhost:27017

##### 2. Backend

```bash
npm install
npm start
```

```
http://localhost:5000/api/topics
```

![Back End](./images/backend.PNG)

##### 3. Frontend

```bash
npm install
npm start
```

```
http://localhost:3000/
```

![Front End](./images/frontend.PNG)

### Dockerizar la aplicación a través de comandos

#### 1. Crear red

```bash
docker network create lemoncode-challenge
docker network ls
```

#### 2. Crear DB

*Volumen y red*

*El MongoDB debe almacenar la información que va generando en un volumen, mapeado a la ruta `/data/db`.*

Este debe de tener una base de datos llamada `TopicstoreDb` con una colección llamada `Topics` (*Nota Ana: esta colección ya está generada dentro del directorio routes del proyecto backend.*). La colección `Topics` debe tener esta estructura:

```json
{
  "_id": { "$oid" : "5fa2ca6abe7a379ec4234883" },
  "topicName" : "Contenedores"
}
```

```bash
docker run -d --name some-mongo `
  -v mongo_data:/data/db `
  --network lemoncode-challenge `
  -p 27017:27017 `
  mongo
```

#### 2. Ejecutar back-end

*El backend debe comunicarse con el mongodb a través de esta URL `mongodb://some-mongo:27017`.*

```bash
docker run -d --name topics-api `
  --network lemoncode-challenge `
  --mount type=bind,source="${PWD}/backend",target=/app `
  -w /app `
  -e DATABASE_URL="mongodb://some-mongo:27017" `
  -e HOST=0.0.0.0 `
  -p 5000:5000 `
  node:20-alpine `
  sh -c "npm install && npm start"
```

#### 3. Ejecutar front-end

*El front-end debe comunicarse con la api a través de `http://topics-api:5000/api/topics`.*

*El front-end debe estar mapeado con el host para ser accesible a través del puerto 8080.*

```bash
docker run -d --name frontend `
  --network lemoncode-challenge `
  --mount type=bind,source="${PWD}/frontend",target=/app `
  -w /app `
  -e API_URI="http://topics-api:5000/api/topics" `
  -p 8080:3000 `
  node:20-alpine `
  sh -c "npm install && npm start"
```

```
http://localhost:5000/api/topics
http://localhost:8080
```

## Ejercicio 2

### Enunciado

Ahora que ya tienes la aplicación del ejercicio 1 dockerizada, utiliza Docker Compose para lanzar todas las piezas a través de este. Debes plasmar todo lo necesario para que esta funcione como se espera: la red que utilizan, el volumen que necesita MongoDB, las variables de entorno, el puerto que expone la web y la API. Además debes indicar qué comandos utilizarías para levantar el entorno, pararlo y eliminarlo.

### Resolución

```
services:

  db:
    image: mongo
    container_name: some-mongo
    restart: always
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=lemoncode-password
    volumes:
      - mongo_data:/data/db
    networks: 
      - lemoncode-challenge
    
  backend:
    container_name: topics-api
    build:
      context: ./backend
    restart: always
    environment:
      DATABASE_URL: mongodb://root:lemoncode-password@some-mongo:27017
      HOST: 0.0.0.0
    volumes:
      - ./backend:/app
    ports:
      - 5000:5000
    networks: 
      - lemoncode-challenge
    depends_on:
      - db

  frontend:
    container_name: frontend
    build:
      context: ./frontend
    restart: always
    environment:
      API_URI: http://topics-api:5000/api/topics
    volumes:
      - ./frontend:/app
    ports:
      - 8080:3000
    networks: 
      - lemoncode-challenge
    depends_on:
      - backend

volumes:
  mongo_data:

networks:
  lemoncode-challenge: 
```