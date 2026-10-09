# Práctica de aula: Docker y Apache

Este documento detalla todos los pasos seguidos para completar la práctica de Docker y Apache utilizando contenedores, Dockerfile y Docker Compose.

# 1. Descargar la imagen de Debian
Se ha descargado la imagen oficial de Debian desde el Docker Hub:
docker pull debian
<img width="1897" height="960" alt="image" src="https://github.com/user-attachments/assets/550abcff-1959-439f-a615-9eda5c0b6c77" />

# 2. Arrancar el contenedor interactivo en segundo plano
Se ha arrancado un contenedor de Debian en modo interactivo (`-it`) y en segundo plano (`-d`), asignándole un nombre y mapeando el puerto 8080 del host con el puerto 80 del contenedor:
docker run -d -it --name mi-contenedor-apache-maria -p 8080:80 debian

# 3. Ejecutar una shell bash en el contenedor
Para acceder al interior del contenedor en ejecución:
docker exec -it mi-contenedor-apache-maria bash

# 4. Instalar y arrancar el servidor Apache
Una vez dentro de la shell del contenedor, se actualizaron los repositorios, se instaló Apache y el navegador en modo texto `elinks`, y se arrancó el servicio:

apt update && apt install -y apache2 elinks
service apache2 start

# 5. Comprobar desde el navegador web
Se accedió desde el navegador web del host a la dirección `http://localhost:8080` para verificar que el servidor web respondía correctamente mostrando la página por defecto de Apache.

# 6. Crear una página HTML personalizada
Se creó un fichero HTML personalizado llamado `Maria.html` dentro de la ruta raíz del servidor web del contenedor (`/var/www/html/`).

# 7. Acceder a la página desde el navegador
Se comprobó el correcto funcionamiento accediendo a la URL correspondiente desde el navegador web.

# 8. Acceder mediante el navegador de línea de comandos `elinks`
Se comprobó la accesibilidad de la página utilizando el navegador web en modo texto dentro del contenedor:
elinks http://localhost/Maria.html

# 9. Crear el fichero Dockerfile para automatizar los pasos
Se creó un fichero `Dockerfile` en la máquina host con las siguientes instrucciones para automatizar la instalación y despliegue:

dockerfile
FROM debian:latest
RUN apt update && apt install -y apache2 
COPY Maria.html /var/www/html/Maria.html
EXPOSE 80
CMD ["apache2ctl", "-D", "FOREGROUND"]

## 10. Crear la imagen a partir del Dockerfile
Se construyó la imagen personalizada utilizando el comando `docker build`:

docker build -t maria-debian-apache .

# 11. Ejecutar el contenedor basado en la nueva imagen
Se levantó un contenedor utilizando la imagen generada por el Dockerfile:

docker run -d -p 8080:80 --name apache-desde-dockerfile maria-debian-apache

# 12. Comando que copia un archivo local a un contenedor
Se utilizó el comando `docker cp` para transferir un fichero HTML local directamente al contenedor en marcha:
docker cp Maria.html mi-contenedor-apache-maria:/var/www/html/Maria.html

# 13. Crear el fichero docker-compose.yml
Se creó un fichero `docker-compose.yml` para automatizar el arranque de Apache asociando un volumen local que mapea la carpeta raíz de los documentos HTML:

services:
  web:
    image: maria-debian-apache:latest
    container_name: apache-maria-compose
    ports:
      - "8082:80"
    volumes:
      - ./html_local:/var/www/html

# 14. Ejecutar el entorno con Docker Compose
Se arrancó el servicio mediante Docker Compose en segundo plano:
docker compose up -d



