# JOIN-ы. Соединения таблиц

Основных видов соединений 5:

- INNER JOIN
- LEFT OUTER JOIN
- RIGHT OUTER JOIN
- FULL OUTER JOIN
- CROSS JOIN

А также дополнительно можно выделить несколько возможностей SQL, которые не являются отдельными видами соединений, но относятся к этой теме:

- SELF JOIN
- NATURAL JOIN
- JOIN LATERAL
- SEMI JOIN и ANTI JOIN

## 1. INNER JOIN

Слово INNER можно опустить 

Тут и далее стоит заметить, что условия соединения могут быть не обязательно всегда равенством.

Выполняет соединения записей, если имеются соответствия в обоих таблицах сразу

Порядок таблиц в запросе не важен

```sql
SELECT p.id,
	t.time_in
FROM Passenger p
	JOIN Pass_in_trip pt ON pt.passenger = p.id
	JOIN Trip t ON t.id = pt.trip
WHERE p.name = 'Steve Martin'
	AND t.town_to = 'London';
```

Интересный момент: за счет того, что мы можем указать в блоке FROM несколько таблиц через запятую, мы можем выполнить соединение на основе WHERE без использования JOIN. Однако такой способ не должен применяться на практике. Это плохая практика. SQL это декларативный язык и мы не должны диктовать СУБД свои условия для выполнения действия. (не должны писать императивно). Плохой пример:

```sql
SELECT b.name AS book_name, a.name AS author_name
FROM book b, author a
WHERE b.author_id = a.author_id
ORDER BY author_name, book_name;
```

## 2. LEFT OUTER JOIN, RIGHT OUTER JOIN

Слово OUTER можно опустить

Выполняет соединение, берет все записи из левой или правой таблиц (в зависимости от использования LEFT, RIGHT и в зависимости от расположения таблиц в самом SQL запросе) и если для другой таблицы нет соответствия, то дополняет ее значением **NULL**

Порядок таблиц важен, результаты могут отличаться!

```sql
SELECT b.name AS book_name,
	a.name AS author_name
FROM book b
	LEFT JOIN author a ON a.id = b.author_id
ORDER BY author_name,
	book_name;
```

## 3. FULL OUTER JOIN

Слово OUTER можно опустить

Возвращает все строки из обеих таблиц, заполняя **`NULL`** значения для тех строк, которые не имеют соответствующих записей в другой таблице. По сути это объединение результатов **`LEFT JOIN`** и **`RIGHT JOIN`**

```sql
select b.name as book_name, a.name as author_name
from book b
full join author a ON a.id = b.author_id
order by author_name, book_name;
```

## 4. CROSS JOIN

Выполняет декартово произведение двух таблиц, то есть каждая строка из первой таблицы соединяется с каждой строкой из второй таблицы.

Может быть полезно, где необходимо получить все возможные пары между двумя таблицами. 

Хороший пример: 

у нас есть футболки разных размеров и варианты принтов на этих футболках, при этом на каждом размере могут быть все варианты принтов. Задача - провести аналитику продаж по каждому размеру И принту. Решение: получили кросс джоин и к нему выполнили лефт джоин по заказам с условием совпадения цвета и размера. 

Как видно в примере, мы не пишем конструкцию ON o.id = b.id для условия соединения

Пример:

```sql
select t.model, c.color, count(o.order_id) orders_count
from t_shirt t
cross join color c
left join t_shirt_order o on (o.color=c.color and o.t_shirt_id=t.t_shirt_id)
group by t.model, c.color
order by orders_count desc;
```

## 2. Не каноничные JOIN-ы

Отдельно можно выделить соединения, которые не входят в число классических существующих в SQL:

### 2.1 SELF JOIN

Объединение таблицы с самой собой.

Предположим, что книги могут ссылаться друг на друга. Например, в книге «Тихий Дон» два тома. Допустим, что каждый том должен быть представлен в таблице отдельно, со своей обложкой, описанием и т.д. Тогда:

```sql
select b.name as book_name, p.name as parent_book_name
from book b
join book p on b.parent_book_id = p.book_id;
```

Выведет:

```sql
|book_name       |parent_book_name|
|----------------|----------------|
|Тихий Дон. Том 1|Тихий Дон       |
|Тихий Дон. Том 2|Тихий Дон       |
```

можно и left join использовать тут, чтобы получить левую таблицу полную, а правая заполниться NULL.

```sql
select b.name as book_name, p.name as parent_book_name
from book b
left join book p on b.parent_book_id = p.book_id;
```

### 2.2 NATURAL JOIN

Соединяет таблицы по всем одноимённым колонкам

просто с **`JOIN`**:

```sql
select o.order_id, t.model, c.color
from t_shirt_order o
join t_shirt t using(t_shirt_id)
join color c using(color);
```

С **`JOIN NATURAL`** можно сделать так — короче:

```sql
select o.order_id, t.model, c.color
from t_shirt_order o
natural join t_shirt t
natural join color c;
```

Вместо указания по каким колонкам соединять таблицы, мы пишем NATURAL JOIN и не указываем колонки, и соединение происходит по одноимённым колонкам, то есть колонкам, которые имеют одинаковые названия в соединяемых таблицах.

Не применяется на практике, так как такой подход является не явным, легко получить ошибку и неверные данные.

### 2.3 **JOIN LATERAL**

Слово **`LATERAL`** может применяться вместе с любой операции **`JOIN`**, например, с **`INNER JOIN`**, **`LEFT JOIN`**, **`RIGHT JOIN`** и т.д., и позволяет правому операнду получить доступ к столбцам левого операнда.

Пример решаемой задачи:

выбирать авторов, для каждого автора выполнить подзапрос в таблицу книг, в этом подзапросе отфильтровать книги конкретного автора из внешнего запроса, отсортировать их по дате и выбрать одну книгу, добавленную позже всего:

```sql
select
    a.name as author,
    b.name as last_added_book
from
    author a
join lateral (
    select name from book
    where book.author_id = a.author_id
    order by created_at desc nulls last
    limit 1
) b on true order by author;
```