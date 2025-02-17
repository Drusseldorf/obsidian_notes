### Оформление HTML-кода

- Отступ перед вложенными элементами — два пробела:
    
    ```html
    <ul>
      <li>элемент списка</li>
    </ul>
    ```
    

- Одиночные теги типа `<br>` не следует закрывать (не надо так: `<br/>`).

- Названия атрибутов пишутся в двойных кавычках: `class="attr-not-single-quotes"`

- Имена классов и идентификаторов пишутся строчными буквами и разделяются дефисом `-`.
    
    Например: `my-class-not-underscore`
    

- Если описание тега длиннее 79 символов — нужен перенос строки. Новая строка отбивается двумя пробелами от начала строки родительского тега.
    
    ```html
    <div
      id="my-id"
      class="my-class-long-name">
      <p>
        Lorem Ipsum Dolor Sit....
      </p>
    </div>
    ```
    
    Если необходимо избежать пробела в текстовой части, строку можно перенести так:
    
    ```html
    <a
      id="my-id"
      class="my-class-long-name"
    >Lorem Ipsum Dolor Sit....</a>
    ```
    

### Django-шаблоны

Код шаблонов оформляется согласно [Coding style Django](https://docs.djangoproject.com/en/dev/internals/contributing/writing-code/coding-style/#template-style).

- До и после переменной ставится пробел:
    
    ```html
    {{ foo }}
    ```
    

- Не ставятся пробелы между фильтрами:
    
    ```html
    {{ name|lower }}
    ```
    

- Теги Django по значимости идентичны HTML-тегам. Отступ перед вложенными элементами — два пробела:
    
    ```html
    {% extends 'base.html' %}
    {% load static %}
    {% block css %}
    {% endblock %}
    
    <title>
      {% block title %}
        Надпись сверху
      {% endblock %}
    </title>
    
    {% block content %}
      <main>
        {% for post in object_list %}    
          <div class="single-post"> 
            <p>Текст поста</p>
          </div>
        {% endfor %}
      </main>
    {% endblock %}
    ```