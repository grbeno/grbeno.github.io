## Django with React Consuming LLM-API

This LLM-API consumer, that I am presenting here, is built upon a Django application using React as static files. With the API consumer, I implemented a language corrector and translator application, which is a kind of playground to try creating something great.

`Django` `ReactJS` `PostgeSQL` `Django REST framework`

You can find here the [GitHub repository](https://github.com/grbeno/dj-rjs-template) of the template, which is the foundation or starting point of the application.

### Setting up the environment

**Download the `.zip` from GitHub**

**Move the files to a new directory. Let's name it `llmlang`.**

**Open CLI and change the directory using the `$ cd ` command**

**Create a virtual environment for the project**
    
```
python -m venv venv
```
**Activate the enviroment**
```
venv\scripts\activate
```
**Install the dependencies**
```
pip install -r requirements.txt
```
**Create an .env file and update with the next environment variables**
    
Generate new key for the `SECRET_KEY`  
```
python -c 'import secrets;print(secrets.token_hex(32))'
```

```
# .env

SECRET_KEY=<generated-secret-key>
SSL_REQUIRE=False

```
**Create new database**
    
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

**Migrate data**
```
python manage.py makemigrations accounts
```
```
python manage.py migrate
```

### Starting the language app development

**Create model**

```python
# langapp/models.py

from django.db import models
from django.utils import timezone

class Language(models.Model):

    timestamp = models.DateTimeField(default=timezone.now)
    # These fields can contain up to 1000 characters to accommodate lengthy user inputs.
    prompt = models.CharField(max_length=1000)
    correction = models.CharField(max_length=1000)
    translation = models.CharField(max_length=1000)

    def __str__(self):
        return self.prompt
```
**Migrate the data**

```
python manage.py makemigrations langapp
```
```
python manage.py migrate lang
```
Install Django REST framework and update `config/settings.py`
```
pip install djangorestframework
```
Add 'rest-framework' to the `INSTALLED_APPS`. And the next setup to the end of the file:
```python
# config/settings.py

# REST Framework settings
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.AllowAny',
    ],
}
```
**langapp/serializers.py**

Create the file then update with this:

```python
# langapp/serializers.py

from rest_framework import serializers
from .models import Language

class LanguageSerializer(serializers.ModelSerializer):
    class Meta:
        model = Language
        fields = '__all__'
```
**langapp/llm-api.py**

You need an API key from OpenAI e.g.
```
pip install openai
```
If you choose a provider other than OpenAI, always read it's documentation to how to apply the API.

Let's integrate and costumize the LLM-API in a new `llm-api.py` file.

```python
# langapp/llm-api.py

import openai
from environs import Env
from rest_framework.response import Response

from .serializers import LanguageSerializer

env = Env()
env.read_env()


class Assistant:

    def __init__(self, request, prompt):
        self.request = request
        self.prompt = prompt
        self.model = 'gpt-4o-mini'
        self.role = """Your response should be the correction of the given prompt. 
        If the prompt is already correct, respond with 'Your english is correct'."""
        self.key = env.str("OPENAI_API_KEY")


    def llm_response(self):
        if self.key:
            openai.api_key = self.key
            response = openai.chat.completions.create(
                model=self.model,
                messages=[
                    {"role": "system", "content": self.role},
                    {"role": "user", "content": self.prompt}
                ],
                temperature=0.6, # 0-1
                max_tokens=512,
            )
            return response.choices[0].message.content
        else:
            return f"@Thank you for the prompt! Possible problem with API key. We are working on it."

    
    def get_data(self):
        correction = self.llm_response()
        self.role="""Anyanyelvi szintű magyarul beszélő vagy és az a feladatod, hogy a szövegeket, amiket a felhasználó ad, 
        magyarra fordíts! Ha magyarul kapod a feladatot, akkor ugyanazt a szöveget válaszold vissza!"""
        translation = self.llm_response()
        data = { 'prompt': self.prompt, 'correction': correction, 'translation': translation }
        
        # run on server
        serializer = LanguageSerializer(data=data, context={'request': self.request})

        if serializer.is_valid(raise_exception=True):
            serializer.save()
            return Response(serializer.data)
```
**langapp/views.py**

```python
# langapp/views.py

from django.views.generic import TemplateView
from rest_framework.views import APIView
from rest_framework.response import Response

from .models import Language
from .serializers import LanguageSerializer
from .llm_api import Assistant


# React home page
class React(TemplateView):
    template_name = 'index.html'


class LangAI(APIView):
	
	serializer_class = LanguageSerializer

	def get(self, request):
		detail = Language.objects.all()
		serializer = LanguageSerializer(detail, many=True)
		return Response(serializer.data)
	
	def post(self, request):
		return Assistant(request).get_data()
		
	def delete(self, request):
		try:
			detail = Language.objects.all()
			detail.delete()
			return Response({'message': 'Item deleted successfully.'})
		except Language.DoesNotExist:
			return Response({'error': 'Item not found.'})
```

**langapp/urls.py**
```python
# langapp/urls.py

from django.urls import path
from .views import LangAI

urlpatterns = [
    path('api/chat/', LangAI.as_view(), name='chat'),
]
```
**config/urls.py**
```python
# config/urls.py

from django.contrib import admin
from django.urls import path, include
from django.urls import re_path
from langapp.views import React


urlpatterns = [
    
    path('admin/', admin.site.urls),
    
    # App
    path("", include("langapp.urls")),

    # React
    re_path(r'^.*', React.as_view(), name='frontend'),
]
```
```
pip install django-cors-headers
```
**config/settings.py**

Add 'corsheaders' to the `INSTALLED_APPS`.

Add 'corsheaders.middleware.CorsMiddleware' to the `MIDDLEWARE`.

### Setting up frontend
