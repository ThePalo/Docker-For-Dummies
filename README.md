# Docker for dummies

* **Docker:** herramienta de creación y gestión de contenedores
* **Contenedor:** es una unidad estándar de software que empaqueta el código y todas sus dependencias para que la aplicación se ejecute de forma rápida y confiable de un entorno informático a otro
* Usa tecnología de linux (*cgroups* y *namespaces*) para simular un contenedor
  * Hace que los procesos ejecutados crean que se ejecutan independientemente en otro sistema
* Docker comparte kernel con el host → hay una dependència de kernel, pero nada más
* **Dockerfile** → *build* → **Imagen** → *run* → **Contenedor**
* Dockerfile: crea una imagen que, al correr, crea un contenedor. 
  * La imagen se compone de:
    * OS
    * Software
    * App

## Comandos
* Descargar imagen `$ docker pull <image_name> `
* Run image: `$ docker run <settings> <image_name>`
  * `-d` → para correr en background
* https://hub.docker.com/ → sitio oficial de imagenes listas para compilar
* Ver contenedores en ejecución: `$ docker ps`
* Iniciar contenedor: `$ docker start <container_id>`
* Parar contenedor: `$ docker stop <container_id>`
* Ver output del contenedor: `$ docker logs <container_id>`
  * App dentro del contenedor no debe escribir en archivo, lo debe hacer en standard output así docker va a cogerlo como logs
* Ejecutar comando en contenedor en ejecución `$ docker exec -it <container_id> <command>`
  * `-i` → sesion interactiva
  * `-t` → terminal

## Usar contenedores
* Crear archivo Dockerfile → se basa en una imagen padre
  * Buscar imagen padre → ejemplo: imagen oficial de go
  * El archivo debe llamarse `Dockerfile`
* Ejemplo Dockerfile 
```dockerfile
FROM node:12.22.1-alpine3.11 # imagen padre → usar imagen oficiales

WORKDIR /app  # directorio donde se copiara toda la app
COPY . .      # copiar el directorio actual al directorio default del container (WORKDIR)
RUN yarn install --production  # corre el comando al compilar la imagen → compilar el código de node

CMD ["node", "/app/src/index.js"]  # especificar el comando que vaya a correr al iniciar contenedor, comando node con argumento /app/src/index.js
```
* Una vez creado el archivo Dockerfile, usar `$ docker build -t <image_name> <directorio_app>` para crear la imagen

* Acceder a puertos: los puertos que escucha la app de docker no són los del host. Para hacer el mapping, se debe correr un docker con la imagen deseada y indicando el mapeo de puertos: `$ docker run -p 3000:3000 <image_name>`
* Persistencia de datos: los datos generados en un contenedor se borran al reiniciar el contenedor. Para que haya persistencia: `$ docker run -v <local_path>:<docker_path> <image_name>`
  * Modificación bidireccional: cualquier cambio en una de los dos ficheros se ve en el otro.
  * Los ficheros también pueden ser directorios

* Subir imagen a https://hub.docker.com/
  * Crear cuenta
  * Tagear imagen: `$ docker tag <image_id> <tag>`
    * <tag> es <user>/<image_name>:<version>
  * Subir imagen: `$ docker push <tag>`

## Correr varios contenedores a la vez
* Lo más común: BD i app en contenedores separados
* Crear network (net de docker): ` $ docker network create <net_name>`
* Crear container para BD conectado a la network (ejemplo con imagen mysql:5.7):
```shell
$ docker run -d \
--network <net_name> --network-alias <alias> \
-v <fichero_docker>:<fichero_original> \
-e MYSQL_ROOT_PASSWORD=<pwd> \
-e MYSQL_DATABASE=<db_name> \
mysql:5.7
```
* Crear container para app conectado a la network:
```shell
$ docker run -dp 3000:3000 \
--network <net_name> \
-e MYSQL_HOST=mysql \
-e MYSQL_USER=root \
-e MYSQL_PASSWORD=<pwd> \
-e MYSQL_DB=<db_name> \
<nombre_imagen>
```

## Docker compose
* Archivo para configuración de docker
* Así queda ordenado y facilmente lejible
* El archivo se debe llamar `docker-compose.yaml`
```yaml
version: "3.7"

services: #declaramos los servicios dentro del docker compose → automáticamente los pone en la misma red

#docker run -dp 3000:3000 --network todo-app -e MYSQL_HOST=mysql -e MYSQL_USER=root -e MYSQL_PASSWORD=secret -e MYSQL_DB=todos getting-started:v2

  app:
    image: pablokbs/getting-started:v2
    ports:
      - 3000:3000
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
 
# docker run -d     --network todo-app --network-alias mysql     -v todo-mysql-data:/var/lib/mysql     -e MYSQL_ROOT_PASSWORD=secret     -e MYSQL_DATABASE=todos     mysql:5.7

  mysql:
    image: mysql:5.7
    volumes:
      - ./todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos
```
* Correr el docker-compose: `$ docker-compose up -d`
* Eliminar contenedores docker-compose: `$ docker-compose down`

 
 *Apuntes sacados del vídeo de Pelado Nerd, [DOCKER 2021 - De NOVATO a PRO! (CURSO COMPLETO)](https://youtu.be/CV_Uf3Dq-EU)*
