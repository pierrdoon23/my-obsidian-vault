
Индексация в PostgreSQL — это механизм ускорения поиска и обработки данных в таблицах. Индексы работают как «оглавление» для данных, позволяя СУБД быстро находить нужные строки без полного сканирования таблицы. Однако индексы требуют дополнительных ресурсов на их поддержку (при вставке, обновлении, удалении данных), поэтому их нужно использовать обдуманно.

---

### **1. Основные типы индексов**
PostgreSQL поддерживает несколько типов индексов, каждый из которых подходит для определенных сценариев:

#### **a. B-tree (Балансированное дерево)**
- **Для чего:** Стандартный индекс для большинства операций (сравнение: `=`, `<`, `>`, `BETWEEN`, `ORDER BY`).
- **Пример:**  
  ```sql
  CREATE INDEX idx_users_email ON users(email);
  ```

#### **b. Hash**
- **Для чего:** Точное совпадение (`=`). Работает быстрее B-tree для простых равенств, но не поддерживает сортировку и диапазоны.
- **Пример:**  
  ```sql
  CREATE INDEX idx_users_id_hash ON users USING HASH(id);
  ```

#### **c. GiST (Generalized Search Tree)**
- **Для чего:** Геометрические данные, полнотекстовый поиск, диапазоны. Подходит для сложных типов данных.
- **Пример для геоданных:**  
  ```sql
  CREATE INDEX idx_points_geo ON points USING GIST(coordinates);
  ```

#### **d. SP-GiST (Space-Partitioned GiST)**
- **Для чего:** Неравномерные структуры данных (например, IP-адреса, точки в нерегулярных сетках).

#### **e. GIN (Generalized Inverted Index)**
- **Для чего:** Составные данные (массивы, JSONB, полнотекстовый поиск). Позволяет искать элементы внутри сложных структур.
- **Пример для JSONB:**  
  ```sql
  CREATE INDEX idx_products_tags ON products USING GIN(tags);
  ```

#### **f. BRIN (Block Range Index)**
- **Для чего:** Большие таблицы с упорядоченными данными (например, временные метки). Работает с диапазонами блоков данных.
- **Пример:**  
  ```sql
  CREATE INDEX idx_sensors_time ON sensors USING BRIN(timestamp);
  ```

---

### **2. Создание индексов**
#### **Базовый синтаксис:**
```sql
CREATE INDEX [CONCURRENTLY] [index_name] ON table_name (column1, column2, ...)
[USING method] -- B-tree, GIN и т.д.
[WITH (option = value)] -- параметры (например, fillfactor)
[WHERE condition]; -- частичный индекс
```

#### **Примеры:**
- **Составной индекс (несколько столбцов):**  
  ```sql
  CREATE INDEX idx_users_name_age ON users(name, age);
  ```
- **Покрывающий индекс (INCLUDE):**  
  ```sql
  CREATE INDEX idx_orders_customer ON orders(customer_id) INCLUDE (order_date);
  ```
  Такой индекс позволяет выполнять запросы только по индексу, без обращения к таблице («index-only scan»).

- **Частичный индекс (только для подмножества данных):**  
  ```sql
  CREATE INDEX idx_orders_active ON orders(status) WHERE status = 'active';
  ```

---

### **3. Когда использовать индексы**
- **Частые поисковые запросы** по условиям в `WHERE`.
- **Сортировка (`ORDER BY`)** и группировка (`GROUP BY`).
- **Соединения таблиц (`JOIN`)** по ключам.
- **Ограничения уникальности** (`UNIQUE CONSTRAINT`).

---

### **4. Лучшие практики**
#### **a. Избегайте избыточности**
- Не создавайте индексы для редко используемых столбцов.
- Удаляйте неиспользуемые индексы (проверяйте через `pg_stat_user_indexes`).

#### **b. Оптимизация для больших таблиц**
- Используйте **BRIN** для упорядоченных данных.
- Для частичных данных применяйте **частичные индексы**.

#### **c. Параметры индексов**
- **CONCURRENTLY**: Создавайте индексы без блокировки таблицы (но дольше):  
  ```sql
  CREATE INDEX CONCURRENTLY idx_users_email ON users(email);
  ```
- **fillfactor**: Задавайте свободное место в страницах индекса для обновлений (например, `WITH (fillfactor=70)`).

#### **d. Обслуживание**
- Перестраивайте индексы командой `REINDEX`:  
  ```sql
  REINDEX INDEX idx_users_email;
  ```
- Обновляйте статистику:  
  ```sql
  ANALYZE table_name;
  ```

---

### **5. Примеры использования**
#### **a. Ускорение поиска в JSONB**
```sql
-- Создание индекса GIN для поиска по ключам JSONB
CREATE INDEX idx_products_metadata ON products USING GIN(metadata);

-- Запрос: найти продукты с тегом "electronics"
SELECT * FROM products WHERE metadata @> '{"tags": ["electronics"]}';
```

#### **b. Полнотекстовый поиск**
```sql
-- Создание индекса GIN для полнотекстового поиска
CREATE INDEX idx_docs_content ON documents USING GIN(to_tsvector('english', content));

-- Запрос: найти документы с словом "database"
SELECT * FROM documents 
WHERE to_tsvector('english', content) @@ to_tsquery('database');
```

---

### **6. Как проверить использование индекса**
Используйте `EXPLAIN ANALYZE`, чтобы убедиться, что индекс применяется:
```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
```
В выводе ищите узлы **Index Scan** или **Index Only Scan**.

---

### **7. Ограничения**
- **Замедление операций записи**: Каждый индекс увеличивает время вставки/обновления.
- **Размер на диске**: Индексы могут занимать больше места, чем сама таблица.
- **Неправильный выбор типа индекса**: Например, B-tree для JSONB не даст эффекта.

---

### **8. Полезные запросы**
- **Список индексов таблицы:**  
  ```sql
  SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'table_name';
  ```
- **Статистика использования индексов:**  
  ```sql
  SELECT * FROM pg_stat_user_indexes WHERE relname = 'table_name';
  ```

Индексация — это мощный инструмент, но его эффективность зависит от понимания структуры данных и паттернов запросов. Всегда тестируйте индексы на реалистичных данных и нагрузке!