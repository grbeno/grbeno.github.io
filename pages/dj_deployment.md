# Deploying Django

### Preparation
*	requirements.txt
    - Create a text file for listing dependencies
	    ```
        pip freeze > requirements.txt
	    ```
*	Git, GitHub
    - Create Git repository locally
    
    - Download Git: [https://git-scm.com/downloads](https://git-scm.com/downloads)
    
    - Install Git on your system
    
    - Check if the installation is succesful
        ```
        git --version
        ```
    
    - Initalize a Git repository in your project directory
        ```
        cd <project path>
        ```
        ```
        git init
        ```
    
    - Create a GitHub account if you haven't got yet: [https://github.com/](https://github.com/)
    
    - Create a new repository `Repository/new`, name the new repo, set to private or keep it public
    
    - Create a `.gitignore` file and add the names of the files to it that should be ignored while pushing to GitHub.
    
    - If you use React as well, copy the content of `frontend/.gitignore` (related to React) to the new `.gitignore` file in the project directory. 
    
    - Delete `frontend/.gitignore`
    
    - Create your `README.md` file and delete `frontend/README.md` (if you don't need it)

    - Connect the local Git repository to the remote GitHub repo
        ``` 
        git remote add origin <copy the path from github>
        ```
    - Add all files that are not ignored by `.gitignore`
        ```
        git add .
        ```
    - Make the first inital commit
        ```
        git commit -m 'init'
        ```
    - Push it to the GitHub
        ```
        git push -u origin main
        ```
*	Collect static files
    - Collect static files before deploying
      ```
      python manage.py collectstatic --noinput
      ```
*	`config/settings.py`
    - Add the url to the `ALLOWED_HOST` (<mysitename.up.railway.app e.g.)
*   Gunicorn
    - [https://pypi.org/project/gunicorn/](https://pypi.org/project/gunicorn/)
        ```
        pipenv install gunicorn
        ```
    - _Procfile_
        ```
        web: gunicorn config.wsgi --log-file
        ```

### Deploying to Railway
 
* Open Railway UI
	- Add new project
	- Add postgres as database
	- Set variables: `SECRET_KEY` `DATABASE_URL`
* Go to settings
	- Generate domain
	- Deploy: custom start command
		```
  		gunicorn.wsgi --log file -
		```
* Set `DOMAIN_NAME` variable then (config/settings.py):
	- Add the domain to `ALLOWED_HOST`
	- Add url to `CSRF_TRUSTED_ORIGINS`
* Create a nixpacks.toml file
	- If you want to deploy also a JS front-end library with node (ReactJS e.g.)
		```
		providers = ["node", "python"]
		```

### Deploying to Heroku

Open Heroku UI
```
cd <project-name>
```
```
heroku login
```
```
heroku git:remote <project-name>
```
If heroku does not support the python version which you would like to use in runtime.txt then ignore it (.slugignore) and use the default on the platform.
```
git push heroku main
```
Overview > heroku postgres > settings > database credentials. Then copy URI to `DATABASE_URL` variable in config vars.
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
