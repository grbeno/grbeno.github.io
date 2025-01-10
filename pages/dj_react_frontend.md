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
If more versions are installed, check the list and select newest one.
```
nvm list
```
Select the newest version
```
nvm use newest
```
Check node is installed
```
node -v
```
Learn more from the [documentation of nvm-windows](https://github.com/coreybutler/nvm-windows/blob/master/README.md).
### Starting a React application and adding it to Django.
Stay in the Django project directory and create the new directory `frontend` (or what you prefer) for React files.

It is possible to install React either in the project directory or in a dedicated `frontend` directory. In the former case, a new directory needs to be created, but the files can later be moved into the project directory.

Here are two possible solutions to start React.
  - **CRA** (_create-react-app_)
  - **Vite** **Preferred*

### 1. CRA (_create-react-app_)
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
    "copy-build": "copyfiles -u 2 build/static/**/* ../static/ && copyfiles -u 1 build/* ../static/"
  },
  "devDependencies": {
    "copyfiles": "^2.4.1"
  },
```
The __-u__ flag in the __copy-build__ script specifies how many directory levels to strip from the source paths. 
```
npm run build
```
### 2. Vite

```
npm create vite@latest frontend
```
Vite asks you to select a framework (choose React) and your preference between JavaScript and TypeScript.
```
cd frontend
```
```
npm install
```
Modify `vite.config.js` according to the next. With this modification static files from the frontend's build directory copied to the Django's static.
```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://vitejs.dev/config/
export default defineConfig({
  
  plugins: [react()],
  
  // This ensures that when you run npm run build, Vite places the CSS and JS  files in the correct location that Django expects.
  
  build: {
    outDir: '../static',  // Output to Django's static folder
    assetsDir: 'assets',  // Put all static assets in an 'assets' directory
  },

},)
```
```
npm run build
```
### Make modifications in Django for both Vite and CRA as well
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

STATICFILES_DIRS = [ str(BASE_DIR.joinpath('static')) ]
```
In the case of Vite
```python
# config/settings.py

STATIC_URL = '/static/'

STATICFILES_DIRS = [ str(BASE_DIR.joinpath('static','assets')) ]
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
    re_path(r'^.*', React.as_view(), name='frontend'),
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
Delete the unnecessary files. In the case of CRA, the following files are included: `App.test.js` `logo.svg` `reportWebVitals.js` `setupTests.js`, and the entire content of the `build` and `public` directories except for `index.hmtl`.

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
```yml

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

