# Глава 3. Обзор FastApi

## Начало работы

### Установка зависимостей:

```
pip install fastapi
pip install uvicorn
```

FastAPI не имеет встроенного веб-сервера, но рекомендуется использовать **Uvicorn**.

### Пример программы с одним эндпоинтом (модуль: hello.py):

аналогично эти функции называют view\ вьюшки \ представления

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/hi")
def greet():
    return "Hello? World?"
```

- **app** — объект FastAPI верхнего уровня, представляющий всё приложение.
- **@app.get("/hi")** — декоратор для маршрута, определяющий, что GET-запросы по пути `/hi` обрабатываются функцией **greet**.
- **greet** — функция обработки пути (может принимать аргументы, об этом позже).

Важно отметить что порядок этих функций важен. Если эндпоинт, который должна обрабатывать функция является частным случаем какого то другого более общего, то частный случай должен стоять первее общего. Например:
/id/last_goods должен стоять до функции которая обрабатывает /id/{goods_id}

### Запуск приложения:

```bash
uvicorn hello:app --reload
```

- **hello** — имя модуля.
- **app** — имя переменной FastAPI.
- **-reload** — опция для авто-перезапуска сервера при изменении кода (удобно при разработке).

Приложение по умолчанию запускается на **localhost:8000**.

### Тестирование:

GET-запрос на [http://localhost:8000/hi](http://localhost:8000/hi) вернёт: `"Hello? World?"`

## URL-пути

Изменим код для использования переменной в пути:

```python
@app.get("/hi/{who}")
def greet(who):
    return f"Hello, {who}"
```

FastAPI сопоставляет переменные в пути (в фигурных скобках) с аргументами функции.

Запрос: [http://localhost:8000/hi/Andrey](http://localhost:8000/hi/Andrey) вернёт: `"Hello, Andrey"`

## Параметры запроса

Параметры запроса передаются как query string: `?name=value`

Изменим код для использования параметра запроса:

```python
@app.get("/hi")
def greet(who: str):
    return f"Hello, {who}"
```

Теперь `who` воспринимается как query параметр. Запрос: [http://localhost:8000/hi?who=Andrey](http://localhost:8000/hi?who=Andrey) вернёт: `"Hello, Andrey"`

Если параметр отсутствует, вернётся ошибка:

```json
{
    "detail": [
        {
            "type": "missing",
            "loc": ["query", "who"],
            "msg": "Field required",
            "input": null}
    ]
}
```

## Тело запроса

POST-запросы позволяют отправлять данные в теле запроса. Изменим код для обработки тела запроса:

```python
from fastapi import FastAPI, Body

app = FastAPI()

@app.post("/hi")
def greet(who: str = Body(embed=True)):
    return f"Hello, {who}"
```

Используем **Body(embed=True)** для обработки данных JSON. Пример POST-запроса:

```json
{
    "who": "Andrey"
}
```

Ответ будет: `"Hello, Andrey"`

А вообще говоря, мы можем создать класс Pydantic, унаследовавшись от BaseModel. И далее передать этот класс вместо Body. Тогда FastApi сразу поймет что ожидается json и какие именно поля, согласно пайдантик модели, которую мы передали в функцию, например:

```python
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()

class UserEmail(BaseModel):
    email: EmailStr

@app.post("/register_email")
def register_email(email: UserEmail):
    return {"success": True,
            **email.model_dump()}
```

## HTTP-заголовки

FastAPI позволяет легко работать с заголовками. Пример:

```python
from fastapi import FastAPI, Header

app = FastAPI()

@app.post("/hi")
def greet(who: str = Header()):
    return f"Hello, {who}"
```

FastAPI автоматически преобразует HTTP-заголовки в формат snake_case. Например, запрос с заголовком **User-Agent** можно обработать так:

```python
@app.post("/agent")
def get_agent(user_agent: str = Header()):
    return user_agent
```

## HTTP-ответы

FastAPI автоматически возвращает JSON-ответы.

### Коды состояния:

- **200** — успешный ответ (по умолчанию).
- **4xx** — ошибки клиента.

Пример изменения статуса:

```python
@app.get("/happy", status_code=201)
def happy():
    return ":)"
```

### Возвращение заголовков:

```python
from fastapi import Response

@app.get("/header/{name}/{value}")
def header(name: str, value: str, response: Response):
    response.headers[name] = value
    return "some body"
```

## Типы ответов

FastAPI поддерживает несколько типов ответов, которые можно импортировать:

- **JSONResponse** (по умолчанию).
- **HTMLResponse**
- **PlainTextResponse**
- **RedirectResponse**
- **FileResponse**
- **StreamingResponse**

### Пример использования кастомного ответа:

```python
from fastapi import Response

@app.get("/custom")
def custom_response():
    return Response(content="Custom response", media_type="text/plain")
```

## Преобразование типов

FastAPI использует **jsonable_encoder** для сериализации сложных объектов, таких как **datetime**.

jsonable_encoder - Преобразует любой объект в формат, который может быть закодирован в JSON. Это используется внутри FastAPI, чтобы убедиться, что все, что вы возвращаете, может быть закодировано в JSON перед отправкой клиенту.
Вы также можете использовать его самостоятельно, например, для преобразования объектов перед сохранением их в базе данных, которая поддерживает только JSON

Пример:

```python
from fastapi.encoders import jsonable_encoder
import datetime

data = datetime.datetime.now()
encoded_data = jsonable_encoder(data)
```

## Модели данных и `response_model`

FastAPI позволяет использовать разные модели данных для ввода и вывода, что удобно для удаления конфиденциальных данных или разделения полей.

Пример:

```python
from pydantic import BaseModel
from datetime import datetime

class TagIn(BaseModel):
    tag: str

class Tag(BaseModel):
    tag: str
    created: datetime
    secret: str

class TagOut(BaseModel):
    tag: str
    created: datetime
```

Пример использования:

```python
@app.get("/tag/{tag_str}", response_model=TagOut)
def get_tag(tag_str: str):
    return {"tag": tag_str, "created": datetime.utcnow(), "secret": "hidden"}
```

В этом примере поле `secret` скрыто для пользователя благодаря использованию **response_model**.