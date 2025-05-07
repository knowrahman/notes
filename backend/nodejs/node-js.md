### What is Node.js?

* **Node.js** is a runtime environment that lets you run JavaScript code *outside* the browser.
* It's built on **Google Chrome’s V8 engine**, which makes it very fast.
* It uses a **non-blocking, event-driven architecture**, making it ideal for building scalable network applications.

#### Example: Hello World in Node.js

Create a file named `hello.js`:

```js
console.log('Hello, Node.js!');
```

Run it using:

```bash
node hello.js
```

You should see:

```
Hello, Node.js!
```

### Step-by-Step Learning Plan

1. **Basics of Node.js**

   * What is Node.js?
   * How it works (event loop, non-blocking I/O).
   * Writing your first script (`hello.js`).

2. **Node.js Core Modules**

   * `fs`, `http`, `path`, `os`, `events`, etc.
   * Hands-on with simple file server, logging, and reading/writing files.

3. **npm and Modules**

   * Installing packages (`npm i`), `package.json`, and versioning.
   * Creating and using custom modules.

4. **Express.js Introduction**

   * What is Express and why use it?
   * Your first Express server.

5. **Routing in Express**

   * `app.get()`, `app.post()`, `req`, `res`.
   * Route parameters and query strings.

6. **Middleware**

   * Built-in, custom, and third-party middleware.
   * `express.json()`, `express.static()`.

7. **Building a REST API**

   * CRUD operations with in-memory data.
   * RESTful principles explained simply.

8. **Advanced Topics**

   * Express Router, MVC structure.
   * Error handling, environment variables, deployment.

---

### Globals in Node.js

In Node.js, **globals** are variables, functions, and objects that are **available in all modules** without needing to `require()` them. Think of them as built-in tools always ready to use.

They are like the air in a room—you don’t need to import it; it's just there.

---

### Key Global Objects in Node.js

| Global                      | Description                                 | Example                      |
| --------------------------- | ------------------------------------------- | ---------------------------- |
| `__dirname`                 | Directory name of the current module        | `console.log(__dirname)`     |
| `__filename`                | Full path of the current file               | `console.log(__filename)`    |
| `require()`                 | Function to import modules                  | `const fs = require('fs')`   |
| `module`                    | Represents the current module               | `console.log(module)`        |
| `exports`                   | Used to export values from a module         | `module.exports = {...}`     |
| `global`                    | Like `window` in browsers; global namespace | `global.name = "Node";`      |
| `process`                   | Info about the current Node process         | `console.log(process.argv)`  |
| `Buffer`                    | Used to work with binary data               | `Buffer.from('hello')`       |
| `setTimeout`, `setInterval` | Timers like in browsers                     | `setTimeout(() => {}, 1000)` |

---

### Examples

1. **`__dirname` and `__filename`:**

```js
console.log('Directory:', __dirname);
console.log('File:', __filename);
```

2. **Using `global`:**

```js
global.message = 'Accessible everywhere';
console.log(global.message);
```

3. **Process info:**

```js
console.log(process.platform); // e.g., 'win32', 'linux'
console.log(process.argv);     // Arguments passed to the script
```

---

### Memory Trick

Use this acronym: **"DR. G.B.P.M."**
(D for `__dirname`, R for `require`, G for `global`, B for `Buffer`, P for `process`, M for `module`)

> "Doctor GBP Makes Node Work."

---


Let’s explore how `require`, `module`, and `exports` work together to organize code into modules in Node.js.

---

### What is `require()`?

`require()` is a **function** used to **import** code from other files or built-in modules.

### What is `module.exports`?

Every file in Node.js is treated as a **separate module**. You use `module.exports` to **expose** parts of that file (functions, objects, variables) so they can be imported elsewhere using `require()`.

---

### Basic Example

#### File: `math.js`

```js
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

// Export functions
module.exports = {
  add,
  subtract
};
```

#### File: `app.js`

```js
const math = require('./math'); // './' means it's a local file

console.log(math.add(5, 3));       // 8
console.log(math.subtract(10, 4)); // 6
```

---

### Behind the Scenes

* When `require('./math')` runs:

  * Node loads `math.js`.
  * It wraps the file in a function behind the scenes:

    ```js
    (function(exports, require, module, __filename, __dirname) {
      // Your code here
    });
    ```
  * This is why you can use `require`, `module`, `exports`, `__dirname`, and `__filename` directly—they’re available as globals inside that wrapper.

