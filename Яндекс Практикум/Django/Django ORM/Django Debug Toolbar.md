
## 1. Middleware и его роль

- **Middleware** — промежуточные слои, которые обрабатывают запросы перед вызовом view-функции и ответы после её работы.
- Запрос проходит по цепочке middleware в заданном порядке, а ответ — в обратном порядке.
- Django Debug Toolbar реализуется как middleware, собирающее информацию о запросах и ответах.

## 2. Установка Django Debug Toolbar

1. **Установка пакета**
    - Активируйте виртуальное окружение и выполните:
        
        ```bash
        pip install django-debug-toolbar==3.8.1
        ```
        
2. **Обновление зависимостей**
    - В корневой папке проекта выполните:
        
        ```bash
        pip freeze > requirements.txt
        ```
        
    - Убедитесь, что пакет появился в файле `requirements.txt`.

## 3. Настройка в файле settings.py

1. **Регистрация приложения**
    - Добавьте `'debug_toolbar'` в список `INSTALLED_APPS`.
    - Разместите его ниже `django.contrib.staticfiles`:
        
        ```python
        INSTALLED_APPS = [
            # ... другие приложения ...
            'debug_toolbar',
        ]
        ```
        
2. **Подключение middleware**
    - Добавьте в конец списка `MIDDLEWARE`:
        
        ```python
        MIDDLEWARE = [
            # ... стандартные middleware ...
            'debug_toolbar.middleware.DebugToolbarMiddleware',
        ]
        ```
        
3. **Ограничение доступа**
    - Укажите IP-адреса, с которых допускается отображение панели:
        
        ```python
        INTERNAL_IPS = [
            '127.0.0.1',
        ]
        ```
        

## 4. Настройка маршрутов в urls.py

- В файле `urls.py` импортируйте настройки и добавьте правило для debug_toolbar при включённом режиме разработки:
    
    ```python
    from django.conf import settings
    from django.urls import include, path
    
    urlpatterns = [
        # ... ваши маршруты ...
    ]
    
    if settings.DEBUG:
        import debug_toolbar
        urlpatterns += [path('__debug__/', include(debug_toolbar.urls))]
    ```
    

## 5. Итог

- Django Debug Toolbar теперь подключён как middleware и отображается только при `DEBUG = True`.
- Панель предоставляет информацию о SQL-запросах, времени обработки, кэшировании и прочих аспектах запроса, что значительно облегчает отладку проекта.

Следуя этому конспекту, вы сможете самостоятельно подключить и настроить Django Debug Toolbar в вашем проекте.