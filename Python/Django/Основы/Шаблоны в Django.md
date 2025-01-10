# Шаблоны в Django

Owner: Андрей Королев
Last edited time: December 19, 2024 10:45 PM

## Шаблоны

### **Что такое шаблоны:**

- Шаблоны — это HTML-страницы с возможностью вставки данных из Python-кода с помощью тегов Django.
- Они делают сайты динамическими.

---

### **Создание шаблонов:**

- Создай папку `templates` в корне проекта или внутри приложения.
- Укажи путь в `settings.py`:
    
    ```python
    TEMPLATES = [
        {
            'DIRS': [BASE_DIR / 'templates'],
            'APP_DIRS': True,
        },
    ]
    ```
    

---

### **Пример использования шаблона:**

- **Шаблон `index.html`:**
    
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>Мой Django проект</title>
    </head>
    <body>
        <h1>Добро пожаловать!</h1>
    </body>
    </html>
    ```
    
- **Функция во `views.py`:**
    
    ```python
    from django.shortcuts import render
    
    def index(request):
        return render(request, 'index.html')
    ```
    
- **Маршрут в `urls.py`:**
    
    ```python
    from django.urls import path
    from blog import views
    
    urlpatterns = [
        path('', views.index),
    ]
    ```
    

---

### **Структура папок для шаблонов:**

- Общая папка:
    
    ```markdown
    templates/
        index.html
    ```
    
- Разделение по приложениям:
    
    ```markdown
    templates/
        blog/
            index.html
    ```
    

---

1. **Функция `render()` vs класс `TemplateResponse`:**
    - **`render()`:** Простой способ рендеринга шаблона.
    - **`TemplateResponse`:** Альтернатива для возврата шаблона.
        
        ```python
        from django.template.response import TemplateResponse
        
        def index(request):
            return TemplateResponse(request, 'blog/index.html')
        ```
        

---

1. **Динамические данные в шаблонах:**
    - Данные можно передать в шаблон через контекст:
        
        ```python
        def index(request):
            return render(request, 'blog/index.html', {'name': 'Django'})
        ```
        
    - В шаблоне:
        
        ```html
        <h1>Привет, {{ name }}!</h1>
        ```
        

---

1. **Заключение:**
Шаблоны позволяют вставлять Python-данные в HTML и настраивать структуру проекта для поддержки динамических веб-страниц. Это основа для создания более сложных приложений.

## Передача данных в шаблоны Django

### Основные концепции:

1. **Передача данных в шаблоны**:
    - Для отображения динамических данных в шаблоне используются переменные, передаваемые из представлений (views).
    - Эти данные передаются через параметр `context` в функцию `render`.
2. **Контекст**:
    - Контекст представляет собой словарь, где ключи – это имена переменных, используемых в шаблоне, а значения – данные, передаваемые из представления.
3. **Использование переменных в шаблоне**:
    - Переменные внутри шаблона обозначаются двойными фигурными скобками: `{{ variable_name }}`.

---

### Примеры с кодом:

1. **Пример передачи простых данных**:

**views.py**:

```python
from django.shortcuts import render

def index(request):
    data = {'header': 'Hello Django', 'message': 'Welcome to Python'}
    return render(request, 'blog/index.html', context=data)
```

**index.html**:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>Учу Django</title>
</head>
<body>
    <h2>{{ header }}</h2>
    <p>{{ message }}</p>
</body>
</html>
```

**Вывод в браузере**:

```css
Hello Django
Welcome to Python
```

---

1. **Пример передачи сложных данных**:

**views.py**:

```python
from django.shortcuts import render

def index(request):
    header = 'Данные пользователя'  # Обычная строка
    langs = ['Python', 'Java', 'C#']  # Список
    user = {'name': 'Tom', 'age': 23}  # Словарь
    address = ('Абрикосовая', 23, 45)  # Кортеж

    data = {'header': header, 'langs': langs, 'user': user, 'address': address}
    return render(request, 'blog/index.html', context=data)
```

**index.html**:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>Учу Django</title>
</head>
<body>
    <h1>{{ header }}</h1>
    <p>Имя: {{ user.name }} Возраст: {{ user.age }}</p>
    <p>Адрес: ул. {{ address.0 }}, д. {{ address.1 }}, кв. {{ address.2 }}</p>
    <p>Языки: {{ langs.0 }}, {{ langs.1 }}, {{ langs.2 }}</p>
