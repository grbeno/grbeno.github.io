# Deploying Django

## Preparation
*	requirements.txt
    - Create a text file for listing dependencies
	    ```
        pip freeze > requirements.txt
	    ```
*	git, github
    - Create git and github repository
    - [https://docs.github.com/en/repositories/creating](https://docs.github.com/en/repositories/creating-and-managing-repositories/quickstart-for-repositories)
*	collectstatic
    - Collect static files before deploying
      ```
      python manage.py collectstatic --noinput
      ```
*	config/settings.py
    - Add the url to the `ALLOWED_HOST` (<site_name>up.railway.app e.g.)
*   Gunicorn
    - [https://pypi.org/project/gunicorn/](https://pypi.org/project/gunicorn/)
        ```
        pipenv install gunicorn
        ```
    - _Procfile_
        ```
        web: gunicorn config.wsgi --log-file
        ```

## Deploying to Railway
 
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
	- Build: custom build command
		```
		CI=false react-scripts build
		```
* Set `DOMAIN_NAME` variable then (config/settings.py):
	- Add the domain to `ALLOWED_HOST`
	- Add url to `CSRF_TRUSTED_ORIGINS`
* Create a nixpacks.toml file
	- If you want to deploy also a JS front-end library with node (ReactJS e.g.)
		```
		providers = ["node", "python"]
		```

## Deploying to Heroku

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

## Docker-compose
 - ...
