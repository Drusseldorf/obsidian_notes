# 11.52 json_agg vs ORM

1. **Создание моделей в Django**
    
    В Django ORM мы создаем модели, описывающие таблицы базы данных. Пример структуры модели для авторов, категорий и книг:
    
    ```python
    from django.db import models
    
    class Author(models.Model):
        author_id = models.BigIntegerField(primary_key=True)
        name = models.CharField(max_length=255)
    
        class Meta:
            db_table = "author___"
    
    class Category(models.Model):
        category_id = models.BigIntegerField(primary_key=True)
        name = models.CharField(max_length=255)
    
        class Meta:
            db_table = "category___"
    
    class Book(models.Model):
        book_id = models.BigIntegerField(primary_key=True)
        name = models.CharField(max_length=255)
        authors = models.ManyToManyField(to=Author, related_name='books')
        categories = models.ManyToManyField(to=Category, related_name='books')
    
        class Meta:
            db_table = "book___"
    ```
    
2. **Инициализация данных**
    
    Создание записей для этих моделей можно реализовать с помощью команды Django management:
    
    ```python
    from books.models import Author, Book, Category
    from django.core.management.base import BaseCommand
    
    class Command(BaseCommand):
        def handle(self, *args, **options):
            Category.objects.all().delete()
            Author.objects.all().delete()
            Book.objects.all().delete()
    
            authors = (
                Author.objects.create(author_id=1, name='Александр Пушин'),
                Author.objects.create(author_id=2, name='Александр Беляев')
            )
    
            categories = (
                Category.objects.create(category_id=1, name='Художественная литература'),
                Category.objects.create(category_id=2, name='Научная фантастика')
            )
    
            book = Book.objects.create(book_id=1, name="Капитанская дочка")
            book.categories.add(categories[0])
            book.authors.add(authors[0])
    
            book = Book.objects.create(book_id=2, name="Ариэль")
            book.categories.add(categories[0], categories[1])
            book.authors.add(authors[1])
    ```
    
3. **Вывод книг с категориями и авторами**
    
    Простейший способ получить и вывести данные в Django:
    
    ```python
    from books.models import Book
    
    def print_books_naive_method():
        books = Book.objects.all()
        for book in books:
            book_name = book.name
            category_names = ", ".join([c.name.lower() for c in book.categories.all()])
            author_names = ", ".join([a.name for a in book.authors.all()])
            print(f"«{book_name}», авторы: {author_names}, категории: {category_names}")
    ```
    
    Этот подход генерирует много SQL-запросов (например, 9 запросов для 9 книг).
    
4. **Оптимизация с `prefetch_related`**
    
    Использование метода `prefetch_related` уменьшает количество SQL-запросов:
    
    ```python
    def print_books_with_prefetch():
        books = Book.objects.prefetch_related("authors", "categories").all()
        for book in books:
            book_name = book.name
            category_names = ", ".join([c.name.lower() for c in book.categories.all()])
            author_names = ", ".join([a.name for a in book.authors.all()])
            print(f"«{book_name}», авторы: {author_names}, категории: {category_names}")
    ```
    
    Здесь количество запросов сокращается до 3, но это всё еще не оптимально.
    
5. **Оптимизация с `json_agg`**
    
    Если нужно вывести данные одним запросом, можно использовать SQL-функцию `json_agg` в Django с помощью `connection.cursor()`:
    
    ```python
    from django.db import connection
    
    def print_books_with_json():
        with connection.cursor() as cursor:
            cursor.execute("""
            SELECT
                b.name AS book_name,
                json_agg(distinct a.*) AS authors,
                json_agg(distinct c.*) AS categories
            FROM book___ b
            LEFT JOIN book___authors USING(book_id)
            LEFT JOIN author___ a USING(author_id)
            LEFT JOIN book___categories USING(book_id)
            LEFT JOIN category___ c USING(category_id)
            GROUP by b.name
            ORDER BY b.name
            """)
            books = cursor.fetchall()
            for book in books:
                book_name = book[0]
                category_names = ", ".join([c["name"].lower() for c in book[2]])
                author_names = ", ".join([a["name"] for a in book[1]])
                print(f"«{book_name}», авторы: {author_names}, категории: {category_names}")
    ```
    
    Этот способ позволяет получить все данные одним запросом, независимо от количества связей.
    

### Итог:

- **Naive Method**: Много SQL-запросов, низкая производительность.
- **`prefetch_related`**: Оптимизирует запросы, но все равно выполняется несколько запросов для связанных данных.
- **`json_agg`**: Максимальная оптимизация — один запрос, который собирает все данные в одном результате.

Этот подход особенно полезен, когда у вас сложные связи многие ко многим между моделями, как в примере с книгами, авторами и категориями.