</body>
</html>
```

**Вывод в браузере**:

```makefile
Данные пользователя
Имя: Tom Возраст: 23
Адрес: ул. Абрикосовая, д. 23, кв. 45
Языки: Python, Java, C#
```

---

### Ключевые моменты:

1. **Словари и списки**:
    - К элементам словаря можно обращаться через точечную нотацию: `{{ dictionary.key }}`.
    - К элементам списка и кортежа – через индексы: `{{ list.0 }}`.
2. **TemplateResponse**:
    - В случае использования класса `TemplateResponse`, данные передаются в шаблон аналогично, через третий параметр конструктора.

## Использование специальных тегов в шаблонах Django

### Основные теги и их применение

---

### 1. Тег `now`: вывод текущей даты и времени

**Синтаксис**:

```html
{% now 'формат_данных' %}
```

**Форматы даты и времени**:

- `Y`: Год в виде четырех цифр.
- `y`: Последние две цифры года.
- `F`: Полное название месяца.
- `M`: Сокращенное название месяца.
- `j`: День месяца (без ведущего нуля).
- `d`: День месяца (с ведущим нулем).
- `H`: Часы (0-24).
- `h`: Часы (0-12).
- `i`: Минуты.
- `s`: Секунды.
- `l`: День недели (текстом).

**Пример в шаблоне**:

```html
<p>Год: {% now 'Y' %}</p>
<p>Дата: {% now 'j/m/Y' %}</p>
<p>Время: {% now 'H:i:s' %}</p>
```

**Вывод**:

```makefile
Год: 2024
Дата: 11/12/2024
Время: 14:35:20
```

---

### 2. Тег `if`: вывод информации по условию

**Синтаксис**:

```html
{% if условие %}
    <p>...</p>
{% elif другое условие %}
    <p>...</p>
{% else %}
    <p>...</p>
{% endif %}
```

**Пример в представлении**:

```python
from django.shortcuts import render

def index(request):
    data = {'age': 50}
    return render(request, 'blog/index.html', context=data)
```

**Пример в шаблоне**:

```html
<p>Анализ возраста клиента:</p>
{% if age > 65 %}
    <p>Клиент достиг пенсионного возраста</p>
{% else %}
    <p>Клиент не является пенсионером</p>
{% endif %}
```

**Вывод**:

```
Анализ возраста клиента:
Клиент не является пенсионером
```

---

### 3. Тег `for`: вывод информации в цикле

**Синтаксис**:

```html
{% for item in collection %}
    <li>{{ item }}</li>
{% empty %}
    <li>Коллекция пуста</li>
{% endfor %}
```

**Пример в представлении**:

```python
from django.shortcuts import render

def index(request):
    cat = ['Python', 'Java', 'JS', 'Go', 'C#', 'Kotlin']
    return render(request, 'blog/index.html', context={'cat': cat})
```

**Пример в шаблоне**:

```html
<p>Категории:</p>
<ul>
{% for i in cat %}
    <li>{{ i }}</li>
{% empty %}
    <li>Категории пусты</li>
{% endfor %}
</ul>
```

**Вывод**:

- Если список `cat` содержит элементы:
    
    ```diff
    Категории:
    - Python
    - Java
    - JS
    - Go
    - C#
    - Kotlin
    ```
    
- Если список `cat` пуст:
    
    ```diff
    Категории:
    - Категории пусты
    ```
    

---

### Резюме:

1. **`now`** – для форматированного вывода текущей даты и времени.
2. **`if`** – для отображения информации по условию.
3. **`for`** – для отображения элементов коллекции в цикле, с обработкой пустых коллекций через тег `{% empty %}`.

## Дополнительные теги в шаблонах Django

### 1. **Тег `with`: задание переменных в шаблоне**

**Синтаксис**:

```html
{% with variable_name='value' %}
    {{ variable_name }}
{% endwith %}
```

**Описание**:

- Тег `with` позволяет объявлять переменные внутри блока.
- Переменные доступны только в пределах блока `{% with %}...{% endwith %}`.

**Пример**:

```html
{% with name='Tom' age=29 %}
    <div>
        <p>Name: {{ name }}</p>
        <p>Age: {{ age }}</p>
    </div>
{% endwith %}
```

**Вывод**:

```makefile
Name: Tom
Age: 29
```

---

### 2. **Тег `autoescape`: управление экранированием HTML**

**Синтаксис**:

```html
{% autoescape on %}
    {{ variable }}
{% endautoescape %}
```

- По умолчанию, экранирование включено (`autoescape on`).
- При отключении (`autoescape off`) переданный HTML будет интерпретирован как код, а не как текст.

**Пример**:
**Шаблон**:

```html
<body>
    {{ body }}

    {% autoescape off %}
        {{ body }}
    {% endautoescape %}
