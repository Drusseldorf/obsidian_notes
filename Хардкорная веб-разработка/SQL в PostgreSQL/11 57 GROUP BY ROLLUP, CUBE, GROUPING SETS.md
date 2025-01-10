# 11.57 GROUP BY ROLLUP, CUBE, GROUPING SETS

### 1. **GROUP BY ROLLUP**

`GROUP BY ROLLUP` позволяет выполнять агрегирование не только по каждой группе, но и по всему набору данных. Это добавляет дополнительную строку в итоговую таблицу, которая будет содержать агрегацию по всему набору данных.

Пример:

```sql
select
    bc.name as category_name,
    count(*) as books_count
from book_category bc
join book_to_category btc using(category_id)
join book b using(book_id)
group by rollup (bc.name)
order by category_name nulls last;
```

Результат:

| category_name | books_count |
| --- | --- |
| Литература по программированию | 1 |
| Научная фантастика | 4 |
| Художественная литература | 8 |
|  | 13 |

Видим, что добавлена строка с общим количеством книг. Однако ошибка здесь в том, что повторяющиеся строки считаются как отдельные записи. Чтобы избежать этого, можно использовать `DISTINCT`:

```sql
select
    bc.name as category_name,
    count(distinct b.book_id) as books_count
from book_category bc
join book_to_category btc using(category_id)
join book b using(book_id)
group by rollup (bc.name)
order by category_name nulls last;
```

Теперь подсчёт будет корректным, и итоговая строка вернёт уникальное количество книг.

### 2. **GROUP BY CUBE**

`GROUP BY CUBE` возвращает все возможные комбинации группировок. Это позволяет выполнять агрегацию по всем возможным группам.

Пример:

```sql
select
    bc.name as category_name,
    a.author_id,
    count(distinct b.book_id) as all_category_books
from book_category bc
join book_to_category btc using(category_id)
join book b using(book_id)
join book_to_author bta using(book_id)
join author a using(author_id)
group by cube (bc.name, a.author_id)
order by category_name nulls last, author_id nulls last;
```

Результат:

| category_name | author_id | all_category_books |
| --- | --- | --- |
| Литература по программированию | 2 | 1 |
| Литература по программированию |  | 1 |
| Научная фантастика | 4 | 2 |
| Научная фантастика | 5 | 2 |
| Научная фантастика |  | 4 |
| Художественная литература | 1 | 2 |
| Художественная литература | 3 | 2 |
| Художественная литература | 4 | 2 |
| Художественная литература | 5 | 2 |
| Художественная литература |  | 8 |
|  | 1 | 2 |
|  | 2 | 1 |
|  | 3 | 2 |
|  | 4 | 2 |
|  | 5 | 2 |
|  |  | 9 |

Здесь видны все возможные группировки — как по категории, так и по автору.

### 3. **GROUP BY GROUPING SETS**

`GROUPING SETS` позволяет явно задавать, какие группировки нужны. Это даёт большую гибкость и контроль по сравнению с `ROLLUP` и `CUBE`.

Пример:

```sql
select
    bc.name as category_name,
    a.author_id,
    count(distinct b.book_id) as all_category_books
from book_category bc
join book_to_category btc using(category_id)
join book b using(book_id)
join book_to_author bta using(book_id)
join author a using(author_id)
group by grouping sets ((), bc.name, a.author_id, (bc.name, a.author_id))
order by category_name nulls last, author_id nulls last;
```

Результат:

| category_name | author_id | all_category_books |
| --- | --- | --- |
| Литература по программированию | 2 | 1 |
| Литература по программированию |  | 1 |
| Научная фантастика | 4 | 2 |
| Научная фантастика | 5 | 2 |
| Научная фантастика |  | 4 |
| Художественная литература | 1 | 2 |
| Художественная литература | 3 | 2 |
| Художественная литература | 4 | 2 |
| Художественная литература | 5 | 2 |
| Художественная литература |  | 8 |
|  | 1 | 2 |
|  | 2 | 1 |
|  | 3 | 2 |
|  | 4 | 4 |
|  | 5 | 4 |
|  |  | 13 |

Здесь мы чётко указываем, какие группировки нам нужны. Это гибкость, которой не обладают `ROLLUP` и `CUBE`.

### 4. **Использование GROUP BY для многомерного анализа**

Иногда требуется проанализировать данные на разных уровнях. Например, сколько книг написал каждый автор, сколько книг принадлежит каждой категории и общее количество книг. Эти группировки можно комбинировать с помощью вышеуказанных методов для эффективного анализа данных.

### Основные моменты:

- **ROLLUP** добавляет итоговые строки по всем группам, упрощая анализ.
- **CUBE** возвращает все возможные комбинации группировок.
- **GROUPING SETS** — наиболее гибкий инструмент, позволяющий явно указать, какие группы нужны.

Эти механизмы помогают делать сложные отчёты и сводные таблицы прямо в SQL, что упрощает интеграцию с различными аналитическими системами.