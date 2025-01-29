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

**react/views.py**

Set also HTTP/HTTPS protocol in `IndexView`.

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

        # Set HTTP/HTTPS protocol
        http_protocol = 'https://' if is_secure else 'http://'
        context['BACKEND_URL'] = f"{http_protocol}{self.request.get_host()}"
        
        # Set WS/WSS protocol
        ws_protocol = 'wss://' if is_secure else 'ws://'
        context['WS_URL'] = f"{ws_protocol}{self.request.get_host()}"
        
        return context
```

Add the `BACKEND_URL|safe` `<script>` to the `<head>` tag in `frontend/index.html` and calling with `windows.BACKEND_URL` in axios.jsx and where it is neccessary.

Add the `WS_URL|safe` `<script>` to the `<head>` tag in `frontend/index.html` and calling with `windows.WS_URL` in Chat.jsx.
