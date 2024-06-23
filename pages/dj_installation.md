---
layout: default
---

# Installation and setting up
This is a normal paragraph following a header. GitHub is a code hosting platform for version control and collaboration. It lets you and others work together on projects from anywhere.

OS: **Windows 11 Pro 64bit**

## Prerequisites

* **Python**
    - [https://www.python.org/downloads/windows/](https://www.python.org/downloads/windows/)
    - Install [pyenv](https://pypi.org/project/pyenv/) if you want to use multiple python versions for your projects.
    
* **IDE**
    - Integrated development environment
    - PyCharm, VS Code, Vim
    
* **Package manager**
    - pip/conda(for Anaconda/Miniconda environment)
    
My choices are **python 3.10**, **VS Code** and **pip**.
Although, I will use **pipenv** for Django web application development, because I also need to setup virtual environment.

### Pipenv

[https://pypi.org/project/pipenv/](https://pypi.org/project/pipenv/)

_Install globally_
```
pip install pipenv
```
_Change the directory according to the project path_
```
cd <the_path_of_your_project_directory>
```
_The installation command creates the Pipfile and Pipfile.lock with the dependencies and version infos_
```
pipenv install  <dependencies>  
```
_Activate the virtual environment_
```
pipenv shell
```

Alternativelly, you can use venv to setup virtual enviroment.

## Django installation
_Installing actual version. Be sure you are in the right directory!_
```
pipenv install django
```
If you want to install a specific version of Django, just use the version number.
```
pipenv install django==5.0.1
```
You can use >, <, <= or >= to specify the version.

## Setting up Django
1.  **Create Project**
    ```
    django-admin startproject config .
    ```
    This command creates a Django project in the actual directory and a config subdirectory inside it with the files `settings.py` `urls.py` `asgi.py` `wsgi.py`.
2.  **Create app**
    ```
    python manage.py startapp <app_name> 
    ```
    Check if the project works well on localhost!
    ```
    python manage.py runserver
    ```
    You should see a django „congratulations” page in the browser at: http://127.0.0.1:8000/ or http://localhost:8000/
3.  **Setting up Database**  
    You can make a decision to continue the default light-weight database engine sqlite3 or chose a larger one such as PostgreSQL, MySQL, MariaDB, Oracle.

    My choice is PostgreSQL.

    If your choice is the same, then you should make the next steps:
	
    **Download & install postgres**
	https://www.postgresql.org/download/windows/

    **Create database**
    ```
    psql -U postgres 
    ```
    ```
    CREATE DATABASE <db_name> WITH OWNER postgres; 
    ```
    _Install dj_database_url_
    ```
    pipenv install dj_database_url
    ```
    You have to set the environment variables for the database!
    My choice for this purpose is [environs](https://pypi.org/project/environs/).
    ```
    pipenv install environs
    ```
    _The env file should look like something similar_
    ```
    # .env

    DATABASE_URL=postgresql://postgres:<password>@localhost:5432/<db_name>
    SSL_REQUIRE=False
    DB_PASSWORD=<password>

    ```
    _config/settings.py_
    ```python
    import dj_database_url
    from environs import Env

    env = Env()
    env.read_env()
                    
    # DB configuered in .env file /DATABASE_URL/ -> dj_database_url.config() returns a dictionary
    db_config = dj_database_url.config(conn_max_age=600, ssl_require=env.bool('SSL_REQUIRE', default=True))
    DATABASES = {'default': db_config}
    ```
    
    Finally, you can migrate the datas into the database.
    ```
    python manage.py makemigrations
    ```
    ```
    python manage.py migrate
    ```
 
 
