# 11.39 LEFT и RIGHT OUTER JOIN

### LEFT JOIN

- **LEFT JOIN** возвращает все строки из левой таблицы и совпадающие строки из правой. Если в правой таблице нет соответствий, возвращаются строки из левой таблицы с `NULL` для отсутствующих значений из правой таблицы.
- Слово **OUTER** не является обязательным и обычно опускается.

Пример:

```sql
SELECT b.name AS book_name, a.name AS author_name
FROM book b
LEFT JOIN author a USING(author_id)
ORDER BY author_name, book_name;
```

### RIGHT JOIN

- **RIGHT JOIN** возвращает все строки из правой таблицы и соответствующие строки из левой таблицы. Если в левой таблице нет совпадений, возвращаются строки из правой с `NULL` для отсутствующих данных из левой.
- Слово **OUTER** также можно опустить.

Пример:

```sql
SELECT b.name AS book_name, a.name AS author_name
FROM book b
RIGHT JOIN author a USING(author_id)
ORDER BY author_name, book_name;
```

### Зачем оба JOIN?

Результаты **RIGHT JOIN** можно получить с помощью **LEFT JOIN**, поменяв таблицы местами, но оба оператора существуют для удобства и читаемости запросов. На практике **LEFT JOIN** используется чаще.