## 1. Задача и назначение Django-форм

- **Стандартный подход (HTML-форма в шаблоне)**: работает, но Django «не знает» о форме.
- **Django Form**:
    1. Генерирует HTML-код формы.
    2. Обрабатывает и валидирует полученные данные.
    3. Упрощает внесение изменений: нужно лишь изменить класс формы, а не вручную править вёрстку во всех шаблонах.

## 2. Создание класса формы

- Создаём файл `forms.py` в приложении.
- Определяем класс, унаследованный от `django.forms.Form`.
    
    ```python
    # birthday/forms.py
    from django import forms
    
    class BirthdayForm(forms.Form):
        first_name = forms.CharField(label='Имя', max_length=20)
        last_name = forms.CharField(
            label='Фамилия',
            required=False,
            help_text='Необязательное поле'
        )
        birthday = forms.DateField(label='Дата рождения')
    ```
    
- **Типы полей**: см. [классы для полей формы](https://docs.djangoproject.com/en/3.2/ref/forms/fields/#built-in-field-classes).

## 3. Подключение формы во view-функцию

1. **Импортировать класс формы**:
    
    ```python
    from .forms import BirthdayForm
    ```
    
2. **Создать экземпляр формы и передать в контекст**:
    
    ```python
    def birthday(request):
        form = BirthdayForm()
        return render(request, 'birthday/birthday.html', {'form': form})
    ```
    
3. **Отобразить форму в шаблоне**:
    
    ```html
    <!-- templates/birthday/birthday.html -->
    {% extends "base.html" %}
    {% block content %}
      <form>
        {{ form.as_p }}  <!-- или form, form.as_table, form.as_ul -->
        <input type="submit" value="Submit">
      </form>
    {% endblock %}
    ```
    

## 4. Форматы вывода формы

- `{{ form }}` 
- `{{ form.as_ul }}` — вывод полей как список `<ul><li>...</li></ul>`.
- `{{ form.as_p }}` — поля в `<p>...</p>`.

## 5. Переопределение лейблов и подсказок

- Аргумент `label` устанавливает текст подписи поля.
- Аргумент `help_text` задаёт краткую подсказку пользователю.

## 6. Виджеты

- Определяют, какой HTML-элемент будет использован для поля (напр., `<input type="date">`).
- Указываются через аргумент `widget`:
    
    ```python
    birthday = forms.DateField(
        label='Дата рождения',
        widget=forms.DateInput(attrs={'type': 'date'})
    )
    ```
    
- Подробнее см. [виджеты в документации](https://docs.djangoproject.com/en/3.2/ref/forms/widgets/#built-in-widgets).