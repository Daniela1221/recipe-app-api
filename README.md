# Paso a paso
# Settiando el proyecto

En GitHub, se configuran los secretos del repositorio, se añade uno para el usuario de DockerHub, y luego se añade otro secreto con el Token creado en DockerHub.

Configurar Docker y Django
- Los comandos se corren en docker

**`docker-compose run --rm app sh -c "python manage.py collectstatic"`**

Las partes cambiables del comando es "app" que representa lo que vamos a estar corriendo, y el comando de django que deseemos utilizar.

Se crean los archivos:
- .dockerignore
- Dockerfile
- app (carpeta vacía)

Se ejecuta el comando `docker build .` en la terminal en la carpeta donde se encuentra el proyecto para crear el docker image.

Se crea el archivo:
- docker-compose.yml

Se ejecuta el comando `docker-compose build` en la terminal en la carpeta donde se encuentra el proyecto para crear la configuración de docker compose.

Se instala flake8 para implementar Liting, es decir, una librería de manejo de errores. Se crea un nuevo archivo requirements.dev.txt. Se crea el archivo .dockerignore. Se modifica el archivo docker-compose.yml y se agrega lo siguiente a la altura de context, dentro de build:

`      args:`
`        - DEV=true`

Se agrega lo siguiente al archivo Dockerfile:
- Debajo de `COPY ./requirements.txt /tmp/requirements.txt`:
**COPY ./requirements.dev.txt /tmp/requirements.dev.txt**

- Arriba de `RUN python -m venv /py && \`:
**ARG DEV=false**

- Debajo de `/py/bin/pip install -r /tmp/requirements.txt && \`:
`if [ $DEV = "true" ]; \ `
`        then /py/bin/pip install -r /tmp/requirements.dev.txt ; \ `
`    fi && \ `

Una vez implementada las modificaciones, se corre el comando `docker-compose build` para implementar los cambios.
Para configurar flake8, creamos el archivo .flake8 dentro de la carpeta app. Luego corremos la línea de código siguiente en la terminal:

**docker-compose run --rm app sh -c "flake8"**

Procedemos a crear el proyecto Django con la línea de código siguiente

**docker-compose run --rm app sh -c "django-admin startproject app ."**

Para visualizar el proyecto en el navegador con docker corremos el comando

**docker-compose up**

