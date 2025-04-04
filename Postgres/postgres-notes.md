Reference video: [![Watch the video]](https://youtu.be/nTQUwghvy5Q)
---

### 1. **What is a Database? (0:03:16)**

A **database** is an organized collection of data that can be easily accessed, managed, and updated. Data in a database is typically stored in tables, and the data can be structured (like in relational databases) or unstructured (like in NoSQL databases). 

Databases are used to store and retrieve large amounts of data efficiently and securely. They are often used in applications ranging from small websites to large enterprise systems.

In a relational database:
- **Data is stored in tables** (structured into rows and columns).
- Each table can have relationships to other tables, which helps you organize and manage data efficiently.

For example, a simple e-commerce database might have tables for **products**, **orders**, and **customers**, where each table is related to the others by keys.

---

### 2. **What is SQL and Relational Database? (0:05:17)**

#### SQL (Structured Query Language)
SQL is the standard language used to interact with a relational database. It allows you to:
- Create and modify databases and tables.
- Insert, update, delete, and query data.
- Define relationships between tables.

Common SQL commands include:
- `CREATE TABLE`: Creates a new table in the database.
- `INSERT INTO`: Adds data into a table.
- `SELECT`: Retrieves data from a table.
- `UPDATE`: Modifies data in a table.
- `DELETE`: Removes data from a table.

#### Relational Database
A **relational database** is a type of database that stores data in a structured way using **tables**. These tables are related to one another, and relationships are often defined by **primary keys** and **foreign keys**.

- **Primary Key**: A unique identifier for each record in a table.
- **Foreign Key**: A column that creates a relationship between two tables by referencing the primary key of another table.

For example:
- You might have a **Customers** table where each customer has a unique ID (`customer_id`), and an **Orders** table that references the `customer_id` as a foreign key to link orders to customers.

---

### 3. **What is PostgreSQL AKA Postgres? (0:09:10)**

**PostgreSQL** (often abbreviated as **Postgres**) is an advanced, open-source relational database management system (RDBMS). It is known for its high level of compliance with SQL standards and advanced features.

Postgres is:
- **ACID Compliant**: Ensures that database transactions are processed reliably.
- **Highly Extensible**: You can add custom data types, functions, and even write code in different programming languages like Python.
- **Supports Advanced Features**: Like **JSON**, **full-text search**, **array data types**, and more.
- **Open-Source**: It’s free to use and backed by a large community.

In addition to traditional relational features, Postgres also supports **NoSQL** features, like storing and querying JSON data, making it very flexible for modern applications.

---

### 4. **PostgreSQL Installation (Mac OS) (0:10:53)**

To install PostgreSQL on a **Mac**, you can follow these steps:

#### Using Homebrew (Recommended)
1. **Install Homebrew**: If you don’t have Homebrew installed, open the terminal and run:
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. **Install PostgreSQL**: Once Homebrew is installed, run:
   ```bash
   brew install postgresql
   ```

3. **Start PostgreSQL**: After installation, start the PostgreSQL service:
   ```bash
   brew services start postgresql
   ```

4. **Verify Installation**: Check if PostgreSQL is installed and running:
   ```bash
   psql --version
   ```

#### Using Postgres.app
You can also install **Postgres.app**, which is a macOS application that provides a full PostgreSQL installation along with a GUI to manage your databases.
1. Download **Postgres.app** from [PostgresApp.com](https://postgresapp.com/).
2. Drag the app to your `Applications` folder and open it.
3. It will automatically set up PostgreSQL and the necessary binaries on your system.

---

### 5. **PostgreSQL Installation (Windows) (0:14:21)**

To install PostgreSQL on **Windows**:

1. **Download the Installer**: Go to the official [PostgreSQL download page](https://www.postgresql.org/download/windows/) and download the installer for Windows.

2. **Run the Installer**: Launch the installer and follow the instructions. You will be asked to:
   - Choose the installation directory.
   - Set the password for the `postgres` superuser.
   - Select the components to install (you can leave the default options).
   - Choose the port number (default is `5432`).
   - Choose the locale (default is fine).

3. **Complete the Installation**: Finish the installation process and restart your computer if required.

4. **Verify Installation**: Open the `SQL Shell (psql)` application and type:
   ```bash
   psql --version
   ```

5. **Using pgAdmin**: PostgreSQL comes with **pgAdmin**, a GUI client to manage your databases, which is installed automatically during setup.

---

### 6. **GUI Clients vs Terminal/CMD Clients (0:17:38)**

- **GUI Clients**: Graphical user interfaces (GUI) are applications with visual representations of your data and allow you to interact with the database without writing complex SQL queries. Examples include **pgAdmin**, **DBeaver**, and **DataGrip**. These are often preferred for beginners or people who prefer working with visual tools.
  
  **Pros**:
  - Easier to use, no need to remember complex commands.
  - Visualize tables, data, and relationships.
  - Helpful for generating queries.

  **Cons**:
  - Can be slower for large queries.
  - Limited flexibility compared to using SQL commands directly.

- **Terminal/CMD Clients**: Using a terminal or command-line interface (CLI) like `psql` (PostgreSQL’s command-line tool) allows you to write SQL queries directly. This is often preferred by advanced users for its speed and flexibility.

  **Pros**:
  - Direct interaction with the database.
  - No extra overhead of a GUI.
  - More control over the database and queries.

  **Cons**:
  - Requires knowledge of SQL.
  - Less visual, which can make it harder to manage complex databases.

---

### 7. **Setup PSQL (Mac OS) (0:21:39)**

To set up `psql` (PostgreSQL's command-line tool) on **Mac OS**:

1. If you installed PostgreSQL via Homebrew, the `psql` tool should already be installed.
2. Open **Terminal** and type:
   ```bash
   psql postgres
   ```
   This opens an interactive session with the default `postgres` database.

3. If you installed PostgreSQL via Postgres.app, the `psql` command will be available in the `/Applications/Postgres.app/Contents/Versions/latest/bin/` directory. Add this directory to your `PATH`:
   ```bash
   export PATH=$PATH:/Applications/Postgres.app/Contents/Versions/latest/bin
   ```

---

### 8. **Setup PSQL (Windows) (0:25:22)**

On **Windows**, after installing PostgreSQL:
1. Open the **SQL Shell (psql)** application that was installed.
2. When prompted:
   - Enter the **server** (usually `localhost` if it’s on your local machine).
   - Enter the **database name** (by default, `postgres`).
   - Enter the **username** (by default, `postgres`).
   - Enter the **password** that you set during installation.

3. After logging in, you can run SQL queries directly from the terminal.

---



* * * * *

### 1\. **How to Create a Database (0:30:15)**

To create a new database in PostgreSQL, you use the `CREATE DATABASE` command. This command will create a new database in your PostgreSQL instance.

#### Syntax:

```
CREATE DATABASE database_name;

```

#### Example:

```
CREATE DATABASE ecommerce;

```

This creates a new database named `ecommerce`.

#### Verifying Database Creation:

You can list all databases to confirm that the database has been created:

```
\l

```

This will list all the databases in your PostgreSQL instance.

To connect to the newly created database:

```
\c ecommerce

```

#### Sample Result:

If the database is created successfully, you will see:

```
You are now connected to database "ecommerce" as user "your_user".

```

* * * * *

### 2\. **How to Connect to Databases (0:33:35)**

Once a database is created, you can connect to it using the `\c` (or `\connect`) command in PostgreSQL's interactive terminal (`psql`).

#### Syntax:

```
\c database_name

```

#### Example:

To connect to the `ecommerce` database:

```
\c ecommerce

```

This will switch your session to the `ecommerce` database.

Alternatively, you can also connect directly when starting `psql` by providing the database name:

```
psql -d ecommerce

```

#### Sample Result:

After running the `\c` command, you'll see:

```
You are now connected to database "ecommerce" as user "your_user".

```

* * * * *

### 3\. **A Very Dangerous Command (0:38:12)**

One of the most dangerous commands in PostgreSQL (and most databases) is the `DROP` command. It is used to **delete** an entire database or table, and it cannot be undone. Always be cautious when using this command.

#### Syntax for Dropping a Database:

```
DROP DATABASE database_name;

```

#### Example:

```
DROP DATABASE ecommerce;

```

This command will **permanently delete** the `ecommerce` database and all of its data. This action cannot be reversed unless you have a backup.

#### Syntax for Dropping a Table:

```
DROP TABLE table_name;

```

#### Example:

```
DROP TABLE users;

```

This will permanently remove the `users` table from the database. All the data in that table will be lost.

* * * * *

### 4\. **How to Create Tables (0:41:37)**

To create a table in PostgreSQL, you use the `CREATE TABLE` command. A table consists of columns, and each column has a name and a data type.

#### Syntax:

```
CREATE TABLE table_name (
    column1 datatype,
    column2 datatype,
    column3 datatype
);

```

#### Example:

```
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

In this example:

-   `id`: A serial number that automatically increments and is used as the primary key.

-   `name`: A text field to store the user's name.

-   `email`: A text field for the user's email, marked as unique (no two users can have the same email).

-   `created_at`: A timestamp indicating when the record was created, with a default value of the current time.

#### Verifying Table Creation:

You can check the structure of your table with the `\d` command:

```
\d users

```

This will show the details of the `users` table, including column names and data types.

#### Sample Result:

After running the `CREATE TABLE` command, PostgreSQL will respond with:

```
CREATE TABLE

```

* * * * *

### 5\. **Creating Tables Without Constraints (0:45:46)**

In some cases, you may want to create a table without any constraints (such as primary keys, foreign keys, etc.) to have more flexibility. Constraints are used to enforce rules on the data, but if you choose not to use them, you can create a table with just basic column definitions.

#### Syntax:

```
CREATE TABLE table_name (
    column1 datatype,
    column2 datatype,
    column3 datatype
);

```

#### Example:

```
CREATE TABLE products (
    id SERIAL,
    name VARCHAR(100),
    price DECIMAL
);

```

This creates a `products` table with three columns but without any constraints. For example, `id` is not a primary key, so there are no restrictions on duplicates or indexing for that column.

#### Sample Result:

After running the command, you'll see:

```
CREATE TABLE

```

* * * * *

### 6\. **Creating Tables with Constraints (0:49:12)**

Constraints are rules that you can apply to columns to enforce certain conditions. These can include primary keys, unique values, and foreign key relationships.

#### Common Constraints:

-   **PRIMARY KEY**: Ensures that each record in the table has a unique identifier.

-   **NOT NULL**: Ensures that a column cannot have NULL values.

-   **UNIQUE**: Ensures that all values in a column are unique.

-   **FOREIGN KEY**: Establishes a relationship between two tables.

#### Syntax:

```
CREATE TABLE table_name (
    column1 datatype CONSTRAINT constraint_name,
    column2 datatype CONSTRAINT constraint_name
);

```

#### Example 1: Table with Primary Key and Unique Constraints:

```
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    hire_date DATE
);

```

In this example:

-   `employee_id` is a **primary key** and will be unique for every record.

-   `name` and `email` are **NOT NULL**, meaning they must have values.

-   `email` is also **unique**, so no two employees can have the same email.

#### Example 2: Table with Foreign Key Constraint:

```
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);

```

Here, `customer_id` is a **foreign key** that links the `orders` table to the `customers` table. It references the `id` column in the `customers` table.

#### Sample Result:

After running these commands, you will see:

```
CREATE TABLE

```

### Summary:

-   **Creating a Database**: You use `CREATE DATABASE` to create a new database.

-   **Connecting to Databases**: Use `\c database_name` in `psql` to connect to a database.

-   **Dangerous Command**: The `DROP` command is dangerous as it permanently deletes databases or tables.

-   **Creating Tables**: You can create tables with basic columns or with constraints like primary keys and foreign keys.



* * * * *

### **1\. Data Types in PostgreSQL**

Data types define the kind of data a column can hold. PostgreSQL supports many types, categorized as follows:

#### **Numeric Types**:

-   **INTEGER** (`int`, `int4`): A standard integer type.

-   **BIGINT** (`int8`): A large integer type, used when the range of `INTEGER` is too small.

-   **SMALLINT** (`int2`): A smaller integer type, with a smaller range than `INTEGER`.

-   **DECIMAL** or **NUMERIC**: Exact numeric data types for storing large numbers with precise values, useful for financial calculations.

-   **REAL**: A single precision floating point number.

-   **DOUBLE PRECISION**: A double precision floating point number.

#### Example:

```
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    price DECIMAL(10, 2)  -- A decimal number with 10 digits in total and 2 digits after the decimal point
);

```

#### **Character Types**:

-   **CHAR(n)**: A fixed-length character string.

-   **VARCHAR(n)**: A variable-length character string with a maximum length `n`.

-   **TEXT**: A variable-length character string with no upper limit.

#### Example:

```
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50),  -- A variable-length string for username (max 50 characters)
    bio TEXT  -- A large string with no limit on length
);

