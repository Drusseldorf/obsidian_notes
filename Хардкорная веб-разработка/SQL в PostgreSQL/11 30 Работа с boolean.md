# 11.30 Работа с boolean

Работа с булевыми значениями (`boolean`) в PostgreSQL позволяет эффективно управлять состояниями записей, такими как подтверждение email, удаление записей и другие двоичные состояния. Рассмотрим основные концепции и примеры использования `boolean` в контексте базы данных.

### Создание таблиц с булевыми значениями

Создадим две таблицы: `rroom_user` и `email`. В таблице `email` используется булевое значение для определения, верифицирован ли email (`is_verified`), а также составной первичный ключ для уникальной комбинации `email` и `user_id`.

```sql
CREATE TABLE rroom_user (
    user_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    birth_date DATE
);

CREATE TABLE email (
    email VARCHAR(320),
    user_id BIGINT REFERENCES rroom_user(user_id),
    is_verified BOOLEAN NOT NULL DEFAULT FALSE,
    PRIMARY KEY (email, user_id)
);
```

**Объяснение:**

- **Составной первичный ключ** (`PRIMARY KEY (email, user_id)`) используется для обеспечения уникальности комбинации `email` и `user_id`. Это полезно, когда один и тот же email может быть связан с разными пользователями, но верифицированным может быть только один из них.
- **Булево поле** `is_verified` используется для хранения информации о верификации email.

### Уникальный индекс для верифицированных email

Для того чтобы обеспечить уникальность верифицированного email, создадим уникальный индекс:

```sql
CREATE UNIQUE INDEX unique_verified_email_per_user
ON email (email)
WHERE is_verified = true;
```

**Пример вставки данных и работы с булевым полем:**

1. Вставляем двух пользователей:
    
    ```sql
    INSERT INTO rroom_user(birth_date) VALUES ('1988-07-29') RETURNING user_id; -- 1
    INSERT INTO rroom_user(birth_date) VALUES ('1996-01-01') RETURNING user_id; -- 2
    ```
    
2. Вставляем одинаковые email для двух разных пользователей:
    
    ```sql
    INSERT INTO email (email, user_id) VALUES ('sterx@rl6.ru', 1);
    INSERT INTO email (email, user_id) VALUES ('sterx@rl6.ru', 2);
    ```
    
    Оба записи будут иметь `is_verified = false`.
    
3. Попробуем верифицировать email у обоих пользователей:
    
    ```sql
    UPDATE email SET is_verified = true WHERE email = 'sterx@rl6.ru'; -- Ошибка, нарушение уникальности
    ```
    
    Ошибка возникает из-за уникального индекса, который допускает только один верифицированный email.
    
    ```sql
    UPDATE email SET is_verified = true WHERE email = 'sterx@rl6.ru' AND user_id = 1; -- Успешно
    ```
    
    Теперь, если попытаться верифицировать email у другого пользователя, снова возникнет ошибка:
    
    ```sql
    UPDATE email SET is_verified = true WHERE email = 'sterx@rl6.ru' AND user_id = 2; -- Ошибка
    ```
    

### Подход с колонкой `is_deleted`

Вместо физического удаления данных из таблицы часто используется подход с пометкой записи как удаленной, что позволяет сохранить исторические данные:

```sql
ALTER TABLE email ADD COLUMN is_deleted BOOLEAN DEFAULT FALSE;

-- Пример выборки только активных записей
SELECT * FROM email WHERE is_deleted = false;
```

Этот подход требует добавления условия `WHERE is_deleted = false` в каждый запрос, который должен учитывать только активные записи.

### Перенос удаленных записей в отдельную таблицу

Альтернативным подходом является перенос удаленных записей в отдельную таблицу:

1. Создаем основную и "удаленную" таблицы:
    
    ```sql
    CREATE TEMP TABLE some_entity (
        entity_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
        value TEXT NOT NULL
    );
    
    CREATE TABLE some_entity_deleted (LIKE some_entity INCLUDING DEFAULTS INCLUDING CONSTRAINTS INCLUDING INDEXES);
    ```
    
2. Вставляем данные и переносим одну запись в таблицу удаленных записей с использованием транзакции:
    
    ```sql
    INSERT INTO some_entity (value) VALUES ('some value');
    INSERT INTO some_entity (value) VALUES ('some another value');
    
    BEGIN;
    INSERT INTO some_entity_deleted SELECT * FROM some_entity WHERE entity_id = 2;
    DELETE FROM some_entity WHERE entity_id = 2;
    COMMIT;
    ```
    
    Транзакция гарантирует, что операция будет выполнена атомарно — либо обе операции (вставка и удаление) будут выполнены, либо ни одна.
    
3. Альтернативный способ с использованием CTE (Common Table Expression):
    
    ```sql
    WITH deleted_rows AS (
        DELETE FROM some_entity
        WHERE entity_id = 3
        RETURNING *
    )
    INSERT INTO some_entity_deleted SELECT * FROM deleted_rows;
    ```
    

### Преимущества и удобство работы с булевыми значениями

- **Отсутствие необходимости постоянного добавления условий `WHERE is_deleted = false`**: Если используется отдельная таблица для удаленных записей, это упрощает запросы, так как не нужно помнить про дополнительное условие.
- **Разделение актуальных и исторических данных**: Это позволяет поддерживать порядок в данных и упрощает их использование.