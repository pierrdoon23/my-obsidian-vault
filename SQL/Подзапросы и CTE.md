
**Подзапросы и CTE (Common Table Expressions)** — это инструменты SQL, позволяющие структурировать сложные запросы, улучшая их читаемость и управляемость. Рассмотрим их особенности, синтаксис и примеры.

---

### **1. Подзапросы (Subqueries)**
Подзапрос — это запрос, вложенный в другой запрос. Может использоваться в:
- `SELECT`,
- `FROM`,
- `WHERE`,
- `HAVING`,
- `JOIN`.

#### **Типы подзапросов**
| **Тип**             | **Описание**                                                                 |
|----------------------|-----------------------------------------------------------------------------|
| **Некоррелированный** | Выполняется один раз, независимо от внешнего запроса. Результат кешируется. |
| **Коррелированный**   | Зависит от внешнего запроса. Выполняется для каждой строки внешнего запроса. |

#### **Примеры**
**a. В `WHERE` (Некоррелированный):**
```sql
-- Найти сотрудников с зарплатой выше средней
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

**b. В `SELECT` (Коррелированный):**
```sql
-- Для каждого отдела вывести количество сотрудников
SELECT 
    d.department_name,
    (SELECT COUNT(*) 
     FROM employees e 
     WHERE e.department_id = d.department_id) AS emp_count
FROM departments d;
```

**c. В `FROM`:**
```sql
-- Использование подзапроса как временной таблицы
SELECT dept_id, avg_salary
FROM (
    SELECT department_id AS dept_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) AS dept_stats
WHERE avg_salary > 50000;
```

**d. В `JOIN`:**
```sql
-- Объединение с результатом подзапроса
SELECT e.name, d.department_name
FROM employees e
JOIN (
    SELECT department_id, department_name 
    FROM departments 
    WHERE location = 'New York'
) AS d ON e.department_id = d.department_id;
```

---

### **2. CTE (Common Table Expressions)**
CTE — это временный результат запроса, который можно использовать в основном запросе. Определяется через `WITH`.

#### **Преимущества CTE:**
- Улучшает читаемость (особенно для сложных запросов).
- Позволяет рекурсию.
- Может быть использован несколько раз в одном запросе.

#### **Синтаксис:**
```sql
WITH cte_name (column1, column2, ...) AS (
    -- Подзапрос
)
SELECT * FROM cte_name;
```

#### **Примеры**
**a. Простой CTE:**
```sql
-- Найти отделы с зарплатой выше средней
WITH avg_salary AS (
    SELECT AVG(salary) AS avg FROM employees
)
SELECT department_id, AVG(salary) AS dept_avg
FROM employees
GROUP BY department_id
HAVING AVG(salary) > (SELECT avg FROM avg_salary);
```

**b. Множественные CTE:**
```sql
WITH 
    dept_stats AS (
        SELECT department_id, COUNT(*) AS emp_count
        FROM employees
        GROUP BY department_id
    ),
    high_budget_depts AS (
        SELECT department_id 
        FROM departments 
        WHERE budget > 100000
    )
SELECT d.department_name, ds.emp_count
FROM dept_stats ds
JOIN high_budget_depts hbd ON ds.department_id = hbd.department_id
JOIN departments d ON ds.department_id = d.department_id;
```

**c. Рекурсивный CTE:**
```sql
-- Иерархия сотрудников (менеджер → подчиненные)
WITH RECURSIVE employee_hierarchy AS (
    -- Анкерный запрос (начало иерархии)
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    -- Рекурсивная часть
    SELECT e.id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM employee_hierarchy;
```

---

### **3. Подзапросы vs CTE**
| **Критерий**       | **Подзапросы**                          | **CTE**                                  |
|---------------------|-----------------------------------------|------------------------------------------|
| **Читаемость**      | Могут усложнить запрос                  | Улучшают структуризацию                 |
| **Многократное использование** | Нет (дублирование кода)           | Да (можно ссылаться несколько раз)      |
| **Рекурсия**        | Не поддерживается                       | Поддерживается через `WITH RECURSIVE`   |
| **Производительность** | Коррелированные могут быть медленными | Оптимизируются аналогично подзапросам   |

---

### **4. Рекомендации**
1. **Используйте CTE** для:
   - Многошаговых преобразований данных.
   - Рекурсивных запросов (оргструктуры, деревья).
   - Улучшения читаемости (особенно с несколькими подзапросами).

2. **Используйте подзапросы** для:
   - Простых условий в `WHERE` или `SELECT`.
   - Быстрых одноразовых вычислений.

3. **Избегайте** избыточных коррелированных подзапросов — они могут замедлить выполнение.

---

### **5. Пример замены подзапроса на CTE**
**Исходный подзапрос:**
```sql
SELECT name, salary
FROM employees
WHERE department_id IN (
    SELECT department_id 
    FROM departments 
    WHERE location = 'London'
);
```

**С CTE:**
```sql
WITH london_departments AS (
    SELECT department_id 
    FROM departments 
    WHERE location = 'London'
)
SELECT e.name, e.salary
FROM employees e
JOIN london_departments ld ON e.department_id = ld.department_id;
```

---

**Итог:**  
Подзапросы и CTE — взаимодополняющие инструменты. Выбор зависит от сложности задачи и требований к читаемости. CTE особенно полезны для структурирования многоэтапных запросов и рекурсии.