```

#### **Date/Time Types**:

-   **DATE**: A date (year, month, day).

-   **TIME**: A time of day (hour, minute, second).

-   **TIMESTAMP**: A date and time (including timezone).

-   **INTERVAL**: A time span (e.g., '1 day', '5 hours').

#### Example:

```
CREATE TABLE events (
    event_id SERIAL PRIMARY KEY,
    event_date DATE,
    start_time TIME,
    end_time TIME
);

```

#### **Boolean Type**:

-   **BOOLEAN**: Stores `TRUE`, `FALSE`, or `NULL`.

#### Example:

```
CREATE TABLE subscriptions (
    subscription_id SERIAL PRIMARY KEY,
    active BOOLEAN DEFAULT TRUE  -- This will default to 'TRUE' if no value is provided
);

```

#### **UUID Type**:

-   **UUID**: A universally unique identifier, commonly used to store unique identifiers that are not tied to any particular database sequence.

#### Example:

```
CREATE TABLE sessions (
    session_id UUID PRIMARY KEY DEFAULT gen_random_uuid()
);

```

#### **JSON Types**:

-   **JSON**: A text-based format for storing JSON data.

-   **JSONB**: A more efficient binary format for JSON storage, allows indexing.

#### Example:

```
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_details JSONB  -- Stores JSON data in a binary format
);

```

#### **Array Types**:

-   **ARRAY**: A collection of elements of any type, including numbers, text, or even other arrays.

#### Example:

```
CREATE TABLE inventory (
    product_id SERIAL PRIMARY KEY,
    colors TEXT[],  -- Array of color names
    sizes TEXT[]    -- Array of available sizes
);

```

* * * * *

### **2\. Constraints in PostgreSQL**

Constraints are rules that restrict the data that can be inserted into a table, ensuring data integrity. Below are the common constraints:

#### **NOT NULL**:

-   Ensures that a column cannot have a `NULL` value.

#### Example:

```
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL  -- This column cannot have NULL values
);

```

#### **UNIQUE**:

-   Ensures that all values in a column (or combination of columns) are distinct.

#### Example:

```
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE  -- Ensures that no two users can have the same username
);

```

#### **PRIMARY KEY**:

-   A combination of `NOT NULL` and `UNIQUE`. It uniquely identifies each record in a table.

#### Example:

```
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,  -- Automatically unique and not null
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

#### **FOREIGN KEY**:

-   Ensures a column's value matches the primary key or a unique key from another table, establishing a relationship between the tables.

#### Example:

```
CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    FOREIGN KEY (order_id) REFERENCES orders(order_id)  -- Foreign key linking order_items to orders table
);

```

#### **CHECK**:

-   Ensures that all values in a column satisfy a specific condition (e.g., a price must be greater than 0).

#### Example:

```
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    price DECIMAL(10, 2),
    CHECK (price > 0)  -- Ensures that the price is always greater than 0
);

```

#### **DEFAULT**:

-   Sets a default value for a column when no value is provided during an `INSERT` statement.

#### Example:

```
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    signup_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP  -- Default value is the current timestamp
);

```

#### **EXCLUSION**:

-   Ensures that certain rows cannot have overlapping values in specified columns. This is commonly used for range-based constraints.

#### Example:

```
CREATE TABLE room_bookings (
    booking_id SERIAL PRIMARY KEY,
    room_number INT,
    start_date DATE,
    end_date DATE,
    EXCLUDE USING gist (room_number WITH =, tsrange(start_date, end_date) WITH &&)
);

```

#### **REFERENCES**:

-   Similar to **FOREIGN KEY**, but can be used to enforce foreign key relationships on other columns, without needing to define a `REFERENCES` constraint explicitly.

* * * * *

### **Adding Constraints to an Existing Table**

You can also add constraints to an existing table using the `ALTER TABLE` command.

#### Example 1: Adding a `NOT NULL` Constraint

```
ALTER TABLE employees
ALTER COLUMN name SET NOT NULL;

```

#### Example 2: Adding a `UNIQUE` Constraint

```
ALTER TABLE users
ADD CONSTRAINT unique_email UNIQUE (email);

```

#### Example 3: Adding a `CHECK` Constraint

```
ALTER TABLE products
ADD CONSTRAINT price_check CHECK (price >= 0);

```

#### Example 4: Adding a `FOREIGN KEY` Constraint

```
ALTER TABLE order_items
ADD CONSTRAINT fk_order FOREIGN KEY (order_id) REFERENCES orders(order_id);

```

* * * * *

### **Summary of Common Data Types and Constraints**

-   **Data Types**:

    -   Numeric types: `INTEGER`, `DECIMAL`, `BIGINT`, `SMALLINT`, etc.

    -   Character types: `VARCHAR`, `CHAR`, `TEXT`.

    -   Date and time: `DATE`, `TIMESTAMP`, `TIME`, `INTERVAL`.

    -   Boolean: `BOOLEAN`.

    -   JSON: `JSON`, `JSONB`.

    -   Arrays: `TEXT[]`, `INT[]`.

-   **Constraints**:

    -   `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `DEFAULT`.

* * * * *

* * * * *

### **1\. FLOAT Data Type**

In PostgreSQL, the `FLOAT` data type is used to store floating-point numbers. It is an approximate numeric type, meaning it is not 100% precise due to the way floating-point numbers are stored in memory.

There are two types of floating-point data types:

-   **`REAL`**: Single precision floating-point number (4 bytes).

-   **`DOUBLE PRECISION`**: Double precision floating-point number (8 bytes).

#### Example:

```
CREATE TABLE measurements (
    measurement_id SERIAL PRIMARY KEY,
    temperature REAL,   -- Single precision
    pressure DOUBLE PRECISION  -- Double precision
);

```

-   **`REAL`** allows you to store floating-point values with **6 digits of precision**.

-   **`DOUBLE PRECISION`** allows you to store floating-point values with **15 digits of precision**.

* * * * *

### **2\. DECIMAL (NUMERIC) Data Type and Precision**

The `DECIMAL` or `NUMERIC` data type is used for exact numeric values with a specified **precision** (total number of digits) and **scale** (number of digits to the right of the decimal point). Unlike `FLOAT`, `DECIMAL` stores numbers exactly and is often used for financial calculations where precision is important.

-   **Precision**: The total number of digits in the number (both to the left and right of the decimal point).

-   **Scale**: The number of digits after the decimal point.

#### Syntax:

```
DECIMAL(precision, scale)

```

-   **`precision`**: The total number of digits in the number.

-   **`scale`**: The number of digits after the decimal point.

For example, `DECIMAL(10, 2)` means a number with **10 total digits**, of which **2 digits are after the decimal point**.

#### Example:

```
CREATE TABLE transactions (
    transaction_id SERIAL PRIMARY KEY,
    amount DECIMAL(10, 2)  -- Precision = 10, Scale = 2
);

```

Here, `amount` can store numbers up to **10 digits**, with **2 digits after the decimal point**. So, you could store values like:

-   `12345678.90`

-   `99999999.99`

-   `0.99`

But **it cannot store**:

-   `123456789.01` (too many digits before the decimal point)

-   `12345.123` (too many digits after the decimal point)

#### More Examples:

-   `DECIMAL(5, 2)` could store `123.45` (3 digits before the decimal point, 2 digits after the decimal point).

-   `DECIMAL(6, 3)` could store `123.456` (3 digits before the decimal point, 3 digits after the decimal point).

-   `DECIMAL(10, 0)` could store `1234567890` (no digits after the decimal point).

### **Precision Example with Data Insertion:**

If you insert a value that exceeds the scale (number of digits after the decimal point), PostgreSQL will round the value to fit within the defined precision and scale.

#### Example of Inserting Data:

```
INSERT INTO transactions (amount) VALUES (123.456);

```

Since `DECIMAL(10, 2)` only allows **2 digits after the decimal point**, this will be rounded to:

```
123.46

```

#### Example of Inserting a Large Value:

```
INSERT INTO transactions (amount) VALUES (123456789.01);

```

Since `DECIMAL(10, 2)` can only hold **10 digits** (including the digits before and after the decimal point), this will **fail** with an error because it exceeds the allowed number of digits.

* * * * *

### **Summary**

-   **`FLOAT` Data Type**: Used for storing approximate numeric values (single or double precision). It stores floating-point numbers, which may not be perfectly precise.

-   **`DECIMAL (NUMERIC)` Data Type**: Used for storing exact numeric values with defined precision and scale. It is preferred for financial data where precision is essential.

    -   **Precision** defines the total number of digits.

    -   **Scale** defines how many digits come after the decimal point.

    
---

### **1. INSERT INTO (0:55:55)**

The **`INSERT INTO`** command is used to insert new records into a table in a PostgreSQL database. It's one of the most commonly used SQL commands.

#### Syntax:
```sql
INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);
```

- **`table_name`**: The name of the table where you want to insert data.
- **`column1, column2, ...`**: The list of columns you want to insert data into.
- **`value1, value2, ...`**: The corresponding values for each column.

If you’re inserting values for every column, you don’t need to specify the column names. However, it’s a good practice to do so, especially when the table has a lot of columns or if you’re leaving certain columns empty (e.g., `NULL`).

#### Example:
```sql
INSERT INTO users (name, email, created_at)
VALUES ('John Doe', 'john.doe@example.com', CURRENT_TIMESTAMP);
```

This command inserts a new row into the `users` table with `name`, `email`, and `created_at` values.

---

### **2. INSERT INTO Example (0:59:14)**

Here’s a more concrete example with a full database table:

#### Table Structure:
```sql
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    hire_date DATE,
    salary DECIMAL(10, 2)
);
```

Now, to insert data into the `employees` table:
```sql
INSERT INTO employees (first_name, last_name, hire_date, salary)
VALUES ('Alice', 'Smith', '2022-01-15', 55000.00);
```

This command will insert a new record for Alice Smith with a hire date of `2022-01-15` and a salary of `55000.00`.

#### Sample Result:
If the insert is successful, PostgreSQL will not return any output but will indicate that the command was successful:
```
INSERT 0 1
```
This means one row was inserted.

---

### **3. Generate 1000 Rows with Mockaroo (1:02:36)**

Mockaroo is a tool used to generate large datasets for testing purposes. It allows you to create CSVs, SQL insert statements, and more. You can use Mockaroo to generate 1000 rows of data for a table, then use the generated SQL insert statements in your PostgreSQL database.

#### Steps:
1. Go to [Mockaroo](https://mockaroo.com/).
2. Select the type of data you need (e.g., names, addresses, dates, etc.).
3. Specify the number of rows (1000 in this case).
4. Select the **SQL** format to generate the `INSERT INTO` statements.
5. Copy the generated SQL and paste it into your PostgreSQL query tool.

For example, Mockaroo might generate SQL like:
```sql
INSERT INTO employees (first_name, last_name, hire_date, salary)
VALUES
('John', 'Doe', '2023-01-01', 60000.00),
('Jane', 'Smith', '2022-05-22', 70000.00),
-- 998 more rows
;
```

---

### **4. SELECT FROM (1:12:28)**

The **`SELECT`** statement is used to retrieve data from a database. It is the most commonly used SQL query.

#### Syntax:
```sql
SELECT column1, column2, ...
FROM table_name;
```

- **`column1, column2, ...`**: The columns you want to select. You can use `*` to select all columns.
- **`FROM table_name`**: The table from which to retrieve the data.

#### Example:
```sql
SELECT first_name, last_name
FROM employees;
```

This retrieves the `first_name` and `last_name` columns from the `employees` table.

#### Sample Result:
```
 first_name | last_name
