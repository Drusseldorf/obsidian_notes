## 1. Зачем нужны представления на основе классов (CBV)

- Django предоставляет набор **типовых классов** для решения стандартных задач: отображение списков, отдельных объектов, редактирование, создание и удаление записей.
- Это экономит код и снижает вероятность ошибок: типичные операции (CRUD) уже реализованы «под капотом», достаточно переопределить пару атрибутов.
- В традиционных представлениях-функциях (FBV) вы вручную пишете логику работы с формами, валидацией, выборкой из БД, пагинацией – а CBV делают это автоматически.

---

## 2. Основная идея CBV

1. Вместо `def my_view(request): ...` создаётся класс `MyView(ClassView)`.
2. В файле `urls.py` вместо `path('...', my_view)` пишем `path('...', MyView.as_view())`.
3. Внутри класса указываем настройки (например, `model =`, `template_name =` и т. п.).
4. Django сам берёт на себя рутину: обращение к базе, пагинацию, переадресацию после сохранения формы и т.д.

---

## 3. Пример: `ListView` (просмотр списка объектов)

### 3.1 Исходная ситуация (FBV)

```python
def birthday_list(request):
    birthdays = Birthday.objects.order_by('id')
    paginator = Paginator(birthdays, 10)
    page_number = request.GET.get('page')
    page_obj = paginator.get_page(page_number)
    context = {'page_obj': page_obj}
    return render(request, 'birthday/birthday_list.html', context)
```

### 3.2 Переписываем на `ListView`

```python
from django.views.generic import ListView

class BirthdayListView(ListView):
    model = Birthday              # Какая модель
    ordering = 'id'               # Сортировка
    paginate_by = 10              # Пагинация (10 на страницу)
```

**В `urls.py`:**

```python
urlpatterns = [
    path('list/', BirthdayListView.as_view(), name='list'),
]
```

**Как это работает:**

- Django ищет шаблон по умолчанию: `<appname>/<modelname>_list.html`. У нас это `birthday/birthday_list.html`.
- В контекст передается `page_obj` (и ещё несколько переменных).
- Пагинация, сортировка и выборка объектов делаются автоматически.

---

## 4. Пример: `CreateView` (создание объектов)

### 4.1 Создание объекта (быстрый старт)

```python
from django.views.generic import CreateView
from .models import Birthday

class BirthdayCreateView(CreateView):
    model = Birthday
    fields = '__all__'            # Или список конкретных полей
    template_name = 'birthday/birthday.html'
    success_url = reverse_lazy('birthday:list')
```

**В `urls.py`:**

```python
urlpatterns = [
    path('', BirthdayCreateView.as_view(), name='create'),
    ...
]
```

**Что произойдёт:**

- При GET-запросе Django сгенерирует форму (по всем указанным полям модели).
- При POST-запросе данные автоматически валидируются и сохраняются, а пользователь будет переадресован на `success_url`.

Однако в таком варианте не будет вашей кастомной логики, например, валидации через `clean_...`, специальных виджетов и т.д. Если вам нужна **уже написанная форма `BirthdayForm`** (модельная с особыми полями, виджетами, валидацией и пр.), то указываем `form_class`, а не `fields`:

```python
from .forms import BirthdayForm

class BirthdayCreateView(CreateView):
    model = Birthday
    form_class = BirthdayForm
    template_name = 'birthday/birthday.html'
    success_url = reverse_lazy('birthday:list')
```

Теперь класс будет использовать вашу форму и всё её поведение.

---

## 5. Пример: `UpdateView` (редактирование объектов)

Очень похоже на `CreateView`, но теперь речь о существующем объекте:

```python
from django.views.generic import UpdateView

class BirthdayUpdateView(UpdateView):
    model = Birthday
    form_class = BirthdayForm
    template_name = 'birthday/birthday.html'
    success_url = reverse_lazy('birthday:list')
```

В `urls.py`:

```python
urlpatterns = [
    path('<int:pk>/edit/', BirthdayUpdateView.as_view(), name='edit'),
    ...
]
```

- `UpdateView` ищет объект с ключом `pk`, автоматически создаёт форму для редактирования.
- При отправке формы делает `form.save()`, обновляя существующую запись.

