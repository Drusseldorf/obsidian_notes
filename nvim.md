### **Вкладки (Tabs)**

Neovim поддерживает вкладки, которые работают как контейнеры для окон.

1. **Открыть новый файл во вкладке:**
    
    ```php
    :tabnew <имя_файла>
    ```
    
    Если файл ещё не существует, он будет создан.
    
2. **Переключение между вкладками:**
    - На следующую вкладку: `gt`
    - На предыдущую вкладку: `gT`
3. **Список вкладок:**
    
    ```ruby
    :tabs
    ```
    
4. **Закрыть текущую вкладку:**
    
    ```ruby
    :tabclose
    ```
    

### **Разделение окна (Splits)**

Можно открыть несколько файлов в одном экране с разделением окон.

1. **Открыть файл в горизонтальном разделе:**
    
    ```bash
    :split <имя_файла>
    ```
    
2. **Открыть файл в вертикальном разделе:**
    
    ```php
    :vsplit <имя_файла>
    ```
    
3. **Перемещение между окнами:**
    - `Ctrl+w w` — Переключение между окнами.
    - `Ctrl+w h` — Влево.
    - `Ctrl+w j` — Вниз.
    - `Ctrl+w k` — Вверх.
    - `Ctrl+w l` — Вправо.
4. **Закрытие окна:**
    
    ```less
    :q
    ```
    

## Переходы в режим вставки

`A` — переходит в конец строки и включает режим ввода.

### Решения некоторых проблем

1. При копировании и вставке текста в конце строк появляется символ ^M
   
   Символы вида `^M` в конце строки — это признак «windows-формата» строк (CRLF), когда в файле используются возвраты каретки `\r` плюс перевод строки `\n`. Linux-системы (и большинство UNIX-инструментов) ожидают только `\n` (LF).
   
   Решение: 
   1. Явно указать формат файла через команду `:set ff=unix`
   2. Провести замену символов через команду `:%s/\r//g`