---

### Shorter Way (using `exports` directly)

You can also do:

```js
exports.add = (a, b) => a + b;
exports.subtract = (a, b) => a - b;
```

But **don’t mix** `exports =` with `module.exports =`—only use one.

---

### Memory Trick

**"RAM"**

* `Require` → to pull in
* `Assign` to `module.exports`
* `Make` functions available

"Use RAM to make your code modular!"

---

Want to try using `require()` with a custom logger module next, or go deeper into the Node.js module system (like caching, file paths, or third-party modules)?


Let’s build a custom **logger module** and use `require()` to import it.

---

### Step-by-Step: Creating and Using a Logger Module

#### 1. Create a file named `logger.js`

```js
// logger.js

function logInfo(message) {
  console.log(`INFO: ${message}`);
}

function logError(message) {
  console.error(`ERROR: ${message}`);
}

// Export the functions
module.exports = {
  logInfo,
  logError
};
```

#### 2. Create a file named `app.js`

```js
// app.js

// Import your custom logger
const logger = require('./logger');

logger.logInfo('Server started successfully.');
logger.logError('Failed to connect to the database.');
```

#### 3. Run it in terminal:

```bash
node app.js
```

**Output:**

```
INFO: Server started successfully.
ERROR: Failed to connect to the database.
```

---

### How It Works

* `logger.js` defines some helper functions.
* `module.exports` exposes those functions.
* `require('./logger')` loads that file and returns the exported object.

---

### Memory Tip

Think of each `.js` file like a **toolbox**:

* You pack tools (`functions`) using `module.exports`.
* You open that toolbox using `require()`.

> "Pack with `module.exports`, unpack with `require()`."

---

Next options:

1. Want to see how Node.js **caches** modules when required?
2. Or want to start working with **built-in modules** like `fs` or `path`?


### Node.js Module Caching (How `require()` Caches Modules)

In Node.js, when you use `require()` to load a module, **it gets loaded and executed only once**. After that, it is **cached in memory**. Any future `require()` calls for the same file will return the **same instance** (not re-executed).

---

### Example: Demonstrating Module Caching

#### 1. Create `logger.js`

```js
console.log('Logger module loaded');

let count = 0;

function log(message) {
  count++;
  console.log(`(${count}) ${message}`);
}

module.exports = log;
```

#### 2. Create `app.js`

```js
const log1 = require('./logger');
log1('First call');

const log2 = require('./logger');
log2('Second call');

console.log('Are log1 and log2 the same?', log1 === log2);
```

#### Run:

```bash
node app.js
```

#### Output:

```
Logger module loaded
(1) First call
(2) Second call
Are log1 and log2 the same? true
```

### What You See:

* The `console.log('Logger module loaded')` runs **only once**.
* Even though `require('./logger')` is called twice, Node returns the **cached instance** the second time.
* Both `log1` and `log2` point to the **same function** object in memory.

---

### Why Caching Matters

* **Improves performance** by avoiding repeated file loading.
* Ensures **shared state** when modules hold data (like config or DB connections).
* But be careful: if your module holds mutable state, all imports share it.

---

### Memory Trick

> Think of `require()` like **checking out a library book**:

* The first person gets the full copy.
* Everyone else just reads the same copy from the reading room (cached).

---

Node.js comes with many **built-in modules** that provide core functionality without needing to install anything. These are sometimes called **core modules**.

---

### Common Built-in Modules

| Module   | Purpose                              | Example Use                   |
| -------- | ------------------------------------ | ----------------------------- |
| `fs`     | File system operations               | Read/write files              |
| `path`   | Handle file and directory paths      | Normalize paths               |
| `http`   | Create web servers                   | Build HTTP server             |
| `os`     | Get operating system info            | Platform, memory, CPUs        |
| `url`    | Parse and format URLs                | Break URL into components     |
| `events` | Work with events and event listeners | Build event-driven logic      |
| `crypto` | Encryption, hashing                  | Password hashing, tokens      |
| `util`   | Utility functions                    | Promisify callbacks, debug    |
| `stream` | Handle streaming data                | Reading big files efficiently |

