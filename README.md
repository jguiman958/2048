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

Ejecutando mediante run sentencias anidades como actualización de repositorios instalación de git, instalación de apache2 y por ultimo borrar contenido de los paquetes.

El siguiente "run" ejecuta un clone para descargar un repositorio desde github siguiente mueve todo el contenido de app a /var/www/html y seguidamente borra todo el contenido de app.

Y por ultimo en el docker se añade por terminal que ese proceso se realice en primer plano.
