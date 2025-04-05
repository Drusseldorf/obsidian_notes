### Основные возможности `unittest` в Django

1. **Наследование**  
    Все тестовые классы должны наследоваться от `django.test.TestCase` (которая в цепочке наследования идёт от `unittest.TestCase`).
    
2. **Поиск тестов**  
    Django ищет файлы, начинающиеся с `test*` (например, `test_routes.py`, `test_models.py`) во всех пакетах, где есть файл `__init__.py`.
    
3. **Формат тестов**
    
    - Логика та же, что и в `unittest`: методы начинаются с `test_`.
        
    - Проверки делаются методами `assertEqual`, `assertTrue`, `assertFalse` и пр.
        

---

### Первые тесты (пример из `test_trial.py`)

```python
# news/tests/test_trial.py
from django.test import TestCase

class Test(TestCase):
    def test_example_success(self):
        self.assertTrue(True)  # Тест всегда успешен

class YetAnotherTest(TestCase):
    def test_example_fails(self):
        self.assertTrue(False)  # Тест всегда упадёт
```

#### Запуск тестов и выборочно

```bash
# Запустить все тесты проекта
python manage.py test

# Запустить только тесты приложения news
python manage.py test news

# Запустить только тесты из файла test_trial.py
python manage.py test news.tests.test_trial

# Запустить только класс Test
python manage.py test news.tests.test_trial.Test

# Запустить конкретный тест в классе YetAnotherTest
python manage.py test news.tests.test_trial.YetAnotherTest.test_example_fails
```

- Параметр `-v` (`--verbosity`) с числами от 0 до 3 позволяет получать более развёрнутый или более краткий вывод.
    

---

### Тестовая база данных

- Django **автоматически** создаёт временную базу данных, на которую накатываются **все миграции**.
    
- Тесты работают только с этой временной БД, «реальная» БД не затрагивается.
    
- По окончании тестов временная база удаляется.
    
- Каждый класс `TestCase` изолирован транзакционно: изменения в одном классе не влияют на другой. То же и для каждого теста внутри класса.
    

---

### Создание объектов для тестов

В `unittest` обычно используют фикстуры — методы класса `TestCase`, где подготавливают данные:

- `setUp()` — запускается **перед каждым** тестом класса.
    
- `setUpClass()` — запускается **один раз** для всего класса (но в Django перед использованием нужно вызывать `super().setUpClass()`).
    
- **В Django** есть **метод `setUpTestData()`**, который удобнее, чем `setUpClass()`:
    
    - Вызывается один раз для всего класса.
        
    - Не требует ручного вызова родительского `super()`.
        
    - Создаёт данные, которые сохраняются для всех тестов класса, при этом каждый тест «откатывается» транзакционно к исходному состоянию.
        

#### Пример использования `setUpTestData()`

```python
# news/tests/test_trial.py
from django.test import TestCase
from news.models import News

class TestNews(TestCase):
    @classmethod
    def setUpTestData(cls):
        cls.news = News.objects.create(
            title='Заголовок новости',
            text='Тестовый текст',
        )

    def test_successful_creation(self):
        news_count = News.objects.count()
        self.assertEqual(news_count, 1)

    def test_title(self):
        self.assertEqual(self.news.title, 'Заголовок новости')
```

Чтобы не нарушать DRY, часто выносят повторяющиеся строки (например, `TITLE`, `TEXT`) в константы класса и используют `cls.TITLE` при создании, а в проверках – `self.TITLE`.

---

### Программный HTTP-клиент (класс `Client`)

- В `django.test` доступен программный клиент, имитирующий работу браузера.
    
- Обращение через `self.client` (доступен по умолчанию).
    
- Можно проверить:
    
    - `response.status_code` (код ответа, например `200`, `404`),
        
    - `response.content` (байтовые данные ответа),
        
    - `response.context` (данные, переданные в шаблон),
        
    - `response.templates` (список шаблонов, задействованных для рендера).
        
- Можно создавать **несколько клиентов**: например, для анонима и для авторизованного пользователя.
    
- Авторизация через `client.force_login(user)`.
    

---

#### Пример теста маршрутов (файл `news/tests/test_routes.py`)

```python
# news/tests/test_routes.py
from http import HTTPStatus
from django.test import TestCase
from django.urls import reverse

class TestRoutes(TestCase):
    def test_home_page(self):
        url = reverse('news:home')
        response = self.client.get(url)
        self.assertEqual(response.status_code, HTTPStatus.OK)
```

- `reverse('news:home')` возвращает реальный путь (например, `'/'`).
    
- Проверка `self.assertEqual(response.status_code, HTTPStatus.OK)` удостоверивается, что главная страница доступна.
    

---

### Best practices от опытных разработчиков (с учётом Django)

1. **Используйте `TestCase` от Django**, а не чистый `unittest.TestCase`, чтобы автоматом создавать/очищать тестовую БД.
    
2. **Работайте с `reverse()`** вместо жёстких адресов: если URL’ы изменятся, не придётся переделывать все тесты.
    
3. **Готовьте общие объекты один раз**: используйте `setUpTestData()`, если данные не меняются от теста к тесту. Для специфических случаев используйте `setUp()`.
    
4. **Разделяйте ответственность**: если один тест проверяет логику модели, другой — логику маршрутов, и т.д.
    
5. **Для проверки логики прав доступа** или работы сессий создавайте **несколько клиентов** — «анонимный» и «авторизованный» (через `force_login`).
    
6. **Проверяйте коды ответов** (особенно редиректы, `302` -> `200`, ошибки `403`, `404` и т.п.) и содержимое (при необходимости) через `response.content` или специальные вспомогательные методы — но избегайте чрезмерного дублирования.
    
7. **Не тестируйте сам Django**: рендеринг стандартных форм, встроенную авторизацию и т.п. — тестировать нужно свою бизнес-логику, а не фреймворк.
    
8. **Держите эксперименты в отдельных файлах** (например, `test_trial.py`) или удаляйте их, чтобы не захламлять проект — либо помечайте декоратором `@unittest.skip()`.
    

---

**Важные детали**

- При использовании `setUpClass()` и `tearDownClass()` в Django нужно обязательно вызывать `super().setUpClass()` / `super().tearDownClass()`.
    
- Метод `setUpTestData()` не требует вызова `super()`.
    
- Атрибуты `response`:
    
    - `content` — байтовые данные HTML-ответа,
        
    - `status_code` — числовой код ответа (200, 302, 403, 404, ...),
        
    - `context` — словарь данных, идущих в шаблон,
        
    - `templates` — список шаблонов, участвующих в рендере.
        

---

**Пример удаления экспериментальных тестов**

```python
# Если не нужны временные тесты — либо удаляем test_trial.py,
# либо помечаем классы и методы декоратором @skip:

import unittest
from django.test import TestCase

@unittest.skip('Временные тесты, пропускаем')
class Test(TestCase):
    def test_example(self):
        pass
```

Так тесты не будут мешаться при проверках и не повлияют на общий отчёт.