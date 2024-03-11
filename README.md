# Dockerizar una web estática y publicarla en docker hub.
## Requisitos necesarios.

```
Como imagen base deberá utilizar la última versión de ubuntu.

Instala el software necesario para poder clonar el repositorio de GitHub donde se encuentra la aplicación web estática.

Clona el repositorio de GitHub donde se encuentra la aplicación web estática en el directorio /usr/share/nginx/html/, que es el directorio que utiliza Nginx, por defecto, para servir el contenido.

El puerto que usará la imagen para ejecutar el servicio de Nginx será el puerto 80.

El comando que se ejecutará al iniciar el contenedor será el comando nginx -g   'daemon off;'.
```

### Contamos con el fichero docker file, el cual contiene lo siguiente.

```
FROM ubuntu:23.04

LABEL author ="Juan José"

RUN apt update \
    && apt install git -y \
    && apt install apache2 -y \
    && rm -rf /var/lib/apt/lists/* 

RUN git clone https://github.com/josejuansanchez/2048.git /app \
    && mv /app/* /var/www/html \
    && rm -rf /app

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

El cual se hace mención a la imagen de ubuntu, con la versión de 23.04

Estableciendo una etiqueta autor.

Ejecutando mediante run sentencias anidadas como actualización de repositorios instalación de git, instalación de apache2 y por ultimo borrar contenido de los paquetes.

El siguiente "run" ejecuta un clone para descargar un repositorio desde github siguiente mueve todo el contenido de app a /var/www/html y seguidamente borra todo el contenido de app.

Y por ultimo en el docker se añade por terminal que ese proceso se realice en primer plano, mediante un entrypoint iniciando el servicio de apache, en esa imagen de ubuntu.

Para crear la imagen a partir del archivo dockerfile debemos emplear lo siguiente...

```
docker build -t nginx-2048 .
```

Una buena práctica es asignarle una etiqueta a la imagen.
```
docker tag nginx-2048 jguiman958/2048:1.0
```
Añadiendo una versión específica.
```
docker tag nginx-2048 josejuansanchez/nginx-2048:latest
```

O una latest.

### Seguidamente se debe publicar la imagen en dockerhub.

Primero iniciamos sesión desde el terminal
```
docker login
```
Y seguidamente cuando ponemos nuestras credenciales.
Realizamos un push:
```
docker push jguiman958/2048:1.0 o latest...
```

También podemos realizarlo mediante github actions, el cual permite realizar cambios de fuera automatica empleando un fichero docker-publish.yml

```
name: Docker

# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.

on:
#  schedule:
#    - cron: '20 0 * * *'
  push:
    branches: [ "main" ]
    # Publish semver tags as releass.
    tags: [ 'v*.*.*' ]
  workflow_dispatch:

env:
  # Use docker.io for Docker Hub if empty
  REGISTRY: docker.io
  # github.repository as <account>/<repo>
  IMAGE_NAME: 2048
  IMAGE_TAG: latest


jobs:
  build:

    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      # This is used to complete the identity challenge
      # with sigstore/fulcio when running outside of PRs.
      id-token: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      # Set up BuildKit Docker container builder to be able to build
      # multi-platform images and export cache
      # https://github.com/docker/setup-buildx-action
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@f95db51fddba0c2d1ec667646a06c2ce06100226 # v3.0.0

      # Login against a Docker registry except on PR
      # https://github.com/docker/login-action
      - name: Log into registry ${{ env.REGISTRY }}
        if: github.event_name != 'pull_request'
        uses: docker/login-action@343f7c4344506bcbf9b4de18042ae17996df046d # v3.0.0
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # Build and push Docker image with Buildx (don't push on PR)
      # https://github.com/docker/build-push-action
      - name: Build and push Docker image
        id: build-and-push
        uses: docker/build-push-action@0565240e2d4ab88bba5387d719585280857ece09 # v5.0.0
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ env.REGISTRY }}/${{ secrets.DOCKERHUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}
          #tags: ${{ steps.meta.outputs.tags }}
          #labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```
Aquí, declaramos desde los permisos, hasta las variables necesarias que incluyen el login usado para iniciar sesión en docker hub, es decir sus credenciales, empleando un token securizado necesario para iniciar sesión, actuando sobre la rama principal del repositorio, asignando un nombre, a este fichero, el cual si visualizamos en actions con nuestro repositorio, podremos ver errores, si lo carga correctamente, etc. Podremos ver cualquier cambio. Haciendo que los cambios al realizar un push, o al realizar un guardado de un fichero, se hagan al instante.
