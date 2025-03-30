**Основная идея**  
Тесты удобно писать с помощью готовых библиотек (например, `unittest` или `pytest`). Ключевые возможности `unittest`:

- Тесты объединяются в классы, унаследованные от `unittest.TestCase`.
    
- Каждый метод-тест начинается с префикса `test_`.
    
- Вместо встроенных `assert` используются методы вида `assertEqual()`, `assertNotEqual()`, `assertRaises()` и т. д.
    
- Запуск тестов — через `unittest.main()` или консольные команды (`python -m unittest`, `python tests.py`, с разными флагами).
    

---

**Пример структуры проекта**

```
любая_директория/
├── code.py           # Файл с тестируемым кодом
└── tests.py          # Файл с тестами
```

---

**Код программы: `code.py`**

```python
from datetime import datetime

def service_100():
    """Возвращает текущее время."""
    current_time = datetime.now()
    return current_time 
```

_(«Неразумный» рефакторинг, при котором тесты проваливаются, выглядит так:)_

```python
from datetime import datetime

def service_100(current_time=datetime.now()):
    """Возвращает текущее время."""
    return current_time
```

---

**Тесты: `tests.py`**

```python
import unittest
from datetime import datetime
from time import sleep

from code import service_100

class TestTimeService(unittest.TestCase):

    def test_time_is_running_out(self):
        first_time = service_100()
        sleep(1)
        second_time = service_100()
        self.assertNotEqual(first_time, second_time)

    def test_result_is_datetime(self):
        result = service_100()
        self.assertIsInstance(result, datetime)

unittest.main()  # Запуск тестов
```

**Запуск**

1. Перейти в директорию с файлом `tests.py` и выполнить `python tests.py`.
    
2. Либо запустить с флагом `-v` (`--verbose`): `python tests.py -v`.
    
3. Либо использовать `python -m unittest` в каталоге, где лежат тесты:
    
    - `python -m unittest` — запускает все тесты во всех файлах, начинающихся на `test`.
        
    - Можно выбирать конкретные файлы, классы или методы тестов, например:
        
        ```bash
        python -m unittest test_one
        python -m unittest test_one test_two
        python -m unittest test_one.TestClass
        python -m unittest test_one.TestClass.test_method
        ```
        
    - Ключ `-f` (`--failfast`) прерывает тестирование при первом же упавшем тесте.
        

---

**Почему «неудачный» рефакторинг проваливает тесты**  
При использовании аргументов по умолчанию с вызовом `datetime.now()` значение вычисляется один раз при импортировании модуля и не обновляется. Тест, который проверяет «изменение времени», падает, так как обе проверки возвращают одинаковое время.

---

**Распространённые методы `TestCase`**

- `assertEqual(x, y)` / `assertNotEqual(x, y)`
    
- `assertTrue(x)` / `assertFalse(x)`
    
- `assertIs(a, b)` / `assertIsNot(a, b)`
    
- `assertIsNone(x)` / `assertIsNotNone(x)`
    
- `assertIn(a, b)` / `assertNotIn(a, b)`
    
- `assertIsInstance(a, type)` / `assertNotIsInstance(a, type)`|
- `assertRaises(ZeroDivisionError, self.calc.divider, 1, 0)` - тип исключения, тестируемый метод, аргументы для метода
    
- И многие другие (`unittest` содержит более тридцати подобных методов).
    

---

**Пропуск и настройка тестов**

```python
import sys
import unittest

class TestExample(unittest.TestCase):
    """Демонстрирует возможности по пропуску тестов."""

    @unittest.skip('Этот тест мы просто пропускаем')
    def test_show_msg(self):
        self.assertTrue(False, 'Значение должно быть истинным')

    @unittest.skipIf(sys.version_info.major == 3 and sys.version_info.minor == 9,
                     'Пропускаем, если питон 3.9')
    def test_python3_9(self):        
        pass

    @unittest.skipUnless(sys.platform.startswith('linux'), 'Тест только для Linux')
    def test_linux_support(self):        
        pass

    @unittest.expectedFailure
    def test_fail(self):
        self.assertTrue(False, 'Ожидаем истинное значение')
```

- `@unittest.skip(reason)` — пропустить тест безусловно.
    
- `@unittest.skipIf(condition, reason)` — пропустить тест, если условие истинно.
    
- `@unittest.skipUnless(condition, reason)` — пропустить тест, если условие ложно.
    
- `@unittest.expectedFailure` — пометить тест как ожидаемо провальный; в отчёте будет отображаться `expected failure`.
    

---

**Ожидаемое падение vs проверка на исключение**

- «Ожидаемое падение» (`@unittest.expectedFailure`) — тест не прерывается исключением; сам тест «лукаво» возвращает «упал — но так и было задумано».
    
- Проверка, что код **обязан** выбросить исключение, делается через `assertRaises`. Пример:
    

```python
import unittest

def division_func(a, b):
    """Функция деления одного числа на другое."""
    return a / b

class TestExample(unittest.TestCase):

    @unittest.expectedFailure
    def test_fail(self):
        self.assertTrue(False, 'Ожидаем истинное значение')

    def test_zero_division(self):
        with self.assertRaises(ZeroDivisionError):
            division_func(1, 0)
```

- Можно добавлять `msg=` в `assertRaises`, чтобы вывести сообщение при неверном результате.
    

---

**Best practices от опытных разработчиков**

1. **Чёткая структура тестов**: каждый тест — отдельная логическая проверка, в названии теста (после `test_`) кратко описывайте ожидаемое поведение.
    
2. **Разделение кода и тестов**: храните тестовые файлы отдельно (например, в папке `tests`) и не смешивайте их в одном файле с основной логикой.
    
3. **Одно утверждение на тест** (по возможности): упавший тест сразу показывает конкретную причину сбоя, не прерывая выполнения остальных тестов.
    
4. **Используйте `setUp()` и `tearDown()`** (или `setUpClass()` и `tearDownClass()`) для инициализации/очистки окружения — убирайте повторяющийся код из тестов.
    
5. **Не тестировать внешние сервисы напрямую**: при необходимости используйте модули типа `unittest.mock`, чтобы эмулировать работу внешних зависимостей.
    
6. **Проверяйте исключения** с помощью `assertRaises` вместо «ожидаемого падения»: так вы явно фиксируете, что конкретная ошибка обязана возникать.
    
7. **Старайтесь поддерживать быстрый старт тестов**: чем быстрее запускается тест-сьют, тем чаще вы можете его прогонять и выявлять проблемы на ранних этапах.
    
8. **Покрытие тестами**: старайтесь тестировать ключевые ветки кода и обрабатывать нетривиальные случаи; настраивайте инструменты подсчёта покрытия (`coverage`) для оценки полноты тестирования.