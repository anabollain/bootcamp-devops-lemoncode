# Ejercicios GitHub Actions

## 1. Crea un workflow CI para el proyecto de frontend - OBLIGATORIO

### Enunciado

Copia el directorio [.start-code/hangman-front](../03-github-actions/.start-code/hangman-front) en el directorio raíz del mismo repositorio que usaste para las clases de GitHub Actions. Si no lo creaste, crea un repositorio nuevo.

Después crea un nuevo workflow que se dispare cuando haya cambios en el proyecto `hangman-front` y exista una nueva pull request (deben darse las dos condiciones a la vez). El workflow ejecutará las siguientes operaciones:

* Build del proyecto de front
* Ejecutar los unit tests

### Resolución

#### 1. Verificar proyecto

```bash
cd hangman-front
npm ci
npm start
```

```
localhost:8080
```

#### 2. Crea el repositorio en GitHub

- Crea un repositorio privado llamado `entrega-github-lemoncode`.

- Clona el repositorio en tu local:

- Copia el directorio  `../03-github-actions/.start-code/hangman-front`.

- Sube cambios al repositorio

```bash
git add .
git commit -m "Initial commit"
git push origin main
```

#### 3. YAML

- Una vez clonado el repositorio en tu local con el contenido de la aplicación, ejecuta el siguiente comando:

```bash
git checkout -b add-basic-workflows
```

- Crea una carpeta `.github/workflows` con un archivo `ci.yaml` con el siguiente contenido:

```yaml
name: CI

on:
  push:
    branches: [main]
    paths: ["hangman-front/**"]
  pull_request:
    branches: [main]
    paths: ["hangman-front/**"]
    
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Build
        working-directory: ./hangman-front
        run: |
          npm ci
          npm run build --if-present
  
  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: test
        working-directory: ./hangman-front
        run: |
          npm ci
          npm test
```

- Sube los cambios al repositorio:

```bash
git add .
git commit -m "Add ci.yaml"
git push origin add-basic-workflows
```

- Crea una PR:

Desde GitHub, generamos una **Pull Request** de `add-basic-workflows` a `main`. Una vez creada, veremos que se ha generado un nuevo **Job** en el apartado **Actions** de forma automática.

#### 4. Soluciona posibles errores

Verás que el job `test` ha fallado...

- Modifica la siguiente línea en el archivo `start-game.spect.tsx` para que el test sea exitoso:

```js
    expect(items).toHaveLength(2);
```

- Sube los cambios y revisa el workflow, verás que ya no hay errores.

#### 5. Utiliza artifacts

- Añade un nuevo archivo `ci-artifacts.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
    paths: ["hangman-front/**"]
  pull_request:
    types: [opened, reopened]
    branches: [main]
    paths: ["hangman-front/**"]
    
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Build
        working-directory: ./hangman-front
        run: |
          npm ci
          npm run build --if-present

      - name: Upload Artifact (dependencies)
        uses: actions/upload-artifact@v4
        with:
          name: dependencies
          path: hangman-front/node_modules/
          include-hidden-files: true
  
  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Download Artifact (dependencies)
        uses: actions/download-artifact@v4
        with:
          name: dependencies
          path: hangman-front/node_modules/

      - name: test
        working-directory: ./hangman-front
        run: |
          chmod -R +x node_modules
          npm test
```

- Sube los cambios

- Revisa workflow

#### 6. Utiliza cache

- Añade un nuevo archivo `ci-cache.yml`:

- Sube los cambios

- Revisa workflow

#### 6. Completar MR

...

---

## 2. Crea un workflow CD para el proyecto de frontend - OBLIGATORIO

### Enunciado

Crea un nuevo workflow que se dispare manualmente y haga lo siguiente:

* Crear una nueva imagen de Docker
* Publicar dicha imagen en el [container registry de GitHub](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)

### Resolución

#### 1. Crear rama

- Creamos una nueva rama `add-cd-workflow` en el repositorio.

#### 2. Crea el archivo `cd.yaml`:

