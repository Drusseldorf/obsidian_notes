# 11.41 CROSS JOIN

`CROSS JOIN` выполняет **декартово произведение** двух таблиц, то есть каждая строка из первой таблицы соединяется с каждой строкой из второй таблицы. Условие соединения здесь не указывается, потому что его нет.

### Пример с книгами и авторами:

```sql
SELECT b.name AS book_name, a.name AS author_name
FROM book b
CROSS JOIN author a
ORDER BY author_name, book_name;
```

Этот запрос возвращает 60 записей, так как у нас 10 книг и 6 авторов (10 * 6 = 60).

### Пример с футболками и цветами:

Декартово произведение удобно для получения всех возможных комбинаций данных, как в примере с футболками и цветами:

```sql
CREATE TEMP TABLE t_shirt (
    t_shirt_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    model VARCHAR(255)
);

CREATE TEMP TABLE color (
    color CHAR(6) PRIMARY KEY
);

CREATE TEMP TABLE t_shirt_order (
    order_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    t_shirt_id BIGINT NOT NULL REFERENCES t_shirt(t_shirt_id),
    color CHAR(6) NOT NULL REFERENCES color(color),
    customer VARCHAR(255) NOT NULL
);

INSERT INTO t_shirt(model) VALUES
    ('футболка с принтом голубя Валеры'),
    ('футболка без принта'),
    ('футболка очень длинная');

INSERT INTO color(color) VALUES
    ('ffffff'), ('000000'), ('ff0000'), ('00ff00'), ('0000ff'), ('eeeeee');

INSERT INTO t_shirt_order (t_shirt_id, color, customer) VALUES
    (1, 'ffffff', 'Хитрован'),
    (2, '000000', 'Продуман'),
    (2, '00ff00', 'Футболкоман'),
    (2, 'ff0000', 'какой-то странный малый');
```

**Получим все комбинации футболок с цветами:**

```sql
SELECT t.model, c.color
FROM t_shirt t
CROSS JOIN color c;
```

**Добавим количество продаж по каждой комбинации:**

```sql
SELECT t.model, c.color, COUNT(o.order_id) AS orders_count
FROM t_shirt t
CROSS JOIN color c
LEFT JOIN t_shirt_order o ON (o.color = c.color AND o.t_shirt_id = t.t_shirt_id)
GROUP BY t.model, c.color
ORDER BY orders_count DESC;
```

### Задача по количеству строк при `JOIN`:

Даны следующие таблицы:

```sql
CREATE TEMP TABLE a (id INT, some_columns TEXT);
CREATE TEMP TABLE b (id INT, some_columns TEXT);

INSERT INTO a(id, some_columns) VALUES
    (1, 'a_1'),
    (1, 'a_2'),
    (3, 'a_3');

INSERT INTO b(id, some_columns) VALUES
    (1, 'b_1'),
    (1, 'b_2'),
    (4, 'b_3'),
    (5, 'b_4');
```

**Сколько строк вернет запрос?**

```sql
SELECT * FROM a JOIN b USING(id);
```

Ответ: **4 строки**.

Объяснение: значение `id = 1` присутствует в обеих таблицах. В таблице `a` есть 2 строки с `id = 1`, и в таблице `b` тоже 2 строки с `id = 1`. В результате `JOIN` даст 4 строки — все возможные комбинации этих строк.

Так как JOIN выполняется по столбцу id, соединяться будут только те строки, у которых значения id совпадают в обеих таблицах.
Значение id = 1 присутствует в обеих таблицах. В таблице a две строки с id = 1, и в таблице b также две строки с id = 1.
При выполнении JOIN получится 4 строки (все возможные комбинации этих строк)
по идее это можно описать так, что сначала берется декартово произведение, а затем данные фильтруются по условию конкретного джоина (это просто логическое объяснение, это так в самой субд не делается)