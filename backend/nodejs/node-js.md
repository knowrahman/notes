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

### Object-Oriented Programming (OOP) Concepts in Node.js – Full Guide

Node.js uses JavaScript, which supports OOP through **prototypes**, **classes**, and **objects**. These OOP concepts are used in both your code and internally by Node itself (like in `EventEmitter`, `fs`, etc.).

Here’s a full breakdown of **core OOP principles** and how to use them in Node.js:

---

## 1. **Class & Object**

* **Class**: Blueprint for creating objects (introduced in ES6)
* **Object**: An instance of a class

### Example:

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hello, I'm ${this.name}`;
  }
}

const rahman = new Person('Rahman');
console.log(rahman.greet()); // Hello, I'm Rahman
```

---

## 2. **Encapsulation**

* Bundles **data (properties)** and **methods (functions)** inside a class.
* Protects internal state by restricting direct access.

### Example with Private Fields:

```js
class BankAccount {
  #balance = 0;

  deposit(amount) {
    this.#balance += amount;
  }

  getBalance() {
    return this.#balance;
  }
}

const acc = new BankAccount();
acc.deposit(100);
console.log(acc.getBalance()); // 100
// console.log(acc.#balance); // Error: private field
```

---

## 3. **Abstraction**

* Hide complex logic behind simple methods
* Show only what’s necessary to the outside world

### Example:

```js
class Vehicle {
  start() {
    this._checkBattery();
    console.log('Vehicle started');
  }

  _checkBattery() {
    console.log('Checking battery...');
  }
}

const v = new Vehicle();
v.start();
// Output: Checking battery... Vehicle started
```

---

## 4. **Inheritance**

* One class inherits features (methods/properties) from another
* Promotes code reuse

### Example:

```js
class Animal {
  speak() {
    return 'Makes sound';
  }
}

class Dog extends Animal {
  speak() {
    return 'Barks';
  }
}

const dog = new Dog();
console.log(dog.speak()); // Barks
```

---

## 5. **Polymorphism**

* Same method name behaves differently in different classes
* Achieved by **overriding** methods

### Example:

```js
class Shape {
  area() {
    return 0;
  }
}

class Square extends Shape {
  constructor(side) {
    super();
    this.side = side;
  }

  area() {
    return this.side * this.side;
  }
}

const s = new Square(4);
console.log(s.area()); // 16
```

---

## 6. **Static Methods and Properties**

* Belong to the class, not instances
* Used for utility or factory functions

### Example:

```js
class MathHelper {
  static square(n) {
    return n * n;
  }
}

console.log(MathHelper.square(5)); // 25
```

---

## 7. **Getters and Setters**

* Controlled access to object properties

### Example:

```js
class User {
  constructor(name) {
    this._name = name;
  }

  get name() {
    return this._name.toUpperCase();
  }

  set name(value) {
    if (value.length < 3) throw new Error('Too short');
    this._name = value;
  }
}

const u = new User('Ali');
console.log(u.name); // ALI
u.name = 'Rahman';
console.log(u.name); // RAHMAN
```

---

## Summary Table

| OOP Concept     | Purpose                                  |
| --------------- | ---------------------------------------- |
| Class & Object  | Create structured entities               |
| Encapsulation   | Hide data, expose behavior               |
| Abstraction     | Hide inner complexity                    |
| Inheritance     | Reuse logic from parent class            |
| Polymorphism    | Override methods with different behavior |
| Static Methods  | Utility functions not tied to instance   |
| Getters/Setters | Controlled access to private data        |

---

### Memory Trick: **"A PIECE" of OOP**

* **A**bstraction
* **P**olymorphism
* **I**nheritance
* **E**ncapsulation
* **C**lasses
* **E**ntry point (object)

---
### Encapsulation in Node.js – All Types Explained

Encapsulation is the OOP principle of **hiding internal implementation details** and **exposing only what’s necessary** to the outside world. This allows better **security**, **modularity**, and **control**.

Node.js (via JavaScript) supports **several ways** to achieve encapsulation.

---

## 1. **Function Scope Encapsulation** (Classic JS Pattern)

Variables declared inside a function are **private** to that function.

### Example:

```js
function Counter() {
  let count = 0; // private variable

  return {
    increment() {
      count++;
    },
    getCount() {
      return count;
    }
  };
}

const c = Counter();
c.increment();
console.log(c.getCount()); // 1
// console.log(c.count); // undefined
```

> Good for module-like behavior without classes.

---

## 2. **ES6 Class with Public Methods (No Privacy)**

Default class fields are **public** in JavaScript.

```js
class User {
  constructor(name) {
    this.name = name; // public
  }

  greet() {
    return `Hello, ${this.name}`;
  }
}
```

> Use this for basic encapsulation and structured code.

---

## 3. **Encapsulation with Getters and Setters**

Use `get` and `set` to expose **controlled access** to internal values.

```js
class Account {
  constructor(balance) {
    this._balance = balance;
  }

  get balance() {
    return this._balance;
  }

  set balance(value) {
    if (value < 0) throw new Error('Invalid balance');
    this._balance = value;
  }
}
```

> Use when you want validation or transformation during access.

---

## 4. **Private Fields with `#` (ES2022)**

The most secure form of encapsulation in modern JavaScript.

```js
class SecureAccount {
  #balance = 0;

  deposit(amount) {
    if (amount > 0) this.#balance += amount;
  }

  getBalance() {
    return this.#balance;
  }
}

const acc = new SecureAccount();
acc.deposit(100);
console.log(acc.getBalance()); // 100
// console.log(acc.#balance); // SyntaxError: Private field
```

> Use for **true privacy**. Not accessible outside the class at all.

---

## 5. **Module Encapsulation (Node.js Module Scope)**

Each file in Node.js has its own **module scope**, which also provides encapsulation.

```js
// db.js
const password = 'secret'; // private
function connect() { console.log('Connected'); }

module.exports = { connect }; // only this is public
```

```js
// app.js
const db = require('./db');
db.connect();
// db.password → undefined
```

> Useful for hiding internal utilities or configs.

---

## Summary of Encapsulation Techniques

| Technique                  | Privacy Level | Use Case                            |
| -------------------------- | ------------- | ----------------------------------- |
| Function scope             | Moderate      | Closures, factory functions         |
| Class with public fields   | None          | Simple OOP structure                |
| Getters/Setters            | Controlled    | Validation, computed values         |
| Private fields (`#`)       | Strong        | Strict encapsulation                |
| Module exports (`require`) | File-based    | Limit what other modules can access |

---

### Memory Trick: Use **"FPGMP"**

* **F**unction scope
* **P**ublic fields
* **G**etters/setters
* **#** (Private fields)
* **M**odule exports

> "Encapsulation in Node.js is a FPGMP system – secure from inside out."

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
### `http` Module in Node.js – Full Breakdown

The `http` module in Node.js allows you to **create servers**, **handle HTTP requests**, and **build web applications** without any third-party frameworks like Express.

It’s one of the **core built-in modules**, so no installation needed:

```js
const http = require('http');
```

---

### What Can You Do With It?

* Build a basic web server
* Handle different types of requests (GET, POST, etc.)
* Send HTML, JSON, or files as responses
* Build APIs manually
* Learn the foundation of Express.js

---

## 1. Creating a Basic HTTP Server

