### Deploying Django Project Using both ASGI and WSGI

This deployment is based on the experiences from the previous two posts, except for a few updates.

**config/settings.py**

```python
# config/settings.py

WSGI_APPLICATION = 'config.wsgi.application'
ASGI_APPLICATION = 'config.asgi.application'
```

**Procfile**

In the `Procfile`, you should add and parameterize both of the servers: My choices are Gunicorn for WSGI and Daphne for ASGI.

```
web: gunicorn config.wsgi:application -b 0.0.0.0 & daphne -b 0.0.0.0 -p $PORT config.asgi:application
```

