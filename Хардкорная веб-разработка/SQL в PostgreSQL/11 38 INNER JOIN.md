# 11.38 INNER JOIN

`INNER JOIN` — это тип соединения таблиц, который возвращает только те строки, где есть совпадения по указанному условию между обеими таблицами.

### Пример использования:

```sql
SELECT b.name AS book_name, a.name AS author_name
FROM book b
INNER JOIN author a ON b.author_id = a.author_id
ORDER BY author_name, book_name;
```

Этот запрос вернёт все книги, у которых есть авторы, и всех авторов, у которых есть книги.

### Ключевые моменты:

- **INNER JOIN** возвращает только совпадающие записи. Если у автора нет книг или у книги нет автора, эти записи в результат не попадут.
- **Псевдонимы** (например, `b` для таблицы `book` и `a` для таблицы `author`) упрощают запись.
- Можно использовать сокращённую форму `USING` для одноимённых столбцов:
    
    ```sql
    SELECT b.name AS book_name, a.name AS author_name
    FROM book b
    JOIN author a USING(author_id)
    ORDER BY author_name, book_name;
    ```
    
- Слово `INNER` можно опустить:
    
    ```sql
    SELECT b.name AS book_name, a.name AS author_name
    FROM book b
    JOIN author a USING(author_id)
    ORDER BY author_name, book_name;
    ```
    
- Альтернативный синтаксис без `JOIN`:
    
    ```sql
    SELECT b.name AS book_name, a.name AS author_name
    FROM book b, author a
    WHERE b.author_id = a.author_id
    ORDER BY author_name, book_name;
    ```
    

Однако, лучше использовать `JOIN`, так как он делает запрос более читаемым и понятным.

WHERE - используется именно для фильтрации, и не должен в общем случае быть использован для соединения таблиц