## Django with React Consuming LLM-API

This LLM-API consumer, that I am presenting here, is built upon a Django application using React as static files. With the API consumer, I implemented a language corrector and translator application, which is a kind of playground to try creating something great.

`Django` `ReactJS` `PostgeSQL` `Django REST framework`

You can find here the [GitHub repository](https://github.com/grbeno/dj-rjs-template) of the template, which is the foundation or starting point of the application.

### Setting up the environment

1. Download the `.zip` from GitHub
2. Move the files to a new directory. Let's name it `llmlang`.
3. Open CLI and change the directory using the `$ cd ` command
4. Create a virtual environment for the project
    
    ```
    python -m venv venv
    ```
5. Activate the enviroment
    ```
    venv\scripts\activate
    ```
6. Install the dependencies
    ```
    pip install -r requirements.txt
    ```
7. Create an .env file and update with the next environment variables
    
    Generate new key for the `SECRET_KEY`  
    ```
    python -c 'import secrets;print(secrets.token_hex(32))'
    ```
    
    ```
    # .env

    SECRET_KEY=<generated-secret-key>
    SSL_REQUIRE=False

    ```
8. Create new database
    
    Installing postgres on your device is necessarry.
    
    Open CLI

    ```
    psql -U postgres
    ```
    Create a database, let's name it langapp.
    ```
    CREATE DATABASE langapp WITH OWNER postgres; 
    ```
    Add `DATABASE_URL=postgresql://postgres:<password>@localhost:5432/langapp` to the `.env`
9. Migrate data
    ```
    python manage.py makemigrations accounts
    ```
    ```
    python manage.py migrate
    ```

### Starting the language app development
