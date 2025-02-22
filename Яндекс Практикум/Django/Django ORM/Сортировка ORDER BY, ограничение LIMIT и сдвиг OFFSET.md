## 1. Сортировка (ORDER BY)

- **В модели через Meta:**  
    Определите порядок сортировки для всех запросов к модели:
    
    ```python
    class Category(models.Model):
        title = models.CharField(max_length=256)
        # ...
        class Meta:
            ordering = ('output_order', 'title')  # ASC: по возрастанию
    ```
    
    - Для обратного порядка поставьте перед полем минус: `ordering = ('-output_order',)`
- **В конкретном запросе:**  
    Используйте метод `.order_by()`:
    
    ```python
    categories = Category.objects.values('id', 'output_order', 'title').order_by('output_order', 'title')
    ```
    
    - Если вы указываете сортировку в `.order_by()`, она переопределяет настройку из Meta.
    - Для отключения сортировки вызовите `order_by()` без параметров.

## 2. Ограничение выборки (LIMIT) и сдвиг (OFFSET)

- **Использование срезов QuerySet:**  
    Python-срезы применяются к QuerySet так же, как к спискам:
    
    ```python
    ice_cream_list = IceCream.objects.values('id', 'title', 'description') \
                        .filter(is_published=True, is_on_main=True) \
                        .order_by('title')[1:4]
    ```
    
    - Здесь `[1:4]` означает:
        - **OFFSET**: пропустить 1 запись (начальный индекс)
        - **LIMIT**: вернуть 4 - 1 = 3 записи
- **SQL-эквивалент:**  
    Запрос будет включать конструкции `LIMIT 3` и `OFFSET 1`.
    

## Итог

- Сортировку можно задать глобально (в классе Meta) или локально (через `.order_by()`).
- Ограничение числа записей и смещение реализуются через срезы QuerySet, что эквивалентно операторам LIMIT и OFFSET в SQL.
- Комбинируйте методы сортировки, фильтрации и срезы для оптимальной выборки нужных данных.