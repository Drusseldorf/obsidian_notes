# Запросы к БД через ORM

В этой заметке мы рассмотрим, как работать с базой данных через SQLAlchemy ORM в асинхронной среде с использованием `async_session`. Это позволяет писать запросы к базе данных на Python, используя синтаксис, приближенный к объектно-ориентированному стилю.

## Добавление пользователя в базу данных

Напишем функцию, которая добавляет `telegram_id` пользователя в базу данных, если его еще нет. Эта функция будет полезна, когда новый пользователь начинает взаимодействовать с ботом.

```python
from app.database.models import async_session  # Импортируем асинхронную сессию для работы с БД
from app.database.models import User  # Импортируем модель User для работы с таблицей пользователей
from sqlalchemy import select  # Импортируем функцию select для создания SQL-запросов

async def set_user(telegram_id: int):
    """
    Функция для добавления нового пользователя в базу данных, если его еще нет.
    :param telegram_id: Уникальный идентификатор пользователя в Telegram
    """
    async with async_session() as session:  # Открываем асинхронную сессию
        # Создаем запрос для поиска пользователя по его telegram_id
        user = await session.scalar(select(User).where(User.telegram_id == telegram_id))

        if not user:  # Если пользователь не найден, добавляем его в базу данных
            session.add(User(telegram_id=telegram_id))  # Создаем новый объект пользователя
            await session.commit()  # Сохраняем изменения в базе данных

```

### Подробности:

- **`async_session()`** — создает новую сессию для работы с базой данных. Использование асинхронной сессии позволяет работать с запросами к БД в неблокирующем режиме, что важно для асинхронного бота.
- **`select(User).where(User.telegram_id == telegram_id)`** — создает SQL-запрос на выборку пользователя с заданным `telegram_id`.
- **`session.scalar()`** — выполняет запрос и возвращает одно значение, если оно существует (или `None`, если результат пустой).
- **`session.add()`** — добавляет новый объект в сессию для последующего сохранения в базе данных.
- **`session.commit()`** — сохраняет изменения в базе данных.

### Использование в хэндлере

Теперь можно вызвать эту функцию в хэндлере команды `/start`, чтобы при первом взаимодействии с ботом сохранять `telegram_id` пользователя в базе данных.

```python
import app.database.requests as rq  # Импортируем модуль с нашими запросами к базе данных
from aiogram.types import Message  # Импортируем тип данных для обработки сообщений
from aiogram.filters import CommandStart  # Импортируем фильтр для команды /start

@router.message(CommandStart())  # Хэндлер для обработки команды /start
async def cmd_start(message: Message):
    """
    Обработчик команды /start. Сохраняет пользователя в базе данных, если он новый.
    """
    await rq.set_user(message.from_user.id)  # Сохраняем пользователя в базе данных
    await message.answer('Приветик', reply_markup=kb.main)  # Отправляем приветственное сообщение

```

Этот подход гарантирует, что новый пользователь будет сохранен в базе данных, так как команда `/start` обычно является первой командой, которую вызывает пользователь для начала работы с ботом.

## Дополнительные примеры запросов

### 1. Получение всех пользователей

Напишем функцию, которая возвращает всех пользователей из базы данных с сортировкой по `telegram_id`.

```python
async def get_all_users():
    """
    Возвращает список всех пользователей, отсортированных по telegram_id.
    """
    async with async_session() as session:
        result = await session.scalars(select(User).order_by(User.telegram_id))
        return result.all()  # Возвращаем всех пользователей в виде списка

```

### 2. Удаление пользователя

Функция для удаления пользователя из базы данных по его `telegram_id`.

```python
async def delete_user(telegram_id: int):
    """
    Удаляет пользователя из базы данных.
    :param telegram_id: Идентификатор пользователя для удаления
    """
    async with async_session() as session:
        user = await session.scalar(select(User).where(User.telegram_id == telegram_id))

        if user:
            await session.delete(user)  # Удаляем пользователя из базы данных
            await session.commit()  # Подтверждаем удаление

```