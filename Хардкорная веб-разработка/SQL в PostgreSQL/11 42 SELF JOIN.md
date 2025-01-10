# 11.42 SELF JOIN

`SELF JOIN` — это соединение таблицы с самой собой. Примером может быть таблица книг, где одна книга может ссылаться на другую как на "родительскую".

### Пример использования SELF JOIN:

Предположим, что книга «Тихий Дон» состоит из двух томов, и каждый том представлен как отдельная запись в таблице. Чтобы связать их, добавим колонку `parent_book_id`, которая будет ссылаться на основную книгу.

```sql
ALTER TABLE book ADD COLUMN parent_book_id BIGINT REFERENCES book(book_id);

INSERT INTO book (name, author_id, category_id, parent_book_id)
SELECT 'Тихий Дон. Том 1', author_id, category_id, book_id
FROM book WHERE name = 'Тихий Дон';

INSERT INTO book (name, author_id, category_id, parent_book_id)
SELECT 'Тихий Дон. Том 2', author_id, category_id, book_id
FROM book WHERE name = 'Тихий Дон';
```

Теперь запрос с `SELF JOIN`:

```sql
SELECT b.name AS book_name, p.name AS parent_book_name
FROM book b
JOIN book p ON b.parent_book_id = p.book_id;
```

Результат:

| book_name | parent_book_name |
| --- | --- |
| Тихий Дон. Том 1 | Тихий Дон |
| Тихий Дон. Том 2 | Тихий Дон |

### Пример с `LEFT JOIN`:

Можно также использовать `LEFT JOIN`, чтобы вернуть все книги, включая те, у которых нет родительской книги:

```sql
SELECT b.name AS book_name, p.name AS parent_book_name
FROM book b
LEFT JOIN book p ON b.parent_book_id = p.book_id;
```

Результат:

| book_name | parent_book_name |
| --- | --- |
| Тихий Дон. Том 1 | Тихий Дон |
| Тихий Дон. Том 2 | Тихий Дон |
| Простой Python |  |

### Пример с `RIGHT JOIN`:

```sql
SELECT b.name AS book_name, p.name AS parent_book_name
FROM book b
RIGHT JOIN book p ON b.parent_book_id = p.book_id;
```

SELF JOIN — это мощный инструмент, когда нужно связать строки внутри одной таблицы, например, для представления иерархий или связанных записей, как в примере с томами книги.