---
layout: default
---

# Installation and setting up
This is a normal paragraph following a header. GitHub is a code hosting platform for version control and collaboration. It lets you and others work together on projects from anywhere.

OS: Windows 11 Pro 64bit

## Prerequisites

*   **Python**
    - https://www.python.org/downloads/windows/
    - Install pyenv if you want to use multiple python versions for your projects.
*   **IDE**
    -	Integrated development environment
    -	PyCharm, VS Code, Vim
*   **Package and dependency manager**
    - pip/conda(for Anaconda/Miniconda environment)

My choices are python 3.10, VS Code and pip.
Although, I will use pipenv for Django web application development, because I also need to setup virtual environment.

### Pipenv

https://pypi.org/project/pipenv/

_install globally_
```
pip install pipenv
```
```
cd <the_path_of_your_project_directory>
```
```
pipenv install  <dependencies>  
```
_activate the virtual environment_
```
pipenv shell
```

Alternativelly, you can use venv to setup virtual enviroment.

## Django installation

## Setting up Django
1.  ### Create Project
2.  ### Create app
3.  ### Setting up Database  
    ...

_config/settings.py_
```py
import dj_database_url
                
# DB configuered in .env file /DATABASE_URL/ -> dj_database_url.config() returns a dictionary
db_config = dj_database_url.config(conn_max_age=600, ssl_require=env.bool('SSL_REQUIRE', default=True))
DATABASES = {'default': db_config}
```
 
 
