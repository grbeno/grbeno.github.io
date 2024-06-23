# Deploying Django

## Preparation
*	requirements.txt
*	git, github
*	collectstatic
*	settings.py
    - ALLOWED_HOST, STATIC vars, CSRF_TRUSTED_ORIGINS ...
* . . . 

## Deploying to Railway
 
 - Open Railway UI
 - Add new project
 - Add postgres as database
 - Set variables: SECRET_KEY, DATABASE_URL
 - Go to settings
 - Generate domain
 - Deploy > custom start command > ``` gunicorn.wsgi --log file - ```
 - Build > custom build command > ``` CI=false react-scripts build ```
 - Set DOMAIN_NAME variable then (config/settings.py):
   - Add the domain to ALLOWED_HOST
   - Add url to CSRF_TRUSTED_ORIGINS
- Create a nixpacks.toml file  > ``` providers = ["node", "python"] ```

## Deploying to Heroku

- Open Heroku UI
- ``` $ cd <project-name> ```
- ``` $ heroku login ```
- ``` $ heroku git:remote <project-name> ```
- If heroku does not support the python version which you would like to use in runtime.txt then ignore it (.slugignore) and use the default on the platform.
- ``` $ git push heroku main ```
- Overview > heroku postgres > settings > database credentials
- Copy URI to DATABASE_URL variable in config vars
- ``` $ heroku run python manage.py migrate ```

## Docker-compose
 - ...
