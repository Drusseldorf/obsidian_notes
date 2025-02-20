## 1. Обработка запроса главной страницы

- **View-функция** `index()` в приложении `homepage` отвечает за запрос к базе.
    
- С помощью Django ORM делается запрос к модели `IceCream`.
    
- Пример кода:
    
    ```python
    # homepage/views.py
    from django.shortcuts import render
    from ice_cream.models import IceCream
    
    def index(request):
        template_name = 'homepage/index.html'
        # Получаем все объекты модели IceCream
        ice_cream_list = IceCream.objects.all()
        context = {
            'ice_cream_list': ice_cream_list,
        }
        return render(request, template_name, context)
    ```
    

## 2. Ленивые запросы и рендеринг шаблона

- **Ленивая загрузка (lazy evaluation)**: SQL-запрос не выполняется, пока данные из QuerySet не запрошены в шаблоне.
    
- Если в шаблоне не используются данные из `context`, запросы не отправляются.
    
- Для проверки можно вывести переменную напрямую, например:
    
    ```html
    <!-- templates/homepage/index.html -->
    <h1>Главная страница</h1>
    {{ ice_cream_list }}
    ```
    
- Для более осмысленного отображения итерация по QuerySet’у:
    
    ```html
    <!-- templates/homepage/index.html -->
    {% for ice_cream in ice_cream_list %}
      <ul>
        <li>ID: {{ ice_cream.id }}</li>
        <li>Опубликовано: {{ ice_cream.is_published }}</li>
        <li>На главную: {{ ice_cream.is_on_main }}</li>
        <li>Название: {{ ice_cream.title }}</li>
        <li>Описание: {{ ice_cream.description }}</li>
        <li>FK wrapper: {{ ice_cream.wrapper_id }}</li>
        <li>FK category: {{ ice_cream.category_id }}</li>
      </ul>
    {% endfor %}
    ```
    

## 3. Оптимизация выборки с методом values()

- Часто на странице нужны не все поля модели.
    
- Метод `.values()` позволяет выбрать только необходимые столбцы (например, `id` и `title`).
    
- Пример изменения view-функции:
    
    ```python
    # homepage/views.py
    from django.shortcuts import render
    from ice_cream.models import IceCream
    
    def index(request):
        template_name = 'homepage/index.html'
        # Получаем только поля 'id' и 'title'
        ice_cream_list = IceCream.objects.values('id', 'title')
        context = {
            'ice_cream_list': ice_cream_list,
        }
        return render(request, template_name, context)
    ```
    
- При использовании `.values()` QuerySet превращается в список словарей:
    
    ```python
    <QuerySet [{'id': 1, 'title': 'Золотое мороженое'}, {'id': 2, 'title': 'Готическое мороженое'}, ...]>
    ```
    
- Соответствующий шаблон:
    
    ```html
    <!-- templates/homepage/index.html -->
    {% for ice_cream in ice_cream_list %}
      <ul>
        <li>ID: {{ ice_cream.id }}</li>
        <li>Название: {{ ice_cream.title }}</li>
      </ul>
    {% endfor %}
    ```
    

## Итог

- **View-функция** делает запрос к модели через `IceCream.objects.all()`, но запрос выполняется только при использовании данных в шаблоне.
- **Ленивые запросы** оптимизируют работу сервера, не отправляя запросы, пока данные не нужны.
- Использование метода **`.values()`** позволяет выбирать только нужные поля для передачи в шаблон, снижая нагрузку на БД.