------------+-----------
 John       | Doe
 Jane       | Smith
 ...
```

---

### **5. ORDER BY (1:15:18)**

The **`ORDER BY`** clause is used to sort the result set by one or more columns. By default, it sorts in ascending order (from lowest to highest). You can also specify **descending** order with `DESC`.

#### Syntax:
```sql
SELECT column1, column2
FROM table_name
ORDER BY column1 [ASC|DESC];
```

#### Example:
```sql
SELECT first_name, salary
FROM employees
ORDER BY salary DESC;
```

This retrieves the `first_name` and `salary` of employees and sorts them by `salary` in **descending** order.

#### Sample Result:
```
 first_name | salary
------------+--------
 Jane       | 80000
 Alice      | 75000
 John       | 60000
 ...
```

---

### **6. DISTINCT (1:19:53)**

The **`DISTINCT`** keyword is used to remove duplicate records from the result set.

#### Syntax:
```sql
SELECT DISTINCT column1, column2
FROM table_name;
```

#### Example:
```sql
SELECT DISTINCT first_name
FROM employees;
```

This returns a list of unique first names from the `employees` table, removing any duplicates.

#### Sample Result:
```
 first_name
------------
 John
 Jane
 Alice
 ...
```

---

### **7. WHERE Clause and AND (1:21:59)**

The **`WHERE`** clause is used to filter records based on specific conditions. The **`AND`** operator allows you to combine multiple conditions.

#### Syntax:
```sql
SELECT column1, column2
FROM table_name
WHERE condition1 AND condition2;
```

#### Example:
```sql
SELECT first_name, salary
FROM employees
WHERE salary > 60000 AND hire_date < '2023-01-01';
```

This selects employees with a salary greater than 60,000 and who were hired before January 1, 2023.

#### Sample Result:
```
 first_name | salary
------------+--------
 Jane       | 70000
 ...
```

---

### **8. Comparison Operators (1:25:29)**

Comparison operators are used to compare values in SQL. Common operators include:
- **`=`**: Equal to
- **`<>`** or **`!=`**: Not equal to
- **`>`**: Greater than
- **`<`**: Less than
- **`>=`**: Greater than or equal to
- **`<=`**: Less than or equal to

#### Syntax:
```sql
SELECT column1, column2
FROM table_name
WHERE column1 > value;
```

#### Example:
```sql
SELECT first_name, salary
FROM employees
WHERE salary >= 60000;
```

This will return employees whose salary is greater than or equal to 60,000.

#### Sample Result:
```
 first_name | salary
------------+--------
 Jane       | 70000
 Alice      | 75000
 ...
```

---

### **Exam Power-ups for Memory:**

- **INSERT INTO**: Always specify the table columns to avoid errors, especially if the table has default values or constraints.
- **SELECT**: Be aware of using `DISTINCT` when you want unique results; it can help you clean up duplicate data.
- **ORDER BY**: Default is ascending order; always specify `DESC` if you want descending order.
- **WHERE**: Use **AND/OR** to combine multiple conditions and **comparison operators** to filter data precisely.

---

* * * * *

### **1\. `SERIAL` and `BIGSERIAL` (Auto-Incrementing Keys)**

In PostgreSQL, **`SERIAL`** and **`BIGSERIAL`** are used to create auto-incrementing integer columns, often used as **primary keys**. These data types allow you to automatically generate unique identifiers for each row without needing to manually specify the value.

#### **`SERIAL`**:

-   **`SERIAL`** is used to create auto-incrementing integer columns, with a range of **1 to 2,147,483,647**.

-   It creates an **integer** column (`INT`) that automatically increments with each new row inserted.

-   The `SERIAL` type uses a **sequence** to generate the next available number.

#### Syntax:

```
column_name SERIAL

```

#### **`BIGSERIAL`**:

-   **`BIGSERIAL`** is similar to **`SERIAL`**, but it uses a **`BIGINT`** data type (8 bytes) for larger values, with a range of **1 to 9,223,372,036,854,775,807**.

-   It's typically used when you expect to insert a very large number of records into the table.

#### Syntax:

```
column_name BIGSERIAL

```

### **Example Table with `SERIAL` and `BIGSERIAL`**

#### Using `SERIAL` for a Primary Key:

```
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

```

Here, `user_id` will automatically increment with each new row inserted.

#### Using `BIGSERIAL` for a Primary Key:

```
CREATE TABLE products (
    product_id BIGSERIAL PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10, 2)
);

```

In this case, `product_id` will be a **BIGINT** and auto-increment with each new product inserted.

* * * * *

### **2\. `UUID` as a Primary Key**

The **`UUID`** (Universally Unique Identifier) data type is an alternative to **`SERIAL`** and **`BIGSERIAL`** for generating unique keys. UUIDs are particularly useful when you need globally unique identifiers that can be generated on different systems or across distributed databases.

A **UUID** is a 128-bit number, represented as a 32-character hexadecimal string, which ensures uniqueness even across different systems and databases. UUIDs are **not sequential** like `SERIAL` and `BIGSERIAL`, but they can be generated in a variety of ways, including using system clocks, random data, or hashes.

#### Syntax:

```
column_name UUID DEFAULT uuid_generate_v4()

```

Here, **`uuid_generate_v4()`** generates a random UUID using version 4 of the UUID standard (based on random numbers).

#### Example Table with `UUID`:

```
CREATE TABLE customers (
    customer_id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(100),
    email VARCHAR(100)
);

```

In this example, each `customer_id` is generated as a **random UUID** automatically when a new row is inserted.

#### Generating UUIDs:

To generate a **UUID** in PostgreSQL, you need to have the **`uuid-ossp`** extension installed. You can do this with:

```
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

```

Then, you can use functions like:

-   `uuid_generate_v4()` --- Generates a random UUID.

-   `uuid_generate_v1()` --- Generates a UUID based on the timestamp and MAC address.

* * * * *

### **3\. Starting from a Custom Serial**

Sometimes, you may want to start the auto-incrementing sequence at a number other than 1 (e.g., you might want it to start from 1000 or any other custom value). PostgreSQL allows you to set the **starting value** of the sequence.

To achieve this, you can use the `ALTER SEQUENCE` command after creating the table or sequence.

#### Example: Creating a Table with a Custom Serial Start

Let's say you want the `user_id` to start at 1000 instead of 1. Here's how you can do it:

1.  **Create the table with `SERIAL`**:

```
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

```

1.  **Set the starting value of the sequence**:

PostgreSQL automatically creates a sequence for `SERIAL` columns. You can alter it to set the starting value.

```
ALTER SEQUENCE users_user_id_seq RESTART WITH 1000;

```

-   `users_user_id_seq` is the automatically created sequence for the `user_id` column in the `users` table.

-   `RESTART WITH 1000` sets the next value of the sequence to 1000.

#### Verifying the Starting Value:

To check the current value of the sequence:

```
SELECT last_value FROM users_user_id_seq;

```

After running the above `ALTER` command, this query should return `1000` as the next value.

* * * * *

### **Example Summary**:

-   **Using `SERIAL`**: Automatically increments with integer values starting from 1.

-   **Using `BIGSERIAL`**: Automatically increments with large integer values, starting from 1.

-   **Using `UUID`**: Uses a globally unique identifier, generated randomly, and often used for distributed systems.

-   **Custom Serial Start**: Use `ALTER SEQUENCE` to set a custom starting point for an auto-incrementing `SERIAL` or `BIGSERIAL` column.

* * * * *

### **Exam Power-ups for Memory:**

-   **`SERIAL`**: Remember, `SERIAL` is an auto-incrementing integer type that is automatically tied to a sequence, with a range of **1 to 2,147,483,647**.

-   **`BIGSERIAL`**: Use **`BIGSERIAL`** for large datasets when the range of `SERIAL` might be exceeded.

-   **`UUID`**: Use **`UUID`** when you need globally unique identifiers, especially useful in distributed systems or when you don't want sequential IDs.

-   **Custom Serial Start**: To start a `SERIAL` sequence at a custom value, use `ALTER SEQUENCE sequence_name RESTART WITH <custom_value>`.

* * * * *
    


* * * * *

### **1\. LIMIT, OFFSET & FETCH (1:29:35)**

These SQL clauses are used to control the number of rows returned in a query, which is especially useful for pagination or when you only need a subset of data.

#### **LIMIT**:

The **`LIMIT`** clause is used to specify the maximum number of rows to return.

#### Syntax:

```
SELECT column1, column2
FROM table_name
LIMIT number_of_rows;

```

#### Example:

```
SELECT first_name, last_name
FROM employees
LIMIT 5;

```

This query returns the first 5 rows of the `employees` table.

#### **OFFSET**:

The **`OFFSET`** clause is used to specify the number of rows to skip before starting to return rows. This is useful for pagination, where you might want to skip a certain number of rows based on the current page.

#### Syntax:

```
SELECT column1, column2
FROM table_name
OFFSET number_of_rows;

```

#### Example:

```
SELECT first_name, last_name
FROM employees
OFFSET 5 LIMIT 5;

```

This skips the first 5 rows and then returns the next 5 rows. Essentially, it is fetching the 6th to the 10th rows.

#### **FETCH**:

**`FETCH`** is part of the SQL standard for controlling pagination and is often used with the **`OFFSET`** clause. It's more explicit than using `LIMIT` and provides more readability in SQL queries.

#### Syntax:

```
SELECT column1, column2
FROM table_name
OFFSET number_of_rows
FETCH FIRST number_of_rows ROWS ONLY;

```

#### Example:

```
SELECT first_name, last_name
FROM employees
OFFSET 5 FETCH FIRST 5 ROWS ONLY;

```

This query skips the first 5 rows and returns the next 5 rows, similar to the `OFFSET` + `LIMIT` combination.

#### Sample Result:

For `LIMIT 5`:

```
 first_name | last_name
------------+-----------
 Alice      | Smith
 Bob        | Johnson
 Charlie    | Brown
 David      | Lee
 Eve        | White