```js
const http = require('http');

const server = http.createServer((req, res) => {
  res.write('Hello from Node.js server!');
  res.end(); // Always call end()
});

server.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

#### Breakdown:

* `http.createServer(callback)` creates the server.
* The callback runs every time the server receives a **request**.
* `req` = Incoming request object
* `res` = Response object used to send data back to the client

---

## 2. Handling URL and Method

```js
const server = http.createServer((req, res) => {
  if (req.method === 'GET' && req.url === '/') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Welcome to the homepage!');
  } else if (req.method === 'GET' && req.url === '/about') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('About us page');
  } else {
    res.writeHead(404, { 'Content-Type': 'text/plain' });
    res.end('404 Not Found');
  }
});
```

---

## 3. Sending JSON Responses

```js
if (req.url === '/api' && req.method === 'GET') {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  const data = { message: 'Hello JSON' };
  res.end(JSON.stringify(data));
}
```

---

## 4. Sending HTML Response

```js
res.writeHead(200, { 'Content-Type': 'text/html' });
res.end('<h1>Welcome</h1><p>This is HTML</p>');
```

---

## 5. Reading Data from a POST Request

To read body data from a POST request:

```js
if (req.method === 'POST' && req.url === '/submit') {
  let body = '';

  req.on('data', chunk => {
    body += chunk;
  });

  req.on('end', () => {
    console.log('Received:', body);
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Data received');
  });
}
```

---

## 6. Setting Headers

Use `res.writeHead(statusCode, headersObject)`:

```js
res.writeHead(200, {
  'Content-Type': 'application/json',
  'Custom-Header': 'Rahman'
});
```

---

## 7. Common Request Properties

| Property      | Description                   |
| ------------- | ----------------------------- |
| `req.method`  | HTTP method (GET, POST, etc.) |
| `req.url`     | URL path of request           |
| `req.headers` | Headers sent by client        |

---

## 8. Common Response Methods

| Method            | Description                        |
| ----------------- | ---------------------------------- |
| `res.write()`     | Write response body in chunks      |
| `res.end()`       | Finalize and send the response     |
| `res.writeHead()` | Set status and headers in one call |

---

### Real-World Analogy

Think of the `http` module like a **waiter** at a restaurant:

* The waiter (`server`) listens for customers (`requests`).
* The customer makes a request: `"GET /menu"`.
* The waiter responds with the menu (`res.write`, `res.end`).
* If the customer asks something unknown, the waiter replies: `"404 Not Found"`.

---

### Memory Trick

Use "**RUMBLE**" to remember key features:

* **R**eq & Res
* **U**RL parsing
* **M**ethod checking
* **B**ody parsing (manually for POST)
* **L**isten on a port
* **E**rror and 404 handling

> "Let your HTTP server RUMBLE with features."

---

Would you like to now build a full REST API (CRUD) using only the `http` module, or compare it directly to Express.js to see why Express is more convenient?
### Node.js Packages and npm – Full Guide

---

### What Is a Package in Node.js?

A **package** is a reusable block of code that can be included in your project. It usually:

* Solves a specific problem (e.g., `lodash` for utilities, `axios` for HTTP requests)
* Contains a `package.json` file describing it
* Can be installed via **npm**

A package can be:

* **Local** (in your project)
* **Global** (installed on your machine)
* **Built-in** (like `fs`, `http` – doesn’t need installing)

---

### What Is `npm`?

`npm` = **Node Package Manager**
It's the default tool for:

* Installing packages
* Managing dependencies
* Running scripts
* Publishing your own packages

When you install Node.js, **npm comes with it**.

---

### `package.json` – The Heart of a Node Project

To create it:

```bash
npm init
```

Or auto-fill everything:

```bash
npm init -y
```

This creates a `package.json` file like:

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "lodash": "^4.17.21"
  }
}
```

---

### Installing Packages

#### 1. Local (for your project)

```bash
npm install axios
```

This:

* Adds it to `node_modules/`
* Adds it to `"dependencies"` in `package.json`

#### 2. Global (system-wide)

```bash
npm install -g nodemon
```

---

### Installing Dev Dependencies

For tools only used during development (like test runners):

```bash
npm install --save-dev jest
```

---

### Using Installed Packages

Example: using `axios`

```js
const axios = require('axios');

axios.get('https://api.github.com').then(res => {
  console.log(res.data);
});
```

---

### `node_modules/` and `.gitignore`

* `node_modules/` contains all installed packages
* You should never push it to Git
* Add this to `.gitignore`:

```
node_modules/
```

---

### Useful npm Commands

| Command                      | What It Does                                |
| ---------------------------- | ------------------------------------------- |
| `npm install`                | Installs all dependencies from package.json |
| `npm install package-name`   | Installs a new package                      |
| `npm install -g`             | Installs globally                           |
| `npm uninstall package-name` | Removes a package                           |
| `npm update`                 | Updates packages to latest allowed version  |
| `npm list`                   | Lists installed packages                    |
| `npm run script-name`        | Runs custom scripts (like `npm run start`)  |

---

### Memory Trick

Use **"PICS-DUG"** to remember:

* **P**ackage.json
* **I**nstall
* **C**reate (`npm init`)
* **S**cripts
* **D**ependencies
* **U**ninstall
* **G**lobal install

> "npm helps your Node.js app PICS-DUG deep into productivity."

---

Great question, Rahman. Understanding the difference between `let` and `var` is essential for writing clean and bug-free JavaScript.

Here’s a clear and detailed breakdown:

---

## ✅ `var` vs `let` in JavaScript

| Feature                      | `var`                                  | `let`                                                |
| ---------------------------- | -------------------------------------- | ---------------------------------------------------- |
| **Scope**                    | Function-scoped                        | Block-scoped                                         |
| **Hoisting**                 | Hoisted (initialized with `undefined`) | Hoisted (but not initialized)                        |
| **Redeclaration**            | Can be redeclared in the same scope    | Cannot be redeclared in the same scope               |
| **Global Object Binding**    | Adds to `window` (in browsers)         | Does **not** add to `window`                         |
| **Temporal Dead Zone (TDZ)** | ❌ Not affected                         | ✅ Exists – reference before declaration throws error |
| **Best Practice**            | Avoid using                            | Preferred over `var`                                 |

---

## 🔍 1. Scope

```js
function testVar() {
  if (true) {
    var x = 10;
  }
  console.log(x); // ✅ Works: 10 (function-scoped)
}

function testLet() {
  if (true) {
    let y = 20;
  }
  console.log(y); // ❌ Error: y is not defined (block-scoped)
}
```

---

## 🔍 2. Hoisting

```js
console.log(a); // undefined
var a = 5;

console.log(b); // ❌ ReferenceError: Cannot access 'b' before initialization
let b = 10;
```

* `var` is **hoisted and initialized** to `undefined`
* `let` is **hoisted but not initialized** → temporal dead zone (TDZ)

---

## 🔍 3. Redeclaration

```js
var name = "Rahman";
var name = "Emma"; // ✅ Allowed

let age = 25;
let age = 30; // ❌ SyntaxError: Identifier 'age' has already been declared
```

---

## 🔍 4. Global Binding

```js
var city = "Sydney";
let country = "Australia";

console.log(window.city);    // ✅ "Sydney"
console.log(window.country); // ❌ undefined
```

---

## ✅ Conclusion

* Always prefer `let` (or `const`) over `var`
* `var` behaves inconsistently due to hoisting and function scope
* `let` is **block-scoped**, safer, and more predictable

---

Great question, Rahman! ES6 (also called ECMAScript 2015) introduced **major improvements** to JavaScript, many of which are essential for writing clean, modern code — especially when working with Node.js, React, or AWS Lambdas.

Here’s a categorized list of the **most important ES6 features**, with code examples:

---

## ✅ 1. `let` and `const`

* Block-scoped variable declarations.

```js
let counter = 1;
const PI = 3.14;
```

---

## ✅ 2. Arrow Functions (`=>`)

* Shorter syntax and lexical `this` binding.

```js
const add = (a, b) => a + b;
```

---

## ✅ 3. Template Literals

* Use backticks `` ` `` for multiline strings and string interpolation.

```js
const name = "Rahman";
console.log(`Hello, ${name}!`);
```

---

## ✅ 4. Default Parameters

```js
function greet(name = "Guest") {
  return `Hello, ${name}`;
}
```

---

## ✅ 5. Destructuring (Objects & Arrays)

```js
// Object
const user = { name: "Rahman", age: 25 };
const { name, age } = user;

// Array
const arr = [1, 2, 3];
const [first, second] = arr;
```

---

## ✅ 6. Rest and Spread Operators (`...`)

### Spread (copy/merge objects or arrays):

```js
const arr1 = [1, 2];
const arr2 = [...arr1, 3]; // [1, 2, 3]

const obj1 = { a: 1 };
const obj2 = { ...obj1, b: 2 }; // { a: 1, b: 2 }
```

### Rest (capture remaining arguments):

```js
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b);
}
```

---

## ✅ 7. Enhanced Object Literals

```js
const name = "Emma";
const user = {
  name,
  greet() {
    return `Hi, ${this.name}`;
  }
};
```

---

## ✅ 8. `for...of` Loop

```js
const arr = [10, 20, 30];
for (const num of arr) {
  console.log(num);
}
```

---

## ✅ 9. Promises (Async Programming)

```js
const getData = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve("Done!"), 1000);
  });
};

getData().then(console.log);
```

---

## ✅ 10. Classes

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hi, I'm ${this.name}`;
  }
}
```

---

## ✅ 11. Modules (`import` / `export`)

Used in ESModules (when `"type": "module"` in package.json)

```js
// math.js
export const add = (a, b) => a + b;

