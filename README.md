# COURSE: BUILD A BACKEND REST API WITH PYTHON & DJANGO - ADVANCED
# 1. Project SetUp

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


# 2. Configure GitHub Actions

Se crea la carpeta .github a la altura de app. Dentro de esta creamos otra carpeta workflows, y aquí creamos el archivo check.yml . Una vez realizado, se corre el comando por consola:

**docker-compose run --rm app sh -c "python manage.py test"**

En Docker Hub generamos un token de acceso, al cual le ponemos el nombre del proyecto. Guardamos la pestaña que nos da el token y vamos a GitHub, donde añadimos dos secretos al repositorio:

DOCKERHUB_USER : se pone el nombre de la cuenta de Docker HUb.
DOCKERHUB_TOKEN : se copia el token proporcinado por Docker Gub y lo ponemos en este campo.

Luego, al hacer commit y pushear el código a GitHub, en el apartado de Actions te salen los commits y si se implementaron correctamente o tienen algún error.


# 3. Test Driven Development with Django

Las test corren por parte de Django test framework y adicionalmente lo tiene Django Rest Framework.
Para correr las pruebas, estas se corren en una base de datos a parte. Aquí se ocupa el archivo test.py que se añade en cada aplicación.

Hay varios tipos de test:
- SimpleTestCase: se usa para tests que no requieren información de la base de datos.
- TestCase: se usa para correr pruebas que requieren la base de datos.

Para ocupar esto en test.py se importa el tipo de test:

**from django.test import SimpleTestCase**

**from app_two import views** o lo que se quiera incorporar al test.

Se define la clase que se va a utilizar: `class ViewsTests(SimpleTestCase):`

Se le implementa el método y la configuración de inputs, para luego ejecutar el código de test.

Se implementa el checkeo de lo que debería arrojar la prueba para probar los resultados con el código `self.assertEqual`.

Una comprobar las tests, se corre el siguiente comando:

**python manage.py test**

Lo cual muestra si las pruebas son correctas o si hubo algún error.

Mocking: sobrescribir o cambiar un comportamiento de las dependencias.


# 4. Configure Database

Se utiliza la base de datos PostgreSQL. Lo cual se configura en el archivo docker-compose.yml ; entonces se agrega al existente archivo lo siguiente:

- Debajo de command (y a la altura de este) se agrega:

`    environment:`
`      - DB_HOST=db`
`      - DB_NAME=devdb`
`      - DB_USER=devuser`
`      - DB_PASS=changeme`
`    depends_on:`
`      - db`

- Debajo de lo último agregado, pero a la altura de app se agrega:

`  db:`
`    image: postgres:13-alpine`
`    volumes:`
`      - dev-db-data:/var/lib/postgresql/data`
`    environment:`
`      - POSTGRES_DB=devdb`
`      - POSTGRES_USER=devuser`
`      - POSTGRES_PASSWORD=changeme`

- A continuación, a la altura de services, se agrega:

`volumes:`
`  dev-db-data:`

Luego de efectuar los cambios, corremos la línea de comando en cmd:

**docker-compose up**

Para guardar los cambios realizados en docker.

Ahora, se instalará postgres (el paquete psycopg2 actúa como intermediario entre postgres y django, pero no es bueno para la puesta en producción, como AWS si se instala la versión binaria).

En el archivo Dockerfile agregamos lo siguiente:

- Debajo de la línea de código `/py/bin/pip install --upgrade pip && \` agregar:

`    apk add --update --no-cache postgresql-client && \`
`    apk add --update --no-cache --virtual .tmp-build-deps \`
`        build-base postgresql-dev musl-dev && \`

- Debajo de la línea de código `rm -rf /tmp && \` agregar:

`    apk del .tmp-build-deps && \`

Luego, en el archivo requirements.txt agregamos `psycopg2>=2.8.6,<2.9`.
Ahora guardamos los cambios en docker escribiendo las siguientes líneas de comando en cmd:

**docker-compose down**
**docker-compose build**