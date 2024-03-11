# Docker-file

Mediante esta práctica se estuvo creando un archivo **Dockerfile** para realizar la construcción (build) de una imagen docker (que luego podría ser llamada por algún archivo docker-compose.yml).
Donde a continuación desgloso la funcionalidad de cada parte del código definido:

En esta primera parte se esta indicando la base sobre la que se construye la imagen, en este  caso **ubuntu:23.04**, si no se especifica from, cogerá una imagen base predeterminada:

```
FROM ubuntu:23.04
```
Mediante esta etiqueta se puede poner el **nombre del autor**:

```
LABEL author="Samuel Hernández"
```
Y en esta primera parte se realización la creación de la primera capa, las cuales son las que vemos descargarse cuando hacemos algún **pull**. En este caso dividimos la lógica de las capas donde en esta primera se hace lo siguiente:

- Instalación de **git**.
- Instalación de **apache**.
- Limpiar la **caché** de los **paquetes**, logrando así un menor tamaño final de la imagen docker.
  
```
RUN apt update \
    && apt install git -y \
    && apt install apache2 -y \
    && rm -rf /var/lib/apt/lists/*
```
En la siguiente capa el objetivo es bajar el reporitorio de GitHub del juego 2048:

- Git clone del repositorio sobre la carpeta **/app**.
- Mover el contenido de **/app/** hacia **/var/www/html**.
- Eliminar el directorio **/app**, pues ya fue movido su contenido.
  
```
RUN git clone https://github.com/josejuansanchez/2048.git /app \
    && mv /app/* /var/www/html \
    && rm -rf /app
```
Aquí se establece el comando predeterminado que se ejecutará cuando se inicie un contenedor basado en esta imagen. En este caso, se inicia el servidor Apache en primer plano utilizando apache2ctl.
Si no se pone **entrypoint**, Docker utilizará la entrada predeterminada definida por la imagen base.
```
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```
# Contenido del Workflow ()

Este contenido es diseñado para realizar la publicación de la imagen en DockerHub, automatizando su proceso de construcción en dicho sitio, así como la automatización de la publicación de la imagen:

```
name: Publicar en dockerhub samuel3101

# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.

on:
  #schedule:
   # - cron: '22 15 * * *'
  push:
    branches: [ "main" ]
  workflow_dispatch:
    # Publish semver tags as releases.
    #tags: [ 'v*.*.*' ]
  #pull_request:
  #  branches: [ "main" ]

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
        uses: docker/login-action@343f7c4344506bcbf9b4de18042ae17996df046d # v3.0.0
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ secrets.docker_username }}
          password: ${{ secrets.docker_TOKEN }}

      # Build and push Docker image with Buildx (don't push on PR)
      # https://github.com/docker/build-push-action
      - name: Build and push Docker image
        id: build-and-push
        uses: docker/build-push-action@0565240e2d4ab88bba5387d719585280857ece09 # v5.0.0
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ env.REGISTRY }}/${{ secrets.docker_username }}/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}
          #tags: ${{ steps.meta.outputs.tags }}
          #labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```