---

### Example: Using `fs` and `path`

#### 1. Using `fs` (File System)

```js
const fs = require('fs');

// Write to a file
fs.writeFileSync('example.txt', 'Hello from Node.js');

// Read from a file
const content = fs.readFileSync('example.txt', 'utf8');
console.log(content);  // Output: Hello from Node.js
```

#### 2. Using `path`

```js
const path = require('path');

console.log('Filename:', __filename);
console.log('Dirname:', __dirname);
console.log('Base name:', path.basename(__filename)); // e.g., 'app.js'
console.log('Extension:', path.extname(__filename));  // e.g., '.js'
```

---

### Memory Trick

> Use this phrase: "**FHP OPEC U**"
> F = `fs`
> H = `http`
> P = `path`
> O = `os`
> P = `process`
> E = `events`
> C = `crypto`
> U = `url`

"**FHP OPEC U** runs the core of Node."

---

Want to dive into a hands-on file system project (like a file logger or JSON DB), or start building a web server using the `http` module?


Let’s build two hands-on projects using Node.js built-in modules:

1. **File Logger** – logs messages to a file.
2. **JSON Database** – a simple system that reads/writes JSON data as a tiny database.

---

### 1. File Logger

#### Goal: Save logs to a file like `log.txt`

#### File: `logger.js`

```js
const fs = require('fs');
const path = require('path');

const logFilePath = path.join(__dirname, 'log.txt');

function log(message) {
  const timestamp = new Date().toISOString();
  const logEntry = `[${timestamp}] ${message}\n`;

  fs.appendFileSync(logFilePath, logEntry, 'utf8');
}

module.exports = log;
```

#### File: `app.js`

```js
const log = require('./logger');

log('Server started...');
log('User logged in');
log('Database connected');
```

#### Run:

```bash
node app.js
```

#### Output in `log.txt`:

```
[2025-05-08T14:00:01.000Z] Server started...
[2025-05-08T14:00:02.000Z] User logged in
[2025-05-08T14:00:03.000Z] Database connected
```

---

### 2. JSON Database

#### Goal: Add and retrieve users from a `db.json` file.

#### File: `db.js`

```js
const fs = require('fs');
const path = require('path');

const dbPath = path.join(__dirname, 'db.json');

// Initialize db if it doesn't exist
if (!fs.existsSync(dbPath)) {
  fs.writeFileSync(dbPath, JSON.stringify({ users: [] }, null, 2));
}

function getUsers() {
  const data = fs.readFileSync(dbPath, 'utf8');
  return JSON.parse(data).users;
}

function addUser(user) {
  const data = JSON.parse(fs.readFileSync(dbPath, 'utf8'));
  data.users.push(user);
  fs.writeFileSync(dbPath, JSON.stringify(data, null, 2));
}

module.exports = { getUsers, addUser };
```

#### File: `dbTest.js`

```js
const db = require('./db');

db.addUser({ id: 1, name: 'Rahman' });
db.addUser({ id: 2, name: 'Ali' });

console.log('Users:', db.getUsers());
```

#### Run:

```bash
node dbTest.js
```

#### Output in `db.json`:

```json
{
  "users": [
    { "id": 1, "name": "Rahman" },
    { "id": 2, "name": "Ali" }
  ]
}
```

---

### Memory Trick

> "Logs to `.txt`, data to `.json` — txt for talk, json for structure."

---

Let’s build a simple **CLI (Command Line Interface) app** in Node.js using `process.argv` and `process.env`.

---

### CLI App Goal: A Simple Note Manager

The app will support:

* Adding a note: `node notes.js add "Buy milk"`
* Listing notes: `node notes.js list`

---

### Step-by-Step

#### 1. Create a file named `notes.js`

```js
const fs = require('fs');
const path = require('path');

const filePath = path.join(__dirname, 'notes.json');

// Initialize file if not present
if (!fs.existsSync(filePath)) {
  fs.writeFileSync(filePath, JSON.stringify([]));
}

const command = process.argv[2];
const message = process.argv[3];

if (command === 'add') {
  const notes = JSON.parse(fs.readFileSync(filePath, 'utf8'));
  notes.push({ text: message, createdAt: new Date().toISOString() });
  fs.writeFileSync(filePath, JSON.stringify(notes, null, 2));
  console.log('Note added:', message);
} else if (command === 'list') {
  const notes = JSON.parse(fs.readFileSync(filePath, 'utf8'));
  console.log('\nYour Notes:\n');
  notes.forEach((note, index) => {
    console.log(`${index + 1}. ${note.text} (${note.createdAt})`);
  });
} else {
  console.log('Usage:\n  node notes.js add "Your note"\n  node notes.js list');
}
```

