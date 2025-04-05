### Плагин `pytest-django`

**Что даёт**:

1. **Упрощает тестирование Django-проектов** на `pytest`.
    
2. **Готовые фикстуры** для взаимодействия с базой данных и клиентом (client, admin_client, admin_user и т.д.).
    
3. **Специальные функции-ассерты**, похожие на методы из `django.test.TestCase` (например, `assertRedirects()`), импортируются из `pytest_django.asserts`.
    

---

### Установка и настройка

```bash
pip install pytest-django==4.5.2
```

- В корне проекта создаём файл `pytest.ini` (или другой конфигурационный файл), прописываем:
    
    ```ini
    [pytest]
    DJANGO_SETTINGS_MODULE = your_project.settings
    ```
    
    где `your_project.settings` – точечная нотация к файлу `settings.py`.
    

**Пример** (для проекта YaNote):

```ini
[pytest]
DJANGO_SETTINGS_MODULE = yanote.settings
```

- Это позволит `pytest-django` «увидеть» Django-проект и настроить тестовую базу данных.
    

---

### Основные фикстуры `pytest-django`

1. **`client`**
    
    - Анонимный `django.test.Client`.
        
    - Пример использования:
        
        ```python
        def test_with_client(client):
            response = client.get('/')
            assert response.status_code == 200
        ```
        
2. **`admin_client`**
    
    - Клиент, авторизованный под суперюзером (логин: `admin`, пароль: `password`).
        
    - Пример:
        
        ```python
        def test_only_for_superuser(admin_client):
            response = admin_client.get('/admin-only/')
            assert response.status_code == 200
        ```
        
3. **`admin_user`**
    
    - Объект суперюзера (логин: `admin`, пароль: `password`), **без** клиента.
        
    - Полезно, если нужно работать именно с юзером (назначить автором статьи и т.п.).
        
4. **`django_user_model`**
    
    - Возвращает **модель** пользователя, которая используется в проекте (обычно `auth.User`, если не переопределён `AUTH_USER_MODEL`).
        
    - Пример:
        
        ```python
        def test_create_user(django_user_model):
            user = django_user_model.objects.create(username='testuser')
            assert user.username == 'testuser'
        ```
        
    - Можно логинить через `client.force_login(user)`.
        
5. **`db`**
    
    - Нужна для тестов, которым требуется доступ к базе данных, но сама по себе обычно не вызывается явно, а подтягивается другими фикстурами.
        
    - Если вы **не используете** вышеперечисленные фикстуры, но **нужен** доступ к базе, применяйте маркер `@pytest.mark.django_db`.
        

---

### Ассерты из `pytest_django.asserts`

Для проверки, характерной для Django-проектов:

```python
from pytest_django.asserts import assertRedirects, assertTemplateUsed, ...
```

- Аналогично `self.assertRedirects()` и др. из `unittest.TestCase`.
    

---

### Как всё работает?

- `pytest-django` **видит** указанный `settings.py` и **поднимает** тестовую БД перед стартом.
    
- Автоматически **прокручивает** тесты в транзакциях для изоляции.
    
- Даёт фикстуры для упрощения создания пользователей, клиент-сессий, аутентификаций.
    

---

### Примерный сценарий использования фикстур

```python
@pytest.mark.django_db
def test_user_can_post_comment(client, django_user_model):
    # Создать пользователя
    user = django_user_model.objects.create(username='TestUser')
    # Залогинить этого пользователя
    client.force_login(user)
    # Запостить комментарий
    response = client.post('/comment/new/', data={'text': 'Hello!'})
    assert response.status_code == 302  # Redirect?
    ...
```

**Что происходит**:

- `@pytest.mark.django_db` даёт доступ к базе.
    
- `client` – анонимный клиент (фикстура).
    
- `django_user_model` – модель User (можно создавать пользователей).
    
- `client.force_login(user)` – «авторизуемся».
    

---

### Сводка по основным фикстурам

- **`client`** – анонимный `Client`.
    
- **`admin_client`** – клиент, залогиненный суперюзером.
    
- **`admin_user`** – объект суперюзера (без клиента).
    
- **`django_user_model`** – сама модель пользователя, которую можно использовать для создания кастомных юзеров.
    
- **`@pytest.mark.django_db`** или фикстура `db` – для тестов, которым нужен доступ к БД, если не используете другие фикстуры, которые её уже подключают.
    

---

**Best practices**:

1. Указывайте `DJANGO_SETTINGS_MODULE` в `pytest.ini`.
    
2. Используйте фикстуру `client` (или `admin_client`) вместо ручного `django.test.Client()`.
    
3. При необходимости создавать дополнительных пользователей применяйте `django_user_model`.
    
4. Для тестов, которым нужна база, но вы не используете фикстуры `client`/`admin_user`, ставьте `@pytest.mark.django_db`.
    
5. Импортируйте нужные ассерты из `pytest_django.asserts` – это делает тесты выразительнее, ближе к `django.test.TestCase`.
    
