
**Расширения PostgreSQL** — это дополнительные модули, которые добавляют новые функции, типы данных, операторы или целые возможности в СУБД. Они позволяют адаптировать PostgreSQL под специфические задачи без изменения ядра системы. Разберем, как работать с расширениями и какие из них наиболее полезны.

---

### **1. Управление расширениями**
#### **Установка**
```sql
-- Установить расширение (требуются права суперпользователя)
CREATE EXTENSION extension_name;
```

#### **Просмотр установленных расширений**
```sql
SELECT * FROM pg_extension;
```

#### **Просмотр доступных расширений**
```sql
SELECT * FROM pg_available_extensions;
```

#### **Обновление**
```sql
ALTER EXTENSION extension_name UPDATE;
```

#### **Удаление**
```sql
DROP EXTENSION extension_name;
```

---

### **2. Популярные расширения**
#### **a. PostGIS**
- **Для чего:** Работа с геопространственными данными (GIS).
- **Установка:**
  ```sql
  CREATE EXTENSION postgis;
  ```
- **Пример:**  
  Хранение координат и расчет расстояний:
  ```sql
  SELECT ST_Distance(
      ST_GeomFromText('POINT(55.7522 37.6156)'), -- Москва
      ST_GeomFromText('POINT(59.9343 30.3351)')  -- Санкт-Петербург
  ) AS distance_km;
  ```

---

#### **b. pgcrypto**
- **Для чего:** Шифрование данных, хеширование.
- **Установка:**
  ```sql
  CREATE EXTENSION pgcrypto;
  ```
- **Пример:**  
  Хеширование пароля:
  ```sql
  INSERT INTO users (username, password) 
  VALUES ('alice', crypt('secret', gen_salt('bf')));
  ```

---

#### **c. uuid-ossp**
- **Для чего:** Генерация UUID (уникальных идентификаторов).
- **Установка:**
  ```sql
  CREATE EXTENSION "uuid-ossp";
  ```
- **Пример:**  
  ```sql
  SELECT uuid_generate_v4(); -- 550e8400-e29b-41d4-a716-446655440000
  ```

---

#### **d. hstore**
- **Для чего:** Хранение данных в формате ключ-значение внутри столбца.
- **Установка:**
  ```sql
  CREATE EXTENSION hstore;
  ```
- **Пример:**  
  ```sql
  CREATE TABLE products (
      id SERIAL PRIMARY KEY,
      attributes HSTORE
  );
  INSERT INTO products (attributes) 
  VALUES ('color => "red", size => "M"');
  ```

---

#### **e. pg_stat_statements**
- **Для чего:** Анализ производительности SQL-запросов.
- **Установка:**
  1. Добавить в `postgresql.conf`:  
     ```
     shared_preload_libraries = 'pg_stat_statements'
     ```
  2. Перезагрузить PostgreSQL.
  3. Выполнить в БД:
     ```sql
     CREATE EXTENSION pg_stat_statements;
     ```
- **Пример:**  
  Просмотр статистики запросов:
  ```sql
  SELECT query, total_time, calls 
  FROM pg_stat_statements 
  ORDER BY total_time DESC 
  LIMIT 10;
  ```

---

#### **f. TimescaleDB**
- **Для чего:** Работа с временными рядами (аналог InfluxDB).
- **Установка:**  
  Требуется отдельная установка через пакетный менеджер ОС.
- **Пример:**  
  Создание гипертаблицы для метрик:
  ```sql
  CREATE TABLE metrics (
      time TIMESTAMPTZ NOT NULL,
      device_id INT,
      temperature DOUBLE PRECISION
  );
  SELECT create_hypertable('metrics', 'time');
  ```

---

### **3. Советы по использованию**
1. **Проверяйте совместимость:** Убедитесь, что расширение поддерживается вашей версией PostgreSQL.
2. **Устанавливайте через contrib:** Многие расширения входят в пакет `postgresql-contrib` (для Linux).
3. **Ограничивайте привилегии:** Не давайте права на установку расширений обычным пользователям.
4. **Тестируйте:** Некоторые расширения могут влиять на производительность.

---

### **4. Полезные запросы**
- **Список установленных расширений:**
  ```sql
  SELECT extname, extversion FROM pg_extension;
  ```
- **Поиск функций расширения:**
  ```sql
  SELECT proname, pg_get_functiondef(oid) 
  FROM pg_proc 
  WHERE proname LIKE 'uuid%';
  ```

---

### **5. Создание своих расширений**
Для продвинутых пользователей PostgreSQL позволяет создавать собственные расширения на языке C или SQL. Процесс включает:
1. Написание кода функций.
2. Создание файла контрольного скрипта (`extension.control`).
3. Упаковку в модуль и установку.

---

**Расширения** превращают PostgreSQL в универсальный инструмент, адаптированный под любые задачи: от геоаналитики до обработки временных рядов. Главное — использовать их обдуманно и следить за актуальностью версий!