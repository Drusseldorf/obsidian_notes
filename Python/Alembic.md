# Alembic

Установить зависимость

```bash
poetry add alembic
```

Генерация новой конфигурации алембик с async шаблоном

```bash
alembic init -t async alembic
```

Создается **папка alembic** и на верхнем уровне **файл alembic.ini**

**alembic.ini** - конфигурация для миграции. Внутри этого файла можно раскомментировать:

```bash
# Uncomment the line below if you want the files to be prepended with date and time
# file_template = %%(year)d_%%(month).2d_%%(day).2d_%%(hour).2d%%(minute).2d-%%(rev)s_%%(slug)s
```

Чтобы названия ревизий строилось по шаблону где дата создания идет первой. Тогда в каталоге они будут лежать упорядоченно. Иначе первым в названии идет ИД ревизии, который рандомный и файлы будут неупорядоченные. 

В папке alembic имеется файл **script.py.mako** - это шаблон, на основе которого либа mako (тянется за алембиком) будет создавать новые миграции.

## Про файл env.py

- `env.py` - Это Python-скрипт, который запускается каждый раз при вызове инструмента миграции Alembic. Как минимум, содержит инструкции для настройки и создания движка SQLAlchemy, получения соединения от этого движка вместе с транзакцией, а затем вызова движка миграции, используя это соединение как источник подключения к базе данных. Скрипт `env.py` является частью сгенерированного окружения, что делает процесс выполнения миграций полностью настраиваемым. Здесь определены точные детали подключения, а также особенности вызова среды миграции. Скрипт можно модифицировать таким образом, чтобы работать с несколькими движками, передавать пользовательские аргументы в среду миграции, загружать и делать доступными специфичные для приложения библиотеки и модели. Alembic включает набор шаблонов инициализации, которые предлагают различные варианты `env.py` для разных сценариев использования.

Файл `env.py` в папке Alembic является ключевым компонентом для управления миграциями баз данных с использованием Alembic в проектах на Python с SQLAlchemy. Он служит для настройки окружения и контекста, в котором будут выполняться миграции.

**Основные функции `env.py`:**

1. **Настройка подключения к базе данных:**
В `env.py` определяется, как Alembic будет подключаться к вашей базе данных. Это может быть строка подключения, взятая из конфигурационного файла, переменной окружения или другого источника.
2. **Определение модели данных:**
Если вы используете автоматическое обнаружение изменений в модели данных (автогенерацию миграций), `env.py` настраивает доступ к метаданным SQLAlchemy (`MetaData`), которые описывают структуру вашей базы данных.
3. **Режимы работы:**
    - **Offline режим:** Позволяет генерировать SQL-скрипты миграций без непосредственного подключения к базе данных.
    - **Online режим:** Выполняет миграции напрямую в базе данных, используя активное подключение.
4. **Настройка контекста миграций:**`env.py` определяет, какие миграции должны быть применены, и управляет контекстом их выполнения, включая обработку транзакций и логирование.

## Создание новой миграции

```bash
alembic revision -m "create accounts table"
```

После выполнения, в папке alembic → versions появится файл, названный по шаблону как в alembic.ini, например: 2024_10_13_1353-fb0ed969b939_create_accounts_table.py

Внутри файла имеется:

revision: str = 'fb0ed969b939’ - ИД этой новой ревизии (миграции)

down_revision: Union[str, None] = None - ИД ревизии на которую нужно откатываться в случае необходимости

В функциях 

```python
def upgrade() -> None:
pass

def downgrade() -> None:
pass
```

Можно написать миграции вручную.

## Автогенерация миграций

Но, так как у нас уже написаны модели для SQLAlchemy, то нам этот файл по сути и не нужен, и его даже можно удалить (речь про файл из пункта выше). Нас больше интересует автогенерация миграций, и это позволяет сделать алембик.

Важно как импортируется базовый класс, иначе ничего не сгенерируется.

Я добавляю в инит файл пакета импорты классов и базового класса затем именно через этот пакет импортирую базовый класс и тогда это работает для автогенерации.

В файле env.py находим строку target_metadata и импортируем сюда наши модели алхимии.

```python
# add your model's MetaData object here
# for 'autogenerate' support
from core.models import Base
target_metadata = Base.metadata
```

Сюда передается именно базовый класс, от которого наследуются наши модели алхимии, вот он для примера:

```python
class Base(DeclarativeBase):
    __abstract__ = (
        True  # causes declarative to skip the production of a table or mapper
    )
    # for the class entirely

    @classmethod  # This decorator never necessary from a runtime perspective,
    # however may be needed in order to support PEP 484 typing tools
    # that don’t otherwise recognize the decorated function as having class-level
    # behaviors for the cls parameter
    @declared_attr.directive  # automatically adding tablename by passing cls name
    def __tablename__(cls) -> str:
        return cls.__name__.lower()

    id: Mapped[int] = mapped_column(
        primary_key=True
    )  # so we dont need to write id in orm models
```

