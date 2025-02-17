# Краткий конспект: Django shell и CRUD-операции

## 1. Django shell

- Специальная консоль, в которую «подгружен» весь Django-проект.
- Запуск: в папке с `manage.py`:
    
    ```bash
    python manage.py shell
    ```
    
- Позволяет импортировать модели и выполнять любые действия с БД напрямую через ORM.

## 2. Операции CRUD

**CRUD** расшифровывается как Create, Read (Retrieve), Update, Delete — четыре базовые операции с данными.

### 2.1 Create (создание)

- Создание записи в таблице через ORM:
    
    ```python
    from ice_cream.models import Category
    
    Category.objects.create(
        title='Категория через shell',
        slug='shell_category'
    )
    ```
    
- В БД будет выполнен SQL-запрос `INSERT ...`.

### 2.2 Read (получение)

1. **Получить все объекты**:
    
    ```python
    Category.objects.all()  # Возвращает QuerySet
    ```
    
2. **Фильтрация**:
    
    ```python
    Category.objects.filter(is_published=True)  # QuerySet
    ```
    
3. **Получить одиночную запись**:
    
    ```python
    Category.objects.get(pk=1)  # Возвращает объект модели
    ```
    

- Если объект не найден, выбрасывается ошибка `DoesNotExist`.

### 2.3 Update (обновление)

4. **Массовое обновление (QuerySet)**:
    
    ```python
    Category.objects.all().update(
        title='Новое название',
        is_published=True
    )
    ```
    
5. **Обновление одиночного объекта**:
    
    ```python
    obj = Category.objects.get(pk=1)
    obj.title = 'Новый заголовок'
    obj.save()  # Данные сохраняются в БД
    ```
    

### 2.4 Delete (удаление)

6. **Удаление одиночного объекта**:
    
    ```python
    obj = Category.objects.get(pk=1)
    obj.delete()
    ```
    
7. **Удаление набора объектов**:
    
    ```python
    Category.objects.all().delete()
    ```
    

## 3. Резюме

- Django shell упрощает отладку и эксперименты с моделями.
- CRUD-операции выполняются через `objects`.
- Методы: `create()`, `all()`, `filter()`, `get()`, `update()`, `save()`, `delete()`.
- Используйте Django shell для тестирования операций с БД, прежде чем внедрять их во view-функции или логику приложения.