// app.js
import { add } from "./math.js";
```

---

## ✅ Bonus: `Map`, `Set`, `WeakMap`, `WeakSet` – New data structures.

---



Want to create and publish your own npm package next? Or explore `npx` and how it helps run packages without installing them globally?
### How to Create and Publish Your Own npm Package (Step-by-Step)

Creating and publishing an npm package is a great way to share reusable code with others — or just modularize your own code. Let’s walk through the full process.

---

### 🔧 1. Setup a New Project

```bash
mkdir my-awesome-package
cd my-awesome-package
npm init
```

Fill out the prompts:

* `name`: must be **unique** on npm (e.g., `rahman-utils`)
* `version`: `1.0.0` is fine
* `main`: your main file (e.g., `index.js`)

Alternatively, skip prompts:

```bash
npm init -y
```

---

### ✍️ 2. Write Your Code

Create `index.js` (or the file you set as `main`):

```js
// index.js
function greet(name) {
  return `Hello, ${name}!`;
}

module.exports = { greet };
```

Now this package exports a simple greeting function.

---

### ✅ 3. Test Your Package Locally (Optional)

Before publishing, test your package in another local project.

1. Link the package:

   ```bash
   npm link
   ```

2. In a separate test project:

   ```bash
   npm link my-awesome-package
   ```

3. Use it:

   ```js
   const { greet } = require('my-awesome-package');
   console.log(greet('Rahman'));  // Hello, Rahman!
   ```

---

### 🗝️ 4. Login to npm

If you don’t have an account yet, sign up at [https://www.npmjs.com/signup](https://www.npmjs.com/signup)

Then:

```bash
npm login
```

Enter your:

* username
* password
* email

---

### 🚀 5. Publish Your Package

```bash
npm publish
```

Done!

Your package is now available at:

```
https://www.npmjs.com/package/<your-package-name>
```

---

### 📦 Updating Your Package

Update your version in `package.json` before publishing again:

* Patch (small fix): `1.0.1`
* Minor (new feature): `1.1.0`
* Major (breaking change): `2.0.0`

Then publish again:

```bash
npm publish
```

---

### 🛑 If You Want to Keep It Private

Add this to `package.json`:

```json
"private": true
```

or publish with a private scope:

```bash
npm publish --access=restricted
```

---
### `package-lock.json` in Node.js – Explained in Detail

---

### What Is `package-lock.json`?

`package-lock.json` is an **automatically generated file** that locks the **exact version** of every installed dependency (and sub-dependency) in your project. It is created when you run:

```bash
npm install
```

It's closely tied to your `package.json`, but they serve different purposes.

---

### Key Differences: `package.json` vs `package-lock.json`

| File                | Purpose                                                      | Editable? |
| ------------------- | ------------------------------------------------------------ | --------- |
| `package.json`      | Declares what packages your project **needs**                | Yes       |
| `package-lock.json` | Records the **exact versions** installed and dependency tree | No (auto) |

---

### Why Is It Important?

1. **Locks dependencies**
   Ensures everyone on your team installs the **same versions**, no surprises.

2. **Deterministic builds**
   `npm install` always installs exactly what's recorded in `package-lock.json`.

3. **Security**
   npm can use it to **audit** your full dependency tree for known vulnerabilities.

4. **Performance**
   Speeds up install times because the dependency tree is already resolved.

---

### Example

If you run:

```bash
npm install lodash
```

* `lodash` appears in your `dependencies` in `package.json`.
* But `package-lock.json` will contain not only `lodash`, but **all sub-packages** it uses, and their exact versions.

---

### Should You Commit `package-lock.json`?

**Yes** — always commit it to version control (e.g., Git), especially in team projects or production apps.

It guarantees:

* Same dependency versions
* Reproducible environments
* Fewer bugs due to version mismatches

---

### What Happens If You Delete It?

If you delete `package-lock.json` and `node_modules/`, and then run:

```bash
npm install
```

npm will re-resolve the versions of all dependencies based on **semver rules** (like `^1.2.3`), which may pull newer versions than before.

---

### Memory Trick

> "`package.json` says what you want, `package-lock.json` locks what you got."

Use the word **"LOCK"**:

* **L**ocks versions
* **O**ptimizes installs
* **C**aptures the full tree
* **K**eeps builds consistent

---
### Event Loop in Node.js – Full Explanation

---

### What Is the Event Loop?

The **event loop** is the core mechanism in Node.js that enables it to perform **non-blocking I/O operations**, even though JavaScript is **single-threaded**.

It allows Node.js to handle **multiple requests** or **tasks concurrently**, without creating new threads for each one.

---

### Why Is It Needed?

JavaScript runs on **one thread** (the main thread). If it waited for slow operations like file reads or HTTP requests, everything would freeze.

The event loop solves this by:

1. Delegating long tasks (I/O, timers) to the system.
2. Continuing to run other code.
3. Executing callback functions when the delegated tasks are done.

---

### The Phases of the Event Loop

The event loop goes through **several phases in a loop**. Each phase has a specific type of task it handles.

#### Phases in Order:

1. **Timers**

   * Executes callbacks scheduled by `setTimeout()` and `setInterval()`.

2. **Pending Callbacks**

   * Executes some system-level operations like TCP errors.

3. **Idle / Prepare**

   * Internal only.

4. **Poll**

   * Retrieves new I/O events.
   * Executes I/O callbacks (file, socket, network, etc.).
   * If no I/O and no timers, it waits.

5. **Check**

   * Executes callbacks from `setImmediate()`.

6. **Close Callbacks**

   * Handles `close` events like socket closing or `process.exit()`.

---

### Visual Flow

```
┌───────────────┐
│   Timers      │ ← setTimeout, setInterval
├───────────────┤
│ Pending Cbs   │ ← System-level tasks
├───────────────┤
│ Idle/Prepare  │ ← Internal only
├───────────────┤
│     Poll      │ ← I/O callbacks
├───────────────┤
│     Check     │ ← setImmediate
├───────────────┤
│ Close Cbs     │ ← Socket/file close
└───────────────┘
```

This cycle repeats **over and over**, executing callbacks when their time or event comes.

---

### Code Example

```js
const fs = require('fs');

setTimeout(() => console.log('setTimeout'), 0);
setImmediate(() => console.log('setImmediate'));

fs.readFile(__filename, () => {
  console.log('File read (I/O)');
  setTimeout(() => console.log('I/O setTimeout'), 0);
  setImmediate(() => console.log('I/O setImmediate'));
});
```

**Output (order may vary slightly):**

```
setTimeout
setImmediate
File read (I/O)
I/O setImmediate
I/O setTimeout
```

Explanation:

* `setTimeout` and `setImmediate` are scheduled.
* File read goes to the I/O phase.
* Inside that I/O callback, new `setTimeout` and `setImmediate` are scheduled.
* I/O `setImmediate` runs before I/O `setTimeout`.

---

### Microtasks: `process.nextTick()` and Promises

These are **not** part of the event loop phases. They run between phases.

```js
console.log('start');

process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('promise'));

console.log('end');
```

**Output:**

```
start
end
nextTick
promise
```

---
### Async Patterns in Node.js – Full Breakdown

Node.js is built for asynchronous, non-blocking code. But you can write async logic in different **patterns**: callback-based, promise-based, or with async/await. Let’s go through each phase, its evolution, and Node's native support.

---

## 1. **Blocking Code (Bad Practice in Node)**

**Blocking code** halts the execution of everything else until it's done. Since Node.js uses a single thread for JavaScript, blocking the thread means nothing else can happen in the meantime.

### Example: Synchronous (blocking) file read

```js
const fs = require('fs');

const data = fs.readFileSync('file.txt', 'utf8');
console.log('File content:', data); // Blocks everything until file is read
```

### Problem:

* If the file is large, it blocks the entire process.
* Bad for servers with multiple users.

---

## 2. **Async Pattern – Promises (Setup Promises)**

**Promises** give a cleaner alternative to callbacks and allow chaining operations.

### Example: Wrapping async code in a Promise

```js
const fs = require('fs');

function readFilePromise(filePath) {
  return new Promise((resolve, reject) => {
    fs.readFile(filePath, 'utf8', (err, data) => {
      if (err) reject(err);
      else resolve(data);
    });
  });
}

readFilePromise('file.txt')
  .then(data => console.log('Promise read:', data))
  .catch(err => console.error('Error:', err));
```

### Advantages:

* Cleaner syntax
* Chainable `.then()` and `.catch()`
* Better error handling than callbacks

---

## 3. **Refactor to Async/Await**

**Async/Await** is built on top of Promises. It looks synchronous but runs asynchronously. It’s the most readable and preferred modern style.

### Refactored version:

```js
const fs = require('fs');

function readFilePromise(filePath) {
  return new Promise((resolve, reject) => {
    fs.readFile(filePath, 'utf8', (err, data) => {
      if (err) reject(err);
      else resolve(data);
    });
  });
}

