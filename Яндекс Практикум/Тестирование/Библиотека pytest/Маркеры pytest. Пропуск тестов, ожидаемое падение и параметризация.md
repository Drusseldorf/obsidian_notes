**Маркеры в `pytest`**  
Маркеры — это специальные декораторы вида `@pytest.mark.<marker>`, позволяющие гибко управлять поведением тестов.

---

### Пропуск тестов (skip, skipif)

1. **`@pytest.mark.skip(reason='...')`**  
    – Пропускает тест без условий.  
    – Полезно, если временно не хочется выполнять тест (внешние сервисы недоступны и т.п.).
    
2. **`@pytest.mark.skipif(condition, reason='...')`**  
    – Пропускает тест по условию, например для версий Python < 3.7.
    

Пропущенные тесты отмечаются буквой **s** (skipped), в отчёте показывается причина пропуска.

---

### Ожидаемое падение (xfail)

1. **`@pytest.mark.xfail(reason='...')`**  
    – Указывает, что тест **ожидаемо** должен провалиться («проблема известна, пока не исправлена»).  
    – В отчёте такой тест помечается буквой **x** (XFAIL).
    
2. Если тест **вдруг прошёл** при наличии `xfail`, то это считается неожиданным успехом (XPASS), и в отчёте это будет отражено.
    

Можно использовать условие:

```python
@pytest.mark.xfail("sys.platform=='win32'", reason='На Windows падает')
```

---

### Параметризация тестов (`@pytest.mark.parametrize`)

Позволяет **запустить один тест** с разными входными данными.

```python
@pytest.mark.parametrize(
    'input_arg, expected_result',
    [
        (4, 5), 
        (3, 5)
    ],
    ids=['First parameter', 'Second parameter']
)
def test_one_more(input_arg, expected_result):
    assert one_more(input_arg) == expected_result
```

- Тест будет выполнен два раза: с `(4,5)` и с `(3,5)`.
    
- Аргументы декоратора:
    
    - Строка (или список) **имён параметров**: `'input_arg, expected_result'`.
        
    - Список **значений**: каждая пара (или кортеж) передаётся функции.
        
    - `ids` (необязательно) даёт осмысленные названия подмножества тестов.
        

#### Мультипараметризация

При использовании нескольких декораторов `@pytest.mark.parametrize`, тест выполняется для **произведений** всех значений:

```python
@pytest.mark.parametrize('x', [1, 2])
@pytest.mark.parametrize('y', ['one', 'two'])
def test_cartesian_product(x, y):
    ...
```

– Количество запусков: `len(x_values) * len(y_values)` → в данном примере **4**.

#### Маркеры в параметрах

Если один из набора параметров ожидается упавшим или пропускается, это можно указать через `pytest.param(..., marks=pytest.mark.xfail/skip)`:

```python
...
@pytest.mark.parametrize(
    'input_arg, expected_result',
    [
        (4, 5), 
        pytest.param(3, 5, marks=pytest.mark.xfail)  # Ожидается падение теста.
    ],
    ids=['First parameter', 'Second parameter',]
)
def test_one_more(input_arg, expected_result):
    ...
```
