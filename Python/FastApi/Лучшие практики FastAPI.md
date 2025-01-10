# Лучшие практики FastAPI

Это перевод статьи по лучшим практикам, которую я нашел на гитхабе

### Структура проекта

Существует множество способов структурировать проект, но лучшая структура — это та, которая последовательна, проста и без сюрпризов.

Многие примерные проекты и учебные материалы делят проект по типу файлов (например, `crud`, `routers`, `models`), что хорошо работает для микросервисов или проектов с небольшими объёмами. Однако этот подход не подошёл для нашего монолита с множеством доменов и модулей.

Структура, которую я нашёл более масштабируемой и развиваемой для этих случаев, вдохновлена проектом Netflix Dispatch с некоторыми небольшими модификациями.

[https://github.com/Netflix/dispatch/blob/master/src/dispatch/database/core.py](https://github.com/Netflix/dispatch/blob/master/src/dispatch/database/core.py)

```bash
fastapi-project
├── alembic/
├── src
│   ├── auth
│   │   ├── router.py
│   │   ├── schemas.py  # модели Pydantic
│   │   ├── models.py  # модели БД
│   │   ├── dependencies.py
│   │   ├── config.py  # локальные настройки
│   │   ├── constants.py
│   │   ├── exceptions.py
│   │   ├── service.py
│   │   └── utils.py
│   ├── aws
│   │   ├── client.py  # модель клиента для внешних сервисов
│   │   ├── schemas.py
│   │   ├── config.py
│   │   ├── constants.py
│   │   ├── exceptions.py
│   │   └── utils.py
│   └── posts
│   │   ├── router.py
│   │   ├── schemas.py
│   │   ├── models.py
│   │   ├── dependencies.py
│   │   ├── constants.py
│   │   ├── exceptions.py
│   │   ├── service.py
│   │   └── utils.py
│   ├── config.py  # глобальные настройки
│   ├── models.py  # глобальные модели
│   ├── exceptions.py  # глобальные исключения
│   ├── pagination.py  # глобальный модуль, напр., для пагинации
│   ├── database.py  # подключение к БД
│   └── main.py
├── tests/
│   ├── auth
│   ├── aws
│   └── posts
├── templates/
│   └── index.html
├── requirements
│   ├── base.txt
│   ├── dev.txt
│   └── prod.txt
├── .env
├── .gitignore
├── logging.ini
└── alembic.ini

```

### Описание структуры:

- **src/** — высший уровень приложения, содержит общие модели, настройки и константы.
- **src/main.py** — корень проекта, инициализирующий FastAPI приложение.
- Каждый пакет имеет собственный `router.py`, `schemas.py`, `models.py` и т.д.
- **router.py** — ядро каждого модуля с маршрутами (endpoints).
- **schemas.py** — модели Pydantic.
- **models.py** — модели БД.
- **service.py** — бизнес-логика, специфичная для модуля.
- **dependencies.py** — зависимости маршрутов.
- **constants.py** — константы и коды ошибок.
- **config.py** — например, переменные окружения.
- **utils.py** — небизнесовые функции, например, нормализация ответов.
- **exceptions.py** — специфичные для модуля исключения.

### Асинхронные маршруты

FastAPI — это асинхронный фреймворк, предназначенный для работы с асинхронными операциями ввода-вывода. Однако FastAPI не ограничивает вас только асинхронными маршрутами, вы можете использовать и синхронные.

Асинхронные маршруты предпочтительны, когда речь идёт о задачах, интенсивных по вводу-выводу.

Пример:

```python
import asyncio
import time
from fastapi import APIRouter

router = APIRouter()

@router.get("/terrible-ping")
async def terrible_ping():
    time.sleep(10)  # блокирующая операция, процесс будет заблокирован
    return {"pong": True}

@router.get("/good-ping")
def good_ping():
    time.sleep(10)  # блокирующая операция, но в отдельном потоке
    return {"pong": True}

@router.get("/perfect-ping")
async def perfect_ping():
    await asyncio.sleep(10)  # неблокирующая операция ввода-вывода
    return {"pong": True}
```

### Задачи с интенсивной нагрузкой на ввод-вывод (I/O):

Асинхронные маршруты, определённые с `async def`, вызываются с помощью `await`, что позволяет избежать блокировки задач во время выполнения.

### Задачи с высокой нагрузкой на CPU:

Для оптимизации CPU-интенсивных задач следует использовать процессы, а не потоки, так как GIL (Global Interpreter Lock) не позволяет эффективно использовать несколько потоков для таких задач.

### Pydantic

Pydantic предоставляет богатый набор инструментов для валидации и преобразования данных. Он поддерживает требуемые и необязательные поля, регулярные выражения, перечисления (enums) и многое другое.

Пример:

```python
from enum import Enum
from pydantic import BaseModel, EmailStr, Field

class MusicBand(str, Enum):
   AEROSMITH = "AEROSMITH"
   QUEEN = "QUEEN"
   ACDC = "AC/DC"

class UserBase(BaseModel):
    first_name: str = Field(min_length=1, max_length=128)
    username: str = Field(min_length=1, max_length=128, pattern="^[A-Za-z0-9-_]+$")
    email: EmailStr
    age: int = Field(ge=18, default=None)
    favorite_band: MusicBand | None = None
```

### Пользовательская модель Pydantic:

Создание глобальной базовой модели позволяет стандартизировать формат datetime или добавлять общие методы.

Пример:

```python
from datetime import datetime
from zoneinfo import ZoneInfo
from pydantic import BaseModel

class CustomModel(BaseModel):
    def serializable_dict(self, **kwargs):
        default_dict = self.dict()
        return jsonable_encoder(default_dict)
```

### Разделение настроек Pydantic:

Лучше разделить настройки на разные модули и домены для улучшения поддерживаемости и организации.

Пример:

```python
from pydantic_settings import BaseSettings

class AuthConfig(BaseSettings):
    JWT_ALG: str
    JWT_SECRET: str

auth_settings = AuthConfig()
```

### Зависимости:

Зависимости FastAPI можно использовать не только для внедрения зависимостей в маршруты, но и для валидации данных против внешних сервисов или БД.

Пример:

```python
async def valid_post_id(post_id: str):
    post = await service.get_by_id(post_id)
    if not post:
        raise PostNotFound()
    return post
```

### Цепочка зависимостей:

Зависимости могут использовать другие зависимости, что уменьшает дублирование кода.

Пример:

```python
async def valid_owned_post(
    post: dict = Depends(valid_post_id),
    token_data: dict = Depends(parse_jwt_data),
):
    if post["creator_id"] != token_data["user_id"]:
        raise UserNotOwner()
    return post
```

### Разделение и переиспользование зависимостей:

FastAPI кэширует результаты зависимостей в пределах одного запроса, что позволяет избежать повторных вызовов.

### Разное:

- **Сериализация:** FastAPI всегда преобразует объекты Pydantic в словари перед сериализацией в JSON.
- **Использование sync SDK:** Если необходимо использовать синхронную библиотеку, используйте её в пуле потоков.

Пример:

```python
from fastapi import FastAPI
from fastapi.concurrency import run_in_threadpool

app = FastAPI()

@app.get("/")
async def call_sync_library():
    await run_in_threadpool(some_sync_function)
```

### Базы данных и миграции:

- Устанавливайте явные правила именования индексов и ключей в БД для согласованности.
- Используйте обратимые миграции с Alembic, давайте миграциям понятные имена.

Пример конвенции для индексов:

```python
POSTGRES_INDEXES_NAMING_CONVENTION = {
    "ix": "%(column_0_label)s_idx",
    "uq": "%(table_name)s_%(column_0_name)s_key",
}
```

### Написание тестов:

Сразу используйте асинхронный клиент для тестирования, чтобы избежать проблем с циклом событий.