---

### How to Run It

#### Add a note:

```bash
node notes.js add "Call mom"
```

#### Add another note:

```bash
node notes.js add "Finish Node.js lesson"
```

#### List all notes:

```bash
node notes.js list
```

#### Output:

```
Your Notes:

1. Call mom (2025-05-08T14:10:21.000Z)
2. Finish Node.js lesson (2025-05-08T14:12:04.000Z)
```

---

### Optional: Use `process.env` for Config

Set a custom file location using an environment variable:

```bash
NOTE_FILE=custom-notes.json node notes.js add "This goes in custom file"
```

Update the script to use `process.env.NOTE_FILE`:

```js
const filePath = path.join(__dirname, process.env.NOTE_FILE || 'notes.json');
```

---

### `os` Module – Access Operating System Info in Node.js

The `os` module in Node.js provides **system-level information** such as CPU details, memory, platform, architecture, and more. It's useful for writing cross-platform tools or diagnostics.

You don’t need to install anything—just:

```js
const os = require('os');
```

---

### Common `os` Methods

| Method          | What It Does                          | Example Output                   |
| --------------- | ------------------------------------- | -------------------------------- |
| `os.platform()` | Returns the OS platform               | `'linux'`, `'win32'`, `'darwin'` |
| `os.arch()`     | Returns CPU architecture              | `'x64'`, `'arm'`                 |
| `os.cpus()`     | Info about each CPU core              | Array with model, speed, times   |
| `os.totalmem()` | Total memory in bytes                 | `8589934592` (8 GB)              |
| `os.freemem()`  | Free memory in bytes                  | `524288000`                      |
| `os.uptime()`   | System uptime in seconds              | `35200`                          |
| `os.hostname()` | Hostname of the system                | `'rahman-laptop'`                |
| `os.type()`     | OS name (`Linux`, `Windows_NT`, etc.) | `'Linux'`                        |
| `os.userInfo()` | Info about current user               | `{ username, homedir, shell }`   |
| `os.homedir()`  | User’s home directory                 | `'/home/rahman'`                 |

---

### Example: `os-info.js`

```js
const os = require('os');

console.log('Platform:', os.platform());
console.log('Architecture:', os.arch());
console.log('CPU Cores:', os.cpus().length);
console.log('Total Memory:', (os.totalmem() / 1024 / 1024).toFixed(2), 'MB');
console.log('Free Memory:', (os.freemem() / 1024 / 1024).toFixed(2), 'MB');
console.log('Uptime:', (os.uptime() / 60).toFixed(2), 'minutes');
console.log('User Info:', os.userInfo());
console.log('Home Directory:', os.homedir());
```

#### Run:

```bash
node os-info.js
```

---
### `path` Module – Handle File and Directory Paths in Node.js

The `path` module helps you **work with file and directory paths** in a way that's safe and works on all operating systems (Windows, Linux, macOS). It prevents bugs from using slashes (`/`, `\`) incorrectly.

```js
const path = require('path');
```

---

### Why Use It?

Because paths differ:

* Linux/Mac: `/home/user/project/file.txt`
* Windows: `C:\Users\user\project\file.txt`

`path` makes it **OS-independent**.

---

### Most Useful Methods

