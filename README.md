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
# Contenido del Workflow

Este contenido es diseñado para realizar la publicación de la imagen en DockerHub, automatizando su proceso de construcción en dicho sitio, así como la automatización de la publicación de la imagen:


