# Proyecto: Biblias - UIII_Biblias_0486

```python
# 1. Crear la carpeta principal del proyecto
mkdir UIII_Biblias_0486

# 2. Navegar a la carpeta del proyecto
cd UIII_Biblias_0486

# 4. Crear el entorno virtual llamado .venv
python3 -m venv .venv

# 5. y 6. Activar el entorno virtual
# (Linux/macOS)
source .venv/bin/activate

# (Windows - Alternativa si el comando anterior falla)
# .venv\Scripts\activate

# 7. Instalar el framework Django
pip install Django

# 8. Crear el proyecto backend_Biblia sin duplicar la carpeta
# El punto final indica que se cree en la carpeta actual (UIII_Biblias_0486)
django-admin startproject backend_Biblia .

# 11. Crear la aplicación app_Biblia
python manage.py startapp app_Biblia

# --- Comandos de Configuración y Base de Datos ---

# Generar los archivos de migración (basados en models.py)
# (Esto asume que ya se configuró INSTALLED_APPS en settings.py y models.py está listo)
python manage.py makemigrations

# Aplicar las migraciones a la base de datos (crea las tablas)
python manage.py migrate

# Crear un superusuario para acceder al panel de administración (admin.py)
python manage.py createsuperuser

# 31. Finalmente, ejecutar el servidor en el puerto 0486
# 9. El servidor estará disponible en la URL http://127.0.0.1:0486/
python manage.py runserver 0.0.0.0:0486

```

## models.py
```python
from django.db import models

# MODELO: CLIENTE_GLOBAL
class ClienteGlobal(models.Model):
    nombre = models.CharField(max_length=100)
    apellido = models.CharField(max_length=100)
    email = models.CharField(max_length=255, unique=True)
    fecha_nacimiento = models.DateField(null=True, blank=True)
    pais_residencia = models.CharField(max_length=100)
    codigo_postal = models.CharField(max_length=15, null=True, blank=True)
    telefono = models.CharField(max_length=20, null=True, blank=True)

    def __str__(self):
        return f"{self.nombre} {self.apellido} ({self.email})"
```

## views.py
```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import ClienteGlobal

def inicio_biblias(request):
    return render(request, 'inicio.html')

def agregar_cliente(request):
    if request.method == 'POST':
        nombre = request.POST.get('nombre')
        apellido = request.POST.get('apellido')
        email = request.POST.get('email')
        pais_residencia = request.POST.get('pais_residencia')
        ClienteGlobal.objects.create(
            nombre=nombre, apellido=apellido, email=email, pais_residencia=pais_residencia
        )
        return redirect('ver_clientes')
    return render(request, 'cliente/agregar_cliente.html')

def ver_clientes(request):
    clientes = ClienteGlobal.objects.all()
    return render(request, 'cliente/ver_clientes.html', {'clientes': clientes})

def actualizar_cliente(request, id):
    cliente = get_object_or_404(ClienteGlobal, id=id)
    return render(request, 'cliente/actualizar_cliente.html', {'cliente': cliente})

def realizar_actualizacion_cliente(request, id):
    cliente = get_object_or_404(ClienteGlobal, id=id)
    if request.method == 'POST':
        cliente.nombre = request.POST.get('nombre')
        cliente.apellido = request.POST.get('apellido')
        cliente.email = request.POST.get('email')
        cliente.pais_residencia = request.POST.get('pais_residencia')
        cliente.save()
        return redirect('ver_clientes')
    return render(request, 'cliente/actualizar_cliente.html', {'cliente': cliente})

def borrar_cliente(request, id):
    cliente = get_object_or_404(ClienteGlobal, id=id)
    if request.method == 'POST':
        cliente.delete()
        return redirect('ver_clientes')
    return render(request, 'cliente/borrar_cliente.html', {'cliente': cliente})
```

## urls.py (app_Biblias)
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_biblias, name='inicio_biblias'),
    path('clientes/', views.ver_clientes, name='ver_clientes'),
    path('clientes/agregar/', views.agregar_cliente, name='agregar_cliente'),
    path('clientes/actualizar/<int:id>/', views.actualizar_cliente, name='actualizar_cliente'),
    path('clientes/realizar_actualizacion/<int:id>/', views.realizar_actualizacion_cliente, name='realizar_actualizacion_cliente'),
    path('clientes/borrar/<int:id>/', views.borrar_cliente, name='borrar_cliente'),
]
```

## urls.py (backend_Biblias)
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Biblias.urls')),
]
```

