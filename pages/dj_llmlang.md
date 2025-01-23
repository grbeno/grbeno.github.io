## Django with React Consuming LLM-API

This LLM-API consumer, that I am presenting here, is built upon a Django application using React as static files. With the API consumer, I implemented a language corrector and translator application, which is a kind of playground to try creating something great.

`Django` `ReactJS` `PostgeSQL` `Django REST framework`

You can find here the [GitHub repository](https://github.com/grbeno/dj-rjs-template) of the template, which is the foundation or starting point of the application.

### Setting up the environment

**Download the `.zip` from GitHub**

**Move the files to a new directory. Let's name it `llmlang`.**

**Open CLI and change the directory using the `$ cd llmlang` command**

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
    
Installing Postgres on your device is necessarry.

Open CLI

```
psql -U postgres
```
Create a database, let's name it langapp.
```
CREATE DATABASE langapp WITH OWNER postgres; 
```
Update `.env`
```
# .env

DATABASE_URL=postgresql://postgres:<password>@localhost:5432/langapp
```
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
Add 'rest-framework' to the `INSTALLED_APPS`. And the next initial setup to the end of the file:
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

You need an API key from OpenAI or another provider.
```
pip install openai
```
If you choose a provider other than OpenAI, always read its documentation to how to apply the API.

Let's integrate and costumize the LLM-API in a new `llm_api.py` file.

```python
# langapp/llm_api.py

import openai
from environs import Env
from rest_framework.response import Response

from .serializers import LanguageSerializer

env = Env()
env.read_env()


class Assistant:

    def __init__(self, request):
        self.request = request
        self.prompt = request.data['prompt']
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

Create this file and update it with the pattern below.

```python
# langapp/urls.py

from django.urls import path
from .views import LangAI

urlpatterns = [
    path('api/chat/', LangAI.as_view(), name='chat'),
]

```
**config/urls.py**

Update the config.urls pattern list with langapp.urls.

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
**CORS**

```
pip install django-cors-headers
```
**config/settings.py**

Add 'corsheaders' to the `INSTALLED_APPS`.

Add 'corsheaders.middleware.CorsMiddleware' to the `MIDDLEWARE`.
> CorsMiddleware should be placed as high as possible, especially before any middleware that can generate responses such as Django’s CommonMiddleware or Whitenoise’s WhiteNoiseMiddleware. If it is not before, it will not be able to add the CORS headers to these responses. [docs](https://github.com/adamchainz/django-cors-headers?tab=readme-ov-file#setup)

### Setting up frontend

```
cd frontend
```
**Install axios**
```
npm install axios
```
**Create `frontend/.env`**
```
# frontend/.env

export VITE_APP_URL=http://localhost:8000

```
**Create src/axios.js and add the code:**
```javascript
// src/axios.jsx

import axios from 'axios';

// Export base url from env variable
const baseURL = import.meta.env.VITE_APP_URL;

// Get csrf token
const getCSRFToken = () => {
    const csrfToken = document.cookie.match(/csrftoken=([\w-]+)/);
    return csrfToken ? csrfToken[1] : null;
};

// Axios instance
const axiosInstance = axios.create({
    baseURL: baseURL,
    timeout: 5000,
    headers: {
        'X-CSRFToken': getCSRFToken(),
        'Content-Type': 'application/json',
        'Accept': 'application/json',
    },
});

export default axiosInstance;

```
**Update `App.jsx`**
```jsx
// src/App.jsx

import React, {useEffect, useState} from 'react';
import axiosInstance from './axios';
import './App.css';

function App() {

  const [response, setResponse] = useState([]);
  const [formData, setFormData] = useState({ prompt: '', });
  const [isLoading, setIsLoading] = useState(false);
  const [fadeIn, setFadeIn] = useState(false);

  // path
  const path = import.meta.env.VITE_APP_URL + '/api/chat/';
  const pathname = window.location.pathname.endsWith('/') ? window.location.pathname.slice(0, -1) : window.location.pathname;

  const postPrompt = (e) => {
    setIsLoading(true);  // spinner on
    e.preventDefault();
    axiosInstance.post(path, {
      prompt: formData.prompt,
    })
    .then((res) => {
      // console.log(res);
      setFormData({ prompt: '', });
      setResponse((response) => [...response, res.data]); 
    })
    .catch((error) => {
      console.log(error);
    });
  };

  const deleteItem = (id) => {
    axiosInstance.delete(path)
    .then((res) => {
      // get answer
      axiosInstance.get(path)
      .then((res) => {
          setResponse(res.data);
      })  
      .catch((error) => {
          console.log(error);
      });  
    })
    .catch((error) => {
      console.log(error);
    });
  };

  // useEffect for getting data
  useEffect(() => {
    // get answer
    console.log(path); // test
    axiosInstance.get(path)
    .then((res) => {
      setResponse(res.data);
    })  
    .catch((error) => {
      console.log(error);
    });
  }, [path]);

  // useEffect for spinner
  useEffect(() => {
    setIsLoading(false);  // spinner off when goes to the bottom of the response list
  }, [response]);

  useEffect(() => {
    // Trigger the fade-in effect when the component mounts
    setFadeIn(true);
  }, []);

   // handle input
  const handleInput = (event) => {
    setFormData({
      ...formData,
      [event.target.name]: event.target.value,
    });
  };

  return (
    <div className="App">
     <h1>Corrector</h1>
     <h1> * </h1>
      <div className="response">
        {window.location.port === '5173' ? (
          <>
            <div className="prompt">This is an english language corrector.</div>
            <div className='correction'>Your English is correct.</div>
            <div className='translation'>Ez egy angol nyelvhelyesség-javító.</div>
          </>
        ) : null}
        {response.map((item) => (
          <div key={item.id} className={`item ${fadeIn ? 'fade-in' : ''}`}> 
            <div className="prompt">{item.prompt}</div>
            <div className='correction'>{item.correction}</div>
            <div className='translation'>{item.translation}</div>
          </div>
        ))}
        {isLoading && <div className="loading">Loading...</div>}
      </div>

      <form onSubmit={postPrompt}>
        <h1> * </h1>
        <textarea
          type="text"
          name="prompt"
          value={formData.prompt}
          onChange={handleInput}
          placeholder="Enter your text"
          required
          />
        <button type="submit">Submit</button>
      </form>
      <button className="delete" onClick={deleteItem}>Delete</button>
    </div>
  );
}

export default App;

```
**Update `App.css`**
```css
/* src/App.css */

body {
  background: #324065;
}

.App {
  text-align: center;
  font-family: 'Courier New', Courier, monospace;
  color: #0c1823;
}

h1 {
  color: #d1a266;
}

.loading {
  color: #d1a266;
}

textarea[type="text"] {
  width: 600px;
  height: 150px;
  font-size: 16px;
  text-align: left;
  vertical-align: top;
  border-radius: 8px;
}

button {
  display: block;
  margin: 0 auto;
  margin-top: 10px;
  width: 605px;
  background-color: #fdd95d;
  color: black;
  padding: 10px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}

.delete {
  
  background-color: #cd4a33;
  
}

.delete:hover {
  background-color: #d21f0a;
}

.prompt,
.correction,
.translation {
  justify-content: center;
  display: flex;
  width: 590px;
  padding: 8px 12px;
  margin: 8px auto;
  border-radius: 8px;
}

.prompt {
  background-color: #add4e5;
}

.correction {
  background-color: #fff7b3;
}

.translation {
  background-color: #add4e5;
  margin-bottom: 32px;
}


```
**Update `main.jsx`**
```jsx
// src/main.jsx

import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import App from './App'

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
)

```
```
npm run build
```
```
cd ..
```
```
python manage.py runserver
```