async function run() {
  try {
    const data = await readFilePromise('file.txt');
    console.log('Async/Await read:', data);
  } catch (err) {
    console.error('Error:', err);
  }
}

run();
```

### Benefits:

* Synchronous look-and-feel
* Easy to write and debug
* No `.then()` chains

---

## 4. **Node's Native Option – fs.promises & util.promisify**

Node.js has started providing native Promise-based APIs, so you don’t have to wrap things manually.

### Option A: `fs.promises` (since Node v10)

```js
const fs = require('fs/promises');

async function run() {
  try {
    const data = await fs.readFile('file.txt', 'utf8');
    console.log('Native fs.promises read:', data);
  } catch (err) {
    console.error('Error:', err);
  }
}

run();
```

### Option B: `util.promisify()` to convert callback-based functions

```js
const fs = require('fs');
const util = require('util');

const readFileAsync = util.promisify(fs.readFile);

async function run() {
  const data = await readFileAsync('file.txt', 'utf8');
  console.log('Promisified read:', data);
}

run();
```

---

### Summary of Async Patterns

| Pattern             | Syntax Style        | Blocking? | Readability | Recommendation          |
| ------------------- | ------------------- | --------- | ----------- | ----------------------- |
| Blocking            | Sync                | Yes       | Simple      | Not for servers         |
| Callbacks           | Nested callbacks    | No        | Hard        | Use only if necessary   |
| Promises            | `.then().catch()`   | No        | Medium      | Good for chaining       |
| Async/Await         | `await` in `async`  | No        | Best        | Recommended             |
| Native Promises API | Built-in in modules | No        | Best        | Always use if available |

---
### Promises and Async/Await in Node.js – Complete Guide

---

## What Is a Promise?

A **Promise** in JavaScript represents a **value that may be available now, or in the future, or never**. It's an object used for **asynchronous operations** like reading a file, making an API call, or querying a database.

---

### States of a Promise

| State         | Description                           |
| ------------- | ------------------------------------- |
| **Pending**   | Initial state, operation not complete |
| **Fulfilled** | Operation completed successfully      |
| **Rejected**  | Operation failed                      |

---

### Creating a Promise

```js
const myPromise = new Promise((resolve, reject) => {
  const success = true;

  if (success) {
    resolve('Task completed');
  } else {
    reject('Something went wrong');
  }
});
```

---

### Consuming a Promise

```js
myPromise
  .then(result => console.log('Success:', result))  // if resolved
  .catch(error => console.log('Error:', error));    // if rejected
```

---

### Real-World Analogy

Imagine you're ordering pizza:

* **Promise created** when you place the order.
* It is **pending** while you're waiting.
* It is **fulfilled** when pizza arrives.
* It is **rejected** if the shop cancels.

---

### Promise Chaining

You can chain `.then()` to perform a sequence of asynchronous operations:

```js
getUser()
  .then(user => getPosts(user.id))
  .then(posts => getComments(posts[0].id))
  .then(comments => console.log(comments))
  .catch(err => console.error(err));
```

---

### Common Pitfall: Callback Hell vs Promise Chain

**Callback Hell** (nested and hard to read):

```js
doTask1(function(result1) {
  doTask2(result1, function(result2) {
    doTask3(result2, function(result3) {
      console.log(result3);
    });
  });
});
```

**Promise Chain** (clean and flat):

```js
doTask1()
  .then(doTask2)
  .then(doTask3)
  .then(console.log);
```

---

## Async/Await – A Cleaner Way to Work With Promises

`async` and `await` are modern JavaScript keywords introduced in ES2017 to work with Promises more elegantly.

### `async` Keyword

Defines a function that **always returns a Promise**.

```js
async function greet() {
  return 'Hello';
}

greet().then(console.log);  // Output: Hello
```

### `await` Keyword

Used **inside async functions only** to wait for a Promise to resolve.

```js
async function main() {
  const result = await myPromise;
  console.log(result);
}
```

### Full Example with Async/Await

```js
const fs = require('fs/promises');

async function readFile() {
  try {
    const data = await fs.readFile('file.txt', 'utf8');
    console.log('File content:', data);
  } catch (error) {
    console.error('Error reading file:', error);
  }
}

readFile();
```

---

### Error Handling with Async/Await

Use `try...catch` blocks to handle rejections:

```js
async function run() {
  try {
    const data = await readData();
    console.log(data);
  } catch (err) {
    console.error('Caught error:', err);
  }
}
```

---

### When to Use What

| Use Case                  | Preferred Tool          |
| ------------------------- | ----------------------- |
| Simple one-off async task | Promise                 |
| Multiple async steps      | Async/Await             |
| Complex control flows     | Async/Await             |
| Top-level script logic    | Async IIFE or `.then()` |

---

### Memory Trick

Use **"PROSE"** for Promises:

* **P**ending
* **R**esolved
* **O**peration
* **S**uccess
* **E**rror

Use **"ACT"** for Async/Await:

* **A**sync keyword
* **C**ontrol with await
* **T**ry/catch for errors

> "Write PROSE for your async code, and ACT clearly with await."

---### Events and EventEmitter in Node.js – Full Explanation

Node.js is built around the idea of **event-driven programming**. It uses the **EventEmitter** class to manage and respond to events across many of its core modules, like `http`, `fs`, and `stream`.

---

## 1. **Events Info (What Are Events in Node.js?)**

In Node.js, an **event** is a signal that something has happened. For example:

* A file finished reading
* A new user connected
* A timer expired
* A request was received

Node uses an internal system called the **Event Loop** to handle these signals efficiently.

---

## 2. **EventEmitter – What It Is**

The `EventEmitter` is a class from the built-in `events` module. It allows you to:

* **Emit events** (i.e., signal something happened)
* **Listen for events** (using `.on()` or `.once()`)

This forms the **publish-subscribe pattern**:

* One part of the code *publishes* an event.
* Other parts *subscribe* and respond.

### Importing EventEmitter:

```js
const EventEmitter = require('events');
```

---

## 3. **EventEmitter – Basic Code Example**

```js
const EventEmitter = require('events');
const emitter = new EventEmitter();

// Listen for a custom event
emitter.on('greet', (name) => {
  console.log(`Hello, ${name}!`);
});

// Emit the event
emitter.emit('greet', 'Rahman');
```

**Output:**

```
Hello, Rahman!
```

### Explanation:

* `emitter.on()` subscribes to the `greet` event.
* `emitter.emit()` triggers the event and passes the argument `'Rahman'`.

---

## 4. **EventEmitter – Additional Info & API Methods**

### Common Methods:

| Method              | Description                       |
| ------------------- | --------------------------------- |
| `on(event, fn)`     | Listen to an event                |
| `once(event, fn)`   | Listen once; remove after trigger |
| `emit(event, args)` | Emit an event                     |
| `removeListener()`  | Manually remove listener          |
| `listenerCount()`   | Count active listeners            |

### Example: `.once()` only fires once

```js
emitter.once('data', () => console.log('Fired once'));
emitter.emit('data');  // Fired
emitter.emit('data');  // Not fired
```

### Example: Remove listener

```js
function logMsg() {
  console.log('Log this');
}
emitter.on('msg', logMsg);
emitter.removeListener('msg', logMsg);
emitter.emit('msg'); // Nothing happens
```

---

## 5. **EventEmitter – `http` Module Example**

The `http` module in Node.js is built using `EventEmitter` internally.

### Example: Listening to HTTP events

```js
const http = require('http');

const server = http.createServer((req, res) => {
  res.end('Hello from server');
});

