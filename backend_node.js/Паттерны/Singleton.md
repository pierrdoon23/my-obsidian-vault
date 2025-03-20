
### Паттерн Singleton (Одиночка) в JavaScript

**Singleton** — это порождающий паттерн проектирования, который гарантирует, что у класса есть только **один экземпляр**, и предоставляет глобальную точку доступа к этому экземпляру.  
Используется, когда требуется централизованное управление ресурсом (например, подключение к БД, логгер, конфигурация приложения).

---

#### **Зачем нужен Singleton?**
- Контроль над созданием экземпляра (только один на всё приложение).
- Глобальный доступ к экземпляру из любой части кода.
- Экономия ресурсов (не создавать повторно «тяжелые» объекты).

---

### Реализация Singleton в JavaScript

#### 1. **Через класс (ES6)**
Используется приватный конструктор и статическое свойство для хранения экземпляра:
```javascript
class Database {
  constructor(config) {
    if (Database.instance) {
      return Database.instance;
    }
    this.config = config;
    Database.instance = this;
    return this;
  }

  connect() {
    console.log(`Подключение к БД: ${this.config.url}`);
  }
}

// Использование
const db1 = new Database({ url: 'postgres://localhost:5432' });
const db2 = new Database({ url: 'mysql://localhost:3306' });

db1.connect(); // "Подключение к БД: postgres://localhost:5432"
db2.connect(); // "Подключение к БД: postgres://localhost:5432" (экземпляр не создается заново)
console.log(db1 === db2); // true
```

---

#### 2. **Через модули Node.js**
Модули в Node.js кешируются, поэтому объект, экспортируемый из модуля, автоматически становится синглтоном:
```javascript
// db.js
let instance = null;

class Database {
  constructor(config) {
    if (!instance) {
      this.config = config;
      instance = this;
    }
    return instance;
  }
}

module.exports = Database;

// В другом файле
const db1 = require('./db');
const db2 = require('./db');
console.log(db1 === db2); // true
```

---

#### 3. **Через замыкание (без классов)**
```javascript
const createLogger = () => {
  let logs = [];

  const addLog = (message) => {
    logs.push(message);
  };

  const getLogs = () => logs;

  return { addLog, getLogs };
};

// Singleton
const logger = (() => {
  let instance;
  return () => {
    if (!instance) {
      instance = createLogger();
    }
    return instance;
  };
})();

// Использование
const log1 = logger();
const log2 = logger();
log1.addLog('Ошибка 404');
console.log(log2.getLogs()); // ["Ошибка 404"]
console.log(log1 === log2); // true
```

---

### **Плюсы и минусы Singleton**
**✅ Плюсы**  
- Гарантирует единственный экземпляр.  
- Удобен для управления общими ресурсами (кеш, настройки, БД).  

**❌ Минусы**  
- Нарушает принцип инверсии зависимостей (DIP из SOLID).  
- Усложняет тестирование (глобальное состояние).  
- Может привести к избыточной связности кода.

---

### **Когда использовать?**
- **Логгеры**: Единый объект для записи логов.  
- **Подключение к БД**: Один пул соединений для всех запросов.  
- **Конфигурация приложения**: Общие настройки для всех модулей.  
- **Кеширование**: Глобальный кеш данных.

---

### **Пример: Singleton для подключения к MongoDB**
```javascript
// mongo.js
const { MongoClient } = require('mongodb');

class MongoDB {
  constructor() {
    if (MongoDB.instance) {
      return MongoDB.instance;
    }

    this.client = new MongoClient(process.env.MONGO_URI);
    this.db = null;
    MongoDB.instance = this;
  }

  async connect() {
    if (!this.db) {
      await this.client.connect();
      this.db = this.client.db('myapp');
    }
    return this.db;
  }
}

module.exports = new MongoDB();

// В другом файле
const mongo1 = require('./mongo');
const mongo2 = require('./mongo');
console.log(mongo1 === mongo2); // true
```

---

### **Важно!**
- **Не злоупотребляйте Singleton**. Часто его можно заменить зависимостями (Dependency Injection).  
- В Node.js модули и так кешируются, поэтому объекты, экспортируемые через `module.exports`, уже являются «синглтонами» для текущего процесса.