- Creamos el archivo `cd.yaml` en el directorio `.github/workflows` con el siguiente contenido:

```yaml
name: CD workflow

on:
  pull_request: # Cuando se crea o actualiza un PR a main
    branches: [main]
    paths: ["hangman-front/**"]
  workflow_dispatch: # Permite ejecutar el workflow manualmente desde la UI de GitHub

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js version
        uses: actions/setup-node@v4
        with:
          node-version: 16
          cache: "npm"
          cache-dependency-path: hangman-front/package-lock.json

      - name: Build
        working-directory: ./hangman-front
        run: |
          npm ci
          npm run build --if-present

      - name: Upload artifact (dist folder)
        uses: actions/upload-artifact@v4
        with:
          name: build-code
          path: hangman-front/dist/

  delivery:
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Download artifact (dist folder)
        uses: actions/download-artifact@v4
        with:
          name: build-code
          path: hangman-front/dist/
          
      - name: Docker login
        uses: docker/login-action@v3
        with:
          username: anabollain
          password: ${{ secrets.DOCKER_PASSWORD }}
 
      - name: Setup Docker builder
        uses: docker/setup-buildx-action@v3
 
      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: ./hangman-front
          push: true
          tags: anabollain/hangman-front-actions-2024:latest
          file: ./hangman-front/Dockerfile.workflow
```

- Sube los cambios al repositorio para que se ejecute el Workflow y hacemos un merge con `main`.

#### 3. Crea credenciales Docker Hub en GitHub

- Accede a tu repositorio en GitHub

- Ve a la configuración del repositorio (Settings)

- Accede a los secretos

En la barra lateral izquierda, desplázate hacia abajo hasta Secrets and variables (Secretos y variables) y haz clic en Actions.

- Agrega los secretos

  - Para DOCKER_USERNAME: Establece el Nombre como DOCKER_USERNAME y el Valor como tu nombre de usuario de Docker Hub.

  - Para DOCKER_PASSWORD: Establece el Nombre como DOCKER_PASSWORD y el Valor como tu contraseña de Docker Hub (o, preferiblemente, un token de acceso para mayor seguridad).

- Guarda los secretos 

Haz clic en Add secret (Agregar secreto) después de ingresar cada uno.

#### 4. Ejecuta el workflow

Manualmente...

---

## 3. Crea un workflow que ejecute tests e2e - OPCIONAL

### Enunciado

