# M7 AE5 – Práctica SQL con Django

## Resumen
Proyecto desarrollado en Django cuyo objetivo es aplicar consultas SQL directas, uso del ORM, índices, anotaciones, cursores y gestión de migraciones, cumpliendo los requisitos de la Actividad Evaluada 5 del Módulo 7.

---

## 1. Creación y activación del entorno virtual

```powershell
python -m venv venv
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\venv\Scripts\Activate.ps1
```

Instalación de dependencias:

```powershell
pip install django
```

---

## 2. Creación del proyecto Django

```powershell
django-admin startproject practica_sql_django
cd practica_sql_django
```

Ejecución inicial del servidor:

```powershell
python manage.py runserver
```

---

## 3. Migraciones iniciales

```powershell
python manage.py migrate
```

Esto aplica las migraciones por defecto de Django (auth, admin, sessions, contenttypes).

---

## 4. Creación de la app `productos`

```powershell
python manage.py startapp productos
```

Se registra la app en `settings.py`:

```python
INSTALLED_APPS = [
    'productos',
]
```

---

## 5. Definición del modelo Producto

```python
class Producto(models.Model):
    nombre = models.CharField(max_length=100, db_index=True)
    precio = models.DecimalField(max_digits=5, decimal_places=2)
    disponible = models.BooleanField(default=True)
```

Migraciones del modelo:

```powershell
python manage.py makemigrations
python manage.py migrate
```

---

## 6. Uso del ORM (Django Shell)

```powershell
python manage.py shell
```

Consultas realizadas:

```python
Producto.objects.all()
Producto.objects.filter(precio__gt=50)
Producto.objects.filter(nombre__startswith="A")
Producto.objects.filter(disponible=True)
```

---

## 7. Inserción de datos de prueba

```python
Producto.objects.create(nombre="Arroz", precio=30, disponible=True)
Producto.objects.create(nombre="Azucar", precio=55, disponible=True)
Producto.objects.create(nombre="Aceite", precio=120, disponible=False)
Producto.objects.create(nombre="Pan", precio=80, disponible=True)
```

---

## 8. Consultas SQL con `raw()`

```python
Producto.objects.raw(
    "SELECT * FROM productos_producto WHERE precio < 100"
)
```

Uso de parámetros para evitar SQL Injection:

```python
Producto.objects.raw(
    "SELECT * FROM productos_producto WHERE precio < %s",
    [100]
)
```

---

## 9. Exclusión de campos

```python
Producto.objects.values('id', 'nombre', 'precio')
```

---

## 10. Uso de `annotate()` (cálculo de impuesto)

```python
from django.db.models import F, ExpressionWrapper, DecimalField

Producto.objects.annotate(
    precio_con_impuesto=ExpressionWrapper(
        F('precio') * 1.16,
        output_field=DecimalField(max_digits=7, decimal_places=2)
    )
)
```

---

## 11. Uso de cursores SQL

```python
from django.db import connection

with connection.cursor() as cursor:
    cursor.execute(
        "INSERT INTO productos_producto (nombre, precio, disponible) VALUES (%s, %s, %s)",
        ["Fideos", 40, True]
    )
```

Consulta directa:

```python
with connection.cursor() as cursor:
    cursor.execute("SELECT nombre, precio FROM productos_producto")
    resultados = cursor.fetchall()
```

---

## 12. Procedimientos almacenados

SQLite no soporta procedimientos almacenados. En motores como PostgreSQL o MySQL, Django permite su uso mediante `cursor.callproc()`.

Ejemplo teórico:

```python
with connection.cursor() as cursor:
    cursor.callproc('obtener_productos_baratos', [100])
```

---

## Conclusión

El proyecto cumple con todos los requisitos solicitados en la Actividad Evaluada AE5, demostrando el uso correcto del ORM de Django, consultas SQL crudas, índices, anotacio