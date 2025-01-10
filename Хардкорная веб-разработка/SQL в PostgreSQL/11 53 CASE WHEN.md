# 11.53 CASE WHEN

Иногда бывает нужно категоризировать данные на основе каких-либо условий. В SQL это можно сделать с помощью оператора `CASE`. Рассмотрим пример, в котором мы категоризируем книги по году их выхода на основе следующих критериев:

- Книги, написанные до 1950 года — "классика"
- Книги, написанные между 1950 и 2010 годами — "современное"
- Книги, написанные позже 2010 года — "новиночка"
- Книги без указания года публикации — "непонятное"

### Пример кода:

```sql
delete from book_detail;

insert into book_detail (book_id, publication_year) values
    (1, 1928),
    (2, 2022),
    (3, 1956),
    (4, 1836),
    (5, 1835),
    (6, 1925),
    (7, 1926),
    (8, 1864)
on conflict (book_id) do update
set publication_year = excluded.publication_year;
```

Этот блок кода сначала очищает таблицу `book_detail` и вставляет данные о годах публикации книг. Если записи с таким `book_id` уже существуют, они обновляются.

### Запрос для категоризации книг:

```sql
select
    b.book_id,
    b.name,
    bd.publication_year,
    case
        when bd.publication_year < 1950 then 'классика'
        when bd.publication_year between 1950 and 2010 then 'современное'
        when bd.publication_year > 2010 then 'новиночка'
    else 'непонятное'
    end era
from book b
left join book_detail bd using(book_id)
order by publication_year desc nulls last;
```

Здесь мы используем оператор `CASE`, который проверяет год публикации книги:

- Если год меньше 1950, то книга помечается как "классика".
- Если год между 1950 и 2010, то книга считается "современной".
- Если год больше 2010, то это "новиночка".
- Если год публикации не указан (NULL), то выводится "непонятное".

Результат запроса выглядит так:

| book_id | name | publication_year | era |
| --- | --- | --- | --- |
| 2 | Python. К вершинам мастерства | 2022 | новиночка |
| 3 | Судьба человека | 1956 | современное |
| 1 | Тихий Дон | 1928 | классика |
| 7 | Остров погибших кораблей | 1926 | классика |
| 6 | Голова профессора Доуэля | 1925 | классика |
| 8 | Путешествие к центру Земли | 1864 | классика |
| 4 | Капитанская дочка | 1836 | классика |
| 5 | Сказка о рыбаке и рыбке | 1835 | классика |
| 9 | Дети капитана Гранта | NULL | непонятное |

### Итоги:

Использование оператора `CASE` в SQL позволяет удобно категоризировать данные по различным критериям. Это упрощает процесс обработки данных и позволяет сразу получать готовые результаты на стороне базы данных, минуя необходимость дополнительных операций в коде на Python или другом языке.

Еще можно вот так

```sql
CASE 
    WHEN count(*) FILTER (WHERE s.average_weight < 50) = 0 THEN '—'
    ELSE count(*) FILTER (WHERE s.average_weight < 50)::text 
END AS light_species,
```

Интересный вариант решения задачки, когда мы при условии что если подсчитали 0 то вернем тире а не 0. Нужно приводить к тексту, так как колонки должны быть однородные

```sql
select
    t.family_name,
    case all_species when 0 then '—' else all_species::text end all_species,
    case light_species when 0 then '—' else light_species::text end light_species,
    case medium_species when 0 then '—' else medium_species::text end medium_species,
    case heavy_species when 0 then '—' else heavy_species::text end heavy_species
from (
    select
        family_name,
        count(species_id) as all_species,
        count(species_id) filter (where average_weight < 50) as light_species,
        count(species_id) filter (where average_weight between 50 and 299) as medium_species,
        count(species_id) filter (where average_weight >= 300) as heavy_species
    from family
    left join genus using(family_id)
    left join species s using(genus_id)
    group by family_name
    order by 1
) t;
```