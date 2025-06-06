## 1. Что такое CSRF-атака

**CSRF (Cross-Site Request Forgery)** — это «межсайтовая подделка запроса», когда пользователь (часто неосознанно) отправляет запрос на сторонний сайт, в котором он уже авторизован. Цель злоумышленника — заставить браузер пользователя выполнить нежелательные действия (изменение данных, перевод денег и т.п.).

### Пример

1. На сайте злоумышленника есть форма, которая отправляет **POST**-запрос в ваш Django-проект (или, допустим, сайт банка).
2. Пользователь случайно попадает на этот сайт, нажимает «Отправить» в «безобидной» форме, а та под капотом указывает `action="https://somebank.com/transfer"` – результат:
    - Запрос уходит от имени пользователя (поскольку в браузере сохранены cookies с авторизацией).
    - Банк считает, что запрос идёт от самого пользователя, и может выполнить опасную операцию (например, перевод средств).

---

## 2. Как Django защищается от CSRF

### 2.1 Вставка CSRF-токена

- Django внедряет в **каждую** HTML-форму (которая отправляет **POST**-запрос) специальное скрытое поле:
    
    ```html
    <input type="hidden" name="csrfmiddlewaretoken" value="...уникальный-токен...">
    ```
    
- Одновременно в **cookies** пользователя сохраняется такой же токен.
- При получении **POST**-запроса Django сравнивает значение токена из формы и токен из cookies.
    - Если они совпадают (и не истекли, и корректны по алгоритму), запрос считается легитимным.
    - Если нет — возвращается ошибка 403 (Forbidden).

### 2.2 GET-запросы не требуют CSRF-токена

- **GET**-запросы, по определению, только читают данные. Django предполагает, что эти действия не меняют состояние сервера, поэтому защита от CSRF здесь не обязательна.
- Если всё же используется метод GET для изменения данных (что само по себе **плохая практика**), защита CSRF уже не сработает. Поэтому **важно** использовать методы запросов по назначению: GET для чтения, POST/PUT/DELETE для изменения.

---

## 3. Демонстрация уязвимости (учебный пример)

1. **guile.html** (находится вне проекта) содержит форму:
    
    ```html
    <form action="http://127.0.0.1:8000/birthday/">
      <!-- поля... -->
      <button type="submit">Отправить</button>
    </form>
    ```
    
2. Если метод формы был GET, запрос проходил и обрабатывался проектом — хоть мы и формально «вне» Django.
3. Если сменить на `method="post"`:
    
    ```html
    <form action="http://127.0.0.1:8000/birthday/" method="post">
      <!-- поля... -->
    </form>
    ```
    
    - То запрос **не пройдёт**. Django выдаст ошибку 403, потому что не находит валидного CSRF-токена.

Таким образом, **без** CSRF-токена любой «чужой» файл может отправить **GET**-запросы без проблем, но для **POST**-запросов (реально меняющих данные) защита включена по умолчанию.

---

## 4. Правильное оформление форм в Django

### 4.1 Пример в шаблоне

Когда вы создаёте форму, которая отправляет **POST**-запрос, обязательно добавляйте тег:

```html
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Отправить</button>
</form>
```

- `{% csrf_token %}` — специальный шаблонный тег, который вставит скрытое поле с токеном.
- При каждом запросе этот токен сверяется с cookies.

### 4.2 Где генерируется токен

- По умолчанию включена middleware `django.middleware.csrf.CsrfViewMiddleware`, которая и выполняет всю логику.
- Токен формируется при загрузке страницы, и его значение записывается как в HTML-форму, так и в cookies.

---

## 5. Бест-практики и рекомендации

1. **Используйте метод POST для «серьёзных» операций**
    
    - Если ваша форма что-то сохраняет или изменяет в БД, делайте это методами POST/PUT/DELETE, чтобы защита CSRF работала.
    - Не используйте GET там, где меняются данные! Иначе защита CSRF **не поможет**.
2. **Не забывайте тег `{% csrf_token %}`**
    
    - Очень частая ошибка новичков: забывают поставить его в шаблоне и удивляются ошибке 403 Forbidden при отправке формы.
3. **Не отключайте CSRF без необходимости**
    
    - Django позволяет временно отключить защиту (декоратор `@csrf_exempt` и т.д.), но это открывает потенциальную уязвимость. Делайте это только осознанно и в редких случаях (например, если используете собственную систему токенов или для интеграции с внешними сервисами).
4. **В продакшене используйте HTTPS**
    
    - Без HTTPS cookies и содержимое запроса могут быть перехвачены. CSRF-токен будет защищать от ложных запросов, но не защитит от «подслушивания».
5. **Проверяйте сессии и авторизацию**
    
    - CSRF-токен работает в связке с cookies, но убедитесь, что у вас правильно настроена аутентификация и авторизация в проекте.
6. **Всегда читайте логику кода во view**
    
    - Даже при наличии CSRF-токена, если view-функция не проверяет права, пользователь может сам себе навредить (или другому пользователю при особых условиях). CSRF защищает _только_ от подделки запросов, а не от всех возможных угроз.