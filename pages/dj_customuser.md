---
layout: default
---

## Custom user model

Sooner or later, the developer needs to customize the user model, so it is highly recommended to use a custom user model. Until the custom user model is done, running the migrate command causes problems. As the documentation says, "Changing AUTH_USER_MODEL after you’ve created database tables is significantly more difficult since it affects foreign keys and many-to-many relationships, for example."

Create a new app for handling custom user model and authentication.
```
python manage.py startapp accounts
```
Custom User class inherits the functionality of AbstractUser class. It can be override or extend later if needed.
```python
# accounts/models.py

from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    pass

```
Update the `INSTALLED_APPS` with the new accounts app and add the custom user model to `AUTH_USER_MODEL`.
```python
# config/settings.py

INSTALLED_APPS = [
  'django.contrib.admin',
  'django.contrib.auth',
  'django.contrib.contenttypes',
  'django.contrib.sessions',
  'django.contrib.messages',
  'django.contrib.staticfiles',
  'helloworld',
  'accounts',  # new app added
]


# At the bottom of the file
AUTH_USER_MODEL = 'accounts.User'

```
Now, you can migrate the data to the database.
```
python manage.py makemigrations accounts
python manage.py migrate
```
Update the user forms (you can find at http://127.0.0.1:8000/admin) with the custom user model.
```python
# accounts/forms.py

from django.contrib.auth import get_user_model
from django.contrib.auth.forms import UserCreationForm, UserChangeForm


class CustomUserCreationForm(UserCreationForm):

    class Meta:
        model = get_user_model()
        fields = ('email', 'username',)


class CustomUserChangeForm(UserChangeForm):

    class Meta:
        model = get_user_model()
        fields = ('email', 'username',)

```
Customize and register the custom user model in admin.py.
```python
# accounts/admin.py

from django.contrib import admin
from django.contrib.auth import get_user_model
from django.contrib.auth.admin import UserAdmin

from .forms import CustomUserCreationForm, CustomUserChangeForm


User = get_user_model()

class CustomUserAdmin(UserAdmin):
    add_form = CustomUserCreationForm
    form = CustomUserChangeForm
    model = User
    list_display = [ 'id', 'username', 'email', 'is_superuser', 'is_active', 'last_login',]

admin.site.register(User, CustomUserAdmin)

```
Create superuser account to log in and handle the Django admin interface.
```
python manage.py createsuperuser
```
This tutorial based on: [https://docs.djangoproject.com/en/5.0/topics/auth/customizing/#substituting-a-custom-user-model](https://docs.djangoproject.com/en/5.0/topics/auth/customizing/#substituting-a-custom-user-model)
