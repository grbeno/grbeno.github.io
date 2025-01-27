## Deploying Async LLM-Chat with Message History

### 1. Redis Memory Channel Layer

There is a Django web application with ReactJS as the frontend, which presents a chatbot with LLM using OpenAI's API.

The chat has a memory layer as well, allowing the user to have longer consistent conversations with the chatbot.

The main tools of the application are [`channels`](https://channels.readthedocs.io/en/stable/index.html) in Django and `WebSocket` in React, which are the key components for implementing asynchronous messaging patterns.

This application works well on localhost, but if we want to deploy it, we need to change the channel layer from In-memory to Redis, as suggested by the [documentation](https://channels.readthedocs.io/en/stable/topics/channel_layers.html#redis-channel-layer).

[GitHub repository is available here](https://github.com/grbeno/llmchat)

#### Change the In-memory to Redis Channel Layer

As the first step, we need to install [`channels_redis`](https://github.com/django/channels_redis/?tab=readme-ov-file#channels_redis)
```
pip install redis channels_redis
```

In order to use Redis for the channel layer, we need to **modify the code** in several places. First of all, we need to update the Django file that contains the class for handling the chat.

```python
# chat/chat_api.py

from openai import OpenAI
from environs import Env
import redis  # Redis Channel Layer
import json  # Redis Channel Layer

# Load the environment variables
env = Env()
env.read_env()

client = OpenAI()
client.api_key=env.str("OPENAI_API_KEY")

class AiChat():

    _role = "You are helpful and friendly. Be short but concise as you can!"
    
    # Redis Channel Layer
    _redis_client = redis.Redis(host='redis', port=6379, db=0)  

    def __init__(self, prompt: str, model: str, channel: str) -> None:
        self.prompt = prompt
        self.model = model
        self.channel = channel

        ## Redis Channel Layer
        
        # Check if the channel exists in Redis
        if not self._redis_client.exists(channel):
            initial_data = [{"role": "system", "content": self._role}]
            self._redis_client.set(channel, json.dumps(initial_data))
        
        # Retrieve the conversation from Redis
        conversation_data = self._redis_client.get(channel)
        self.conversation = json.loads(conversation_data) if conversation_data else initial_data


    def chat(self) -> str:
        if self.prompt:
            # The conversation is going on ...
            # Adding prompt to chat history
            self.conversation.append({"role": "user", "content": self.prompt})
            # Redis Channel Layer
            self._redis_client.set(self.channel, json.dumps(self.conversation))
            # The OpenAI's chat completion generates answers to your prompts.
            completion = client.chat.completions.create(
                model=self.model,
                messages=self.conversation
            )
            answer = completion.choices[0].message.content
            # Adding answer to chat history
            self.conversation.append({"role": "assistant", "content": answer})
            # Redis Channel Layer
            self._redis_client.set(self.channel, json.dumps(self.conversation))
            return answer

```

The places where I modified the original code are marked with the **_# Redis Channel Layer_** comment.

Next, we need to change the channel layer in `./config/settings.py` from In-memory to Redis Channel Layer and set the host to the Redis port.

```python
# config/settings.py

from environs import Env

# Load the environment variables
env = Env()
env.read_env()

# ...

# Channels

CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels_redis.core.RedisChannelLayer",
        "CONFIG": {
            "hosts": [(env.str('REDISHOST', default="redis"), 6379)],
        },           
     }    
 }

```

The "REDISHOST" environment variable provided from the cloud platform, while "redis" is the default for local development, as you will see in the next section.

### 2. Developing

In order to run Redis without installation, I am using Redis as a service in a `docker-compose.yml` file with a `Dockerfile` for the container image.

The local environment and specific Python files are not necessary for the container, so I could use a `.dockerignore` file in this case. However, especially for development purposes, hot reloading is more convenient.  

Before building the Docker image, I update the requirements.txt file with the latest installations.

```
pip freeze > requirements.txt
```
#### Dockerfile
```docker
# Pull base image
FROM python:3.11-slim-bullseye

# Set environment variables
ENV PIP_DISABLE_PIP_VERSION_CHECK 1
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Set work directory
WORKDIR /app

# Install dependencies
COPY ./requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy project
COPY . /app/

EXPOSE 8000
```
#### docker-compose.yml
```yml
services:
  backend:
    build: .
    container_name: llmchat
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/app  # hot reloading, overrides .dockerignore!
    ports:
      - 8000:8000
    depends_on:
      - redis
  redis:
    image: redis:latest
    container_name: llmchat_redis
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - ./redis.conf:/usr/local/etc/redis/redis.conf
    ports:
      - '6379:6379'
```
#### Run the application with the next Docker commands: 
```
docker build -t llmchat-prod .
```
```
docker-compose build
```
```
docker-compose up
```
### 3. Deploying

This application will be deployed to [Railway](https://railway.com/new).

#### Create Procfile

```
web: daphne -b 0.0.0.0 -p 8080 config.asgi:application
```

#### Install additional packages/libs required by daphne server

```
pip install twisted[tls,http2]
```

#### Environment variables

Django

Set DEBUG=True, SECRET_KEY in `.env`. Update also `config/settings.py` with `SECRET_KEY = env.str('SECRET_KEY')` and `DEBUG = env.bool('DEBUG', default=False)`.

React

I also set the environment variables in Django for use in React.

Create a Django app with the name "React".

```
python manage.py startapp react
```
After the app is done, don't forget to add it to `INTSALLED_APPS=[ ... ]` in `config/settings.py`.

react/views.py
```python
# react/views.py

from django.views.generic import TemplateView

class IndexView(TemplateView):
    template_name = 'index.html'

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        
        # Check for forwarded protocol (for proxies like Railway)
        forwarded_proto = self.request.META.get('HTTP_X_FORWARDED_PROTO')
        is_secure = self.request.is_secure() or forwarded_proto == 'https'
        
        # Set WS/WSS protocol
        ws_protocol = 'wss://' if is_secure else 'ws://'
        context['WS_URL'] = f"{ws_protocol}{self.request.get_host()}"
        
        return context
```
Modify the url pattern in `config/urls.py`

```python
# config/urls.py
# ...
from react.views import IndexView

urlpatterns = [
  
  # ...
  
  re_path(r'^.*$', IndexView.as_view()),

]
```
Update files on the frontend as well.

Add this `<script>` to the `<head>` tag in `frontend/index.html`

```jsx
<script>
  window.WS_URL = "{{ WS_URL|safe }}";
</script>
```

Finally, modify the websocket url in `src/Chat.jsx`, call `window.WS_URL` instead of the hard coded url.

#### Serving static files

```python
# config/settings.py

# Static files ...

STATIC_ROOT = str(BASE_DIR.joinpath('staticfiles'))  # production
```
Adding whitenoise to collect the static files of the project

```
pip install whitenoise
```
```python
# config/settings.py

MIDDLEWARE = [
    # ...
    "django.middleware.security.SecurityMiddleware",
    "whitenoise.middleware.WhiteNoiseMiddleware",
    # ...
]
```
Collect static files with Django command
```
python manage.py collectstatic --noinput
```
#### Freezing to requirements before deploying
```
pip freeze > requirements.txt
```
#### GitHub
Commit and Push the files to a repo in GitHub.

#### Configuring Railway

- For building use nixpacks, so add `Dockerfile` `docker-compose.yml` and any Docker-related files for example `.dockerignore` to `.gitignore`

- Create `nixpacks.toml` for building
  ```toml
  providers = ["python"]
  ```
  _We don't need "node" among the providers, because we are using React as static files on the backend._

- The builder in Railway recognizes the nixpacks file and starts to build.

- Set Railway environment variables: SECRET_KEY, OPENAI_API_KEY
, REDISHOST

- Set Custom Start Command in Settings > Deploy: `web: daphne -b 0.0.0.0 -p 8080 config.asgi:application`. Websocket port and URL port should be the same.

- Update `src/Chat.jsx` with the new websocket url: `'wss://... .up.railway.app/ws/chat/';`. _Don't forget to run build and collectstatic!_

- Set ALLOWED_HOSTS=['... .up.railway.app'] and CSRF_TRUSTED_ORIGINS=['https://... .up.railway.app'] in `config/settings.py` with the Railway's data.

- Drag & drop `docker-compose.yml` onto your project canvas, which should contain only the Redis service.

**If any errors occur, check the build and deploy logs in Deployments > View logs!**