| Method                   | What It Does                        | Example Output                        |
| ------------------------ | ----------------------------------- | ------------------------------------- |
| `path.basename(p)`       | File name from path                 | `'app.js'`                            |
| `path.dirname(p)`        | Folder name of the path             | `'/users/rahman/project'`             |
| `path.extname(p)`        | Extension from file                 | `'.js'`                               |
| `path.join(...parts)`    | Join paths safely                   | `'users/rahman/app.js'`               |
| `path.resolve(...parts)` | Resolve absolute path               | `'/Users/rahman/app.js'`              |
| `path.isAbsolute(p)`     | Checks if path is absolute          | `true/false`                          |
| `path.sep`               | Returns path separator (`/` or `\`) | `'/'` on Mac/Linux, `'\\'` on Windows |

---

### Example: `path-demo.js`

```js
const path = require('path');

const filePath = '/users/rahman/project/app.js';

console.log('Base:', path.basename(filePath));     // app.js
console.log('Dir:', path.dirname(filePath));       // /users/rahman/project
console.log('Ext:', path.extname(filePath));       // .js
console.log('Joined:', path.join('users', 'rahman', 'app.js')); // users/rahman/app.js
console.log('Resolved:', path.resolve('app.js'));  // /absolute/path/to/app.js
console.log('Is absolute?', path.isAbsolute(filePath)); // true
console.log('Separator:', path.sep);               // / or \
```

---
### `fs` Module – File System Operations in Node.js

The `fs` (File System) module allows Node.js to **read**, **write**, **delete**, and **manipulate files and folders** on your system. It includes both **synchronous** and **asynchronous** versions of almost every method.

```js
const fs = require('fs');
```

---

### Common `fs` Methods

| Method            | Purpose                      | Sync Version          |
| ----------------- | ---------------------------- | --------------------- |
| `fs.readFile()`   | Read a file asynchronously   | `fs.readFileSync()`   |
| `fs.writeFile()`  | Write/overwrite a file       | `fs.writeFileSync()`  |
| `fs.appendFile()` | Add content to a file        | `fs.appendFileSync()` |
| `fs.existsSync()` | Check if a file exists       | —                     |
| `fs.unlink()`     | Delete a file                | `fs.unlinkSync()`     |
| `fs.mkdir()`      | Create a new folder          | `fs.mkdirSync()`      |
| `fs.readdir()`    | Read contents of a folder    | `fs.readdirSync()`    |
| `fs.stat()`       | Get info about a file/folder | `fs.statSync()`       |

---

### Examples

#### 1. Write and Read a File

```js
const fs = require('fs');

// Write to a file
fs.writeFileSync('note.txt', 'Hello, FS Module!');

// Read the file
const content = fs.readFileSync('note.txt', 'utf8');
console.log(content);  // Hello, FS Module!
```

---

#### 2. Append to a File

```js
fs.appendFileSync('note.txt', '\nAppended line.');
```

---

#### 3. Delete a File

```js
fs.unlinkSync('note.txt');
```

---

#### 4. Create and Read a Folder

```js
fs.mkdirSync('my-folder');
fs.writeFileSync('my-folder/info.txt', 'Inside the folder');
const files = fs.readdirSync('my-folder');
console.log(files); // ['info.txt']
```

---

#### 5. File/Folder Info

```js
const stats = fs.statSync('my-folder/info.txt');
console.log('Is file?', stats.isFile());
console.log('Size in bytes:', stats.size);
```

---
### `fs` Module – Sync vs Async in Node.js

Node.js gives you **two styles** of working with files using the `fs` module:

* **Synchronous (Sync)**: Blocking – waits for the task to finish before moving on.
* **Asynchronous (Async)**: Non-blocking – starts the task, continues with other code, and uses a callback when done.

---

### When to Use

| Style | Use Case                                 | Pros                    | Cons                                |
| ----- | ---------------------------------------- | ----------------------- | ----------------------------------- |
| Sync  | Small scripts, quick file ops            | Simpler code            | Blocks the event loop               |
| Async | Servers, APIs, anything production-grade | Non-blocking, efficient | Requires callbacks or `async/await` |

---

### Example: Writing a File

#### Synchronous (Blocking)

```js
const fs = require('fs');

fs.writeFileSync('sync.txt', 'This is sync');
console.log('Done writing (sync)');
```

**Output order:**

```
Done writing (sync)
```

#### Asynchronous (Non-Blocking)

```js
const fs = require('fs');

fs.writeFile('async.txt', 'This is async', (err) => {
  if (err) throw err;
  console.log('Done writing (async)');
});

console.log('This runs *before* file write finishes');
```

**Output order:**

```
This runs *before* file write finishes  
Done writing (async)
```

---

### Example: Reading a File

#### Sync

```js
const data = fs.readFileSync('sync.txt', 'utf8');
console.log('Read (sync):', data);
```

#### Async

```js
fs.readFile('async.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log('Read (async):', data);
});
```

---

### Real-world Analogy

Imagine you’re a chef:

* **Sync**: You cook one dish, wait until it's done, then start the next one. Guests wait.
* **Async**: You start cooking a dish, then prep another while the first is baking. Much faster overall.

---

### Why Node.js Is a Non-Blocking Language (Event Loop & I/O Explained Simply)

Node.js is **non-blocking** by design, which means it can handle **many tasks at once** without waiting for one to finish before starting another. This is made possible by two things:

---

### 1. **Event Loop**

The **event loop** is a system inside Node.js that manages asynchronous operations like file reads, network requests, and timers.

#### How It Works (Simplified):

1. You ask Node to read a file.
2. Node **starts** the file read and immediately continues to run other code.
3. Once the file is ready, Node **"loops back"** and executes the callback you gave for it.

---

### 2. **Non-Blocking I/O**

In most programming languages, reading a file or making a network request **blocks** the entire thread—everything else waits.

But in Node.js:

* **I/O operations** like reading files, querying databases, or talking to APIs **don’t block** the main thread.
* They’re **delegated** to the OS or a background thread.
* Node listens for completion and then runs the associated callback.

---

### Why Is Node.js Non-Blocking?

1. **Single-threaded architecture**

   * Node runs on a **single thread** using JavaScript.
   * Blocking operations would freeze everything—so non-blocking is necessary.

2. **Built for scalability**

   * Perfect for I/O-heavy applications (APIs, web servers, real-time apps).
   * Can handle **thousands of requests** without creating a thread per user (unlike PHP or Java).

---

### Real-World Analogy

**Restaurant model**:

* A blocking language is like a waiter taking one order and waiting in the kitchen.
* Node.js is like a waiter who takes your order, passes it to the kitchen, and immediately goes to serve the next table. When your food is ready, they bring it to you.

---

### Code Example

#### Non-blocking (Node-style)

```js
const fs = require('fs');