// Attach event listeners
server.on('connection', (socket) => {
  console.log('New connection established');
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### What’s happening:

* When a new client connects, the server **emits a `connection` event**.
* You can listen to this event using `server.on('connection', ...)`.

---

### Real-World Analogy

Imagine a **radio station**:

* The station **broadcasts (emits)** an event ("new song").
* Radios **tuned in (on)** hear and react.

---

### Memory Trick

Use the acronym **"OER"** for EventEmitter:

* **O** = `.on()` to listen
* **E** = `.emit()` to trigger
* **R** = `.removeListener()` to clean up

> "With OER, you Own Every Response."

---


Let's build a **mini chat engine using EventEmitters** to simulate how multiple users can send and receive messages in a simple way—all within Node.js (no networking needed yet).

---

## Mini Chat Engine with EventEmitters

This will help you understand:

* How events can simulate message broadcasts
* How different parts of your app can react to events independently

---

### Step 1: Create a ChatManager

```js
// chat.js
const EventEmitter = require('events');

class ChatManager extends EventEmitter {
  sendMessage(user, message) {
    this.emit('message', { user, message });
  }

  join(user) {
    this.emit('join', user);
  }

  leave(user) {
    this.emit('leave', user);
  }
}

module.exports = ChatManager;
```

---

### Step 2: Use the ChatManager

```js
// app.js
const ChatManager = require('./chat');

const chat = new ChatManager();

// Listen for new messages
chat.on('message', ({ user, message }) => {
  console.log(`[${user}]: ${message}`);
});

// Listen for users joining
chat.on('join', (user) => {
  console.log(`${user} has joined the chat.`);
});

// Listen for users leaving
chat.on('leave', (user) => {
  console.log(`${user} has left the chat.`);
});

// Simulate a chat
chat.join('Rahman');
chat.sendMessage('Rahman', 'Hello, everyone!');
chat.join('Ali');
chat.sendMessage('Ali', 'Hey Rahman!');
chat.leave('Rahman');
```

---

### Output:

```
Rahman has joined the chat.
[Rahman]: Hello, everyone!
Ali has joined the chat.
[Ali]: Hey Rahman!
Rahman has left the chat.
```

---

### Explanation:

* `ChatManager` extends `EventEmitter`, giving it `.on()` and `.emit()` methods.
* We listen for custom events like `message`, `join`, and `leave`.
* When those events are emitted, the listeners respond in real-time.

---

### Real-World Use Cases

* Chat servers (WebSocket based)
* Notification systems
* Pub-sub architectures
* Event-driven microservices

---
### Node.js HTTP System – Complete Guide (Request/Response Cycle + Practical Setup)

This covers everything about building HTTP servers using Node.js without frameworks like Express.

---

## 1. **HTTP Request/Response Cycle**

This is the foundation of how the web works.

### Step-by-Step:

1. **Client** (browser or tool like Postman) sends a **request** to a server.
2. Server **receives the request**.
3. Server **processes it** (reads URL, method, headers, body).
4. Server sends a **response** (HTML, JSON, text, etc.).
5. Client **receives the response** and displays it.

---

## 2. **HTTP Messages**

There are two types of HTTP messages:

| Type     | Sent By | Description                                               |
| -------- | ------- | --------------------------------------------------------- |
| Request  | Client  | Contains method (GET, POST), path, headers, optional body |
| Response | Server  | Contains status code (200, 404), headers, and body        |

---

### HTTP Request Example:

```http
GET /about HTTP/1.1
Host: localhost:3000
User-Agent: Mozilla/5.0
Accept: text/html
```

### HTTP Response Example:

```http
HTTP/1.1 200 OK
Content-Type: text/html

<html><body>Hello!</body></html>
```

---

## 3. **Starter Project – Install & Setup**

Create a folder and initialize npm:

```bash
mkdir http-server-app
cd http-server-app
npm init -y
```

Create `app.js`:

```bash
touch app.js
```

---

## 4. **Starter Overview**

Directory structure:

```
http-server-app/
├── app.js
├── index.html
├── about.html
└── package.json
```

This simple app will:

* Serve HTML pages based on URL path
* Use built-in `http` and `fs` modules

---

## 5. **HTTP Basics – Create a Simple Server**

### app.js

```js
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Welcome to my server');
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000');
});
```

---

## 6. **HTTP Headers**

Headers are key-value pairs sent along with the request or response.

### Example:

```js
res.writeHead(200, {
  'Content-Type': 'text/html',
  'X-Powered-By': 'Node.js'
});
```

Common Response Headers:

* `Content-Type`: MIME type (`text/html`, `application/json`)
* `Set-Cookie`: Set cookies
* `Location`: Used for redirects

Common Request Headers:

* `User-Agent`
* `Accept`
* `Content-Type`
* `Authorization`

---

## 7. **HTTP Request Object**

The `req` object (first param in `createServer`) holds info about the incoming request.

### Important properties:

```js
req.url       // '/about'
req.method    // 'GET'
req.headers   // { host: ..., connection: ... }
```

You can route based on these properties.

---

## 8. **Serving an HTML File**

Create `index.html`:

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head><title>Home</title></head>
<body><h1>Hello from HTML file!</h1></body>
</html>
```

### Update `app.js` to serve the file:

```js
const fs = require('fs');
const path = require('path');
const http = require('http');

const server = http.createServer((req, res) => {
  if (req.url === '/') {
    const filePath = path.join(__dirname, 'index.html');
    fs.readFile(filePath, (err, content) => {
      if (err) {
        res.writeHead(500);
        res.end('Server Error');
      } else {
        res.writeHead(200, { 'Content-Type': 'text/html' });
        res.end(content);
      }
    });
  } else {
    res.writeHead(404);
    res.end('Not Found');
  }
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000');
});
```

---

## 9. **Full App Example with Routing**

Create `about.html`:

```html
<!DOCTYPE html>
<html>
<head><title>About</title></head>
<body><h1>About Page</h1></body>
</html>
```

### Final `app.js`:

```js
const http = require('http');
const fs = require('fs');
const path = require('path');

const server = http.createServer((req, res) => {
  const fileMap = {
    '/': 'index.html',
    '/about': 'about.html'
  };

  const file = fileMap[req.url];

  if (file) {
    const filePath = path.join(__dirname, file);
    fs.readFile(filePath, (err, content) => {
      if (err) {
        res.writeHead(500);
        res.end('Server Error');
      } else {
        res.writeHead(200, { 'Content-Type': 'text/html' });
        res.end(content);
      }
    });
  } else {
    res.writeHead(404, { 'Content-Type': 'text/plain' });
    res.end('404 Not Found');
  }
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000');
});
```

---

### Summary

| Concept           | Description                                     |
| ----------------- | ----------------------------------------------- |
| Request           | Data sent from client to server                 |
| Response          | Data sent from server to client                 |
| Headers           | Meta info in request/response                   |
| Routing           | Use `req.url` to decide what to respond with    |
| HTML file serving | Use `fs.readFile()` to send HTML to the browser |

---

### Express vs HTTP Module in Node.js – Complete Comparison

---

## What Is Express?

**Express** is a **minimal and flexible web framework** built on top of Node.js’s native `http` module. It simplifies server-side logic like routing, middleware, request parsing, and response handling.

---

## Why Use Express?

The built-in `http` module is **low-level** — you have to manually handle:

* Routing
* Parsing URLs and query strings
* Handling JSON/form data
* Setting headers
* Error handling

**Express** abstracts all of this and gives you clean, concise APIs to work with.

---

## Code Comparison – `http` vs `express`

### 1. Basic Server

#### With `http` module:

```js
const http = require('http');

http.createServer((req, res) => {
  if (req.url === '/' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Welcome');
  }
}).listen(3000);
```

#### With `express`:

```js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Welcome');
});

app.listen(3000);
```

---

### 2. Parsing JSON or Form Data

#### In `http` (manual parsing required):

```js
let body = '';
req.on('data', chunk => (body += chunk));
req.on('end', () => {
  const data = JSON.parse(body); // if JSON
});
```

#### In `express` (built-in middleware):

```js
app.use(express.json());
app.post('/data', (req, res) => {
  console.log(req.body);  // auto-parsed
  res.send('Received');
});
```

---

## Feature-by-Feature Comparison

| Feature             | `http` Module                   | `express` Framework                     |
| ------------------- | ------------------------------- | --------------------------------------- |
| Routing             | Manual                          | Built-in (`app.get`, `app.post`, etc.)  |
| JSON body parsing   | Manual (event-based)            | Built-in with `express.json()`          |
| Middleware support  | None (must be written manually) | Full support for third-party middleware |
| Static file serving | Manual (using `fs`)             | `express.static()` built-in             |
| Template engines    | Not supported directly          | Supported (EJS, Pug, etc.)              |
| URL/Query parsing   | Manual                          | Auto-parsed via `req.query`             |
| Error handling      | Manual                          | Centralized error middleware            |
| Learning curve      | Low-level, verbose              | Higher-level, readable                  |
| Use in real apps    | Not practical alone             | Industry standard                       |

---

## Summary

| `http`                 | `express`               |
| ---------------------- | ----------------------- |
| Low-level API          | High-level abstraction  |
| Manual routing         | Built-in routing        |
| Verbose and repetitive | Clean and readable code |
| Good for learning      | Great for production    |

---
### Express.js Basics and Practical Guide

---

## 1. **Express Basics**

Express is a **web framework for Node.js** that simplifies:

* Routing
* Middleware handling
* Serving static files
* Building APIs and SSR apps

### Installation

```bash
npm install express
```

### Basic Setup (`app.js`)

```js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Welcome to Express!');
});

