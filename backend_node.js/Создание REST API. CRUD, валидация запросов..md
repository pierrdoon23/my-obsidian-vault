
---

## **Создание REST API в Node.js: CRUD, валидация запросов**

### 1. **Базовая настройка**
Создадим простое REST API для управления пользователями с операциями CRUD.  
Используем **Express.js** и **express-validator** для валидации.

```bash
npm install express express-validator
```

#### `app.js`:
```javascript
import express from 'express';
import { body, validationResult } from 'express-validator';

const app = express();
app.use(express.json());

let users = [
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' },
];

// Маршруты
app.get('/users', (req, res) => {
  res.json(users);
});

// Запуск сервера
const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});
```

---

### 2. **Реализация CRUD**

#### **Create (POST)**
Добавим валидацию для полей `name` и `email`:
```javascript
app.post(
  '/users',
  [
    body('name').notEmpty().withMessage('Name is required'),
    body('email').isEmail().withMessage('Invalid email'),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    const newUser = { id: users.length + 1, ...req.body };
    users.push(newUser);
    res.status(201).json(newUser);
  }
);
```

#### **Read (GET by ID)**
```javascript
app.get('/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ message: 'User not found' });
  res.json(user);
});
```

#### **Update (PUT)**
```javascript
app.put(
  '/users/:id',
  [
    body('name').optional().notEmpty().withMessage('Name is required'),
    body('email').optional().isEmail().withMessage('Invalid email'),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    const user = users.find(u => u.id === parseInt(req.params.id));
    if (!user) return res.status(404).json({ message: 'User not found' });

    Object.assign(user, req.body);
    res.json(user);
  }
);
```

#### **Delete (DELETE)**
```javascript
app.delete('/users/:id', (req, res) => {
  const index = users.findIndex(u => u.id === parseInt(req.params.id));
  if (index === -1) return res.status(404).json({ message: 'User not found' });

  users.splice(index, 1);
  res.status(204).send(); // 204 No Content
});
```

---

### 3. **Валидация запросов**
Используем `express-validator` для проверки входящих данных:
- `body()`: Валидация полей тела запроса.
- `query()` / `params()`: Для query-параметров и параметров пути.
- Кастомные сообщения об ошибках.

**Пример валидации email и пароля**:
```javascript
body('password')
  .isLength({ min: 6 })
  .withMessage('Password must be at least 6 characters'),
```

---

### 4. **Обработка ошибок**
Добавим middleware для централизованной обработки ошибок:
```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ message: 'Internal Server Error' });
});
```

---

### 5. **Тестирование API**
Используйте **Postman** или **curl**:

#### Создание пользователя:
```bash
curl -X POST -H "Content-Type: application/json" -d '{"name":"Charlie","email":"charlie@test"}' http://localhost:3000/users
```
**Ответ при ошибке**:
```json
{
  "errors": [
    {
      "msg": "Invalid email",
      "param": "email",
      "location": "body"
    }
  ]
}
```

---

### 6. **Рекомендации**
1. **Структура проекта**:  
   Разделите код на файлы:
   - `routes/users.js` – маршруты.
   - `controllers/users.js` – логика обработки.
   - `middlewares/validation.js` – валидация.

2. **База данных**:  
   Замените массив `users` на подключение к PostgreSQL/MongoDB (см. предыдущие разделы).

3. **Дополнительно**:  
   - Добавьте аутентификацию (JWT/OAuth).  
   - Используйте лимитер запросов (`express-rate-limit`).  
   - Включите CORS (`cors`).  

---