## admin.py
```python
from django.contrib import admin
from .models import ClienteGlobal

admin.site.register(ClienteGlobal)
```

## settings.py (fragmento importante)
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app_Biblias',
]
```

## base.html
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema de Biblias</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    {% include 'header.html' %}
    {% include 'navbar.html' %}
    <div class="container mt-4">
        {% block content %}{% endblock %}
    </div>
    {% include 'footer.html' %}
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

## navbar.html
```html
<nav class="navbar navbar-expand-lg navbar-dark bg-primary">
    <div class="container-fluid">
        <a class="navbar-brand" href="#">Sistema de Administración de Biblias</a>
        <div class="collapse navbar-collapse">
            <ul class="navbar-nav">
                <li class="nav-item"><a class="nav-link" href="{% url 'inicio_biblias' %}">Inicio</a></li>
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">Clientes</a>
                    <ul class="dropdown-menu">
                        <li><a class="dropdown-item" href="{% url 'agregar_cliente' %}">Agregar Cliente</a></li>
                        <li><a class="dropdown-item" href="{% url 'ver_clientes' %}">Ver Clientes</a></li>
                    </ul>
                </li>
            </ul>
        </div>
    </div>
</nav>
```

## footer.html
```html
<footer class="bg-dark text-white text-center fixed-bottom py-2">
    <p>© {{ year }} Creado por Ing. Jorge Dominguez, CBTIS 128</p>
</footer>
```

## inicio.html
```html
{% extends 'base.html' %}
{% block content %}
<h2>Bienvenido al Sistema de Biblias</h2>
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/8/8b/Bible_open.jpg/800px-Bible_open.jpg" class="img-fluid" alt="Biblias">
{% endblock %}
```

## agregar_cliente.html
```html
{% extends 'base.html' %}
{% block content %}
<h2>Agregar Cliente</h2>
<form method="POST">
    {% csrf_token %}
    <input type="text" name="nombre" placeholder="Nombre" required><br>
    <input type="text" name="apellido" placeholder="Apellido" required><br>
    <input type="email" name="email" placeholder="Correo" required><br>
    <input type="text" name="pais_residencia" placeholder="País de residencia" required><br>
    <button type="submit">Guardar</button>
</form>
{% endblock %}
```

## ver_clientes.html
```html
{% extends 'base.html' %}
{% block content %}
<h2>Lista de Clientes</h2>
<table class="table table-striped">
    <thead>
        <tr>
            <th>Nombre</th>
            <th>Apellido</th>
            <th>Email</th>
            <th>Acciones</th>
        </tr>
    </thead>
    <tbody>
        {% for c in clientes %}
        <tr>
            <td>{{ c.nombre }}</td>
            <td>{{ c.apellido }}</td>
            <td>{{ c.email }}</td>
            <td>
                <a href="{% url 'actualizar_cliente' c.id %}">Editar</a> |
                <a href="{% url 'borrar_cliente' c.id %}">Borrar</a>
            </td>
        </tr>
        {% endfor %}
    </tbody>
</table>
{% endblock %}
```

## actualizar_cliente.html
```html
{% extends 'base.html' %}
{% block content %}
<h2>Actualizar Cliente</h2>
<form method="POST">
    {% csrf_token %}
    <input type="text" name="nombre" value="{{ cliente.nombre }}"><br>
    <input type="text" name="apellido" value="{{ cliente.apellido }}"><br>
    <input type="email" name="email" value="{{ cliente.email }}"><br>
    <input type="text" name="pais_residencia" value="{{ cliente.pais_residencia }}"><br>
    <button type="submit">Actualizar</button>
</form>
{% endblock %}
```

## borrar_cliente.html
```html
{% extends 'base.html' %}
{% block content %}
<h2>Borrar Cliente</h2>
<p>¿Seguro que deseas eliminar a {{ cliente.nombre }} {{ cliente.apellido }}?</p>
<form method="POST">
    {% csrf_token %}
    <button type="submit">Sí, eliminar</button>
    <a href="{% url 'ver_clientes' %}">Cancelar</a>
</form>
{% endblock %}
```

