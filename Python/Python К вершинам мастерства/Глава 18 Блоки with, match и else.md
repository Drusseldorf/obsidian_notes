# Глава 18. Блоки with, match и else

### Конспект главы "with, match, and else Blocks" на русском

### Введение

Эта глава описывает особенности управления потоком выполнения в Python, которые не так распространены в других языках:

1. **`with`**: оператор контекстного управления.
2. **`match/case`**: паттерн-матчинг для обработки выражений.
3. **`else`**: использование в `for`, `while` и `try`.

Эти конструкции делают код более выразительным и удобным.

---

### Контекстные менеджеры и оператор `with`

Контекстные менеджеры обеспечивают автоматическое выполнение операций до и после блока `with`. Это упрощает работу с ресурсами и уменьшает вероятность ошибок.

### Протокол контекстного менеджера

Контекстные менеджеры используют методы:

- `__enter__` – выполняется при входе в блок `with`.
- `__exit__` – выполняется при выходе из блока, даже если произошла ошибка.

Пример работы с файлом:

```python
with open('example.txt') as file:
    content = file.read()
```

**Тонкости:**

- Вызов `open()` возвращает объект `TextIOWrapper`.
- Метод `__enter__` возвращает ссылку на сам файл.
- При выходе из блока автоматически вызывается `__exit__`, закрывающий файл.
- Если исключение возникает в блоке `with`, метод `__exit__` получает информацию об исключении через параметры `exc_type`, `exc_value` и `traceback`. Метод может подавить исключение, вернув `True`, или позволить ему распространиться, вернув `None`.
- Если исключение происходит в `__enter__`, выполнение `with` не начнётся, и `__exit__` не будет вызван.
- Исключения в `__exit__` не подавляются автоматически и могут прервать выполнение программы.

**Мысли автора:**
Контекстные менеджеры упрощают управление ресурсами, делая код компактным и безопасным.

### Кастомный контекстный менеджер

Пример создания своего менеджера:

```python
class CustomManager:
    def __enter__(self):
        print("Entering context")
        return self

    def __exit__(self, exc_type, exc_value, traceback):  # Параметры метода __exit__: exc_type — тип исключения, exc_value — объект исключения с данными, traceback — стек вызовов в момент возникновения исключения.
        print("Exiting context")
        if exc_type:
            print(f"Handled exception: {exc_type}")
            return True  # подавить исключение

```

Использование:

```python
with CustomManager() as manager:
    print("Inside block")
```

**Тонкости:**

- Можно обрабатывать исключения внутри `__exit__`, чтобы избежать их распространения.

**Мысли автора:**
Создание собственных контекстных менеджеров – полезный способ внедрить автоматическое управление ресурсами.

### `@contextmanager`

Упрощение создания контекстных менеджеров через декоратор:

```python
from contextlib import contextmanager

@contextmanager
def reverse_output():
    import sys
    original_write = sys.stdout.write

    def reversed_write(text):
        original_write(text[::-1])

    sys.stdout.write = reversed_write
    try:
        yield
    finally:
        sys.stdout.write = original_write
```

Использование:

```python
with reverse_output():
    print("Hello")  # выведет "olleH"
```

**Тонкости:**

- Код до `yield` выполняется при входе в блок `with`.
- Код после `yield` – при выходе из блока.
- `yield` обёрнут в блок `try-finally`, чтобы гарантировать выполнение завершающего кода в `finally`, даже если в блоке `with` или внутри генератора возникает исключение. Это предотвращает некорректное состояние, например, если изменения в `sys.stdout.write` не будут восстановлены.

**Мысли автора:**
Декоратор `@contextmanager` позволяет сократить количество шаблонного кода при создании контекстных менеджеров.

### Полезные утилиты из `contextlib`

- **`redirect_stdout`*** и ****`redirect_stderr`** – перенаправление вывода.
- **`suppress`** – игнорирование исключений.
- **`ExitStack`** – управление множеством контекстов.

**Тонкости:**

- `ExitStack` полезен, если заранее неизвестно количество контекстов.
- `suppress` позволяет временно подавлять определённые исключения.

**Мысли автора:**
Библиотека `contextlib` предоставляет инструменты для работы с контекстами, делая код гибким и читаемым.

---

### `else` в циклах и `try`

### Использование в циклах

`else` выполняется, если цикл завершился без `break`:

```python
for item in items:
    if condition(item):
        break
else:
    print("Условие не выполнено для всех элементов")

# Пример с while
x = 0
while x < 5:
    if x == 3:
        break
    x += 1
else:
    print("Цикл завершён без break")
```

**Тонкости:**

- Логика `else` в циклах противоположна if/else: блок `else` выполняется после успешного завершения цикла.

**Мысли автора:**`else` в циклах часто упрощает код, устраняя необходимость дополнительных флагов.

### Использование в `try`

`else` выполняется, если в блоке `try` не возникло исключений:

```python
try:
    result = risky_operation()
except ValueError:
    handle_error()
else:
    finalize(result)
```

**Тонкости:**

- Блок `else` следует использовать для кода, который не должен быть внутри `try`.