```

* * * * *

### **2\. IN (1:32:43)**

The **`IN`** operator is used to check if a value matches any value within a list of values. It simplifies multiple **`OR`** conditions.

#### Syntax:

```
SELECT column1, column2
FROM table_name
WHERE column1 IN (value1, value2, value3, ...);

```

#### Example:

```
SELECT first_name, department
FROM employees
WHERE department IN ('Sales', 'Marketing', 'HR');

```

This will return employees whose department is either **Sales**, **Marketing**, or **HR**.

#### Sample Result:

```
 first_name | department
------------+------------
 John       | Sales
 Jane       | Marketing
 Alice      | HR

```

#### Notes:

-   **`IN`** can be used with numbers, strings, dates, and other data types.

-   It can also be used with a subquery:

```
SELECT name
FROM products
WHERE category_id IN (SELECT category_id FROM categories WHERE active = TRUE);

```

* * * * *

### **3\. BETWEEN (1:35:43)**

The **`BETWEEN`** operator is used to filter the result set within a certain range. It is inclusive, meaning it includes the boundary values.

#### Syntax:

```
SELECT column1, column2
FROM table_name
WHERE column1 BETWEEN value1 AND value2;

```

#### Example:

```
SELECT first_name, hire_date
FROM employees
WHERE hire_date BETWEEN '2022-01-01' AND '2023-01-01';

```

This query retrieves employees who were hired between January 1, 2022, and January 1, 2023, including those dates.

#### Sample Result:

```
 first_name | hire_date
------------+------------
 Alice      | 2022-03-15
 Bob        | 2022-06-20
 Charlie    | 2022-11-30

```

#### Notes:

-   **`BETWEEN`** can be used with numeric, date, and time ranges.

-   **`BETWEEN`** is inclusive, so it includes both the start and end values.

* * * * *

### **4\. LIKE and ILIKE (1:37:45)**

The **`LIKE`** operator is used for pattern matching in string columns. It allows you to find records that match a specified pattern.

#### **`LIKE`**:

By default, **`LIKE`** is case-sensitive. It uses two wildcard characters:

-   **`%`**: Represents zero or more characters.

-   **`_`**: Represents a single character.

#### Syntax:

```
SELECT column1, column2
FROM table_name
WHERE column1 LIKE pattern;

```

#### Example:

```
SELECT first_name, last_name
FROM employees
WHERE first_name LIKE 'J%';

```

This query will return all employees whose first name starts with "J" (e.g., John, Jane, Jack).

#### Sample Result:

```
 first_name | last_name
------------+-----------
 John       | Doe
 Jane       | Smith
 Jack       | Brown

```

#### **`ILIKE`**:

The **`ILIKE`** operator is similar to **`LIKE`**, but it is **case-insensitive**, which means it will match patterns regardless of uppercase or lowercase letters.

#### Syntax:

```
SELECT column1, column2
FROM table_name
WHERE column1 ILIKE pattern;

```

#### Example:

```
SELECT first_name, last_name
FROM employees
WHERE first_name ILIKE 'j%';

```

This query will return all employees whose first name starts with **'j'** or **'J'**.

#### Sample Result:

```
 first_name | last_name
------------+-----------
 John       | Doe
 Jane       | Smith
 Jack       | Brown

```

* * * * *

### **Summary of Concepts**

-   **`LIMIT`, `OFFSET`, and `FETCH`**: These clauses are used to control the number of rows returned by a query. `LIMIT` specifies the max number of rows, `OFFSET` skips a certain number of rows, and `FETCH` is used for standard pagination.

-   **`IN`**: A shorthand for multiple `OR` conditions, it checks if a value is within a set of specified values.

-   **`BETWEEN`**: Filters data within a specified range, including the boundary values.

-   **`LIKE` and `ILIKE`**: Used for pattern matching in strings. `LIKE` is case-sensitive, while `ILIKE` is case-insensitive.

* * * * *

### **Exam Power-ups for Memory:**

-   **`LIMIT`**: Useful for pagination. Always remember that `LIMIT` alone won't skip any rows --- you need `OFFSET` for skipping.

-   **`IN`**: Great for checking against a list of values. It can also be used with subqueries.

-   **`BETWEEN`**: Remember, `BETWEEN` is inclusive, so it includes both the start and end values.

-   **`LIKE`**: Use `%` for any number of characters and `_` for a single character in pattern matching.

-   **`ILIKE`**: Use **`ILIKE`** for case-insensitive pattern matching in PostgreSQL.

* * * * *


### **What Are Aggregate Functions?**

Aggregate functions perform a calculation on a set of values and return a single value. They are typically used in conjunction with the **`GROUP BY`** clause to group rows based on certain columns and then apply an aggregate function to each group.

Here are the most commonly used aggregate functions in PostgreSQL:

* * * * *

### **1\. COUNT()**

The **`COUNT()`** function returns the number of rows that match a specified condition. It can also be used with `DISTINCT` to count unique values.

#### Syntax:

```
SELECT COUNT(column_name)
FROM table_name;

```

#### Example 1: Count the Total Number of Employees

```
SELECT COUNT(*)
FROM employees;

```

This returns the total number of rows in the `employees` table.

#### Sample Result:

```
 count
-------
 100

```

#### Example 2: Count Distinct Salaries

```
SELECT COUNT(DISTINCT salary)
FROM employees;

```

This returns the number of distinct salary values in the `employees` table.

#### Sample Result:

```
 count
-------
 15

```

* * * * *

### **2\. SUM()**

The **`SUM()`** function calculates the total sum of a numeric column.

#### Syntax:

```
SELECT SUM(column_name)
FROM table_name;

```

#### Example 1: Total Salary Expense

```
SELECT SUM(salary)
FROM employees;

```

This returns the total sum of the `salary` column in the `employees` table.

#### Sample Result:

```
 sum
------
 500000

```

#### Example 2: Total Sales Amount

```
SELECT SUM(sales_amount)
FROM sales;

```

This returns the total sales amount from the `sales` table.

#### Sample Result:

```
 sum
------
 1200000

```

* * * * *

### **3\. AVG()**

The **`AVG()`** function returns the average value of a numeric column.

#### Syntax:

```
SELECT AVG(column_name)
FROM table_name;

```

#### Example 1: Average Salary

```
SELECT AVG(salary)
FROM employees;

```

This returns the average salary of all employees.

#### Sample Result:

```
 avg
------
 50000

```

#### Example 2: Average Product Price

```
SELECT AVG(price)
FROM products;

```

This returns the average price of products in the `products` table.

#### Sample Result:

```
 avg
------
 25.75

```

* * * * *

### **4\. MIN()**

The **`MIN()`** function returns the smallest value in a column.

#### Syntax:

```
SELECT MIN(column_name)
FROM table_name;

```

#### Example 1: Minimum Salary

```
SELECT MIN(salary)
FROM employees;

```

This returns the minimum salary in the `employees` table.

#### Sample Result:

```
 min
-----
 30000

```

#### Example 2: Minimum Product Price

```
SELECT MIN(price)
FROM products;

```

This returns the minimum price of products in the `products` table.

#### Sample Result:

```
 min
-----
 5.00

```

* * * * *

### **5\. MAX()**

The **`MAX()`** function returns the largest value in a column.

#### Syntax:

```
SELECT MAX(column_name)
FROM table_name;

```

#### Example 1: Maximum Salary

```
SELECT MAX(salary)
FROM employees;

```

This returns the highest salary in the `employees` table.

#### Sample Result:

```
 max
-----
 120000

```

#### Example 2: Maximum Product Price

```
SELECT MAX(price)
FROM products;

```

This returns the highest price of products in the `products` table.

#### Sample Result:

```
 max
-----
 100.00

```

* * * * *

### **6\. VARIANCE()**

The **`VARIANCE()`** function calculates the variance of a numeric column. Variance measures the spread of numbers around the mean. It is useful in statistical analysis.

#### Syntax:

```
SELECT VARIANCE(column_name)
FROM table_name;

```

#### Example 1: Variance in Salaries

```
SELECT VARIANCE(salary)
FROM employees;

```

This calculates the variance in the `salary` column in the `employees` table.

#### Sample Result:

```
 variance
----------
 10000000

```

* * * * *

### **7\. STDDEV()**

The **`STDDEV()`** function calculates the standard deviation, which is the square root of the variance. It shows how much variation exists from the average.

#### Syntax:

```
SELECT STDDEV(column_name)
FROM table_name;

```

#### Example 1: Standard Deviation of Salaries

```
SELECT STDDEV(salary)
FROM employees;

```

This calculates the standard deviation of salaries in the `employees` table.

#### Sample Result:

```
 stddev
--------
 3162.28

```

* * * * *

### **8\. STRING_AGG()**

The **`STRING_AGG()`** function is used to concatenate values from multiple rows into a single string, separated by a delimiter.

#### Syntax:

```
SELECT STRING_AGG(column_name, 'delimiter')
FROM table_name;

```

#### Example 1: Concatenate All Employee Names

```
SELECT STRING_AGG(first_name, ', ')
FROM employees;

```

This concatenates all employee first names in the `employees` table, separated by a comma.

#### Sample Result:

```
 John, Jane, Alice, Bob, ...

```

* * * * *

### **9\. ARRAY_AGG()**

The **`ARRAY_AGG()`** function returns the values of a column as an array.

#### Syntax:

```
SELECT ARRAY_AGG(column_name)
FROM table_name;

```

#### Example 1: Array of Employee Names

```
SELECT ARRAY_AGG(first_name)
FROM employees;

```

This returns an array containing the names of all employees.

#### Sample Result:

```
{John, Jane, Alice, Bob, ...}

```

* * * * *

### **10\. COUNT(*) vs COUNT(column_name)**

-   **`COUNT(*)`**: Counts the total number of rows, including those with `NULL` values.

-   **`COUNT(column_name)`**: Counts the non-`NULL` values in a specific column.

#### Example 1: COUNT(*) (Counting all rows)

```
SELECT COUNT(*)
FROM employees;

```

This counts all rows in the `employees` table, regardless of `NULL` values.

#### Sample Result:

```
 count
-------
 100

```

#### Example 2: COUNT(column_name) (Counting non-NULL values)

```
SELECT COUNT(salary)
FROM employees;

```

This counts the number of non-`NULL` salary values.

#### Sample Result:

```
 count
-------
 95

```

* * * * *

### **Summary of Aggregate Functions**

-   **`COUNT()`**: Counts rows or non-`NULL` values.

-   **`SUM()`**: Sums numeric values.

-   **`AVG()`**: Calculates the average of numeric values.

-   **`MIN()`**: Returns the smallest value.

-   **`MAX()`**: Returns the largest value.

-   **`VARIANCE()`**: Calculates the variance.

-   **`STDDEV()`**: Calculates the standard deviation.

-   **`STRING_AGG()`**: Concatenates values from multiple rows into a single string.

-   **`ARRAY_AGG()`**: Returns values as an array.

* * * * *

### **Exam Power-ups for Memory:**

-   **`COUNT(*)` vs `COUNT(column)`**: **`COUNT(*)`** counts all rows, while **`COUNT(column)`** counts non-`NULL` values in a specific column.

-   **`SUM()`**, **`AVG()`**, **`MIN()`**, **`MAX()`**: Used to perform basic statistical operations like summing, averaging, or finding the extreme values in a dataset.

-   **`VARIANCE()`** and **`STDDEV()`**: Used for statistical analysis, showing the spread or variation in a dataset.

-   **`STRING_AGG()`** and **`ARRAY_AGG()`**: Useful for string concatenation or aggregating values into arrays.

* * * * *


* * * * *

### **What is `GROUP BY`?**

The **`GROUP BY`** clause is used in SQL to arrange identical data into groups. It is commonly used with aggregate functions (such as `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`) to perform calculations on each group of data, rather than on the entire result set.

**`GROUP BY`** allows you to summarize or aggregate data based on one or more columns. It is typically used when you want to:

-   Perform aggregate calculations on data within specific groups.

-   Organize results into categories, such as grouping sales by region, or employees by department.

### **Syntax of `GROUP BY`**

```
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1;

