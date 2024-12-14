# Deploying Django

### Preparation
*   Complete the `.env` file with additional variables

    If your `.env` file doesn't exist yet, create one and use a tool to load the variables. My choice is `environs`
    ```
    pip install environs
    ```
    In the case of using `dj-database-url` to connect to postgres database, the `.env` file already includes at least one variable `DATABASE_URL`.

    Consider adding the following variables to the `.env` file before deploying: `SECRET_KEY`, `DEBUG`, and other data or passwords, as well as API keys that you don't want to share in production.

    For generating secret key use this command:
    ```
    python -c 'import secrets;print(secrets.token_hex(32))'
    ```
    Set `DEBUG` to true in development mode, and to false by default.
    ```python
    # config/settings.py

    # SECURITY WARNING: keep the secret key used in production secret!
    SECRET_KEY = env.str('SECRET_KEY')

    # SECURITY WARNING: don't run with debug turned on in production!
    DEBUG = env.bool('DEBUG', default=False)
    ```
    After deploying you might have to set these variables in your choosen service or cloud platform as well.
*	**Git, GitHub**
     
     Create Git repository locally

     Download Git: [https://git-scm.com/downloads](https://git-scm.com/downloads)

     Install Git on your system

     Check if the installation is succesful
    ```
    git --version
    ```
    
    Initalize a Git repository in your project directory
        
    ```
    cd <project path>
    ```
    ```
    git init
    ```
    
    Create a GitHub account if you haven't got yet: [https://github.com/](https://github.com/)

    Create a new repository `Repository/new`, name the new repo, set to private or keep it public

    Create a `.gitignore` file and add the names of the files to it that should be ignored while pushing to GitHub.

    *Typically, the `.env` file, virtual environment, and other non-public data, is placed in this file.*

    If you use React as well, copy the content of `frontend/.gitignore` (related to React) to the new `.gitignore` file in the project directory. 

    Delete `frontend/.gitignore`

    Create your `README.md` file and delete `frontend/README.md` (if you don't need it)

    Connect the local Git repository to the remote GitHub repo
    ``` 
    git remote add origin <copy the path from github>
    ```
    Add all files that are not ignored by `.gitignore`
    ```
    git add .
    ```
    Make the first inital commit
    ```
    git commit -m 'init'
    ```
    Push it to the GitHub
    ```
    git push -u origin main
    ```
*	**Collect static files**
    
    Add `STATIC_ROOT` to `config/settings.py`
    
    Install whitenoise
    ```
    pipenv install whitenoise
    ```
    ```
    MIDDLEWARE = [
        # ...
        "django.middleware.security.SecurityMiddleware",
        "whitenoise.middleware.WhiteNoiseMiddleware",
        # ...
    ]
    ```
    Collect static files before deploying
    ```
    python manage.py collectstatic --noinput
    ```
*	`config/settings.py`
    - Add the url to the `ALLOWED_HOST` (<mysitename.up.railway.app e.g.) or set to '*' temporarily.
*   **Gunicorn**
    [https://pypi.org/project/gunicorn/](https://pypi.org/project/gunicorn/)
    ```
    pipenv install gunicorn
    ```
    _Procfile_
    ```
    web: gunicorn config.wsgi --log-file -
    ```
* **requirements.txt**
    Create a text file for listing dependencies
	```
    pip freeze > requirements.txt
	```

### Deploying to Railway
 
* Open Railway **Dashboard**
	
    Add **New** Project
	
    Add Postgres as database \
    
    Select **Deploy PostgreSQL** option
    
    **Create +** and select the GitHub repo to deploy
	
    Set **Variables**
    `SECRET_KEY` from your `.env` file\
    `DATABASE_URL` from Railway (*offered automatically*)
* Go to **Settings**
    **Networking\Public Networking**
    Generate or add a custom one
	**Deploy\Custom Start Command**
	```
  	gunicorn config.wsgi
	```
* Update `config/settings.py`
	
    Add the **domain** to `ALLOWED_HOST`
	
    Add the **url** to `CSRF_TRUSTED_ORIGINS`
* Create a `nixpacks.toml` file
	If you want to deploy also a JS front-end library with NodeJS (ReactJS e.g.)
	```
	providers = ["node", "python"]
	```

### Deploying to Heroku

Create an account on Heroku\
Install **Heroku CLI**
```
cd <project-name>
```
Login to Heroku
```
heroku login
```
Open Heroku **Dashboard**\
Create **New Project** in Heroku
```
heroku git:remote <project-name>
```
If heroku does not support the python version which you would like to use in runtime.txt then ignore it (.slugignore) and use the default on the platform.

Overview > heroku postgres > settings > database credentials. Then copy URI to `DATABASE_URL` variable in config vars.
Add `SECRET_KEY` and `DISABLE_COLLECTSTATIC` and setting to 1, if needed.

If you want to deploy also a JS front-end library with NodeJS (ReactJS e.g.)
> A Node.js app on Heroku requires a 'package.json' at the root of the directory structure.
> If you are trying to deploy a Node.js application, ensure that this
> file is present at the top level directory.

- Add **heroku/nodejs** Buildpack

Add your heroku domain to the `ALLOWED_HOST` in `config/settings.py`

Finally,
```
git push heroku main
```
```
heroku run python manage.py migrate
```

### Docker

__Sign up/in Docker Hub__ \
[https://app.docker.com/signup](https://app.docker.com/signup)

__Download and install Docker Desktop:__ 
[https://docs.docker.com/desktop/install/windows-install/](https://docs.docker.com/desktop/install/windows-install/)

Check if the installation is successful, then run the 'hello world' image.
```
docker --version
```
```
docker run hello-world
```
Create a requirements text file, if you haven't created one yet, that will be used by the Dockerfile to install the dependencies in the container.
```
pip freeze > requirements.txt
```
    
__Dockerfile__
```docker
# Pull base image
FROM python:3.11-slim-bullseye

# Set environment variables
ENV PIP_DISABLE_PIP_VERSION_CHECK 1
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Set work directory
WORKDIR /code

# Install dependencies
COPY ./requirements.txt .
RUN pip install -r requirements.txt

# Copy project
COPY . /code/
```
__docker-compose.yml__
```docker
services:

backend:
    build: .
    container_name: my_helloworld
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
    - .:/code
    ports:
    - 8000:8000
    depends_on:
    - db
    environment:
    - "SECRET_KEY=${SECRET_KEY}" # .env
    - "DEBUG=True"
    - "DATABASE_URL=postgres://postgres:postgres@db:5432/postgres"
    - "SSL_REQUIRED=False"

db:
    image: postgres:16.2
    container_name: my_helloworld_db
    ports:
    - '5432:5432'
    volumes:
    - postgres_data:/var/lib/postgresql/data/
    environment:
    - "POSTGRES_HOST_AUTH_METHOD=trust"

volumes:
postgres_data:
```
According to the content of compose yaml file, generate secret key and add to the `.env` file.
```python
python -c 'import secrets;print(secrets.token_hex(32))'
```
```python
# .env

SECRET_KEY="<generated with the command above>"
DEBUG=True  # Set the `DEBUG` variable as well
```
Modify secret key and the debug in the config/settings.py
```python
# config/settings.py

SECRET_KEY = env.str('SECRET_KEY')
DEBUG = env.bool('DEBUG', default=False)
```
Build docker-compose
```
docker-compose build
```
Use Django commands to migrate data and create superuser
```
docker-compose exec backend python manage.py migrate
```

```
docker-compose exec backend python manage.py createsuperuser
```

Start services based on docker-compose.yml
```
docker-compose up -d
```
Open the application in the web-browser
```
http://localhost:8000/
```
To turn off the services
```
docker-compose down
```
To run any Django commands
```
docker-compose exec <django_service_name> python manage.py <command>
```

__Use this repo's README for more information about the Dockerfile and docker-compose.yml.__\
[https://github.com/grbeno/django-postgres-docker](https://github.com/grbeno/django-postgres-docker)
