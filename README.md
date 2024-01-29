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

Ahora, en settings.py registramos la nueva base de datos instalada, en el apartado de DATABASE, quedando de la siguiente forma:

`DATABASES = {`
`    'default': {`
`        'ENGINE': 'django.db.backends.postgresql',`
`        'HOST': os.environ.get('DB_HOST'),`
`        'NAME': os.environ.get('DB_NAME'),`
`        'USER': os.environ.get('DB_USER'),`
`        'PASSWORD': os.environ.get('DB_PASS'),`
`    }`
`}`

Y borramos la base de datos proporcinada por Django de sqlite3.

Creamos otra app llamada "core" con el comando:

**docker-compose run --rm app sh -c "python manage.py startapp core"**

Borramos los archivos views.py y tests.py de la aplicación core, para luego crear la carpeta tests y dentro de ella creamos el archivo __init__.py . Por último regitramos la nueva aplicación en setings.py

Creamos la carpeta management > commands donde cada una tiene el archivo __init__.py y dentro de commands creamos el archivo wait_for_db.py con el fin de crear una función que haga  Django esperar a que la base de datos se cargue, ya que esta tiene un delay tras ocupar Docker.

Luego implementamos una prueba en la carpeta test para comprobar que el comando hace lo que esperamos.

Una vez lista la prueba, corremos el comando en la cmd:

**docker-compose run --rm app sh -c "python manage.py test"**

Probamos directamente el comando creado en la cmd:

**docker-compose run --rm app sh -c "python manage.py wait_for_db"**

Luego corroboramos que el linting (manejo de errores) esté funcionando correctamente.

**docker-compose run --rm app sh -c "python manage.py test && flake8"**

Para ir finalizando la sección, modificamos el docker-compose.yml en la linea de comando que se encuentra asociada al apartado "command", de la siguiente forma:

`    command: >`
`      sh -c "python manage.py wait_for_db &&`
`             python manage.py migrate &&`
`             python manage.py runserver 0.0.0.0:8000"`

Y se modifica el archivo checks.yml en el apartado "name: Test", quedando de la siguiente forma:

`      - name: Test`
`        run: docker-compose run --rm app sh -c "python manage.py wait_for_db && python manage.py test"`

Finalizando con actualizando los cambios en docker:

**docker-compose down**

**docker-compose up**


# 5. Create User Model

Se crea una prueba para probar la implementación de un modelo usuario en la app core.

Ahora se crea el modelo usuario en models.py, y se registra en settings.py al final del código, aplicando la variable siguiente:

**AUTH_USER_MODEL = 'core.User'**

Luego se corren las migraciones:

**docker-compose run --rm app -sh c "python manage.py makemigrations**

**docker-compose run --rm app -sh c "python manage.py wait_for_db && python manage.py migrate**

Al existir previamente la base de datos, arrojará un error así que buscamos la base de datos con el comando:

**docker volume ls**

Y el último agregado será nuestra base de datos, entonces:

**docker volume rm nombre_base_datos**

**docker-compose down**

Podemos corroborar que se ha removido, volviendo a ejecutar:

**docker volume ls**

Se vuelven a correr las migraciones y se prueban las test:

**docker-compose run --rm app sh -c "python manage.py wait_for_db && python manage.py migrate"**

**docker-compose run --rm app sh -c "python manage.py test"**

A continuación se crea una prueba para normalizar el email, de tal forma que no importa si tiene matúsculas o no, lo acepte igual.
Luego se modifica el archivo models.py, en la clase UserManager, dentro del método create_user, cambios:

**user = self.model(email=email, **extra_field)**

Por:

**user = self.model(email=self.normalize_email(email), **extra_field)**

Hacemos correr las pruebas para ver si la implementación es correcta con:

**docker-compose run --rm app sh -c "python manage.py test"**

Ahora hacemos que la variable email sea obligatoria para todos los usuarios, y si no tiene este campo, se levante un error.
Primero se genera una test, y luego en models.py se crea una línea por sobre `user = self.model(email=self.normalize_email(email), **extra_field)`, añadiendo:

`        if not email:`
`            raise ValueError('User must have an email address.')`

Luego corremos las pruebas para corroborar la implementación.

Ahora, se creará un super user, de lo cual se genera una test y luego dentro del modelos UserManager, debajo del método crete_user, se agrega el método create_superuser de la siguiente forma:

`    def create_superuser(self, email, password):`
`        """Create and return a new superuser."""`
`        user = self.create_user(email, password)`
`        user.is_staff = True`
`       user.is_superuser = True`
`        user.save(using=self._db)`
` `
`        return user`

Para finalizar esta sección, aplicamos los cambios en docker

**docker-compose up**

Podemos ver que el servidor corre correctamente y arroja la página de admin login.
Para agregar el admin, corremos el siguiente comando:

**docker-compose run --rm app sh -c "python manage.py createsuperuser**

Los datos ocupados para admin son:

- email: admin@example.com
- pass: 1234


# 6. SetUp Django Admin

Se crea una test y se implementa con el comando `docker-compose run --rm app sh -c "python manage.py test"`.

En admin.py se importa el modelo User y además se crea una clase UserAdmin para ordenar el contenido del admin de Django. 

`from django.contrib import admin`
`from django.contrib.auth.admin import UserAdmin as BaseUserAdmin`
`from django.utils.translation import gettext_lazy as _`
` `
`from core import models`
` `
` `
`class UserAdmin(BaseUserAdmin):`
`    """Define the admin pages for users."""`
`    ordering = ['id']`
`    list_display = ['email', 'name']`

Luego se registra el modelo User y al clase UserAdmin. Se comprueban los cambios con el siguiente comando:

**docker-compose up**

Se vuelve a modificar en admin.py para que el admin de Django lea y no detecte que faltan parámetros por definir, y setearlo de la forma que queremos. Para esto, dentro de la clase UserAdmin, debajo de la línea de código `list_display = ['email', 'name']`, agregamos:

`    fieldsets = (`
`        (None, {'fields': ('email', 'password')}),`
`        (`
`            _('Permissions'),`
`            {`
`                'fields': (`
`                    'is_active',`
`                    'is_staff',`
`                    'is_superuser',`
`                )`
`            }`
`        ),`
`        (_('Important dates'), {'fields': ('last_login',)}),`
`    )`
`    readonly_fields = ['last_login']`

Y volvemos a montar el servidor (con esta modificación, se deja de tener errores en tests_admin).

**docker-compose up**

Para finalizar la sección, se configura la creación de usuarios desde el admin de Django.
En admin.py, en la clase UserAdmin, debajo de la línea de código `readonly_fields = ['last_login']`, se agrega lo siguiente:

`    add_fieldsets = (`
`        (None, {`
`            'classes': ('wide',),`
`            'fields': (`
`                'email',`
`                'password1',`
`                'password2',`
`                'name',`
`                'is_active',`
`                'is_staff',`
`                'is_superuser',`
`            )`
`        }),`
`    )`

Y montamos el servidor:

**docker-compose up**


# 7. API Documentation

Se va documentar de forma automática mediante una librería de Django Rest Framework, la cual es django rest framework spectacular. 

Abriremos requirements.txt y agregaremos la siguiente línea de código:

**drf-spectacular>=0.15.1,<0.16**

Luego volveremos a montar docker:

**docker-compose build**

E inscribimos la app en settings.py como `drf-spectacular` y `rest_framework`. Además, al final del archivo incluimos:

`REST_FRAMEWORK = {`
`    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',`
`}`

En urls.py de la app, inscribimos la librería de spectacular, entonces agregamos un nuevo import: 

`from drf_spectacular.views import (`
`    SpectacularAPIView,`
`    SpectacularSwaggerView,`
`)`

Y se añaden dos paths:

`    path('api/schema/', SpectacularAPIView.as_view(), name='api-schema'),`
`    path('api/docs/', SpectacularSwaggerView.as_view(url_name='api-schema'),name='api-docs'),`

Levantamos el server de docker:

**docker-compose up**

Y podemos ver la interacción en `http://localhost:8000/api/docs/`.


# 8. Build User API