```

-   **`column1`**: The column(s) you want to group by.

-   **`aggregate_function(column2)`**: The aggregate function you want to apply to another column (e.g., `COUNT()`, `SUM()`, `AVG()`, etc.).

-   **`table_name`**: The name of the table you're querying.

* * * * *

### **Using `GROUP BY` with Aggregate Functions**

When you use `GROUP BY`, you typically combine it with aggregate functions to summarize data.

#### **1\. COUNT() Example**

Let's start with a basic example using **`COUNT()`**. This counts the number of rows in each group.

##### Example: Counting Employees by Department

```
SELECT department, COUNT(*)
FROM employees
GROUP BY department;

```

In this example:

-   The query groups the data by `department`.

-   The `COUNT(*)` function counts how many employees are in each department.

#### Sample Result:

```
 department  | count
-------------+-------
 Sales       | 10
 Marketing   | 5
 HR          | 3

```

The result shows the number of employees in each department.

* * * * *

#### **2\. SUM() Example**

You can also use **`SUM()`** to calculate the total of a numeric column for each group.

##### Example: Total Salary Expense by Department

```
SELECT department, SUM(salary)
FROM employees
GROUP BY department;

```

In this case:

-   The query groups the data by `department`.

-   The `SUM(salary)` function calculates the total salary for each department.

#### Sample Result:

```
 department  | sum
-------------+--------
 Sales       | 800000
 Marketing   | 250000
 HR          | 150000

```

This result shows the total salary expense for each department.

* * * * *

#### **3\. AVG() Example**

The **`AVG()`** function is used to find the average value of a column for each group.

##### Example: Average Salary by Department

```
SELECT department, AVG(salary)
FROM employees
GROUP BY department;

```

This query:

-   Groups the employees by their `department`.

-   Calculates the average salary within each department.

#### Sample Result:

```
 department  | avg
-------------+------
 Sales       | 80000
 Marketing   | 50000
 HR          | 50000

```

This shows the average salary for each department.

* * * * *

#### **4\. MIN() and MAX() Example**

You can use the **`MIN()`** and **`MAX()`** functions to find the smallest or largest value in a group.

##### Example 1: Minimum Salary by Department

```
SELECT department, MIN(salary)
FROM employees
GROUP BY department;

```

##### Example 2: Maximum Salary by Department

```
SELECT department, MAX(salary)
FROM employees
GROUP BY department;

```

Both queries:

-   Group the employees by `department`.

-   Return the minimum or maximum salary within each department.

#### Sample Result:

```
 department  | min | max
-------------+-----+------
 Sales       | 50000 | 120000
 Marketing   | 40000 | 70000
 HR          | 35000 | 50000

```

The result shows the lowest and highest salary in each department.

* * * * *

### **Using `GROUP BY` with Multiple Columns**

You can group by multiple columns by separating the column names with commas. This is useful when you want to group by combinations of fields.

##### Example: Grouping by Department and Job Title

```
SELECT department, job_title, COUNT(*)
FROM employees
GROUP BY department, job_title;

```

This query:

-   Groups the data by both `department` and `job_title`.

-   Uses `COUNT(*)` to count how many employees hold each job title within each department.

#### Sample Result:

```
 department  | job_title | count
-------------+-----------+------
 Sales       | Manager   | 2
 Sales       | Associate | 8
 Marketing   | Manager   | 2
 HR          | Associate | 3

```

This result shows the count of employees grouped by both their department and job title.

* * * * *

### **`GROUP BY` with `HAVING`**

The **`HAVING`** clause is used to filter the results after the `GROUP BY` has been applied. It works like the `WHERE` clause but for groups.

#### Example 1: Filtering Groups Using `HAVING`

```
SELECT department, AVG(salary)
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;

```

In this query:

-   We group by `department`.

-   We use `AVG(salary)` to calculate the average salary for each department.

-   The `HAVING` clause filters out departments with an average salary less than or equal to 50,000.

#### Sample Result:

```
 department  | avg
-------------+------
 Sales       | 80000

```

In this case, only the `Sales` department has an average salary greater than 50,000.

#### Example 2: Filtering Groups with `HAVING` and `COUNT()`

```
SELECT department, COUNT(*)
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;

```

This filters departments that have more than 5 employees.

#### Sample Result:

```
 department  | count
-------------+-------
 Sales       | 10
 Marketing   | 8

```

* * * * *

### **Using `GROUP BY` with `ORDER BY`**

You can combine **`GROUP BY`** with **`ORDER BY`** to sort the results after grouping.

#### Example 1: Sorting by Grouped Aggregated Value

```
SELECT department, AVG(salary)
FROM employees
GROUP BY department
ORDER BY AVG(salary) DESC;

```

This query:

-   Groups the employees by department.

-   Calculates the average salary for each department.

-   Orders the results by the average salary in descending order.

#### Sample Result:

```
 department  | avg
-------------+------
 Sales       | 80000
 Marketing   | 50000
 HR          | 45000

```

* * * * *

### **`GROUP BY` Best Practices**

-   **Columns in `GROUP BY`**: All columns in the `SELECT` list that are not part of an aggregate function must be included in the `GROUP BY` clause.

-   **Use `HAVING` for Filtering Groups**: Always use `HAVING` to filter aggregated results after the `GROUP BY` operation.

-   **Performance Considerations**: Grouping can be resource-intensive, especially with large datasets. Make sure indexes are used appropriately on the columns being grouped.

* * * * *

### **Summary of Key Concepts**

-   **`GROUP BY`**: Used to group rows based on one or more columns and perform aggregate functions on those groups.

-   **Common Aggregate Functions**:

    -   `COUNT()`: Counts the number of rows.

    -   `SUM()`: Calculates the sum of numeric values.

    -   `AVG()`: Computes the average of numeric values.

    -   `MIN()`, `MAX()`: Finds the minimum and maximum values.

-   **`HAVING`**: Filters groups after the `GROUP BY` clause.

-   **`ORDER BY`**: Sorts the result set after grouping.

* * * * *

### **Exam Power-ups for Memory:**

-   **`GROUP BY`**: Always group by columns that are not part of aggregate functions.

-   **`HAVING`**: Use `HAVING` to filter grouped results, not `WHERE`.

-   **Performance**: Use **indexes** to speed up queries involving `GROUP BY`, especially when working with large datasets.

* * * * *

```
SELECT department, COUNT(*)
FROM employees
GROUP BY department;

```

### **Order of Execution in SQL Queries**

1.  **FROM**: The first step is the **`FROM`** clause, where the database retrieves data from the `employees` table. This is where the raw data comes from.

2.  **JOINs** (if any): If the query had any `JOIN` operations, they would be executed after the **`FROM`** clause. But since your query does not involve any `JOIN` operations, this step is skipped.

3.  **WHERE** (if any): If there were a **`WHERE`** clause in the query, it would filter rows based on the conditions specified. It applies before any grouping, so only the rows that meet the conditions would be considered for aggregation in the next step. Again, since this query does not have a `WHERE` clause, this step is skipped.

4.  **GROUP BY**: The **`GROUP BY`** clause groups the data based on the `department` column. This step organizes all rows from the `employees` table into groups where each group consists of employees in the same department.

5.  **HAVING** (if any): If there were a **`HAVING`** clause, it would filter the grouped results after **`GROUP BY`**. For example, you could have used **`HAVING COUNT(*) > 5`** to only return departments with more than 5 employees. However, since this query doesn't have a `HAVING` clause, this step is skipped.

6.  **SELECT**: The **`SELECT`** clause comes after **`GROUP BY`** and **`HAVING`** (if any). In this step, the database evaluates the expressions listed in the **`SELECT`** clause. Here, it is returning:

    -   The `department` column, which is part of the grouping.

    -   The count of rows (i.e., employees) per department, which is calculated by the `COUNT(*)` aggregate function.

7.  **ORDER BY** (if any): If the query had an **`ORDER BY`** clause, it would be executed next to sort the results. Since there is no **`ORDER BY`** clause in your query, this step is skipped.

8.  **LIMIT/OFFSET** (if any): The **`LIMIT`** and **`OFFSET`** clauses are applied last. These limit the number of rows returned by the query and control pagination (if used). Since there are no **`LIMIT`** or **`OFFSET`** clauses in your query, this step is also skipped.

### **Summary of Execution Order**

The execution order of your query is as follows:

1.  **FROM** (`employees` table) -- Retrieve data from the table.

2.  **GROUP BY** (`department`) -- Group the rows by the `department` column.

3.  **SELECT** -- For each group, calculate the `COUNT(*)` and return the `department` and count.

4.  **(Optional)** **HAVING** -- If applied, filters the grouped results.

5.  **(Optional)** **ORDER BY** -- If applied, sorts the results.

6.  **(Optional)** **LIMIT/OFFSET** -- If applied, limits the number of rows and controls pagination.

* * * * *

### **Order of Execution for Example Query:**

```
SELECT department, COUNT(*)
FROM employees
GROUP BY department;

```

1.  **FROM**: Retrieve all rows from the `employees` table.

2.  **GROUP BY**: Group the rows by `department`.

3.  **SELECT**: For each group (department), calculate the number of employees using `COUNT(*)`.

4.  **(No HAVING clause)**: No filtering is done on the grouped data.

5.  **(No ORDER BY clause)**: No sorting is done on the result.

6.  **(No LIMIT/OFFSET)**: No rows are excluded or limited.

* * * * *

### **Conclusion**

This is the standard execution order for SQL queries. Understanding the order helps ensure that you use clauses like **`WHERE`**, **`HAVING`**, **`ORDER BY`**, and others correctly.


### **1\. SELECT Statement Syntax Order**

The general syntax order for a **`SELECT`** query is as follows:

```
SELECT [DISTINCT] column1, column2, ...
FROM table_name
[WHERE condition]
[GROUP BY column1, column2, ...]
[HAVING condition]
[ORDER BY column1 [ASC|DESC], column2 [ASC|DESC], ...]
[LIMIT number] [OFFSET number];