app.listen(3000, () => {
  console.log('Server is running on http://localhost:3000');
});
```

---

## 2. **Express – App Example with Multiple Routes**

```js
app.get('/', (req, res) => {
  res.send('Home Page');
});

app.get('/about', (req, res) => {
  res.send('About Page');
});

app.post('/contact', (req, res) => {
  res.send('Contact form submitted');
});
```

---

## 3. **Express – Serving Static Files**

To serve HTML, CSS, JS, or images from a folder like `public/`:

### Step 1: Create a `public` directory

```
public/
├── index.html
├── style.css
└── image.png
```

### Step 2: Serve it with `express.static`

```js
app.use(express.static('public'));
```

Now you can visit:

* `/index.html`
* `/style.css`
* `/image.png`

---

## 4. **API vs SSR (Server-Side Rendering)**

| Feature   | API                                      | SSR                                             |
| --------- | ---------------------------------------- | ----------------------------------------------- |
| Output    | JSON or data                             | HTML pages                                      |
| Usage     | SPAs, mobile apps, client-side rendering | Full HTML websites rendered from backend        |
| Example   | `res.json({ name: 'Rahman' })`           | `res.send('<h1>Welcome</h1>')` or use templates |
| Rendering | Client handles rendering                 | Server prepares and sends HTML                  |

### Express as API:

```js
app.get('/api/user', (req, res) => {
  res.json({ id: 1, name: 'Rahman' });
});
```

### Express as SSR:

```js
app.get('/', (req, res) => {
  res.send('<h1>Welcome to SSR page</h1>');
});
```

---

## 5. **Params and Query Strings in Express**

### URL Params (`:id`)

Used to capture dynamic values in the URL path.

```js
app.get('/user/:id', (req, res) => {
  res.send(`User ID: ${req.params.id}`);
});
```

**Request:** `/user/123`
**Response:** `User ID: 123`

---

### Query Strings (`?key=value`)

Used to send optional data in the URL.

```js
app.get('/search', (req, res) => {
  const { q } = req.query;
  res.send(`You searched for: ${q}`);
});
```

**Request:** `/search?q=express`
**Response:** `You searched for: express`

---

### Combine Both

```js
app.get('/user/:id/profile', (req, res) => {
  const { id } = req.params;
  const { tab } = req.query;
  res.send(`User ID: ${id}, Tab: ${tab}`);
});
```

**Request:** `/user/42/profile?tab=settings`
**Response:** `User ID: 42, Tab: settings`

---

### When to Use Params vs Query Strings in Express

Understanding **when to use `req.params` vs `req.query`** depends on the nature of the data you're passing in the URL.

---

## 1. **Use `params` for Required, Structured Data in the URL Path**

URL parameters (`req.params`) are part of the **route pattern**. They are:

* **Required**
* Used to **identify a resource** (user, product, post)
* Typically used in **RESTful URLs**

### Example:

```js
app.get('/user/:id', (req, res) => {
  res.send(`User ID: ${req.params.id}`);
});
```

**Request:** `/user/42`
**Result:** `User ID: 42`

> Use when the route **won’t make sense** without it.

---

## 2. **Use `query` for Optional Filters, Sorting, Search, and Flags**

Query strings (`req.query`) are:

* **Optional**
* Used to modify or filter a request
* Come **after a `?` in the URL**
* Support multiple key-value pairs

### Example:

```js
app.get('/search', (req, res) => {
  res.send(`Search query: ${req.query.q}`);
});
```

**Request:** `/search?q=shoes&sort=price`
**Result:** `Search query: shoes`

> Use for **filtering, searching, sorting, pagination**, etc.

---

### Visual Comparison

| Purpose            | Use `params`            | Use `query`                |
| ------------------ | ----------------------- | -------------------------- |
| Resource ID        | `/user/:id` → `/user/1` | Not ideal                  |
| Filtering/search   | Not suitable            | `/products?category=books` |
| Pagination         | Not suitable            | `/posts?page=2&limit=10`   |
| Required data      | Yes                     | No                         |
| Optional modifiers | No                      | Yes                        |

---

### Example: Combine Both

```js
app.get('/product/:id', (req, res) => {
  const { id } = req.params;
  const { review } = req.query;

  res.send(`Product ID: ${id}, Review: ${review}`);
});
```

**Request:** `/product/9?review=true`
**Result:** `Product ID: 9, Review: true`

---

### Memory Trick

Use **"PRQ"**:

* **P**arams → **Path data**
* **R**equired
* **Q**uery → **Optional data**

> "Params for paths, Query for questions."


### Middleware in Express.js – Full Breakdown

Middleware is one of the **most powerful features** of Express. It gives you control over the **request/response lifecycle** and lets you handle tasks like logging, authentication, error handling, etc.

---

## 1. **Middleware – Setup**

### What Is Middleware?

Middleware is a **function** that has access to:

```js
(req, res, next)
```

It can:

* Read and modify `req` and `res`
* End the response (`res.send`, `res.end`, etc.)
* Or pass control to the next middleware using `next()`

---

### Middleware Function Example

```js
const express = require('express');
const app = express();

const logger = (req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next(); // pass control to next middleware
};

app.use(logger);

app.get('/', (req, res) => {
  res.send('Home Page');
});

app.listen(3000);
```

---

## 2. **`app.use()` Explained**

`app.use()` is used to **register middleware**:

* For **every route** by default
* Or for specific **paths** if given

### Example: Apply globally

```js
app.use((req, res, next) => {
  console.log('Global middleware');
  next();
});
```

### Example: Apply on specific path

```js
app.use('/admin', (req, res, next) => {
  console.log('Admin access');
  next();
});
```

> This will only run for routes that start with `/admin`

---

## 3. **Multiple Middleware Functions**

You can use multiple middleware for a single route.

### Chaining with `next()`:

```js
const stepOne = (req, res, next) => {
  console.log('Step One');
  next();
};

const stepTwo = (req, res, next) => {
  console.log('Step Two');
  next();
};

app.get('/process', stepOne, stepTwo, (req, res) => {
  res.send('Done processing');
});
```

**Output when visiting `/process`:**

```
Step One
Step Two
Done processing
```

---

## 4. **Additional Middleware Info**

### Built-in Middleware

| Middleware             | Purpose                                    |
| ---------------------- | ------------------------------------------ |
| `express.json()`       | Parses JSON request bodies                 |
| `express.urlencoded()` | Parses form data (`x-www-form-urlencoded`) |
| `express.static()`     | Serves static files like HTML, CSS         |

### Example:

```js
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static('public'));
```

---

### Third-Party Middleware

Install and use external packages like:

* `morgan`: HTTP logger
* `helmet`: Set secure headers
* `cors`: Enable CORS
* `cookie-parser`: Parse cookies

Example:

```js
const morgan = require('morgan');
app.use(morgan('dev'));
```

---

### Custom Middleware Use Cases

* Logging
* Request validation
* Authentication
* Setting custom headers
* Timing/logging performance
* Error handling

---

### Memory Trick

Use **"LACE"** to remember middleware flow:

* **L**ogger
* **A**uthentication
* **C**ustom logic
* **E**rror handler

> "Wrap your app in LACE for layered control."

---
### Middleware Chaining & Terminal Middleware in Express.js – Complete Guide

---

## What Is Middleware Chaining?

In Express, **multiple middleware functions can be chained** for a single route or globally using `app.use()`. Middleware chaining allows you to:

* Run logic in **layers**
* **Pass control** to the next layer with `next()`
* Optionally **end the chain** by sending a response (terminal middleware)

---

### Middleware Flow

Every middleware function follows the signature:

```js
(req, res, next) => { ... }
```

You must call:

* `next()` → passes control to the next middleware
* `res.send()` or `res.end()` → ends the response and the chain

---

## Example: Chaining Multiple Middleware

```js
const express = require('express');
const app = express();

// First middleware
const stepOne = (req, res, next) => {
  console.log('Step One');
  next(); // pass to stepTwo
};

// Second middleware
const stepTwo = (req, res, next) => {
  console.log('Step Two');
  next(); // pass to final handler
};

// Final handler
const finalHandler = (req, res) => {
  res.send('Response from final handler');
};

app.get('/chain', stepOne, stepTwo, finalHandler);
```

### Output on `/chain`:

```
Step One
Step Two
=> Response from final handler
```

---

## What Is Terminal Middleware?

A **terminal middleware** is the one that **ends the request-response cycle** — typically with:

* `res.send()`
* `res.json()`
* `res.redirect()`
* `res.end()`

Once called, **no further middleware runs**.

---

### Example: Terminal Middleware Short-circuits the Chain

```js
const shortCircuit = (req, res, next) => {
  if (req.query.stop === 'true') {
    return res.send('Stopped early');
  }
  next(); // only runs if stop !== 'true'
};