fs.readFile('bigfile.txt', 'utf8', (err, data) => {
  console.log('File read complete');
});

console.log('Doing other stuff while file reads...');
```

**Output:**

```
Doing other stuff while file reads...
File read complete
```

#### Blocking (Sync)

```js
const fs = require('fs');

const data = fs.readFileSync('bigfile.txt', 'utf8');
console.log('File read complete');
```

**Output:**

```
(waits until file is read)
File read complete
```

---

### What Does "Node.js Is Single-Threaded" Mean?

When we say **Node.js is single-threaded**, we mean:

> Node.js runs all **JavaScript code** on a **single thread** — one operation at a time, in a single sequence.

---

### What Is a Thread?

A **thread** is a unit of execution.

* Most traditional languages like Java or C# use **multi-threading**: they create **one thread per task** (e.g., one per user or one per request).
* Node.js uses **just one thread** for running your code, no matter how many users or requests come in.

---

### Then How Does It Handle Many Things at Once?

This is where **non-blocking I/O** and the **event loop** come in.

Node uses:

* **A single main thread** for executing your JavaScript.
* **Background threads** from a pool (called the **libuv thread pool**) for I/O-heavy tasks (like file reads, DB queries).

---

### Internally: Node Is Not Entirely Single-Threaded

* **JavaScript execution** = Single-threaded.
* **I/O operations** = Offloaded to background threads.

So it looks like Node is doing everything at once, but it's doing:

* JavaScript logic in 1 thread.
* Background tasks using system threads (abstracted from you).

---

### Real-World Analogy

* Imagine a chef (the single thread) in a kitchen.
* He prepares meals (JavaScript).
* For time-consuming tasks like boiling rice (I/O), he hands it to a helper (background thread).
* He keeps cooking other meals while the rice boils.

---

### Code Example

```js
const fs = require('fs');

console.log('Start');

// File read is sent to background thread
fs.readFile('file.txt', 'utf8', (err, data) => {
  console.log('File read complete');
});

console.log('End');
```

**Output:**

```
Start
End
File read complete
```

> Node didn't wait for the file—it moved on because the main thread stayed free.

---

### Memory Trick

> "**Single-threaded brain, multi-threaded hands.**"

Think of Node like a **brain** that can **delegate tasks to helpers**, but can only **think about one thing at a time**.

---
