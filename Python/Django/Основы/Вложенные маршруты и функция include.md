# Вложенные маршруты и функция include

Owner: Андрей Королев
Last edited time: December 5, 2024 11:33 PM

### Вложенные маршруты и функция `include`

Функция `include()` в Django позволяет создавать вложенные маршруты, что упрощает структуру URL. Это полезно для группировки маршрутов с одинаковым начальным сегментом.

### Синтаксис:

```python
include(pattern_list)
```

`pattern_list` — это список маршрутов, определенных с помощью `path()` и/или `re_path()`.

### Изменение структуры представлений (`views.py`)

Создадим функции-представления для отображения главной страницы и страниц товаров:

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse('Главная страница')

def products(request):
    return HttpResponse('Список товаров')

def new(request):
    return HttpResponse('Новые товары')

def top(request):
    return HttpResponse('Наиболее популярные товары')
```

### Настройка главного файла маршрутов (`urls.py` проекта)

В главном файле маршрутов подключаем приложение `blog`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')),
]
```

### Создание маршрутов в приложении (`blog/urls.py`)

Определим маршруты для приложения:

```python
from django.urls import path, include
from blog import views

product_patterns = [
    path('', views.products),
    path('new/', views.new),
    path('top/', views.top),
]

urlpatterns = [
    path('', views.index),
    path('products/', include(product_patterns)),
]
```

- `product_patterns` — это отдельный набор маршрутов для товаров.
- Функция `include()` связывает эти маршруты с родительским шаблоном `products/`.

### Итоговая структура маршрутов:

- `http://127.0.0.1:8000` → главная страница.
- `http://127.0.0.1:8000/products/` → список товаров.
- `http://127.0.0.1:8000/products/new/` → новые товары.
- `http://127.0.0.1:8000/products/top/` → популярные товары.

### Преимущества использования `include()`:

- Позволяет избежать дублирования сегментов URL.
- Упрощает изменение корневого маршрута приложения.

Пример: Если в `urls.py` главного проекта изменить путь к приложению `blog` на `blog/`, все маршруты будут корректно обновлены:

```python
urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls')),
]
```

**Новая структура URL:**

- `http://127.0.0.1:8000/blog/`
- `http://127.0.0.1:8000/blog/products/`
- `http://127.0.0.1:8000/blog/products/new/`
- `http://127.0.0.1:8000/blog/products/top/`

---

### Передача параметров в вложенные маршруты

Если родительский маршрут принимает параметры, они передаются всем вложенным маршрутам. Например, добавим параметр `id` для работы с товарами.

### Представления (`views.py`):

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse('Главная страница')

def products(request, id):
    return HttpResponse(f'Товар {id}')

def comments(request, id):
    return HttpResponse(f'Комментарии о товаре {id}')

def questions(request, id):
    return HttpResponse(f'Вопросы о товаре {id}')
```

### Маршруты приложения (`blog/urls.py`):

```python
from django.urls import path, include
from blog import views

product_patterns = [
    path('', views.products),
    path('comments/', views.comments),
    path('questions/', views.questions),
]

urlpatterns = [
    path('', views.index),
    path('products/<int:id>/', include(product_patterns)),
]
```

**Объяснение:**

- Параметр `<int:id>` передается всем вложенным маршрутам.
- Например, запрос на `http://127.0.0.1:8000/products/1/questions/` обработается функцией `questions` с `id=1`.