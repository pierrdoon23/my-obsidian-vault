
Node.js — среда выполнения JavaScript на стороне сервера, основанная на асинхронной, событийно-ориентированной архитектуре. Это позволяет эффективно работать с I/O-операциями (сеть, файлы) и масштабироваться. Рассмотрим ключевые особенности:

---

### 1. **Буферы (Buffers)**
Буферы — структуры данных для работы с бинарными данными (например, изображения, файлы).  
**Особенности**:
- Создаются фиксированного размера.
- Хранят сырые байты (не кодируются в строки).
- Используются там, где требуется низкоуровневая работа с данными.

#### Примеры:
**Создание буфера:**
```javascript
// Создание буфера из строки (кодировка UTF-8)
const buf1 = Buffer.from("Hello, Node.js!");
console.log(buf1); // <Buffer 48 65 6c 6c 6f 2c 20 4e 6f 64 65 2e 6a 73 21>

// Создание буфера заданного размера
const buf2 = Buffer.alloc(10); // 10 байт, заполненных нулями
buf2.write("ABCD"); // Запись данных
console.log(buf2); // <Buffer 41 42 43 44 00 00 00 00 00 00>
```

**Чтение данных из буфера:**
```javascript
const buf = Buffer.from("Привет");
console.log(buf.toString()); // "Привет" (по умолчанию UTF-8)
console.log(buf.toString('hex')); // hex-представление: d09f d180 d0b8 d0b2 d0b5 d182
```

**Конвертация в JSON:**
```javascript
const buf = Buffer.from("test");
console.log(JSON.stringify(buf)); // {"type":"Buffer","data":[116,101,115,116]}
```

---

### 2. **Потоки (Streams)**
Потоки позволяют обрабатывать данные по частям, не загружая их целиком в память.  
**Типы потоков**:
- **Readable** — чтение данных (например, файл, HTTP-запрос).
- **Writable** — запись данных (например, файл, HTTP-ответ).
- **Duplex** — чтение и запись (например, сокет).
- **Transform** — модификация данных на лету (например, сжатие).

#### Примеры:
**Чтение файла через поток:**
```javascript
const fs = require('fs');

const readableStream = fs.createReadStream('input.txt', 'utf8');

readableStream.on('data', (chunk) => {
  console.log('Получен блок данных:', chunk);
});

readableStream.on('end', () => {
  console.log('Чтение завершено.');
});
```

**Запись в файл через поток:**
```javascript
const writableStream = fs.createWriteStream('output.txt');

writableStream.write('Первая строка\n');
writableStream.write('Вторая строка\n');
writableStream.end(); // Завершение записи
```

**Конвейер потоков (Pipe):**
```javascript
const zlib = require('zlib');

// Сжатие файла
fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'));
```

---

### 3. **Модуль `child_process`**
Позволяет создавать дочерние процессы для выполнения системных команд или других приложений.  
**Методы**:
- **`spawn`** — запуск процесса с потоковым I/O (подходит для больших данных).
- **`exec`** — запуск команды с буферизированным выводом (удобен для коротких команд).
- **`execFile`** — запуск исполняемого файла.
- **`fork`** — запуск модуля Node.js как отдельного процесса (общение через IPC).

#### Примеры:
**Использование `spawn`:**
```javascript
const { spawn } = require('child_process');

// Запуск команды ls (Linux/macOS) или dir (Windows)
const ls = spawn('ls', ['-lh', '/usr']);

ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

ls.stderr.on('data', (data) => {
  console.error(`stderr: ${data}`);
});

ls.on('close', (code) => {
  console.log(`Процесс завершен с кодом ${code}`);
});
```

**Использование `exec`:**
```javascript
const { exec } = require('child_process');

exec('node -v', (error, stdout, stderr) => {
  if (error) {
    console.error(`Ошибка: ${error.message}`);
    return;
  }
  console.log(`Версия Node.js: ${stdout}`);
});
```

**Использование `fork`:**
```javascript
// parent.js
const { fork } = require('child_process');
const child = fork('child.js');

child.on('message', (msg) => {
  console.log('Сообщение от ребенка:', msg);
});

child.send({ hello: 'world' });

// child.js
process.on('message', (msg) => {
  console.log('Сообщение от родителя:', msg);
  process.send({ response: 'ok' });
});
```

---

### Итог
- **Буферы** — работа с бинарными данными.
- **Потоки** — обработка больших данных без нагрузки на память.
- **`child_process`** — выполнение внешних команд и параллельных задач.

Эти механизмы делают Node.js мощным инструментом для работы с файлами, сетью и системными процессами.