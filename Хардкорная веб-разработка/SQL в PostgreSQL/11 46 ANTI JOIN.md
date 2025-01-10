# 11.46 ANTI JOIN

**ANTI JOIN** — это тип соединения, который возвращает строки из одной таблицы, для которых **не существует** ни одной соответствующей строки в другой таблице. Он противоположен **SEMI JOIN**, который возвращает строки, для которых соответствие существует.

### Пример:

Найдём книги, для которых нет авторов:

```sql
SELECT b.name
FROM book b
WHERE NOT EXISTS (
    SELECT 1 FROM author a WHERE a.author_id = b.author_id
);
```

### NOT EXISTS vs NOT IN:

ANTI JOIN часто реализуется с использованием конструкции `NOT EXISTS`. Однако важно помнить, что этот синтаксис не всегда идентичен `NOT IN`.

Пример использования `NOT IN`:

```sql
SELECT b.name
FROM book b
WHERE b.author_id NOT IN (
    SELECT UNNEST(array[1, 2, 3, 4, 5, 6]) AS author_id
);
```

Этот запрос вернёт все книги, для которых `author_id` не находится в массиве `[1, 2, 3, 4, 5, 6]`.

Однако, если в подзапросе будет `NULL`, результат `NOT IN` может вернуть неожиданные результаты:

```sql
SELECT b.name
FROM book b
WHERE b.author_id NOT IN (
    SELECT UNNEST(array[1, 2, 3, 4, 5, 6, NULL]) AS author_id
);
```

Этот запрос внезапно вернёт пустой результат, потому что наличие `NULL` в массиве вызывает неопределённое поведение при сравнении. Любое сравнение с `NULL` возвращает `NULL`, а не `TRUE` или `FALSE`.

### Решение с `NOT EXISTS`:

```sql
SELECT b.name
FROM book b
WHERE NOT EXISTS (
    SELECT UNNEST(array[1, 2, 3, 4, 5, 6, NULL]) AS author_id
    WHERE author_id = b.author_id
);
```

Этот запрос корректно вернёт книги без авторов, игнорируя возможные `NULL` в подзапросе.

### Вывод:

Использование `NOT EXISTS` безопаснее и предсказуемое, особенно если в подзапросе могут встречаться `NULL`.

---

### Пример задачи:

Достанем даты наблюдений в формате `ДД.ММ.ГГГГ` (колонка в результирующей таблице должна называться `observation_date_formatted`) и имена всех наблюдателей (колонка должна называться `observer_name`) по тем из них, которые не наблюдали птиц в 2022 году, но наблюдали в другие года. Сортировка по имени наблюдателя и затем по дате наблюдения.

### Решение:

```sql
SELECT observer_name, TO_CHAR(observation_date, 'DD.MM.YYYY') AS observation_date_formatted
FROM observations o
WHERE NOT EXISTS (
    SELECT 1 FROM observations o2
    WHERE EXTRACT('year' FROM o2.observation_date) = 2022
    AND o2.observer_name = o.observer_name
)
ORDER BY observer_name, observation_date;
```

### Разбор решения:

1. **Основной запрос:**
    - Выбираем из таблицы `observations`:
        - Имя наблюдателя `observer_name`.
        - Дату наблюдения, преобразованную в формат `ДД.ММ.ГГГГ` с помощью функции `TO_CHAR()`.
2. **Условие `WHERE NOT EXISTS`:**
    - Используем подзапрос с `NOT EXISTS`, чтобы выбрать только тех наблюдателей, которые **не наблюдали** птиц в 2022 году:
        - Подзапрос выбирает всех наблюдателей, которые имели наблюдения в 2022 году.
        - Если наблюдатель присутствует в подзапросе, то основной запрос исключает его из результатов.
3. **Функция `EXTRACT`:**
    - Извлекает год из даты наблюдения с помощью `EXTRACT('year' FROM o2.observation_date)`.
4. **Сортировка:**
    - Сортируем результаты по имени наблюдателя, а затем по дате наблюдения.