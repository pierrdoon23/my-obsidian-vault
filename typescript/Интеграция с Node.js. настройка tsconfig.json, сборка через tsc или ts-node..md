
Интеграция TypeScript с Node.js позволяет разрабатывать серверные приложения с строгой типизацией. Разберем настройку проекта, работу с `tsconfig.json`, сборку через `tsc` и запуск через `ts-node`.

---

### 1. **Инициализация проекта**
Создайте проект и установите зависимости:
```bash
npm init -y
npm install typescript @types/node --save-dev
npm install ts-node nodemon --save-dev # Для разработки
```

---

### 2. **Настройка `tsconfig.json`**
Создайте файл конфигурации TypeScript:
```bash
npx tsc --init
```

Отредактируйте `tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "ES2022", // Современный стандарт JavaScript
    "module": "CommonJS", // Node.js использует CommonJS
    "outDir": "./dist", // Директория для скомпилированных файлов
    "rootDir": "./src", // Исходные файлы
    "strict": true, // Строгая проверка типов
    "esModuleInterop": true, // Совместимость с CommonJS/ES модулями
    "skipLibCheck": true, // Ускоряет компиляцию
    "sourceMap": true, // Карты кода для отладки
    "moduleResolution": "node" // Поиск модулей как в Node.js
  },
  "include": ["src/**/*.ts"], // Какие файлы компилировать
  "exclude": ["node_modules"] // Игнорируемые директории
}
```

---

### 3. **Структура проекта**
```
project/
├─ src/
│  ├─ index.ts         # Основной файл
├─ dist/               # Скомпилированные JS-файлы (создастся автоматически)
├─ package.json
└─ tsconfig.json
```

---

### 4. **Сборка через `tsc`**
Компиляция TypeScript в JavaScript:
```bash
npx tsc
```

#### Скрипты в `package.json`:
```json
{
  "scripts": {
    "build": "tsc", // Компиляция
    "start": "node dist/index.js" // Запуск скомпилированного кода
  }
}
```
- **Запуск**: `npm run build && npm start`.

---

### 5. **Запуск через `ts-node`**
**ts-node** выполняет TypeScript-код без предварительной компиляции.  
Добавьте в `package.json`:
```json
{
  "scripts": {
    "dev": "ts-node src/index.ts"
  }
}
```
- **Запуск**: `npm run dev`.

---

### 6. **Автоматическая перезагрузка с `nodemon`**
Для отслеживания изменений в коде:
```json
{
  "scripts": {
    "dev:watch": "nodemon --watch 'src/**/*.ts' --exec 'ts-node' src/index.ts"
  }
}
```
- **Запуск**: `npm run dev:watch`.

---

### 7. **Пример кода (`src/index.ts`)**
```typescript
import express from 'express';

const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hello, TypeScript!');
});

app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`);
});
```

---

### 8. **Обработка статических файлов и шаблонов**
Если вы используете, например, `express`:
```typescript
import path from 'path';

app.use(express.static(path.join(__dirname, 'public')));
```

---

### 9. **Отладка в VS Code**
Создайте `.vscode/launch.json`:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug TS",
      "runtimeArgs": ["-r", "ts-node/register"],
      "args": ["${workspaceFolder}/src/index.ts"]
    }
  ]
}
```

---

### 10. **Деплой в продакшен**
1. Скомпилируйте проект: `npm run build`.
2. Запустите скомпилированные файлы из `dist/`:
```bash
node dist/index.js
```

---

### Итог
- **tsconfig.json** — центральный конфиг для TypeScript.
- **tsc** — компиляция в JavaScript для продакшена.
- **ts-node** — запуск TS-кода в разработке без сборки.
- **nodemon** — автоматическая перезагрузка при изменениях.

Такой подход сочетает удобство разработки и надежность типизации.