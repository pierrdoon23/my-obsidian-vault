
---

## **Основы Node.js: Event Loop, модули (CommonJS и ESM), файловая система (fs)**

### 1. **Event Loop**
**Event Loop** — это механизм, который позволяет Node.js выполнять асинхронные операции без блокировки основного потока. Несмотря на однопоточность, Node.js эффективно обрабатывает множество задач за счет неблокирующего ввода-вывода (I/O).

#### Как работает Event Loop:
1. **Фазы Event Loop**:
   - **Таймеры**: Обработка `setTimeout` и `setInterval`.
   - **Pending callbacks**: Выполнение колбэков системных операций (например, ошибки сети).
   - **Idle/Prepare**: Внутренние технические фазы.
   - **Poll**: Ожидание новых событий (например, входящие HTTP-запросы).
   - **Check**: Вызов колбэков `setImmediate`.
   - **Close**: Обработка закрывающих событий (например, `socket.on('close')`).

2. **Пример порядка выполнения**:
```javascript
setTimeout(() => console.log('Timeout'), 0);
setImmediate(() => console.log('Immediate'));
```
Результат может варьироваться, но в основном цикле:
- `setImmediate` выполнится в фазе **Check**.
- `setTimeout` — в фазе **Таймеры**.

---

### 2. **Модули: CommonJS vs ESM**
#### **CommonJS** (стандарт Node.js до версии 14):
- Использует `require` для импорта и `module.exports` для экспорта.
- **Пример**:
  ```javascript
  // math.js
  const add = (a, b) => a + b;
  module.exports = { add };

  // app.js
  const { add } = require('./math.js');
  console.log(add(2, 3)); // 5
  ```

#### **ES Modules (ESM)** (современный стандарт):
- Использует `import/export`.
- Включение ESM:
  - Добавить `"type": "module"` в `package.json`.
  - Или использовать расширение `.mjs` для файлов.
- **Пример**:
  ```javascript
  // math.mjs
  export const add = (a, b) => a + b;

  // app.mjs
  import { add } from './math.mjs';
  console.log(add(2, 3)); // 5
  ```

#### **Сравнение**:
| **Параметр**      | **CommonJS**                     | **ESM**                          |
|--------------------|----------------------------------|----------------------------------|
| Синтаксис          | `require/module.exports`         | `import/export`                  |
| Загрузка модулей   | Синхронная (во время выполнения) | Асинхронная (во время парсинга)  |
| Совместимость      | Поддерживается везде             | Требует настройки (`type: module`) |
| Динамический импорт| `require()`                      | `import('module')`               |

---

### 3. **Файловая система (fs)**
Модуль `fs` позволяет работать с файлами. Есть синхронные и асинхронные методы.

#### **Асинхронные методы** (не блокируют поток):
```javascript
import { readFile, writeFile } from 'fs/promises';

// Чтение файла
readFile('file.txt', 'utf8')
  .then(data => console.log(data))
  .catch(err => console.error(err));

// Запись в файл
await writeFile('new.txt', 'Hello Node.js');
```

#### **Синхронные методы** (блокируют поток):
```javascript
import { readFileSync } from 'fs';

const data = readFileSync('file.txt', 'utf8');
console.log(data);
```

#### **Работа с путями (модуль `path`)**:
```javascript
import path from 'path';

const fullPath = path.join(__dirname, 'data', 'file.txt');
console.log(fullPath); // /project/data/file.txt
```

#### **Пример проверки существования файла**:
```javascript
import { access } from 'fs/promises';

async function checkFileExists(file) {
  try {
    await access(file);
    return true;
  } catch {
    return false;
  }
}
```

---

### **Итог**
- **Event Loop** — сердце асинхронности Node.js, управляет выполнением операций.
- **Модули**:
  - CommonJS — классический подход с `require`.
  - ESM — современный стандарт с `import/export`.
- **Файловая система**:
  - Используйте асинхронные методы для избежания блокировки.
  - `fs.promises` упрощает работу с промисами.
  - Всегда обрабатывайте ошибки в асинхронных операциях.