```

Let's break it down step by step:

* * * * *

### **Order of SQL Clauses**

1.  **SELECT**:

    -   The **`SELECT`** clause defines the columns that will be returned by the query.

    -   Optionally, you can use **`DISTINCT`** to ensure that duplicate rows are eliminated from the result set.

    ```
    SELECT column1, column2, ...

    ```

2.  **FROM**:

    -   The **`FROM`** clause specifies the table(s) from which to retrieve the data.

    ```
    FROM table_name

    ```

3.  **WHERE**:

    -   The **`WHERE`** clause filters the rows before grouping or aggregation happens.

    -   This is where you can define conditions that need to be met for rows to be included in the query result.

    ```
    WHERE condition

    ```

4.  **GROUP BY**:

    -   The **`GROUP BY`** clause is used to group rows based on one or more columns. It's often used in conjunction with aggregate functions (e.g., `COUNT()`, `SUM()`, `AVG()`).

    ```
    GROUP BY column1, column2, ...

    ```

5.  **HAVING**:

    -   The **`HAVING`** clause is used to filter groups after they've been created by the **`GROUP BY`** clause. It's used when you want to filter based on aggregate functions.

    -   **`HAVING`** is applied **after** **`GROUP BY`**.

    ```
    HAVING condition

    ```

6.  **ORDER BY**:

    -   The **`ORDER BY`** clause is used to sort the result set based on one or more columns.

    -   You can specify **`ASC`** (ascending) or **`DESC`** (descending) order for sorting.

    ```
    ORDER BY column1 [ASC|DESC], column2 [ASC|DESC]

    ```

7.  **LIMIT**:

    -   The **`LIMIT`** clause restricts the number of rows returned by the query. It is commonly used for pagination or when you want to fetch a specific number of rows.

    -   **`LIMIT`** is applied **last** in the query execution process.

    ```
    LIMIT number

    ```

8.  **OFFSET**:

    -   The **`OFFSET`** clause is used to skip a specified number of rows before returning the result.

    -   It is often used for pagination, where you can combine **`LIMIT`** and **`OFFSET`** to fetch data in chunks (pages).

    ```
    OFFSET number

    ```

* * * * *

### **Example Query with All Clauses:**

Here's an example of a query that utilizes all of these clauses:

```
SELECT department, COUNT(*)
FROM employees
WHERE salary > 50000
GROUP BY department
HAVING COUNT(*) > 5
ORDER BY department ASC
LIMIT 10 OFFSET 20;

```

### **Explanation:**

1.  **`SELECT department, COUNT(*)`**: Specifies the columns to return, which in this case are `department` and the count of employees.

2.  **`FROM employees`**: Specifies the table from which to retrieve the data.

3.  **`WHERE salary > 50000`**: Filters out employees whose salary is less than or equal to 50,000.

4.  **`GROUP BY department`**: Groups the data by the `department` column.

5.  **`HAVING COUNT(*) > 5`**: Filters out departments that have fewer than 6 employees after grouping.

6.  **`ORDER BY department ASC`**: Sorts the result set by `department` in ascending order.

7.  **`LIMIT 10 OFFSET 20`**: Fetches 10 rows starting from the 21st row (for pagination).

* * * * *

### **Summary of Clause Order in SQL**

1.  **SELECT**

2.  **FROM**

3.  **WHERE**

4.  **GROUP BY**

5.  **HAVING**

6.  **ORDER BY**

7.  **LIMIT**

8.  **OFFSET**

### **Important Points to Remember:**

-   **`WHERE`** filters rows before grouping, while **`HAVING`** filters after grouping.

-   **`ORDER BY`** can be used to sort the final result set.

-   **`LIMIT`** and **`OFFSET`** are typically used for controlling the number of rows returned, especially in pagination scenarios.

* * * * *

### **Exam Power-ups for Memory:**

-   **Clause Order**: Always remember the correct order: **`SELECT` -> `FROM` -> `WHERE` -> `GROUP BY` -> `HAVING` -> `ORDER BY` -> `LIMIT/OFFSET`**.

-   **`HAVING` vs `WHERE`**: Use **`WHERE`** to filter individual rows, and **`HAVING`** to filter groups after aggregation.

-   **Pagination**: Use **`LIMIT`** and **`OFFSET`** for pagination when dealing with large datasets.

* * * * *


---

### **1. Alias (2:09:43)**

An **alias** is a temporary name given to a table or a column in a query. Aliases are often used to make queries more readable or when you need to refer to a table or column with a shorter name.

#### **For Columns**:
You can give a column a temporary name using the `AS` keyword.

#### Syntax:
```sql
SELECT column_name AS alias_name
FROM table_name;
```

#### Example 1: Column Alias for Calculated Column
```sql
SELECT salary * 12 AS yearly_salary
FROM employees;
```
Here, `yearly_salary` is an alias for the calculated column (`salary * 12`), making the result more understandable.

#### Sample Result:
```
 yearly_salary
---------------
 60000
 72000
 50000
```

#### Example 2: Table Alias
You can also create an alias for a table, especially when you are working with joins or long table names.

```sql
SELECT e.first_name, e.last_name, e.salary
FROM employees AS e;
```
Here, `e` is an alias for the `employees` table.

#### Sample Result:
```
 first_name | last_name | salary
------------+-----------+--------
 John       | Doe       | 50000
 Jane       | Smith     | 60000
```

---

### **2. COALESCE (2:12:32)**

The **`COALESCE()`** function returns the first non-`NULL` value in a list of arguments. It is used to handle `NULL` values in your query results and substitute them with a default value.

#### Syntax:
```sql
COALESCE(value1, value2, ..., valueN)
```

#### Example 1: Handling `NULL` Values in a Salary Column
```sql
SELECT first_name, COALESCE(salary, 30000) AS adjusted_salary
FROM employees;
```
If the `salary` is `NULL`, the **`COALESCE()`** function will substitute it with `30000`.

#### Sample Result:
```
 first_name | adjusted_salary
------------+-----------------
 John       | 50000
 Jane       | 60000
 Alice      | 30000
```

In this example, if an employee has a `NULL` salary, it is replaced by `30000`.

#### Example 2: Using `COALESCE()` for Multiple Columns
```sql
SELECT first_name, COALESCE(phone_number, email, 'No Contact Info') AS contact_info
FROM employees;
```
This returns the `phone_number` if available, otherwise the `email`, and if both are `NULL`, it returns `'No Contact Info'`.

#### Sample Result:
```
 first_name | contact_info
------------+-------------
 John       | 123-456-7890
 Jane       | jane@example.com
 Alice      | No Contact Info
```

---

### **3. NULLIF (2:16:15)**

The **`NULLIF()`** function returns `NULL` if two expressions are equal. Otherwise, it returns the first expression. This is useful to prevent division by zero or other conditions where you want to return `NULL` for certain cases.

#### Syntax:
```sql
NULLIF(expression1, expression2)
```

#### Example 1: Preventing Division by Zero
```sql
SELECT sales / NULLIF(quantity, 0) AS sales_per_item
FROM products;
```
Here, `NULLIF(quantity, 0)` ensures that if `quantity` is `0`, the division will not occur and will return `NULL`.

#### Sample Result:
```
 sales_per_item
-----------------
 500
 NULL
 250
```

In the second row, if `quantity` was `0`, the `NULLIF()` function prevents the division by zero and returns `NULL`.

#### Example 2: Handling Equal Values
```sql
SELECT name, NULLIF(salary, 0) AS non_zero_salary
FROM employees;
```
This query returns `NULL` for employees who have a salary of `0`.

#### Sample Result:
```
 name   | non_zero_salary
--------+-----------------
 John   | 50000
 Jane   | 60000
 Alice  | NULL
```

---

### **4. Timestamps and Dates Course (2:20:21)**

In PostgreSQL, **`DATE`** and **`TIMESTAMP`** types are used to store date and time information. `TIMESTAMP` stores both the date and time, while `DATE` only stores the date (year, month, and day).

#### **`DATE`**:
- Stores only the date (no time).
- Example: `2023-04-05`.

#### **`TIMESTAMP`**:
- Stores both the date and time.
- Example: `2023-04-05 14:30:00`.

#### Example 1: Using `DATE`
```sql
SELECT first_name, hire_date
FROM employees
WHERE hire_date > '2022-01-01';
```
This query selects employees hired after January 1, 2022.

#### Sample Result:
```
 first_name | hire_date
------------+-----------
 John       | 2023-03-15
 Jane       | 2022-06-20
```

#### Example 2: Using `TIMESTAMP`
```sql
SELECT first_name, hire_date, last_login
FROM employees
WHERE last_login > '2023-01-01 00:00:00';
```
This query selects employees who logged in after January 1, 2023, at midnight.

#### Sample Result:
```
 first_name | hire_date  | last_login
------------+------------+---------------------
 John       | 2023-03-15 | 2023-04-05 10:00:00
 Jane       | 2022-06-20 | 2023-03-30 08:30:00
```

---

### **5. Adding and Subtracting with Dates (2:23:21)**

In PostgreSQL, you can add or subtract intervals (such as days, months, years) to/from `DATE` or `TIMESTAMP` columns.

#### **Adding/Subtracting Days**

#### Example 1: Adding Days to a Date
```sql
SELECT first_name, hire_date, hire_date + INTERVAL '30 days' AS hire_plus_30_days
FROM employees;
```
This query adds 30 days to the `hire_date` for each employee.

#### Sample Result:
```
 first_name | hire_date  | hire_plus_30_days
------------+------------+-------------------
 John       | 2023-03-15 | 2023-04-14
 Jane       | 2022-06-20 | 2022-07-20
```

#### Example 2: Subtracting Days from a Date
```sql
SELECT first_name, hire_date, hire_date - INTERVAL '15 days' AS hire_minus_15_days
FROM employees;
```
This query subtracts 15 days from the `hire_date`.

#### Sample Result:
```
 first_name | hire_date  | hire_minus_15_days
------------+------------+-------------------
 John       | 2023-03-15 | 2023-03-01
 Jane       | 2022-06-20 | 2022-06-05
```

#### **Adding/Subtracting Time (e.g., Hours, Minutes)**

#### Example 3: Adding Time to a Timestamp
```sql
SELECT first_name, last_login, last_login + INTERVAL '2 hours' AS last_login_plus_2_hours
FROM employees;
```
This adds 2 hours to the `last_login` timestamp.

#### Sample Result:
```
 first_name | last_login          | last_login_plus_2_hours
------------+---------------------+-------------------------
 John       | 2023-04-05 10:00:00 | 2023-04-05 12:00:00
 Jane       | 2023-03-30 08:30:00 | 2023-03-30 10:30:00
```

---

### **Summary**

- **Alias**: Temporary names for columns and tables for easier reference.
- **COALESCE**: Returns the first non-`NULL` value in a list of expressions.
- **NULLIF**: Returns `NULL` if two expressions are equal, otherwise returns the first expression.
- **Timestamps and Dates**: Use `DATE` for dates, and `TIMESTAMP` for both date and time.
- **Adding/Subtracting Dates**: Use **`INTERVAL`** to add or subtract time (e.g., days, hours) to/from date or timestamp columns.

---

### **Exam Power-ups for Memory:**

- **Alias**: Use aliases to simplify complex queries or make them more readable.
- **COALESCE**: Great for replacing `NULL` values with default values, especially in financial reports.
- **NULLIF**: Prevents unwanted results like division by zero by returning `NULL` for specific conditions.
- **Timestamps**: Understand the difference between `DATE` and `TIMESTAMP`, and use **`INTERVAL`** to add or subtract time.
- **Date Arithmetic**: Date arithmetic with **`INTERVAL`** is a powerful tool for manipulating dates.

---

* * * * *

### **1\. Extracting Fields From Timestamp (2:25:58)**

In PostgreSQL, you can **extract specific components** (such as year, month, day, hour, minute, second) from a `TIMESTAMP` or `DATE` field using the **`EXTRACT()`** function. This allows you to isolate individual parts of a date or timestamp for analysis.

#### **Syntax of `EXTRACT()`**:

```
EXTRACT(field FROM source)

```

-   **`field`**: The specific part of the date or time you want to extract. Common fields include `year`, `month`, `day`, `hour`, `minute`, `second`, etc.

-   **`source`**: The `TIMESTAMP` or `DATE` column (or expression) from which to extract the field.

* * * * *

#### **Example 1: Extracting Year from a Timestamp**

```
SELECT first_name, last_login, EXTRACT(YEAR FROM last_login) AS year_joined
FROM employees;

