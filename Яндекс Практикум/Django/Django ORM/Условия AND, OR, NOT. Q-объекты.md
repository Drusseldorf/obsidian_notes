## 1. Объединение условий через метод filter()

- **Простая фильтрация:**  
    Передавайте несколько условий через запятую — они объединяются через логический AND.
    
    ```python
    IceCream.objects.filter(is_published=True, is_on_main=True)
    ```
    
- **Альтернативный вариант:**  
    Последовательные вызовы filter() также объединяют условия через AND.
    
    ```python
    IceCream.objects.filter(is_published=True).filter(is_on_main=True)
    ```
    

## 2. Q-объекты: гибкие запросы с операторами

- **Импорт Q-объектов:**
    
    ```python
    from django.db.models import Q
    ```
    
- **Логический оператор AND:**  
    Объединение условий через `&`:
    
    ```python
    IceCream.objects.filter(Q(is_published=True) & Q(is_on_main=True))
    ```
    
- **Логический оператор OR:**  
    Объединение условий через `|`:
    
    ```python
    IceCream.objects.filter(Q(is_published=True) | Q(is_on_main=True))
    ```
    
- **Логический оператор NOT:**  
    Отрицание условия через `~`:
    
    ```python
    IceCream.objects.filter(Q(is_published=True) & ~Q(is_on_main=False))
    ```
    
- **Группировка условий:**  
    Для улучшения читаемости и корректного приоритета условий используйте скобки:
    
    ```python
    IceCream.objects.filter(
        (Q(is_on_main=True) & Q(is_published=True)) |
        (Q(title__contains='пломбир') & Q(is_published=True))
    )
    ```
    
    Этот запрос выбирает объекты, у которых **is_published=True** и **либо is_on_main=True, либо title содержит 'пломбир'**.

## 3. Приоритет операторов

- **Приоритет:**  
    `~` (NOT) > `&` (AND) > `|` (OR).  
    Явная группировка условий с помощью скобок помогает избежать ошибок приоритетности и делает код понятнее.