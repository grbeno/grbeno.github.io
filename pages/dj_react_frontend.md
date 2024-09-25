---
layout: default
---

# Adding React to Django

OS: **Windows 11 Pro 64bit**

### First steps
Install node on Windows OS using [nvm-windows](https://github.com/coreybutler/nvm-windows/releases).

__For nvm-windows:__ After clicking on the link, download the `nvm-setup.zip` directory (Assets), then unzip it and run the installer!

Install node

```
nvm install <node_version>
```
Check node is installed
```
node -v
```
If more versions are installed, check the list and select newest one.
```
nvm list
```
```
nvm use newest
```
Learn more from the [documentation of nvm-windows](https://github.com/coreybutler/nvm-windows/blob/master/README.md).
### Initialize React and adding to the Django
Stay in the Django project directory and create the new directory `frontend` (or what you prefer) for React files.
```
npx create-react-app@latest frontend
```
```
cd frontend
```
```
npm install copyfiles
```
Modify `package.json` according to the next. With this modification static files from the frontend's `build` directory copied to the Django's `static`.
```json
"scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build && npm run copy-build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "copy-build": "copyfiles -u 1 build/**/* ../static/"
  },
  "devDependencies": {
    "copyfiles": "^2.4.1"
  },
```
```
npm run build
```
### Do some modifications in Django
Add the `static` path to TEMPLATES.
```python
# config/settings.py

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [str(BASE_DIR.joinpath('static')),],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```
Set staticfiles directory
```python
# config/settings.py

STATIC_URL = '/static/'

STATICFILES_DIRS = [ str(BASE_DIR.joinpath('static', 'static')) ]
```
Set url for React template view
```python
# config/urls.py

from django.contrib import admin
from django.urls import path, re_path
from helloworld.views import React

urlpatterns = [
    path('admin/', admin.site.urls),

    # React
    re_path(r'^.*$', React.as_view(), name='frontend'),
]
```
Create React template view
```python
# app/views.py

from django.views.generic import TemplateView

# React home page
class React(TemplateView):
    template_name = 'index.html'
```

Finally, run the Django application with React frontend
``` 
cd ..
```
``` 
python manage.py runserver
```
### Start your custom UI in React
Delete the unnecessary files: `App.test.js` `logo.svg` `reportWebVitals.js` `setupTests.js`

Delete the dependencies with the files above from `index.js`.

Delete all rules except .App from `App.css`.

```css
/* App.css */

.App {
  text-align: center;
}
```
Start customizing the `App.js`
```javascript
// App.js

import './App.css';

function App() {
  return (
    <div className="App">
     <h1> Hello World </h1>
    </div>
  );
}

export default App;
```
After modification go to `frontend` directory and run build
```
cd frontend
```
```
npm run build
```
Go back to project directory and run the app
```
cd ..
```
```
python manage.py runserver
```

### Docker
__Dockerfile__ \
Create `Dockerfile` inside the `frontend/` directory.
```docker
FROM node:20.15-bullseye-slim

WORKDIR /frontend

COPY package.json package-lock.json ./
RUN npm install

COPY . /frontend/
RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```
Complete the `docker-compose.yml` in the project directory with the frontend service
```docker

frontend:
    build: ./frontend
    container_name: helloworld_react_frontend
    volumes:
      - .:/frontend
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_URL=http://localhost:8000
    depends_on:
      - backend
      
```
__More about Docker [here](https://grbeno.github.io/pages/dj_deployment.html).__

### Github
In order for the frontend directory to be accessible in the project's GitHub repository, the following steps need to be taken.

- remove `.git` directory from the `frontend/` (_*Make sure that hidden files are visible!*_)
- copy content from the `./frontend/.gitignore` and paste it to the `./gitignore`
- delete `./frontend/.gitignore` file
- delete `./frontend/Readme.md`

