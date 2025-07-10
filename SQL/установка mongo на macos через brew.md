
## Как установить MongoDB локально (сервер + CLI)

---

### 1. Добавь официальный MongoDB tap в brew
```bash
brew tap mongodb/brew
```

---

### 2. Установи сервер MongoDB Community Edition (и он же включает `mongosh` — CLI)
```bash
brew install mongodb-community@8.0
```

**ВАЖНО:** в новых версиях `mongo` CLI заменили на `mongosh` (Mongo Shell).  
Он автоматически идёт вместе с сервером через brew.

---

### 3. Запусти MongoDB сервер как сервис
```bash
brew services start mongodb-community@8.0
```
(Тогда MongoDB будет стартовать сама при включении Mac.)

Или просто для ручного старта:
```bash
mongod --config /usr/local/etc/mongod.conf
```

---

### 4. Подключись к серверу через CLI
```bash
mongosh
```

**Если всё правильно — увидишь что-то типа:**
```bash
Current Mongosh Log ID: ...
Connecting to: mongodb://127.0.0.1:27017
Using MongoDB: 8.0.x
```

Теперь ты в консоли MongoDB!

---

## Сводка основных команд:

| Что сделать                     | Команда                                                                 |
|----------------------------------|-------------------------------------------------------------------------|
| Установить сервер               | `brew install mongodb-community@8.0`                                   |
| Стартовать сервер как сервис    | `brew services start mongodb-community@8.0`                            |
| Подключиться к базе через CLI   | `mongosh`                                                               |
| Остановить сервер               | `brew services stop mongodb-community@8.0`                             |

---

**Небольшой бонус:** если вдруг CLI (`mongosh`) отдельно понадобится, его можно поставить и так:
```bash
brew install mongosh
```
но обычно через `mongodb-community` он уже идёт встроенным.

---