app.get('/check', shortCircuit, (req, res) => {
  res.send('Finished normally');
});
```

* `/check?stop=true` → responds with `"Stopped early"`
* `/check` → responds with `"Finished normally"`

---

## Global vs Route-Specific Chaining

### Global

```js
app.use(logger);       // applies to all routes
app.use(authMiddleware);
```

### Route-Specific

```js
app.get('/dashboard', authMiddleware, (req, res) => {
  res.send('Dashboard');
});
```

---

## Middleware Chain Rule of Thumb

| Middleware Action               | Result                            |
| ------------------------------- | --------------------------------- |
| Calls `next()`                  | Passes control to next middleware |
| Sends response                  | Ends chain (terminal middleware)  |
| Throws error                    | Goes to error-handling middleware |
| Misses `next()` or `res.send()` | Hangs forever (no response)       |

---
Let’s build a **JWT-based login system with user credentials stored in a JSON file** (simulating a database).

---

## Project Goals

1. **Store users in a JSON file**
2. **Authenticate using username & password**
3. **Generate a JWT token on login**
4. **Protect routes with `authenticate` and `authorize` middleware**

---

## Step 1: Setup Files

Project structure:

```
project/
├── app.js
├── users.json
├── auth.js
└── utils.js
```

---

## Step 2: `users.json` – Simulate a User Database

```json
[
  { "id": 1, "username": "rahman", "password": "1234", "role": "admin" },
  { "id": 2, "username": "ali", "password": "abcd", "role": "user" }
]
```

---

## Step 3: `utils.js` – Load and Find Users

```js
const fs = require('fs');
const path = require('path');

const loadUsers = () => {
  const data = fs.readFileSync(path.join(__dirname, 'users.json'), 'utf8');
  return JSON.parse(data);
};

const findUser = (username, password) => {
  const users = loadUsers();
  return users.find(user => user.username === username && user.password === password);
};

module.exports = { loadUsers, findUser };
```

---

## Step 4: `auth.js` – JWT Auth and Role Middleware

```js
const jwt = require('jsonwebtoken');
const SECRET_KEY = 'secret-key';

const authenticate = (req, res, next) => {
  const token = req.headers['authorization'];
  if (!token) return res.status(401).json({ error: 'No token provided' });

  try {
    const decoded = jwt.verify(token, SECRET_KEY);
    req.user = decoded;
    next();
  } catch {
    return res.status(403).json({ error: 'Invalid token' });
  }
};

const authorize = (role) => {
  return (req, res, next) => {
    if (!req.user || req.user.role !== role) {
      return res.status(403).json({ error: 'Access denied: insufficient role' });
    }
    next();
  };
};

module.exports = { authenticate, authorize, SECRET_KEY };
```

---

## Step 5: `app.js` – The Full App

```js
const express = require('express');
const { authenticate, authorize, SECRET_KEY } = require('./auth');
const { findUser } = require('./utils');
const jwt = require('jsonwebtoken');

const app = express();
app.use(express.json());

// Login route
app.post('/login', (req, res) => {
  const { username, password } = req.body;
  const user = findUser(username, password);

  if (!user) return res.status(401).json({ error: 'Invalid credentials' });

  const token = jwt.sign({ id: user.id, name: user.username, role: user.role }, SECRET_KEY, { expiresIn: '1h' });
  res.json({ token });
});

// Protected dashboard route
app.get('/dashboard', authenticate, (req, res) => {
  res.send(`Hello ${req.user.name}, welcome to your dashboard.`);
});

// Admin-only route
app.get('/admin', authenticate, authorize('admin'), (req, res) => {
  res.send('Welcome to the admin panel');
});

app.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

---

## Test It

1. **Login (POST):**

```bash
curl -X POST http://localhost:3000/login -H "Content-Type: application/json" -d '{"username":"rahman", "password":"1234"}'
```

2. **Use token:**

```bash
curl -H "Authorization: <your-token>" http://localhost:3000/dashboard
curl -H "Authorization: <your-token>" http://localhost:3000/admin
```

---

### Summary

| Feature           | Purpose                             |
| ----------------- | ----------------------------------- |
| `users.json`      | Simulated user database             |
| `login` route     | Validates credentials and gives JWT |
| `authenticate`    | Verifies token and user data        |
| `authorize(role)` | Restricts access based on role      |

---
### HTTP Methods in Express – GET and POST in Detail

Let’s break down how **GET** and **POST** methods work in Express, including both form-based and JavaScript (AJAX/fetch) submissions.

---

## 1. **GET Method – Read or Fetch Data**

* Used to **retrieve data**
* Data can be sent via **URL (query strings or route params)**
* Safe and idempotent (won’t change data)

### Example:

```js
const express = require('express');
const app = express();

app.get('/greet', (req, res) => {
  const name = req.query.name || 'Guest';
  res.send(`Hello, ${name}`);
});
```

**Visit in browser:**
`http://localhost:3000/greet?name=Rahman`
→ Output: `Hello, Rahman`

---

## 2. **POST Method – Submit or Send Data**

* Used to **send data to the server**
* Typically used for forms, logins, creating resources
* Data is sent in the **body** of the request

### Setup Middleware to Parse Body

Add this to enable JSON and form data parsing:

```js
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
```

---

## 3. **POST – Form Submission Example**

### Step 1: HTML Form (public/index.html)

```html
<!DOCTYPE html>
<html>
<head><title>Form Example</title></head>
<body>
  <form method="POST" action="/submit-form">
    <input type="text" name="username" placeholder="Enter name" />
    <input type="submit" value="Submit" />
  </form>
</body>
</html>
```

### Step 2: Serve Static HTML

```js
app.use(express.static('public'));
```

### Step 3: Handle Form Submission

```js
app.post('/submit-form', (req, res) => {
  const { username } = req.body;
  res.send(`Form received. Hello, ${username}`);
});
```

---

## 4. **POST – JavaScript (AJAX / fetch) Example**

Used in SPAs, dynamic UIs, and APIs. No page reload required.

### Step 1: HTML + JavaScript

```html
<!DOCTYPE html>
<html>
<body>
  <input id="nameInput" type="text" placeholder="Enter name" />
  <button onclick="submitName()">Send</button>

  <script>
    function submitName() {
      const username = document.getElementById('nameInput').value;
      fetch('/submit-js', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username })
      })
      .then(res => res.text())
      .then(data => alert(data));
    }
  </script>
</body>
</html>
```

### Step 2: Handle AJAX POST in Express

```js
app.post('/submit-js', (req, res) => {
  const { username } = req.body;
  res.send(`Received via JavaScript: ${username}`);
});
```

---

## Summary: GET vs POST

| Feature         | GET                              | POST                       |
| --------------- | -------------------------------- | -------------------------- |
| Used For        | Reading / fetching               | Submitting / sending       |
| Where Data Goes | URL (query string)               | Request body               |
| Data Size       | Limited                          | Large (forms, files, JSON) |
| Visibility      | Visible in URL                   | Hidden in body             |
| Use in Browser  | Direct link or form (method=GET) | Form or JS (fetch, axios)  |
| Server Access   | `req.query` / `req.params`       | `req.body`                 |

---

### Memory Trick

Use **"GET is for Grabbing, POST is for Pushing."**

---
### Express.js – Full Guide to PUT, DELETE, and Routers with Controllers

This guide walks you through:

1. How to use **PUT** and **DELETE** methods in Express
2. How to organize your app using **Routers** and **Controllers** (modular structure)

---

## 1. **PUT Method – Update Existing Data**

**PUT** is used to **replace or update** an existing resource, like updating a user's profile.

### Example:

```js
app.use(express.json());

app.put('/users/:id', (req, res) => {
  const { id } = req.params;
  const { name, email } = req.body;
  res.send(`User ${id} updated with name: ${name}, email: ${email}`);
});
```

### Request:

```http
PUT /users/42
Content-Type: application/json

{
  "name": "Rahman",
  "email": "rahman@email.com"
}
```

> Use PUT when replacing all or most of a resource.

---

## 2. **DELETE Method – Remove Data**

**DELETE** is used to **remove a resource**, like deleting a user or a product.

### Example:

```js
app.delete('/users/:id', (req, res) => {
  const { id } = req.params;
  res.send(`User ${id} deleted`);
});
```

### Request:

```http
DELETE /users/42
```

> Use DELETE for resource removal, and return 204 or confirmation message.

---

## 3. **Express Router – Setup**

As your app grows, it's best to separate routes into modules.

### Step 1: Create a Router File

