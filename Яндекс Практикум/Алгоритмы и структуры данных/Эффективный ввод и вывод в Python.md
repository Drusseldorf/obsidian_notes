# Эффективный ввод и вывод в Python

### Введение

Эффективность программ зависит от типа ограничений:

1. **CPU-bound**: Ограничения по скорости выполнения процессорных операций.
2. **Memory-bound**: Ограничения по объёму потребляемой памяти.
3. **I/O-bound**: Ограничения по скорости ввода/вывода данных.

При решении задач на платформе, такой как Яндекс Контест, часто ключевым становится скорость ввода-вывода, так как платформа может не принять решение, если оно выполнено медленно.

---

### Эффективный ввод данных

### Input vs sys.stdin.readline

- **`input()`** удобен, но относительно медленный. Он поддерживает подсказки для ввода.
- **`sys.stdin.readline()`** быстрее, так как не выводит подсказки. Однако, она считывает символ перевода строки `\n`, который нужно удалять с помощью `.rstrip()`.

### Пример: Использование `sys.stdin.readline`

```python
import sys

name = sys.stdin.readline().rstrip()
print(f'Привет, {name}!')
```

---

### Сравнение скорости `input()` и `sys.stdin.readline()`

Скрипт для тестирования:

```python
# check.py
import sys

# Используем input()
for _ in range(10**6):
    input()

# Используем sys.stdin.readline()
for _ in range(10**6):
    sys.stdin.readline().rstrip()
```

Запуск через терминал:

```bash
$ time yes | python check.py
```

Результаты:

- **`input()`**: Время выполнения ~0.427 секунд.
- **`sys.stdin.readline()`**: Время выполнения ~0.176 секунд (почти в 2.5 раза быстрее).

---

### Чтение массивов с помощью `sys.stdin.readline()`

```python
import sys

elements_count = int(sys.stdin.readline().rstrip())
data = [None] * elements_count

for index in range(elements_count):
    data[index] = sys.stdin.readline().rstrip()

print(data)
```

---

### Эффективный вывод данных

### Сокращение количества обращений к ОС

Каждый вызов `print()` — это отдельное обращение к ОС. Чтобы минимизировать это, строки объединяют через `\n` перед выводом:

```python
print('Раз, два, три, четыре, пять\nВышел\nPython\nПогулять\n')
```

Этот подход быстрее, но требует больше памяти.

---

### Практика: Объединение эффективного ввода и вывода

### Задача

На вход подаётся количество строк и пары чисел в каждой строке. Нужно вывести их сумму.

**Пример входных данных:**

```
3
5 6
7 8
9 10
```

**Пример выходных данных:**

```
11
15
19
```

### Решение с использованием `sys.stdin.readline()` и сбором строк

```python
import sys

def main():
    num_lines = int(sys.stdin.readline().rstrip())
    output_numbers = [None] * num_lines

    for i in range(num_lines):
        line = sys.stdin.readline().rstrip()
        first, second = map(int, line.split())
        output_numbers[i] = str(first + second)

    print('\n'.join(output_numbers))

if __name__ == '__main__':
    main()
```

---

### Альтернативное решение с распаковкой списка

Используем `print(*list, sep='\n')` для построчного вывода:

```python
import sys

def main():
    num_lines = int(sys.stdin.readline().rstrip())
    output_numbers = [None] * num_lines

    for i in range(num_lines):
        line = sys.stdin.readline().rstrip()
        first, second = map(int, line.split())
        output_numbers[i] = first + second

    print(*output_numbers, sep='\n')

if __name__ == '__main__':
    main()
```

---

### Вывод данных в файл

Метод `join()` универсален и подходит не только для печати, но и для записи в файл:

```python
with open('output.txt', 'w') as file:
    file.write('\n'.join(map(str, output_numbers)))
```

---

### Основные выводы

1. **`sys.stdin.readline()`** эффективнее `input()` для больших объёмов данных.
2. Используйте `.rstrip()` для удаления лишних символов перевода строки.
3. Объединяйте строки перед выводом, чтобы уменьшить количество вызовов `print()`.
4. Метод `join()` универсален для вывода, включая запись в файл.

Этот подход ускоряет ввод и вывод данных, что критично для задач с жёсткими временными ограничениями.