```

This query extracts the **year** from the `last_login` column for each employee.

#### Sample Result:

```
 first_name | last_login          | year_joined
------------+---------------------+-------------
 John       | 2023-04-05 10:00:00 | 2023
 Jane       | 2022-03-30 08:30:00 | 2022
 Alice      | 2021-06-15 09:45:00 | 2021

```

#### **Example 2: Extracting Month and Day from a Timestamp**

```
SELECT first_name, last_login,
       EXTRACT(MONTH FROM last_login) AS month_joined,
       EXTRACT(DAY FROM last_login) AS day_joined
FROM employees;

```

This query extracts the **month** and **day** from the `last_login` timestamp.

#### Sample Result:

```
 first_name | last_login          | month_joined | day_joined
------------+---------------------+--------------+------------
 John       | 2023-04-05 10:00:00 | 4            | 5
 Jane       | 2022-03-30 08:30:00 | 3            | 30
 Alice      | 2021-06-15 09:45:00 | 6            | 15

```

#### **Example 3: Extracting Hour and Minute from a Timestamp**

```
SELECT first_name, last_login,
       EXTRACT(HOUR FROM last_login) AS hour_logged_in,
       EXTRACT(MINUTE FROM last_login) AS minute_logged_in
FROM employees;

```

This query extracts the **hour** and **minute** from the `last_login` timestamp.

#### Sample Result:

```
 first_name | last_login          | hour_logged_in | minute_logged_in
------------+---------------------+----------------+------------------
 John       | 2023-04-05 10:00:00 | 10             | 0
 Jane       | 2022-03-30 08:30:00 | 8              | 30
 Alice      | 2021-06-15 09:45:00 | 9              | 45

```

* * * * *

### **2\. Age Function (2:27:28)**

The **`AGE()`** function in PostgreSQL is used to calculate the interval (the difference) between two dates or timestamps. It is commonly used to calculate an **age** or the difference between a given date and the current date.

#### **Syntax of `AGE()`**:

```
AGE(timestamp1, timestamp2)

```

-   **`timestamp1`**: The first timestamp or date (the later date).

-   **`timestamp2`**: The second timestamp or date (the earlier date).

If you only pass one argument to **`AGE()`**, it will calculate the interval between the provided timestamp and the current date and time.

* * * * *

#### **Example 1: Calculating Age Based on Birthdate**

Let's calculate the **age** of employees based on their `birthdate` column.

```
SELECT first_name, birthdate, AGE(birthdate) AS age
FROM employees;

```

This query calculates the age by subtracting the `birthdate` from the current date.

#### Sample Result:

```
 first_name | birthdate  | age
------------+------------+---------------------
 John       | 1990-05-15 | 32 years 11 mons 20 days
 Jane       | 1985-06-25 | 37 years 9 mons 10 days
 Alice      | 2000-08-10 | 22 years 8 mons 5 days

```

#### **Example 2: Calculating Time Difference Between Two Timestamps**

```
SELECT first_name, last_login, AGE(last_login) AS time_since_last_login
FROM employees;

```

This calculates the difference between the `last_login` timestamp and the current date.

#### Sample Result:

```
 first_name | last_login          | time_since_last_login
------------+---------------------+------------------------
 John       | 2023-04-05 10:00:00 | 0 years 0 mons 1 days
 Jane       | 2022-03-30 08:30:00 | 1 years 0 mons 6 days
 Alice      | 2021-06-15 09:45:00 | 1 years 9 mons 19 days

```

* * * * *

### **Summary of Key Concepts**

-   **`EXTRACT()`**: Extracts specific fields (e.g., year, month, day, hour) from `TIMESTAMP` or `DATE` values.

-   **`AGE()`**: Calculates the difference (interval) between two dates or timestamps, or between a date and the current date.

* * * * *

### **Exam Power-ups for Memory:**

-   **`EXTRACT()`**: Remember that **`EXTRACT(field FROM source)`** lets you break down a timestamp into components. It's useful for time-based analysis like calculating yearly, monthly, or daily trends.

-   **`AGE()`**: Use **`AGE()`** to easily calculate the time difference, such as calculating someone's age or the time elapsed between two timestamps.

* * * * *
* * * * *

### **1\. What Are Primary Keys? (2:29:24)**

A **primary key** is a column (or combination of columns) in a database table that uniquely identifies each row in that table. A primary key must have the following properties:

-   **Uniqueness**: No two rows can have the same value for the primary key column(s).

-   **Non-nullability**: The primary key column(s) cannot contain `NULL` values, ensuring that each row has a valid identifier.

### **Why are Primary Keys Important?**

-   **Uniquely Identifies Records**: It provides a way to uniquely identify each record in a table, ensuring data integrity.

-   **Enforces Data Integrity**: By ensuring that no duplicate or null values are inserted into the primary key column, it helps maintain the integrity of the database.

-   **Helps with Relationships**: The primary key is often used to establish relationships between tables, typically by being referenced as a foreign key in another table.

* * * * *

### **2\. Understanding Primary Keys (2:31:23)**

A primary key is not only a unique identifier for a row but also plays a crucial role in ensuring data relationships in relational databases.

#### **Types of Primary Keys**:

-   **Single Column Primary Key**: A single column is used as the primary key.

-   **Composite Primary Key**: A combination of two or more columns is used to form a primary key. This is useful when no single column can uniquely identify a row.

#### **Example 1: Single Column Primary Key**

```
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    hire_date DATE
);

```

Here, the **`employee_id`** column is the primary key, and it ensures that each employee has a unique identifier.

#### **Example 2: Composite Primary Key**

```
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

```

In this case, the combination of **`order_id`** and **`product_id`** serves as the primary key. This ensures that each product in an order is uniquely identified.

#### Sample Result:

-   In the first example, `employee_id` uniquely identifies each employee.

-   In the second example, the combination of `order_id` and `product_id` uniquely identifies each item in an order.

* * * * *

### **3\. Adding Primary Key (2:36:26)**

You can define a primary key when you create a table, or you can add it to an existing table using the `ALTER TABLE` statement.

#### **Syntax to Add Primary Key When Creating a Table:**

```
CREATE TABLE table_name (
    column1 datatype PRIMARY KEY,
    column2 datatype,
    ...
);

```

#### **Example 1: Adding Primary Key When Creating a Table**

```
CREATE TABLE students (
    student_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    date_of_birth DATE
);

```

In this example, **`student_id`** is set as the primary key when the table is created.

#### **Syntax to Add Primary Key to an Existing Table:**

```
ALTER TABLE table_name
ADD CONSTRAINT constraint_name PRIMARY KEY (column1, column2, ...);

```

#### **Example 2: Adding Primary Key to Existing Table**

```
ALTER TABLE employees
ADD CONSTRAINT pk_employee_id PRIMARY KEY (employee_id);

```

This query adds a primary key to the `employee_id` column of the existing `employees` table.

#### Sample Result:

-   When adding a primary key to a table, PostgreSQL will enforce the uniqueness and non-nullability of the specified column(s).

-   The query will execute successfully if the data in the table does not violate the primary key constraints.

* * * * *

### **4\. Unique Constraints (2:40:55)**

A **unique constraint** ensures that the values in a column (or combination of columns) are unique across all rows in a table. Unlike primary keys, unique constraints allow `NULL` values but ensure that all non-`NULL` values are distinct.

#### **Syntax**:

```
CREATE TABLE table_name (
    column_name datatype UNIQUE,
    ...
);

```

#### **Example 1: Adding a Unique Constraint on a Column**

```
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    first_name VARCHAR(100),
    last_name VARCHAR(100)
);

```

Here, **`email`** has a unique constraint, ensuring that no two customers can have the same email address.

#### **Example 2: Adding a Unique Constraint to an Existing Table**

```
ALTER TABLE customers
ADD CONSTRAINT unique_email UNIQUE (email);

```

This adds a unique constraint to the `email` column in the existing `customers` table.

#### Sample Result:

-   If you attempt to insert a duplicate value in a column with a unique constraint, PostgreSQL will raise an error.

Example of error:

```
ERROR: duplicate key value violates unique constraint "unique_email"

```

* * * * *

### **5\. Check Constraints (2:49:15)**

A **check constraint** ensures that values in a column meet a specific condition. It's used to enforce domain integrity by limiting the values that can be stored in a column.

#### **Syntax**:

```
CREATE TABLE table_name (
    column_name datatype CHECK (condition),
    ...
);

```

#### **Example 1: Adding a Check Constraint for Age**

```
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    birthdate DATE,
    age INT CHECK (age >= 18)
);

```

This ensures that the `age` column only accepts values greater than or equal to 18.

#### **Example 2: Adding a Check Constraint for Valid Salary**

```
ALTER TABLE employees
ADD CONSTRAINT check_salary CHECK (salary >= 30000);

```

This adds a check constraint that ensures that the `salary` column has a value greater than or equal to 30,000.

#### Sample Result:

-   If you try to insert a value that doesn't satisfy the check condition, PostgreSQL will raise an error.

Example of error:

```
ERROR: new row for relation "employees" violates check constraint "check_salary"

```

* * * * *

### **Summary of Key Concepts**

-   **Primary Keys**: Ensure each row in a table has a unique identifier and prevent `NULL` values.

    -   **Composite Primary Keys**: Use multiple columns to create a unique identifier for each row.

-   **Adding Primary Key**: You can define a primary key when creating a table or add it later using `ALTER TABLE`.

-   **Unique Constraints**: Ensure uniqueness of column values, but unlike primary keys, allow `NULL` values.

-   **Check Constraints**: Ensure that the values in a column meet specific conditions.

* * * * *

### **Exam Power-ups for Memory:**

-   **Primary Key**: Remember that **primary keys** are **unique** and **non-nullable**. They help maintain data integrity and establish relationships.

-   **Unique Constraints**: **Unique constraints** allow `NULL` values, but enforce uniqueness for non-`NULL` values.

-   **Check Constraints**: Use **check constraints** to enforce domain rules, such as limiting values to a specific range.

* * * * *

* * * * *

### **1\. `ALTER TABLE` Statement**

The **`ALTER TABLE`** statement is used to modify the structure of an existing table. It can be used for several operations:

-   **Adding Columns**

-   **Dropping Columns**

-   **Renaming Columns**

-   **Modifying Columns**

-   **Adding Constraints**

-   **Dropping Constraints**

* * * * *

### **2\. Adding Columns with `ALTER TABLE`**

You can use **`ALTER TABLE`** to add a new column to an existing table.

#### **Syntax**:

```
ALTER TABLE table_name
ADD column_name datatype;

```

#### **Example 1: Adding a Single Column**

```
ALTER TABLE employees
ADD middle_name VARCHAR(100);

```

This adds a `middle_name` column to the `employees` table. The `middle_name` column is of type `VARCHAR(100)`.

#### **Sample Result**:

-   The `employees` table will now have an additional `middle_name` column.

#### **Example 2: Adding Multiple Columns**

```
ALTER TABLE employees
ADD phone_number VARCHAR(20),
    hire_date DATE;

```

This adds two new columns: `phone_number` and `hire_date` to the `employees` table.

#### **Sample Result**:

-   The `employees` table will now have both the `phone_number` and `hire_date` columns.

* * * * *

### **3\. Dropping Columns with `ALTER TABLE`**

You can use **`ALTER TABLE`** to remove a column from a table.

#### **Syntax**:

```
ALTER TABLE table_name
DROP COLUMN column_name;