</body>
```

**Представление**:

```python
from django.shortcuts import render

def index(request):
    return render(request, 'blog/index.html', context={'body': '<h1>Hello World!</h1>'})
```

**Результат**:

1. **С экранированием (по умолчанию)**:
    
    ```css
    <h1>Hello World!</h1>
    ```
    
2. **Без экранирования (внутри autoescape off)**:
HTML отображается как заголовок:
    
    ```
    Hello World!
    ```
    

---

### 3. **Комментарии в шаблонах**

- **Многострочные комментарии**:
    
    ```html
    {% comment %}
        Этот текст будет проигнорирован шаблонизатором
    {% endcomment %}
    ```
    
- **Однострочные комментарии**:
    
    ```html
    {# Это однострочный комментарий #}
    ```
    

**Пример**:

```html
<p>Текст</p>
{# Этот блок будет проигнорирован #}
<p>Другой текст</p>
```

**Вывод**:

```
Текст
Другой текст
```

---

### Вывод:

1. **`with`**: Используется для задания временных переменных в блоке.
2. **`autoescape`**: Управляет экранированием HTML, делает вывод безопасным или позволяет отображать HTML как код.
3. **`comment`**: Для добавления комментариев в шаблоны, которые игнорируются при рендеринге.

## Статичные файлы

- **Статичные файлы (Static Files)**
    - Статичными файлами считаются файлы, не изменяющиеся динамически и не зависящие от контекста запроса:
        - CSS-файлы
        - Изображения
        - JavaScript-файлы
        - Нередко также статическими делают отдельные HTML-блоки (хедер, футер и др.)
    - В Django уже настроен базовый функционал для работы со статикой:
        - В `settings.py` определена переменная `STATIC_URL`, указывающая путь к статике, по умолчанию:
            
            ```python
            STATIC_URL = 'static/'
            ```
            
        - В `INSTALLED_APPS` по умолчанию подключено приложение `django.contrib.staticfiles`, которое отвечает за работу со статикой.
    - В шаблоне для использования статичных файлов:
        1. Вверху шаблона нужно добавить тег:
            
            ```
            {% load static %}
            ```
            
        2. Для вставки файлов использовать конструкцию:
            
            ```
            <img src="{% static 'images/django_logo.png' %}" alt="Django Logo">
            ```
            
    - Структура проекта может выглядеть так:
        
        ```arduino
        myproject/
          blog/
            static/
              images/
                django_logo.png
            templates/
              blog/
                ...
        ```
        
    - При необходимости указать дополнительные или альтернативные пути к статике, в `settings.py` можно добавить:
        
        ```python
        STATICFILES_DIRS = [
          BASE_DIR / 'static',
          '/var/www/static/',
          '/somefolder/'
        ]
        ```
        
- **TemplateView**
    - Если нужно просто вернуть пользователю содержимое шаблона без обработки данных во вьюхе, можно использовать класс `TemplateView`.
    - Пример подключения в `urls.py`:
        
        ```python
        from django.urls import path
        from django.views.generic import TemplateView
        from blog import views
        
        urlpatterns = [
          path('', views.index),
          path('about/', TemplateView.as_view(template_name='blog/about.html')),
          path('contact/', TemplateView.as_view(template_name='blog/contact.html')),
        ]
        ```
        
    - В пути к шаблону может указываться и папка, например: `blog/about.html`.
    - Параметр `extra_context` позволяет передать в шаблон дополнительные данные:
        
        ```python
        path('about/', TemplateView.as_view(
          template_name='blog/about.html',
          extra_context={'header': 'О сайте'}
        ))
        ```
        
    - В самом шаблоне можно теперь использовать переменную `{{ header }}`:
        
        ```html
        <!DOCTYPE html>
        <html>
        <head>
            <meta charset="utf-8" />
            <title>Учу Django</title>
        </head>
        <body>
            <h1>{{ header }}</h1>
        </body>
        </html>
        ```