Туда мы импортируем именно этот класс.

Далее в файле env.py, после определения target_metadata, переопределяем sqlalchemy.url (это настройка из alembic.ini, и тут мы ее переопределяем).

Как переопределяем? Через config.set_main_option (это некий алембик объект, который уже импортирован в env.py, вот его описание из док стрингов: 

```python
Set an option programmatically within the 'main' section.
This overrides whatever was in the .ini file
```

).

Для начала импортируем сюда в енв.пу наши настройки, где мы храним url подключения к базе данных, затем:

```python
from core.config import settings

config.set_main_option("sqlalchemy.url", settings.db_url)
```

Теперь когда все это мы настроили мы можем уже выполнить команду АВТОМИГРАЦИИ:

```python
 alembic revision --autogenerate -m "create products table"
```

Так же создается файл миграции в папке versions, откроем его и увидим…:

```python
def upgrade() -> None:
    # ### commands auto generated by Alembic - please adjust! ###
    pass
    # ### end Alembic commands ###

def downgrade() -> None:
    # ### commands auto generated by Alembic - please adjust! ###
    pass
    # ### end Alembic commands ###
```

А почему он все еще пустой, но появились какие то комментарии. Почему же так вышло?

Потому что мы на самом деле ничего и не меняли в модельках базы, таблицах. Ничего не добавляли а сама таблица уже существует, вот и получается что алембик сравнил то что имеется с тем, что ему предлагается мигрировать и понял что уже актуально все в БД.

Удалим этот файл миграции (в целом пока мы не применяли файл скрипта миграции мы можем спокойно его удалять) и в целях обучения дропнем нашу таблицу.

Теперь после того как мы выполним

```python
alembic revision --autogenerate -m "create products table"
```

У нас в файле миграции уже появится скрипт

```python
def upgrade() -> None:
    # ### commands auto generated by Alembic - please adjust! ###
    op.create_table('product',
    sa.Column('name', sa.String(), nullable=False),
    sa.Column('description', sa.String(), nullable=False),
    sa.Column('price', sa.Integer(), nullable=False),
    sa.Column('id', sa.Integer(), nullable=False),
    sa.PrimaryKeyConstraint('id')
    )
    # ### end Alembic commands ###

def downgrade() -> None:
    # ### commands auto generated by Alembic - please adjust! ###
    op.drop_table('product')
    # ### end Alembic commands ###
```

Не сложно догадаться по названию методов и аргументам, что он узнал что не хватает нашей таблицы product и из моделек взял все данные для создания таблицы.

В качестве отката он делает скрипт дропа таблицы, что логично, ибо ее и не было до этой миграции. 

## Black

Сам скрипт миграции генерируется не по PEP8, что можно исправить в ini файле алембик.

Там можно подключить блэк форматтер, просто раскоментируя строчки настроек:

```python
# format using "black" - use the console_scripts runner, against the "black" entrypoint
hooks = black
black.type = console_scripts
black.entrypoint = black
black.options = -l 89 REVISION_SCRIPT_FILENAME
```

## Выполняем миграцию

При выполнении миграции алембик создает (если ее еще нет) таблицу в нашей бд и добавляет туда информацию о миграции. (вообще таблица создается еще при команду alembic revision): alembic_version там он хранит version_num - ИД ревизий. 

Важно, что после выполнения миграции нам уже нельзя удалять скрипты миграции из папки versions ибо это вызовет конфликт историй.

Алембик ведь по сути такой аналог гита но для версионирования состояний БД.

Поэтому обязательно все миграции схем, все что генерирует алембик ДОЛЖНО добавляться в гит репозиторий.

Выполним команду

```python
alembic upgrade head
```

И вуаля, в нашей бд создалась таблица product со всеми столбцами, как было указано в модельке алхимии

## Выполняем откат миграции

Выполняем команду

```python
alembic downgrade -1
```

ключ -1 означает что мы хотим откатиться на 1 ревизию назад, то есть на последнюю предыдущую, которая была до новой.

После чего проверим нашу БД и увидим, что таки таблица product действительно пропала

## Слежение за версиями миграций, переход на разные миграции

Для вывода на экран всех миграций: 

```python
alembic history
```

Вывод вида:

```bash
98794b5b511f -> 086c6ecaaa6d (head), create table profile
23d66f46984b -> 98794b5b511f, Create Post table and foreign key user
7aaf7745a42b -> 23d66f46984b, Create User table
<base> -> 7aaf7745a42b, create products table
```

head - показывает последнюю ревизию.

base - показывает текущую примененную.

Для вывода текущей актуальной миграции:

```python
alembic current
```

Откатиться на N транзакций назад

```python
alembic downgrade -N
```