Crea un workflow que se lance de la manera que elijas y ejecute los tests e2e que encontrarás en [este enlance](../03-github-actions/.start-code/hangman-e2e/e2e/). Puedes usar [Docker Compose](https://docs.docker.com/compose/gettingstarted/) o [Cypress action](https://github.com/cypress-io/github-action) para ejecutar los tests.

#### Como ejecutar los tests e2e

* Tanto el front como la api se deben estar corriendo

```bash
docker run -d -p 3001:3000 hangman-api
docker run -d -p 8080:8080 -e API_URL=http://localhost:3001 hangman-front
```

* Los tests se ejecutan desde el directorio `hangman-e2e/e2e` haciendo uso del comando `npm run open`

```bash
cd hangman-e2e/e2e
npm run open
```

### Resolución

- Crear...

---

## 4. Crea una custom JavaScript Action - OPCIONAL

### Enunciado

Crea una custom JavaScript Action que se ejecute cada vez que una `issue` tenga la etiqueta `motivate`. La acción deberá pintar por consola un mensaje motivacional. Puedes usar [esta API](https://type.fit) gratuita. Puedes encontrar más información de como crear una custom JS action en [este enlace](https://docs.github.com/es/actions/creating-actions/creating-a-javascript-action).

```bash
curl https://type.fit/api/quotes
```

### Resolución

#### 1. Crear y clonar el repositorio

Una acción personalizada en GitHub se define dentro de un repositorio, por lo que el primer paso es:

- Crear un repositorio público llamado `get-motivational-message`

- Clonarlo a tu entorno local:

```bash
git clone https://github.com/anabollain/get-motivational-message.git
cd get-motivational-message
```

#### 2. Iniciar el proyecto de Node.js

Desde la raíz del proyecto, inicializamos el entorno de Node.js:

```bash
npm init -y
```

#### 3. Crear el archivo `action.yaml`

Este archivo define la **metadata** de la acción: nombre, descripción, entradas, salidas y cómo se ejecuta.

```yaml
name: 'Get Motivational Message'
description: 'Get random motivational message'
inputs:
  api_url:
    description: 'API Url'
    required: true
    default: https://zenquotes.io/api/random
outputs:
  message:
    description: 'Motivational message'
runs:
  using: 'node20'
  main: 'index.js'
```

#### 4. Instalar dependencias necesarias

Instalamos los **paquetes** necesarios para interactuar con los inputs, outputs y contexto de GitHub Actions:

```bash
npm i @actions/core @actions/github
```

#### 5. Crear el archivo `index.js`

Este script implementa la **lógica** de la acción. Valida los inputs y devuelve un precio simulado para oro o plata.

```js
// Importamos el módulo 'core' de GitHub Actions para manejar inputs y outputs
const core = require('@actions/core');
// Importamos el módulo 'github' para acceder al contexto del workflow (opcional aquí, pero útil para depuración)
const github = require('@actions/github');
// Importamos 'node-fetch' para hacer solicitudes HTTP
const fetch = require('node-fetch');

async function run() {
  try {
    // Obtenemos el valor del input 'api_url' definido en action.yaml
    const apiUrl = core.getInput('api_url');
    console.log(`API URL: ${apiUrl}`);

    let motivationalMessage = 'Stay strong and keep going!';

    try {
      const response = await fetch(apiUrl);
      if (!response.ok) {
        throw new Error(`API request failed with status ${response.status}`);
      }

      const data = await response.json();

      if (data[0]?.q) {
        motivationalMessage = data[0].q;
      } else {
        console.warn('Warning: API responded but no quote found. Using default message.');
      }

    } catch (fetchError) {
      console.warn('Warning: Failed to fetch from API, using default message.');
      console.warn(fetchError.message);
    }

    // Establecemos la salida de la acción, que podrá usarse en otros pasos del workflow
    core.setOutput('motivational_message', motivationalMessage);

    // Mostramos el payload del evento del workflow (útil para depuración)
    const payload = JSON.stringify(github.context.payload, undefined, 2);
    console.log(`Event payload: ${payload}`);

  } catch (error) {
    // Si ocurre un error, se marca el paso como fallido y se imprime el mensaje
    core.setFailed(error.message);
  }

}

run();
```


*💡 Mejora futura: En lugar de valores simulados, podríamos integrar una API externa para obtener precios en tiempo real.*

### 6. Crear el archivo `README.md`

Este archivo explica **cómo utilizar la acción** (se puede consultar el template en la documentación oficial). Aquí un ejemplo de contenido:

```markdown
# get-motivational-message

This action gets gold or silver current price per ounce in USD/EUR.

## Inputs

### `api_url`

**Required** The name of the API Rest.

## Outputs

### `message`

Random motivational message.

## Example usage

uses: anabollain/get-motivational-message@1.0.0
with:
  api_url: 'https://type.fit/api/quotes'

```

#### 7. Publicar el release en GitHub

Para publicar una versión de la acción, crea un tag con:

```bash
git tag -a -m "Get motivational message action initial release" v1.0.0
git push --follow-tags
```

#### 8. Utilizar la custom action en la aplicación `hangman-api`

Creamos el archivo `.github/workflows/test-custom-action.yaml` con el siguiente contenido:

```yaml
name: Test get motivational message custom action

on:
  workflow_dispatch:

jobs:
  get_commodity_price:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        api_url: ["https://type.fit/api/quotes"]

    steps:
      - name: Get motivational message
        id: motivational_message
        uses: anabollain/get-motivational-message-action@v1.0.0
        with:
          commodity: ${{ matrix.api_url }}
      - name: Output motivational message
        run: echo "${{ steps.commodity_price.outputs.motivational_message }}"
```