```

#### **Example 1: Dropping a Column**

```
ALTER TABLE employees
DROP COLUMN middle_name;

```

This removes the `middle_name` column from the `employees` table.

#### **Sample Result**:

-   The `middle_name` column will be removed from the `employees` table.

* * * * *

### **4\. Renaming Columns with `ALTER TABLE`**

You can use **`ALTER TABLE`** to rename an existing column.

#### **Syntax**:

```
ALTER TABLE table_name
RENAME COLUMN old_column_name TO new_column_name;

```

#### **Example 1: Renaming a Column**

```
ALTER TABLE employees
RENAME COLUMN hire_date TO employment_date;

```

This renames the `hire_date` column to `employment_date` in the `employees` table.

#### **Sample Result**:

-   The `hire_date` column will now be renamed to `employment_date`.

* * * * *

### **5\. Modifying Columns with `ALTER TABLE`**

You can modify an existing column's data type or other properties.

#### **Syntax**:

```
ALTER TABLE table_name
ALTER COLUMN column_name SET DATA TYPE new_datatype;

```

#### **Example 1: Modifying a Column's Data Type**

```
ALTER TABLE employees
ALTER COLUMN salary SET DATA TYPE DECIMAL(15, 2);

```

This changes the `salary` column to `DECIMAL(15, 2)`.

#### **Sample Result**:

-   The `salary` column now has the new data type `DECIMAL(15, 2)`.

#### **Example 2: Modifying a Column to Not Null**

```
ALTER TABLE employees
ALTER COLUMN email SET NOT NULL;

```

This modifies the `email` column to enforce that it cannot have `NULL` values.

#### **Sample Result**:

-   The `email` column now enforces the `NOT NULL` constraint.

* * * * *

### **6\. Adding Constraints with `ALTER TABLE`**

You can use **`ALTER TABLE`** to add various types of constraints, such as `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, or `UNIQUE`.

#### **Syntax for Adding Constraints**:

```
ALTER TABLE table_name
ADD CONSTRAINT constraint_name constraint_type (column_name);

```

#### **Example 1: Adding a Primary Key**

```
ALTER TABLE employees
ADD CONSTRAINT pk_employee_id PRIMARY KEY (employee_id);

```

This adds a `PRIMARY KEY` constraint on the `employee_id` column in the `employees` table.

#### **Example 2: Adding a Foreign Key Constraint**

```
ALTER TABLE orders
ADD CONSTRAINT fk_customer_id FOREIGN KEY (customer_id)
REFERENCES customers(customer_id);

```

This adds a `FOREIGN KEY` constraint on the `customer_id` column in the `orders` table, referencing the `customer_id` column in the `customers` table.

#### **Example 3: Adding a Unique Constraint**

```
ALTER TABLE employees
ADD CONSTRAINT unique_email UNIQUE (email);

```

This adds a `UNIQUE` constraint to the `email` column to ensure all email addresses are unique.

#### **Sample Result**:

-   The specified constraints are added to the table, and PostgreSQL ensures they are enforced when inserting or updating rows.

* * * * *

### **7\. Dropping Constraints with `ALTER TABLE`**

You can use **`ALTER TABLE`** to remove constraints from a table.

#### **Syntax**:

```
ALTER TABLE table_name
DROP CONSTRAINT constraint_name;

```

#### **Example 1: Dropping a Primary Key**

```
ALTER TABLE employees
DROP CONSTRAINT pk_employee_id;

```

This removes the `PRIMARY KEY` constraint on the `employee_id` column.

#### **Example 2: Dropping a Foreign Key**

```
ALTER TABLE orders
DROP CONSTRAINT fk_customer_id;

```

This removes the `FOREIGN KEY` constraint from the `customer_id` column in the `orders` table.

#### **Sample Result**:

-   The specified constraints are dropped, and PostgreSQL will no longer enforce those rules.

* * * * *

### **8\. Renaming a Table with `ALTER TABLE`**

You can rename a table using the `ALTER TABLE` statement.

#### **Syntax**:

```
ALTER TABLE old_table_name
RENAME TO new_table_name;

```

#### **Example 1: Renaming a Table**

```
ALTER TABLE employees
RENAME TO staff;

```

This renames the `employees` table to `staff`.

#### **Sample Result**:

-   The table `employees` is now renamed to `staff`.

* * * * *

### **Summary of Key Concepts for `ALTER` Table**

-   **Adding Columns**: Use `ADD COLUMN` to add a new column to an existing table.

-   **Dropping Columns**: Use `DROP COLUMN` to remove an existing column.

-   **Renaming Columns**: Use `RENAME COLUMN` to change a column's name.

-   **Modifying Columns**: Use `ALTER COLUMN` to modify a column's data type or constraints.

-   **Adding Constraints**: You can add primary keys, foreign keys, unique, and check constraints using `ADD CONSTRAINT`.

-   **Dropping Constraints**: Use `DROP CONSTRAINT` to remove constraints.

-   **Renaming Tables**: Use `RENAME TO` to rename a table.

* * * * *

### **Exam Power-ups for Memory:**

-   **`ALTER TABLE`** is versatile: Use it to add, drop, modify columns, and manage constraints.

-   **Adding Constraints**: Use `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, and `UNIQUE` constraints to enforce data integrity.

-   **Renaming Columns/Tables**: Remember, you can rename columns and tables as needed, but it's important not to disrupt relationships with foreign keys or dependent objects.

-   **Modifying Columns**: You can change column types (e.g., data type) and constraints, but always ensure there is no data conflict (like changing from `INTEGER` to `DATE`).

* * * * *

* * * * *

### **1\. What Are Primary Keys? (2:29:24)**

A **primary key** is a column (or combination of columns) in a database table that uniquely identifies each row in that table. A primary key must have the following properties:

-   **Uniqueness**: No two rows can have the same value for the primary key column(s).

-   **Non-nullability**: The primary key column(s) cannot contain `NULL` values, ensuring that each row has a valid identifier.

### **Why are Primary Keys Important?**

-   **Uniquely Identifies Records**: It provides a way to uniquely identify each record in a table, ensuring data integrity.

-   **Enforces Data Integrity**: By ensuring that no duplicate or null values are inserted into the primary key column, it helps maintain the integrity of the database.

-   **Helps with Relationships**: The primary key is often used to establish relationships between tables, typically by being referenced as a foreign key in another table.

* * * * *

### **2\. Understanding Primary Keys (2:31:23)**

A primary key is not only a unique identifier for a row but also plays a crucial role in ensuring data relationships in relational databases.

#### **Types of Primary Keys**:

-   **Single Column Primary Key**: A single column is used as the primary key.

-   **Composite Primary Key**: A combination of two or more columns is used to form a primary key. This is useful when no single column can uniquely identify a row.

#### **Example 1: Single Column Primary Key**

```
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    hire_date DATE
);

```

Here, the **`employee_id`** column is the primary key, and it ensures that each employee has a unique identifier.

#### **Example 2: Composite Primary Key**

```
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

```

In this case, the combination of **`order_id`** and **`product_id`** serves as the primary key. This ensures that each product in an order is uniquely identified.

#### Sample Result:

-   In the first example, `employee_id` uniquely identifies each employee.

-   In the second example, the combination of `order_id` and `product_id` uniquely identifies each item in an order.

* * * * *

### **3\. Adding Primary Key (2:36:26)**

You can define a primary key when you create a table, or you can add it to an existing table using the `ALTER TABLE` statement.

#### **Syntax to Add Primary Key When Creating a Table:**

```
CREATE TABLE table_name (
    column1 datatype PRIMARY KEY,
    column2 datatype,
    ...
);

```

#### **Example 1: Adding Primary Key When Creating a Table**

```
CREATE TABLE students (
    student_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    date_of_birth DATE
);

```

In this example, **`student_id`** is set as the primary key when the table is created.

#### **Syntax to Add Primary Key to an Existing Table:**

```
ALTER TABLE table_name
ADD CONSTRAINT constraint_name PRIMARY KEY (column1, column2, ...);

```

#### **Example 2: Adding Primary Key to Existing Table**

```
ALTER TABLE employees
ADD CONSTRAINT pk_employee_id PRIMARY KEY (employee_id);

```

This query adds a primary key to the `employee_id` column of the existing `employees` table.

#### Sample Result:

-   When adding a primary key to a table, PostgreSQL will enforce the uniqueness and non-nullability of the specified column(s).

-   The query will execute successfully if the data in the table does not violate the primary key constraints.

* * * * *

### **4\. Unique Constraints (2:40:55)**

A **unique constraint** ensures that the values in a column (or combination of columns) are unique across all rows in a table. Unlike primary keys, unique constraints allow `NULL` values but ensure that all non-`NULL` values are distinct.

#### **Syntax**:

```
CREATE TABLE table_name (
    column_name datatype UNIQUE,
    ...
);

```

#### **Example 1: Adding a Unique Constraint on a Column**

```
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    first_name VARCHAR(100),
    last_name VARCHAR(100)
);

```

Here, **`email`** has a unique constraint, ensuring that no two customers can have the same email address.

#### **Example 2: Adding a Unique Constraint to an Existing Table**

```
ALTER TABLE customers
ADD CONSTRAINT unique_email UNIQUE (email);

```

This adds a unique constraint to the `email` column in the existing `customers` table.

#### Sample Result:

-   If you attempt to insert a duplicate value in a column with a unique constraint, PostgreSQL will raise an error.

Example of error:

```
ERROR: duplicate key value violates unique constraint "unique_email"

```

* * * * *

### **5\. Check Constraints (2:49:15)**

A **check constraint** ensures that values in a column meet a specific condition. It's used to enforce domain integrity by limiting the values that can be stored in a column.

#### **Syntax**:

```
CREATE TABLE table_name (
    column_name datatype CHECK (condition),
    ...
);

```

#### **Example 1: Adding a Check Constraint for Age**

```
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    birthdate DATE,
    age INT CHECK (age >= 18)
);

```

This ensures that the `age` column only accepts values greater than or equal to 18.

#### **Example 2: Adding a Check Constraint for Valid Salary**

```
ALTER TABLE employees
ADD CONSTRAINT check_salary CHECK (salary >= 30000);

```

This adds a check constraint that ensures that the `salary` column has a value greater than or equal to 30,000.

#### Sample Result:

-   If you try to insert a value that doesn't satisfy the check condition, PostgreSQL will raise an error.

Example of error:

```
ERROR: new row for relation "employees" violates check constraint "check_salary"

```

* * * * *

### **Summary of Key Concepts**

-   **Primary Keys**: Ensure each row in a table has a unique identifier and prevent `NULL` values.

    -   **Composite Primary Keys**: Use multiple columns to create a unique identifier for each row.

-   **Adding Primary Key**: You can define a primary key when creating a table or add it later using `ALTER TABLE`.

-   **Unique Constraints**: Ensure uniqueness of column values, but unlike primary keys, allow `NULL` values.

-   **Check Constraints**: Ensure that the values in a column meet specific conditions.

* * * * *

### **Exam Power-ups for Memory:**

-   **Primary Key**: Remember that **primary keys** are **unique** and **non-nullable**. They help maintain data integrity and establish relationships.

-   **Unique Constraints**: **Unique constraints** allow `NULL` values, but enforce uniqueness for non-`NULL` values.

-   **Check Constraints**: Use **check constraints** to enforce domain rules, such as limiting values to a specific range.

* * * * *