---

## 6. Пример: `DeleteView` (удаление объектов)

```python
from django.views.generic import DeleteView

class BirthdayDeleteView(DeleteView):
    model = Birthday
    success_url = reverse_lazy('birthday:list')
```

В `urls.py`:

```python
urlpatterns = [
    path('<int:pk>/delete/', BirthdayDeleteView.as_view(), name='delete'),
    ...
]
```

**Шаблон** для подтверждения удаления по умолчанию ищется по схеме `<appname>/<modelname>_confirm_delete.html`.

- На этой странице обычно только одна кнопка «Удалить» (POST-запрос).
- После удаления пользователь переадресуется на `success_url`.

---

## 7. Сокращение кода за счёт МИКСИНОВ

Когда вы заметите, что в `CreateView`, `UpdateView` и `DeleteView` повторяются те же настройки (`model`, `success_url`, `template_name` и т. п.), имеет смысл вынести их в класс-миксин. Пример:

```python
class BirthdayMixin:
    model = Birthday
    success_url = reverse_lazy('birthday:list')

class BirthdayFormMixin:
    form_class = BirthdayForm
    template_name = 'birthday/birthday.html'

class BirthdayCreateView(BirthdayMixin, BirthdayFormMixin, CreateView):
    pass

class BirthdayUpdateView(BirthdayMixin, BirthdayFormMixin, UpdateView):
    pass

class BirthdayDeleteView(BirthdayMixin, DeleteView):
    pass
```

- Миксины **«подмешивают»** (mixin) атрибуты и методы к основному классу.
- Порядок наследования важен: миксины идут первыми, потом идёт родительский класс из Django (`CreateView`, `UpdateView` и т.п.).

---

## 8. Особенности шаблонов в CBV

1. **Имя шаблона**
    - Для `ListView`: `<model>_list.html` по умолчанию.
    - Для `CreateView`: `<model>_form.html` по умолчанию.
    - Для `UpdateView`: `<model>_form.html` по умолчанию.
    - Для `DeleteView`: `<model>_confirm_delete.html` по умолчанию.
    - Если имя у вас другое, надо явно указать `template_name = "... .html"`.
2. **Переменные в контексте**
    - `ListView` передаёт `page_obj`, `object_list` и т.п.
    - `CreateView` / `UpdateView` передают `form` (и `object`, если редактирование).
    - `DeleteView` передаёт `object` (и `<modelname>`), но не `form`.

---

## 9. Советы и бест-практики

1. **Выбирайте правильный generic-класс**
    
    - `ListView` для списков, `DetailView` для отдельного объекта, `CreateView`/`UpdateView`/`DeleteView` для CRUD.
    - Если задача нестандартная — наследуйтесь от `View` или `TemplateView` и пишите логику вручную.
2. **Используйте `form_class` вместо `fields`, когда нужна расширенная логика**
    
    - Так вы сможете подключить собственный `ModelForm`, где уже описаны валидации, виджеты, методы `clean_<field>` и т.д.
3. **Настраивайте `success_url`**
    
    - Или переопределяйте метод `get_success_url()`, если целевой адрес зависит от данных (например, редиректить на detail-страницу только что созданного объекта).
4. **Сокращайте код миксинами**
    
    - Если у вас несколько представлений для одной модели с одинаковыми настройками — выносите их в миксин.
5. **Переопределяйте методы при необходимости**
    
    - Например, `form_valid()`, `get_queryset()`, `get_context_data()`. Это позволяет тонко настраивать поведение CBV.
6. **Не бойтесь смотреть в документацию или код Django**
    
    - CBV могут показаться магическими, но это логичный набор методов. К примеру, [документация по generic CBV](https://docs.djangoproject.com/en/4.2/topics/class-based-views/generic-display/) очень подробная.
7. **Осторожно при переименовании шаблонов**
    
    - Если вы не используете «стандартные» имена шаблонов, всегда указывайте `template_name`.
8. **Когда CBV кажутся перегруженными**
    
    - Если логика становится слишком хитрой (много методов переопределяете, много условных ветвлений), иногда проще вернуться к FBV либо комбинировать. Или разбить на миксины.