#### `routes/userRoutes.js`

```js
const express = require('express');
const router = express.Router();

// Example route
router.get('/', (req, res) => {
  res.send('Get all users');
});

router.post('/', (req, res) => {
  res.send('Create a new user');
});

module.exports = router;
```

### Step 2: Register Router in `app.js`

```js
const express = require('express');
const userRoutes = require('./routes/userRoutes');

const app = express();
app.use(express.json());

app.use('/users', userRoutes);

app.listen(3000, () => {
  console.log('Server running at http://localhost:3000');
});
```

### Test:

* `GET /users`
* `POST /users`

---

## 4. **Express Router – With Controllers (Separation of Concerns)**

### Goal: Keep logic out of route files

---

### Step 1: Controller File

#### `controllers/userController.js`

```js
const getAllUsers = (req, res) => {
  res.send('Fetched all users');
};

const createUser = (req, res) => {
  res.send('User created');
};

const updateUser = (req, res) => {
  const { id } = req.params;
  res.send(`Updated user ${id}`);
};

const deleteUser = (req, res) => {
  const { id } = req.params;
  res.send(`Deleted user ${id}`);
};

module.exports = { getAllUsers, createUser, updateUser, deleteUser };
```

---

### Step 2: Route File

#### `routes/userRoutes.js`

```js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');

router.get('/', userController.getAllUsers);
router.post('/', userController.createUser);
router.put('/:id', userController.updateUser);
router.delete('/:id', userController.deleteUser);

module.exports = router;
```

---

### Step 3: app.js – Register Routes

```js
const express = require('express');
const userRoutes = require('./routes/userRoutes');

const app = express();
app.use(express.json());
app.use('/users', userRoutes);

app.listen(3000);
```

---

## Summary

| Method     | Purpose                | Express Usage                             |
| ---------- | ---------------------- | ----------------------------------------- |
| GET        | Retrieve data          | `app.get('/path', handler)`               |
| POST       | Create new data        | `app.post('/path', handler)`              |
| PUT        | Replace/update data    | `app.put('/path/:id', handler)`           |
| DELETE     | Remove data            | `app.delete('/path/:id', handler)`        |
| Router     | Modular route grouping | `express.Router()` + `app.use('/x', ...)` |
| Controller | Handle business logic  | Separate files for cleaner codebase       |

---

### Memory Trick

Use **"CRUD-RC"**:

* **C**reate (POST)
* **R**ead (GET)
* **U**pdate (PUT)
* **D**elete (DELETE)
* **R**outer
* **C**ontroller

> "Organize your app with CRUD-RC for scalable structure."

---
Great — we'll build a full-featured Node.js application using PostgreSQL, with:

* **Authentication and Authorization** (JWT-based)
* **CRUD operations**
* **Modular structure** (routes, models, controllers, middleware)
* **Best practices** in folder layout, security, and separation of concerns

---

## 📦 Stack Overview

| Feature   | Tool                           |
| --------- | ------------------------------ |
| Server    | Node.js + Express              |
| Database  | PostgreSQL                     |
| ORM       | Prisma (type-safe, modern ORM) |
| Auth      | JWT (JSON Web Token)           |
| Passwords | bcrypt (secure hashing)        |

---

## 📁 Project Structure

```
project/
├── src/
│   ├── controllers/
│   ├── middleware/
│   ├── models/         (Prisma schema)
│   ├── routes/
│   ├── services/
│   ├── utils/
│   └── index.js        (entry point)
├── prisma/
│   └── schema.prisma
├── .env
├── package.json
└── README.md
```

---

## 🔧 Step-by-Step Setup (High-Level Plan)

### 1. Initialize project

```bash
npm init -y
npm install express prisma @prisma/client jsonwebtoken bcrypt dotenv
npx prisma init
```

---

### 2. Configure PostgreSQL connection

`.env`:

```
DATABASE_URL="postgresql://postgres:password@localhost:5432/mydb"
JWT_SECRET="super-secret-key"
```

---

### 3. Create Prisma schema

`prisma/schema.prisma`:

```prisma
model User {
  id       Int    @id @default(autoincrement())
  email    String @unique
  password String
  role     String @default("user")
  posts    Post[]
}

model Post {
  id      Int    @id @default(autoincrement())
  title   String
  content String
  userId  Int
  user    User   @relation(fields: [userId], references: [id])
}
```

Then:

```bash
npx prisma migrate dev --name init
npx prisma generate
```

---

### 4. Build the App

We'll now proceed to build the application **module-by-module**, including:

* User Registration/Login (with hashed passwords)
* JWT Middleware
* Role-based Authorization
* CRUD for Posts
* Best-practice separation

---
```js
// src/index.js
require('dotenv').config();
const express = require('express');
const app = express();
const userRoutes = require('./routes/userRoutes');
const postRoutes = require('./routes/postRoutes');
const { errorHandler } = require('./middleware/errorMiddleware');

app.use(express.json());

app.use('/api/users', userRoutes);
app.use('/api/posts', postRoutes);

app.use(errorHandler);

app.listen(3000, () => console.log('Server running on http://localhost:3000'));


// src/routes/userRoutes.js
const express = require('express');
const router = express.Router();
const { register, login, getMe } = require('../controllers/userController');
const { authenticate } = require('../middleware/authMiddleware');

router.post('/register', register);
router.post('/login', login);
router.get('/me', authenticate, getMe);

module.exports = router;


// src/routes/postRoutes.js
const express = require('express');
const router = express.Router();
const { getPosts, getPost, createPost, updatePost, deletePost } = require('../controllers/postController');
const { authenticate, authorize } = require('../middleware/authMiddleware');

router.get('/', getPosts);
router.get('/:id', getPost);
router.post('/', authenticate, createPost);
router.put('/:id', authenticate, authorize, updatePost);
router.delete('/:id', authenticate, authorize, deletePost);

module.exports = router;


// src/controllers/userController.js
const prisma = require('../utils/prisma');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');

exports.register = async (req, res, next) => {
  try {
    const { email, password } = req.body;
    const hashedPassword = await bcrypt.hash(password, 10);
    const user = await prisma.user.create({ data: { email, password: hashedPassword } });
    res.status(201).json({ id: user.id, email: user.email });
  } catch (err) {
    next(err);
  }
};

exports.login = async (req, res, next) => {
  try {
    const { email, password } = req.body;
    const user = await prisma.user.findUnique({ where: { email } });
    if (!user || !(await bcrypt.compare(password, user.password))) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    const token = jwt.sign({ id: user.id, role: user.role }, process.env.JWT_SECRET, { expiresIn: '1h' });
    res.json({ token });
  } catch (err) {
    next(err);
  }
};

exports.getMe = async (req, res) => {
  const user = await prisma.user.findUnique({ where: { id: req.user.id }, select: { id: true, email: true, role: true } });
  res.json(user);
};


// src/controllers/postController.js
const prisma = require('../utils/prisma');

exports.getPosts = async (req, res) => {
  const posts = await prisma.post.findMany({ include: { user: true } });
  res.json(posts);
};

exports.getPost = async (req, res) => {
  const post = await prisma.post.findUnique({ where: { id: Number(req.params.id) }, include: { user: true } });
  if (!post) return res.status(404).json({ error: 'Post not found' });
  res.json(post);
};

exports.createPost = async (req, res) => {
  const { title, content } = req.body;
  const post = await prisma.post.create({
    data: { title, content, userId: req.user.id }
  });
  res.status(201).json(post);
};

exports.updatePost = async (req, res) => {
  const post = await prisma.post.update({
    where: { id: Number(req.params.id) },
    data: req.body
  });
  res.json(post);
};

exports.deletePost = async (req, res) => {
  await prisma.post.delete({ where: { id: Number(req.params.id) } });
  res.status(204).end();
};


// src/middleware/authMiddleware.js
const jwt = require('jsonwebtoken');
const prisma = require('../utils/prisma');

exports.authenticate = async (req, res, next) => {
  const token = req.headers.authorization;
  if (!token) return res.status(401).json({ error: 'No token' });

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch {
    res.status(403).json({ error: 'Invalid token' });
  }
};

exports.authorize = async (req, res, next) => {
  const post = await prisma.post.findUnique({ where: { id: Number(req.params.id) } });
  if (!post || post.userId !== req.user.id) return res.status(403).json({ error: 'Forbidden' });
  next();
};


// src/middleware/errorMiddleware.js
exports.errorHandler = (err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: 'Internal Server Error' });
};


// src/utils/prisma.js
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();
module.exports = prisma;


```
=======
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

>>>>>>> c0d3ce5935547974e03b36e8d6e7423254e32378