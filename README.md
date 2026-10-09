Este documento detalla todos los pasos seguidos para completar la práctica de Docker y Apache utilizando contenedores, Dockerfile y Docker Compose.

# 1. Descargar la imagen de Debian
Se ha descargado la imagen oficial de Debian desde el Docker Hub:
docker pull debian
<img width="1897" height="960" alt="image" src="https://github.com/user-attachments/assets/550abcff-1959-439f-a615-9eda5c0b6c77" />
<img width="1018" height="257" alt="image" src="https://github.com/user-attachments/assets/4ec6e16b-bfa4-4999-af92-c5ff4bc65ee2" />
docker images debian:trixie-backports 
<img width="1468" height="120" alt="image" src="https://github.com/user-attachments/assets/8a1153fc-0729-4792-812f-05be7dca2ee0" />
<img width="1892" height="562" alt="image" src="https://github.com/user-attachments/assets/fb8ae299-70ca-4377-a593-ccbb0010e783" />

# 2. Arrancar el contenedor interactivo en segundo plano (detached)
Se ha arrancado un contenedor de Debian en modo interactivo (`-it`) y en segundo plano (`-d`), asignándole un nombre y mapeando el puerto 8080 del host con el puerto 80 del contenedor:
docker run -d -it --name mi-contenedor-apache-maria -p 8080:80 debian
<img width="1591" height="310" alt="image" src="https://github.com/user-attachments/assets/db56f7d4-426e-4486-be95-14a9ff6842e9" />
Comprobación:
Comando: docker ps – muestra que el contenedor esta activo.
<img width="1470" height="140" alt="image" src="https://github.com/user-attachments/assets/93bd21f8-7f63-4af7-9df3-0725088697df" />
<img width="1550" height="147" alt="image" src="https://github.com/user-attachments/assets/32392fd5-6854-4f84-9e9c-0c77d6a8afed" />

# 3. Ejecutar una shell bash en el contenedor
Para acceder al interior del contenedor en ejecución:
docker exec -it mi-contenedor-apache-maria bash
<img width="1120" height="157" alt="image" src="https://github.com/user-attachments/assets/60003167-cb32-460b-906e-74d76223df73" />

# 4. Instalar y arrancar el servidor Apache
Una vez dentro de la shell del contenedor, se actualizaron los repositorios, se instaló Apache y el navegador en modo texto `elinks`, y se arrancó el servicio:

apt update && apt install -y apache2 elinks
<img width="1378" height="310" alt="image" src="https://github.com/user-attachments/assets/f881d628-4268-4420-8592-c20c410fe6d7" />
service apache2 start
<img width="1368" height="123" alt="image" src="https://github.com/user-attachments/assets/79898e2f-9e68-4b79-8f89-d78af8532dd8" />

# 5. Comprobar desde el navegador web
Se accedió desde el navegador web del host a la dirección `http://localhost:8080` para verificar que el servidor web respondía correctamente mostrando la página por defecto de Apache.
<img width="1597" height="957" alt="image" src="https://github.com/user-attachments/assets/75ed2ecf-6074-47cc-8f55-99da732936ec" />

# 6. Crear una página HTML personalizada
Se creó un fichero HTML personalizado llamado `Maria.html` dentro de la ruta raíz del servidor web del contenedor (`/var/www/html/`).
<img width="1377" height="125" alt="image" src="https://github.com/user-attachments/assets/d3a38915-5f5f-4077-847b-2a22b7b5d0a7" />

# 7. Acceder a la página desde el navegador
Se comprobó el correcto funcionamiento accediendo a la URL correspondiente desde el navegador web.
<img width="1205" height="317" alt="image" src="https://github.com/user-attachments/assets/8570b61e-610c-4382-94e2-67917eb90cbf" />

# 8. Acceder mediante el navegador de línea de comandos `elinks`
Se comprobó la accesibilidad de la página utilizando el navegador web en modo texto dentro del contenedor:
elinks http://localhost/Maria.html
<img width="1377" height="322" alt="image" src="https://github.com/user-attachments/assets/cb0294b9-9866-49e0-a9bc-09b1866d3b1e" />
<img width="1385" height="308" alt="image" src="https://github.com/user-attachments/assets/bc8ec0db-fbc0-42cb-8a35-9b17cf3e5d99" />

# 9. Crear el fichero Dockerfile para automatizar los pasos
Se creó un fichero `Dockerfile` en la máquina host con las siguientes instrucciones para automatizar la instalación y despliegue:

dockerfile
FROM debian:latest
RUN apt update && apt install -y apache2 
COPY Maria.html /var/www/html/Maria.html
EXPOSE 80
CMD ["apache2ctl", "-D", "FOREGROUND"]
<img width="1147" height="312" alt="image" src="https://github.com/user-attachments/assets/87fd6d90-5f77-44a7-9054-77fd6cf7b1c5" />

# 10. Crear la imagen a partir del Dockerfile
Se construyó la imagen personalizada utilizando el comando `docker build`:

docker build -t maria-debian-apache .
<img width="1152" height="422" alt="image" src="https://github.com/user-attachments/assets/a71c6e54-6124-4b01-971f-d402655323dd" />

# 11. Ejecutar el contenedor basado en la nueva imagen
Se levantó un contenedor utilizando la imagen generada por el Dockerfile:
<img width="1153" height="237" alt="image" src="https://github.com/user-attachments/assets/b6e9bab0-a11a-45c4-9149-8d676d37ed90" />

docker run -d -p 8080:80 --name apache-desde-dockerfile maria-debian-apache
<img width="1168" height="93" alt="image" src="https://github.com/user-attachments/assets/227731ae-7455-4ae4-a2ef-a229d40e5492" />

# 12. Comando que copia un archivo local a un contenedor
Se utilizó el comando `docker cp` para transferir un fichero HTML local directamente al contenedor en marcha:
docker cp Maria.html mi-contenedor-apache-maria:/var/www/html/Maria.html
<img width="1181" height="77" alt="image" src="https://github.com/user-attachments/assets/72bd8bff-72ce-4085-b340-a5b245059a8f" />

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
<img width="1562" height="275" alt="image" src="https://github.com/user-attachments/assets/51e55b76-ae2b-4cbc-bcac-5e8b3675592a" />




