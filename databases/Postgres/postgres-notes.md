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

* * * * *

### **1\. Unique Constraints (2:40:55)**

A **unique constraint** ensures that all values in a column or a combination of columns are distinct across all rows in a table. It's similar to the **primary key** but has a key difference: a unique constraint allows `NULL` values, while a primary key does not.

#### **Why Use Unique Constraints?**

-   **Ensure Data Uniqueness**: They ensure that no two rows in a table can have the same value for the constrained column(s), but allow `NULL` values.

-   **Flexibility**: Unlike primary keys, multiple `NULL` values are allowed in columns with a unique constraint (since `NULL` is not considered equal to another `NULL`).

* * * * *

#### **Syntax for Unique Constraint**:

```
CREATE TABLE table_name (
    column_name datatype UNIQUE
);

```

Or, to add a unique constraint to an existing table:

```
ALTER TABLE table_name
ADD CONSTRAINT constraint_name UNIQUE (column_name);

```

* * * * *

#### **Example 1: Adding a Unique Constraint When Creating a Table**

```
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    first_name VARCHAR(100),
    last_name VARCHAR(100)
);

```

Here, the **`email`** column is constrained to be unique, meaning no two users can have the same email address.

#### Sample Result:

-   If you try to insert two rows with the same `email`, PostgreSQL will throw a **duplicate key error**.

```
INSERT INTO users (email, first_name, last_name) VALUES
('john.doe@example.com', 'John', 'Doe'),
('john.doe@example.com', 'Jane', 'Doe');

```

Error:

```
ERROR:  duplicate key value violates unique constraint "users_email_key"

```

* * * * *

#### **Example 2: Adding a Unique Constraint to an Existing Table**

```
ALTER TABLE users
ADD CONSTRAINT unique_email UNIQUE (email);

```

This adds a unique constraint to the `email` column in the `users` table after the table has been created.

#### Sample Result:

-   The constraint ensures that all email addresses in the `users` table are unique.

* * * * *

#### **Example 3: Composite Unique Constraint**

You can also apply the unique constraint to a combination of columns (a composite unique constraint).

```
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    product_id INT,
    UNIQUE (customer_id, product_id)
);

```

In this case, the combination of `customer_id` and `product_id` must be unique, but you can have multiple rows for the same `customer_id` as long as the `product_id` is different.

#### Sample Result:

-   If you attempt to insert two orders with the same `customer_id` and `product_id`, it will violate the unique constraint.

* * * * *

### **2\. Check Constraints (2:49:15)**

A **check constraint** ensures that values in a column meet a specific condition. It allows you to define custom validation rules for your data, ensuring that the data in your database conforms to certain criteria.

#### **Why Use Check Constraints?**

-   **Data Integrity**: Ensures that the values in a column are within a specific range or meet a particular condition (e.g., a positive number or valid date).

-   **Prevent Invalid Data**: It helps prevent the insertion of invalid data that might otherwise not be caught by other constraints.

* * * * *

#### **Syntax for Check Constraint**:

```
CREATE TABLE table_name (
    column_name datatype CHECK (condition)
);

```

Or, to add a check constraint to an existing table:

```
ALTER TABLE table_name
ADD CONSTRAINT constraint_name CHECK (condition);

```

* * * * *

#### **Example 1: Adding a Check Constraint for Age**

Let's say you want to make sure that the `age` column only accepts values greater than or equal to 18 (no one can be younger than 18).

```
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    age INT CHECK (age >= 18)
);

```

This `CHECK` constraint ensures that no employee can be younger than 18 years old.

#### Sample Result:

-   If you try to insert an employee with an age less than 18, PostgreSQL will reject the insertion.

```
INSERT INTO employees (name, age) VALUES ('John Doe', 17);

```

Error:

```
ERROR:  new row for relation "employees" violates check constraint "employees_age_check"

```

* * * * *

#### **Example 2: Adding a Check Constraint for Salary Range**

Let's say you want to ensure that the `salary` column has values between `30,000` and `200,000`.

```
ALTER TABLE employees
ADD CONSTRAINT salary_check CHECK (salary BETWEEN 30000 AND 200000);

```

This check constraint ensures that any salary inserted into the `employees` table must be between `30,000` and `200,000`.

#### Sample Result:

-   If you try to insert a salary outside this range, PostgreSQL will reject the insertion.

```
INSERT INTO employees (name, salary) VALUES ('Jane Doe', 25000);

```

Error:

```
ERROR:  new row for relation "employees" violates check constraint "salary_check"

```

* * * * *

#### **Example 3: Check Constraint with String Length**

You can also use check constraints to enforce string length conditions.

```
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100) CHECK (LENGTH(product_name) > 3)
);

```

This ensures that the `product_name` column must have more than 3 characters.

#### Sample Result:

-   If you try to insert a product with a name that has fewer than 4 characters, PostgreSQL will reject the insertion.

```
INSERT INTO products (product_name) VALUES ('TV');

```

Error:

```
ERROR:  new row for relation "products" violates check constraint "products_product_name_check"

```

* * * * *

### **Summary of Key Concepts**

-   **Unique Constraints**: Ensure that all values in a column (or combination of columns) are distinct across all rows in a table. **Allow `NULL` values**, but **ensure non-`NULL` values are unique**.

-   **Check Constraints**: Define conditions that the data must satisfy. They are used to enforce domain integrity by allowing only values that meet specific criteria (e.g., ranges, string lengths, valid dates).

* * * * *

### **Exam Power-ups for Memory**:

-   **Unique Constraints**: They ensure **data uniqueness**, but unlike **primary keys**, they **allow `NULL` values**.

-   **Check Constraints**: These are useful for **validating data** and ensuring that only **valid** values are inserted into the table. You can apply complex conditions (e.g., ranges, comparisons).

-   **Composite Unique Constraint**: Allows enforcing uniqueness over multiple columns, not just one.

* * * * *


* * * * *

### **1\. How to Delete Records (2:54:45)**

The **`DELETE`** statement is used to remove one or more rows from a table based on a specified condition. When using the `DELETE` statement, it's essential to specify the correct condition using the **`WHERE`** clause, otherwise, you could accidentally delete all rows in the table.

#### **Syntax**:

```
DELETE FROM table_name
WHERE condition;

```

-   **`table_name`**: The name of the table from which you want to delete records.

-   **`condition`**: A condition that specifies which rows should be deleted. Without the `WHERE` clause, **all records** in the table will be deleted.

#### **Example 1: Deleting a Single Record**

```
DELETE FROM employees
WHERE employee_id = 10;

```

This query deletes the employee with the `employee_id` of 10 from the `employees` table.

#### Sample Result:

-   After running the query, the employee with `employee_id = 10` will be removed from the table.

#### **Example 2: Deleting Multiple Records**

```
DELETE FROM employees
WHERE department = 'Sales';

```

This query deletes all employees in the `Sales` department from the `employees` table.

#### Sample Result:

-   All rows with `department = 'Sales'` will be deleted from the table.

#### **Example 3: Deleting All Records from a Table (Without Deleting the Table)**

```
DELETE FROM employees;

```

This query deletes **all rows** from the `employees` table but **keeps the structure** of the table intact.

#### Sample Result:

-   After running this query, the table will be empty, but the table itself will remain in the database.

#### **Important Considerations for `DELETE`**:

-   **`WHERE` Clause**: Always use a `WHERE` clause to prevent deleting all rows in the table.

-   **Performance**: Deleting many rows can be resource-intensive, especially on large tables. Consider using **`TRUNCATE`** if you need to delete all rows quickly, but note that `TRUNCATE` is a more permanent operation (i.e., it does not fire triggers).

* * * * *

### **2\. How to Update Records (3:01:36)**

The **`UPDATE`** statement is used to modify existing records in a table. Similar to `DELETE`, it's important to specify a **`WHERE`** condition to target only the rows you want to update. Without the `WHERE` clause, all rows in the table will be updated.

#### **Syntax**:

```
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;

```

-   **`table_name`**: The name of the table where the update will be applied.

-   **`column1 = value1`**: Specifies the column to update and the new value.

-   **`condition`**: The condition to specify which rows should be updated.

#### **Example 1: Updating a Single Record**

```
UPDATE employees
SET salary = 70000
WHERE employee_id = 5;

```

This query updates the salary of the employee with `employee_id = 5` to `70000`.

#### Sample Result:

-   The salary of the employee with `employee_id = 5` will be updated to `70000`.

#### **Example 2: Updating Multiple Records**

```
UPDATE employees
SET department = 'Marketing'
WHERE department = 'Sales';

```

This query updates the department of all employees who are currently in the `Sales` department to `Marketing`.

#### Sample Result:

-   All rows with `department = 'Sales'` will be updated to `department = 'Marketing'`.

#### **Example 3: Updating Multiple Columns**

```
UPDATE employees
SET salary = 75000, department = 'HR'
WHERE employee_id = 10;

```

This query updates both the `salary` and `department` columns for the employee with `employee_id = 10`.

#### Sample Result:

-   The `salary` will be updated to `75000`, and the `department` will be updated to `HR` for the employee with `employee_id = 10`.

* * * * *

#### **Example 4: Updating All Records in a Table (Be Careful!)**

```
UPDATE employees
SET department = 'General';

```

This query updates the **`department`** column for **all rows** in the `employees` table, setting the value of `department` to `'General'`.

#### Sample Result:

-   All employees will now have the department set to `General`.

#### **Important Considerations for `UPDATE`**:

-   **`WHERE` Clause**: Always use a `WHERE` clause to limit which rows will be updated. Without it, **all rows** in the table will be updated.

-   **Performance**: Updating many rows, especially in large tables, can be resource-intensive. Make sure the columns you are updating are indexed if you're applying filters on them (e.g., in the `WHERE` clause).

-   **Atomicity**: The `UPDATE` statement is atomic; all updates either happen fully or not at all. If an error occurs while updating, no changes are made to the table.

* * * * *

### **Combining `DELETE` and `UPDATE` in Practice**

#### Example: Deleting Specific Records After an Update

Suppose you want to **increase the salary** of employees in the `Sales` department and then **delete employees** who have a salary above a certain threshold:

```
UPDATE employees
SET salary = salary * 1.1
WHERE department = 'Sales';

DELETE FROM employees
WHERE salary > 80000;

```

In this case:

-   The first query updates the salary of all employees in the `Sales` department by **10%**.

-   The second query deletes employees whose updated salary is greater than **80,000**.

#### Sample Result:

-   After running both queries, employees in the `Sales` department will have their salary increased by 10%, and those with a salary above 80,000 will be removed from the table.

* * * * *

### **Summary of Key Concepts**

-   **DELETE**:

    -   Removes rows from a table.

    -   Be sure to use the `WHERE` clause to specify which rows to delete. Without it, all rows will be deleted.

    -   **Performance Tip**: If you need to delete all rows quickly, consider using **`TRUNCATE`** instead of `DELETE`.

-   **UPDATE**:

    -   Modifies existing records in a table.

    -   Always use the `WHERE` clause to avoid updating all rows.

    -   You can update multiple columns at once.

* * * * *

### **Exam Power-ups for Memory**:

-   **`DELETE`**: Always **use a `WHERE` clause** with `DELETE` to prevent removing all data. If you need to delete all rows, consider using **`TRUNCATE`** for better performance.

-   **`UPDATE`**: Use **`WHERE`** to limit updates to specific records. It's also helpful to update multiple columns in one go (e.g., `salary` and `department`).

-   **Transactional Integrity**: Both **`UPDATE`** and **`DELETE`** are atomic operations, meaning they either complete fully or not at all.

* * * * *

* * * * *

### **1\. On Conflict Do Nothing (3:05:55)**

In PostgreSQL, the **`ON CONFLICT DO NOTHING`** clause is used in **`INSERT`** statements to handle **conflicts** that arise when inserting a row that would violate a constraint (such as a **unique constraint** or a **primary key constraint**).

When a conflict occurs, instead of throwing an error, **`DO NOTHING`** will simply ignore the conflicting insert and proceed without making any changes to the table.

#### **Why Use `ON CONFLICT DO NOTHING`?**

-   **Prevent Duplicate Inserts**: It is particularly useful when you want to avoid inserting duplicate records without raising an error.

-   **Efficient Bulk Inserts**: When performing bulk inserts, you might encounter duplicate keys (e.g., inserting the same rows from multiple sources). Using **`DO NOTHING`** avoids errors from duplicate rows.

#### **Syntax**:

```
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...)
ON CONFLICT (column_name) DO NOTHING;

```

-   **`column_name`**: The column(s) that are subject to a conflict (typically the unique or primary key column).

-   **`DO NOTHING`**: The action to take when a conflict occurs --- no changes are made.

* * * * *

#### **Example 1: Inserting Data with Conflict Handling**

Let's say we have a `users` table where the `email` column is unique, and we want to insert data while avoiding duplicates.

```
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    first_name VARCHAR(100),
    last_name VARCHAR(100)
);

-- Inserting users
INSERT INTO users (email, first_name, last_name)
VALUES
('john.doe@example.com', 'John', 'Doe'),
('jane.smith@example.com', 'Jane', 'Smith')
ON CONFLICT (email) DO NOTHING;

```

In this case:

-   The `email` column is unique, so if an attempt is made to insert a user with an existing email, PostgreSQL will **ignore** that insertion instead of raising an error.

-   The query will **insert** the users whose emails don't conflict and **do nothing** for those whose emails already exist in the table.

#### Sample Result:

-   If the email `'john.doe@example.com'` already exists, that row will be ignored, and only the non-conflicting rows will be inserted.

* * * * *

#### **Example 2: Bulk Insert with Conflict Handling**

```
INSERT INTO users (email, first_name, last_name)
VALUES
('john.doe@example.com', 'John', 'Doe'),
('alice.jones@example.com', 'Alice', 'Jones'),
('bob.brown@example.com', 'Bob', 'Brown')
ON CONFLICT (email) DO NOTHING;

```

In this case:

-   If `john.doe@example.com` already exists, it will be ignored, and the new users `'alice.jones@example.com'` and `'bob.brown@example.com'` will be inserted.

#### Sample Result:

-   Only the rows with unique emails will be inserted. Rows with duplicate emails will be skipped.

* * * * *

### **2\. Upsert (3:11:09)**

An **Upsert** operation is a combination of an **`INSERT`** and an **`UPDATE`**. It inserts a new row if the specified key (such as a primary key or unique key) does not already exist, and it updates the existing row if the key already exists.

In PostgreSQL, you can perform an **upsert** using the **`ON CONFLICT`** clause with the **`DO UPDATE`** action.

#### **Why Use Upsert?**

-   **Efficient Handling of Existing and New Data**: It's useful when you want to either insert new records or update existing records based on a conflict (e.g., when data already exists and needs to be updated).

-   **No Need for Separate `INSERT` and `UPDATE`**: It eliminates the need to check whether a record already exists and perform either an `INSERT` or `UPDATE` accordingly.

#### **Syntax**:

```
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...)
ON CONFLICT (column_name) DO UPDATE
SET column1 = value1, column2 = value2, ...;

```

-   **`column_name`**: The column(s) that are subject to a conflict (usually a unique or primary key).

-   **`DO UPDATE`**: The action to perform if a conflict occurs. The values in the specified columns are updated.

* * * * *

#### **Example 1: Basic Upsert**

Let's say we want to upsert data into the `users` table, where the `email` column is unique. If the email already exists, we'll update the `first_name` and `last_name`.

```
INSERT INTO users (email, first_name, last_name)
VALUES
('john.doe@example.com', 'John', 'Doe')
ON CONFLICT (email) DO UPDATE
SET first_name = EXCLUDED.first_name, last_name = EXCLUDED.last_name;

```

-   The query tries to insert a user with the email `john.doe@example.com`. If the email already exists, the existing row will be updated with the new `first_name` and `last_name`.

#### Sample Result:

-   If `john.doe@example.com` already exists, the `first_name` and `last_name` will be updated with the new values.

-   If it doesn't exist, a new row will be inserted.

* * * * *

#### **Example 2: Upsert with Multiple Columns**

```
INSERT INTO users (email, first_name, last_name)
VALUES
('john.doe@example.com', 'John', 'Doe'),
('alice.jones@example.com', 'Alice', 'Jones')
ON CONFLICT (email) DO UPDATE
SET first_name = EXCLUDED.first_name, last_name = EXCLUDED.last_name;

```

In this example:

-   If a conflict occurs on the `email` column, the existing record will be updated with the new `first_name` and `last_name`.

-   If no conflict occurs, a new row will be inserted.

#### Sample Result:

-   If `john.doe@example.com` already exists, it will be updated with the new values. If not, both new rows will be inserted.

* * * * *

#### **Example 3: Using Upsert with Complex Updates**

Let's say you have a table for **orders** with a `quantity` column, and you want to update the `quantity` if the order already exists.

```
INSERT INTO orders (order_id, product_id, quantity)
VALUES (101, 1, 10)
ON CONFLICT (order_id, product_id) DO UPDATE
SET quantity = orders.quantity + EXCLUDED.quantity;

```

-   The query inserts a new order. If an order already exists with the same `order_id` and `product_id`, it **updates** the `quantity` by adding the existing quantity to the new quantity (i.e., it increases the quantity for that order).

#### Sample Result:

-   If the `order_id = 101` and `product_id = 1` already exists, the `quantity` will be updated (i.e., old `quantity + new` quantity`).

-   If the combination does not exist, a new order will be inserted.

* * * * *

### **Summary of Key Concepts**

-   **`ON CONFLICT DO NOTHING`**: Used in `INSERT` statements to avoid errors when inserting data that violates a unique constraint. It ignores conflicting rows.

-   **Upsert** (`ON CONFLICT DO UPDATE`): Allows for inserting new records or updating existing records based on conflicts. It simplifies managing data without needing separate `INSERT` and `UPDATE` queries.

* * * * *

### **Exam Power-ups for Memory:**

-   **`ON CONFLICT DO NOTHING`**: Useful for preventing duplicate records when inserting, especially in bulk operations. Avoids errors due to unique constraints.

-   **Upsert (`DO UPDATE`)**: Provides a powerful way to insert new records or update existing ones with a single query, streamlining data management.

-   **EXCLUDED**: In the **`DO UPDATE`** part of the upsert, **`EXCLUDED`** refers to the row that was proposed for insertion but caused the conflict.

* * * * *


* * * * *

### **What is a Relationship in Databases? (3:16:41)**

In relational databases, a **relationship** is a connection between two or more tables. Relationships are fundamental for organizing data in a structured manner. They allow you to represent how different pieces of data are related to one another.

There are **three types of relationships** between tables:

1.  **One-to-One Relationship**: A single record in one table is associated with a single record in another table.

2.  **One-to-Many Relationship**: A single record in one table can be associated with multiple records in another table.

3.  **Many-to-Many Relationship**: Multiple records in one table can be associated with multiple records in another table.

These relationships are typically established using **foreign keys**, which are references from one table to another.

* * * * *

### **Foreign Keys: The Key to Relationships**

A **foreign key** is a column (or a set of columns) in one table that uniquely identifies a row in another table. It's essentially a way of establishing and enforcing a relationship between two tables.

-   A **primary key** in one table is referenced by a **foreign key** in another table.

-   Foreign keys help enforce **referential integrity**, ensuring that the data in your database stays consistent and that you cannot insert a record into a table that doesn't correspond to a record in another table.

#### **Why are Foreign Keys Important?**

-   **Enforce Consistency**: They ensure that relationships between tables are maintained and that invalid data isn't inserted.

-   **Data Integrity**: By enforcing relationships, foreign keys help prevent situations where a record in one table references a non-existent record in another table.

* * * * *

### **1\. One-to-Many Relationship**

In a **one-to-many** relationship, a single record in one table can be associated with multiple records in another table. This is the most common type of relationship in databases.

#### **Example 1: One-to-Many Relationship**

Let's take an example where we have two tables:

-   **`departments`**: Each department can have many employees.

-   **`employees`**: Each employee works for one department.

#### **Step 1: Create the `departments` table**

```
CREATE TABLE departments (
    department_id SERIAL PRIMARY KEY,
    department_name VARCHAR(100)
);

```

Here, the `department_id` column is the **primary key** that uniquely identifies each department.

#### **Step 2: Create the `employees` table with a foreign key**

```
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);

```

In this example:

-   The `employees` table has a **foreign key** `department_id`, which references the `department_id` in the `departments` table.

-   This creates a **one-to-many** relationship: A department can have many employees, but each employee belongs to only one department.

#### **Sample Data Insertion**

```
-- Insert departments
INSERT INTO departments (department_name)
VALUES ('HR'), ('Engineering'), ('Marketing');

-- Insert employees with department assignments
INSERT INTO employees (first_name, last_name, department_id)
VALUES
('John', 'Doe', 1),  -- John works in HR
('Jane', 'Smith', 2),  -- Jane works in Engineering
('Bob', 'Brown', 2);  -- Bob works in Engineering

```

#### Sample Result:

-   **`departments` table**:

```
 department_id | department_name
---------------+----------------
 1             | HR
 2             | Engineering
 3             | Marketing

```

-   **`employees` table**:

```
 employee_id | first_name | last_name | department_id
-------------+------------+-----------+---------------
 1           | John       | Doe       | 1
 2           | Jane       | Smith     | 2
 3           | Bob        | Brown     | 2

```

In this example, `John` works in the `HR` department (department ID 1), while `Jane` and `Bob` work in the `Engineering` department (department ID 2). The `department_id` column in the `employees` table is the **foreign key** that links each employee to their respective department.

* * * * *

### **2\. One-to-One Relationship**

In a **one-to-one** relationship, each record in one table is associated with exactly one record in another table. This is less common than the one-to-many relationship but can be useful in specific scenarios.

#### **Example 2: One-to-One Relationship**

Let's take an example where we have two tables:

-   **`users`**: Each user has one profile.

-   **`profiles`**: Each profile belongs to one user.

#### **Step 1: Create the `users` table**

```
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(100) UNIQUE
);

```

#### **Step 2: Create the `profiles` table with a foreign key**

```
CREATE TABLE profiles (
    profile_id SERIAL PRIMARY KEY,
    user_id INT UNIQUE,
    bio TEXT,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

```

Here:

-   The `profiles` table has a **foreign key** `user_id`, which references the `user_id` in the `users` table.

-   The `user_id` in the `profiles` table is marked as **unique**, ensuring that each user can only have one profile.

#### **Sample Data Insertion**

```
-- Insert users
INSERT INTO users (username)
VALUES ('john_doe'), ('jane_smith');

-- Insert profiles
INSERT INTO profiles (user_id, bio)
VALUES (1, 'Hello, I am John. I love coding!'),
       (2, 'Hi, I am Jane. I enjoy photography.');

```

#### Sample Result:

-   **`users` table**:

```
 user_id | username
---------+----------
 1       | john_doe
 2       | jane_smith

```

-   **`profiles` table**:

```
 profile_id | user_id | bio
------------+---------+--------------------------------------
 1          | 1       | Hello, I am John. I love coding!
 2          | 2       | Hi, I am Jane. I enjoy photography.

```

Here, each user has exactly one profile. The `user_id` in the `profiles` table is the **foreign key** that links each profile to its user.

* * * * *

### **3\. Many-to-Many Relationship**

In a **many-to-many** relationship, multiple records in one table can be related to multiple records in another table. To handle this, you typically use a **junction table** (or **bridge table**), which holds the foreign keys from both tables.

#### **Example 3: Many-to-Many Relationship**

Let's take an example where we have two tables:

-   **`students`**: Each student can enroll in multiple courses.

-   **`courses`**: Each course can have multiple students.

#### **Step 1: Create the `students` table**

```
CREATE TABLE students (
    student_id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

```

#### **Step 2: Create the `courses` table**

```
CREATE TABLE courses (
    course_id SERIAL PRIMARY KEY,
    course_name VARCHAR(100)
);

```

#### **Step 3: Create the `enrollments` table (Junction Table)**

```
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);

```

In this case:

-   The `enrollments` table is the **junction table** that connects `students` and `courses`.

-   Both `student_id` and `course_id` are foreign keys in the `enrollments` table, referencing the `students` and `courses` tables, respectively.

#### **Sample Data Insertion**

```
-- Insert students
INSERT INTO students (name)
VALUES ('John Doe'), ('Alice Smith'), ('Bob Brown');

-- Insert courses
INSERT INTO courses (course_name)
VALUES ('Math 101'), ('History 201'), ('Computer Science 301');

-- Enroll students in courses
INSERT INTO enrollments (student_id, course_id)
VALUES (1, 1), (1, 3), (2, 2), (3, 1), (3, 3);

```

#### Sample Result:

-   **`students` table**:

```
 student_id | name
------------+-------------
 1          | John Doe
 2          | Alice Smith
 3          | Bob Brown

```

-   **`courses` table**:

```
 course_id | course_name
-----------+-------------
 1         | Math 101
 2         | History 201
 3         | Computer Science 301

```

-   **`enrollments` table** (Junction Table):

```
 student_id | course_id
------------+-----------
 1          | 1
 1          | 3
 2          | 2
 3          | 1
 3          | 3

```

Here:

-   **John Doe** is enrolled in **Math 101** and **Computer Science 301**.

-   **Alice Smith** is enrolled in **History 201**.

-   **Bob Brown** is enrolled in **Math 101** and **Computer Science 301**.

* * * * *

### **Adding Relationships Between Tables**

To add relationships between tables, you need to:

1.  Define **foreign key constraints**.

2.  Use the **`REFERENCES`** keyword to establish a relationship between the foreign key column in the child table and the primary key column in the parent table.

#### Example: Adding a Foreign Key to an Existing Table

Let's say we already have the `employees` table (from previous examples), and we want to establish a relationship between `employees` and `departments`.

```
ALTER TABLE employees
ADD CONSTRAINT fk_department FOREIGN KEY (department_id)
REFERENCES departments(department_id);

```

This adds a foreign key constraint on the `department_id` column in the `employees` table, linking it to the `department_id` column in the `departments` table.

* * * * *

### **Conclusion: Relationships and Foreign Keys**

-   **Relationships**: Establish how tables are connected to one another. Common types include **one-to-many**, **one-to-one**, and **many-to-many**.

-   **Foreign Keys**: Used to enforce relationships between tables. A foreign key is a column in one table that references the primary key of another table.

-   **One-to-Many**: A single record in one table can be related to multiple records in another table.

-   **Many-to-Many**: Multiple records in one table can be related to multiple records in another table, usually handled by a junction table.

* * * * *

### **Exam Power-ups for Memory**:

-   **Foreign Keys**: They ensure referential integrity, linking data across tables.

-   **Junction Tables**: Use junction tables to implement **many-to-many** relationships.

-   **One-to-Many**: Use foreign keys to implement **one-to-many** relationships (e.g., departments and employees).

* * * * *


### **1\. What is a Foreign Key Column?**

A **foreign key** column in a table is a reference to the **primary key** of another table. It establishes a link between the two tables. The main role of foreign keys is to enforce **referential integrity** by ensuring that only valid values, corresponding to rows in another table, can be inserted into the foreign key column.

### **2\. What Happens When You Update a Foreign Key Column?**

When you update a foreign key column, you are essentially changing the value that links the record to another record in a different table. **Referential integrity** must be maintained, meaning:

-   The updated foreign key value must still exist as a valid primary key in the referenced table.

-   If the referenced primary key value no longer exists or is changed incorrectly, it can break the relationship and lead to **orphaned records**.

* * * * *

### **3\. Updating Foreign Key Columns in PostgreSQL**

To **update foreign key columns**, you need to ensure that the updated value already exists in the referenced table, or the update may fail due to the foreign key constraint.

#### **Steps to Update Foreign Key Columns**:

1.  **Ensure the referenced value exists in the parent table.**

2.  **Use the `UPDATE` statement** to change the foreign key value.

3.  **Ensure the change adheres to the foreign key constraint** (i.e., the new foreign key value must be a valid primary key in the referenced table).

* * * * *

### **4\. Example Scenario**

Let's consider the following tables and scenario:

-   **`departments`** table (parent table):

    -   `department_id`: Primary key

    -   `department_name`

-   **`employees`** table (child table):

    -   `employee_id`: Primary key

    -   `first_name`, `last_name`: Employee information

    -   `department_id`: Foreign key linking to `departments.department_id`

#### **Table Definitions**:

```
-- Create the departments table
CREATE TABLE departments (
    department_id SERIAL PRIMARY KEY,
    department_name VARCHAR(100)
);

-- Create the employees table with a foreign key to departments
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);

```

* * * * *

### **5\. Example 1: Updating the Foreign Key in the Employees Table**

Let's say you want to change the department of an employee. You will update the **`department_id`** in the **`employees`** table, ensuring that the new department exists in the **`departments`** table.

#### **Step 1: Insert some sample data**

```
-- Insert departments
INSERT INTO departments (department_name)
VALUES ('Sales'), ('Engineering'), ('Marketing');

-- Insert employees
INSERT INTO employees (first_name, last_name, department_id)
VALUES
('John', 'Doe', 1),  -- John works in Sales
('Jane', 'Smith', 2);  -- Jane works in Engineering

```

At this point:

-   **`John`** is in the **Sales** department (department_id = 1).

-   **`Jane`** is in the **Engineering** department (department_id = 2).

* * * * *

#### **Step 2: Update the Foreign Key**

Now, let's say that **John** has been promoted and transferred to the **Engineering** department. We need to update his `department_id` in the **`employees`** table.

```
UPDATE employees
SET department_id = 2  -- Transfer John to the Engineering department
WHERE first_name = 'John' AND last_name = 'Doe';

```

#### Sample Result:

-   **`John`** will now be associated with the **Engineering** department (department_id = 2).

-   **`Jane`** remains in the **Engineering** department.

#### After the Update, the tables look like:

-   **`departments` table**:

```
 department_id | department_name
---------------+----------------
 1             | Sales
 2             | Engineering
 3             | Marketing

```

-   **`employees` table**:

```
 employee_id | first_name | last_name | department_id
-------------+------------+-----------+---------------
 1           | John       | Doe       | 2
 2           | Jane       | Smith     | 2

```

In this case, the foreign key constraint ensures that only valid `department_id` values are allowed. If you try to update to a `department_id` that does not exist, the update will fail.

* * * * *

### **6\. Example 2: Invalid Update (Foreign Key Constraint Violation)**

Let's attempt an invalid update where we try to assign an employee to a department that doesn't exist.

```
UPDATE employees
SET department_id = 999  -- Invalid department_id
WHERE first_name = 'John' AND last_name = 'Doe';

```

#### Error:

```
ERROR:  insert or update on table "employees" violates foreign key constraint "employees_department_id_fkey"
DETAIL:  Key (department_id)=(999) is not present in table "departments".

```

In this case:

-   PostgreSQL throws an error because `999` is not a valid `department_id` in the `departments` table.

-   The **foreign key constraint** prevents the update because it would violate referential integrity.

* * * * *

### **7\. Handling Foreign Key Updates with Cascade Options**

You can specify **cascading actions** when defining foreign key constraints. These actions determine what happens to the child table when the referenced record in the parent table is updated or deleted.

#### **Cascading Options**:

-   **`ON UPDATE CASCADE`**: Automatically updates the foreign key in the child table when the referenced primary key is updated.

-   **`ON DELETE CASCADE`**: Automatically deletes records in the child table when the referenced record in the parent table is deleted.

* * * * *

#### **Example 3: Using `ON UPDATE CASCADE`**

Let's say we want to ensure that if the **`department_id`** in the `departments` table is updated, the corresponding foreign key values in the `employees` table are also updated.

```
-- Create departments table with ON UPDATE CASCADE
CREATE TABLE departments (
    department_id SERIAL PRIMARY KEY,
    department_name VARCHAR(100)
);

-- Create employees table with ON UPDATE CASCADE
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id) ON UPDATE CASCADE
);

```

Now, if we update the `department_id` in the **`departments`** table, the corresponding `department_id` in the **`employees`** table will be automatically updated.

#### **Step 1: Insert Data**

```
-- Insert departments and employees
INSERT INTO departments (department_name) VALUES ('Sales'), ('Engineering');
INSERT INTO employees (first_name, last_name, department_id) VALUES ('John', 'Doe', 1), ('Jane', 'Smith', 2);

```

#### **Step 2: Update the Primary Key in the Parent Table**

Let's update the `department_id` in the **`departments`** table.

```
UPDATE departments
SET department_id = 3
WHERE department_name = 'Sales';

```

#### Sample Result:

-   In this case, because of the **`ON UPDATE CASCADE`** option, all records in the **`employees`** table that had `department_id = 1` (Sales) will have their `department_id` updated to `3`.

-   The **`employees`** table will automatically update the department for **John** (who was in Sales).

* * * * *

### **Summary of Key Concepts**

-   **Foreign Keys**: Used to establish relationships between tables. A foreign key in the child table references the primary key in the parent table.

-   **Referential Integrity**: Ensures that the foreign key values in the child table always correspond to valid primary key values in the parent table.

-   **Updating Foreign Key Columns**: You can update a foreign key value, but it must exist in the referenced table. If you try to reference a non-existing key, the update will fail.

-   **Cascading Updates**: The `ON UPDATE CASCADE` option ensures that when a primary key in the parent table is updated, the corresponding foreign keys in the child table are automatically updated as well.

* * * * *

### **Exam Power-ups for Memory:**

-   **Foreign Key Updates**: Always ensure that the new foreign key value exists in the referenced table before updating a foreign key.

-   **Referential Integrity**: Foreign keys help maintain the integrity of relationships by ensuring that the foreign key value always corresponds to a valid primary key.

-   **Cascading**: Use **`ON UPDATE CASCADE`** and **`ON DELETE CASCADE`** to ensure consistency between related tables.

* * * * *

### **Understanding SQL Joins: A Comprehensive Guide**

In SQL, a **join** is used to combine rows from two or more tables based on a related column. Joins are crucial when you want to work with data from multiple tables in a relational database. Let's go through the different types of joins available in SQL and explain them in detail, along with examples and use cases.

* * * * *

### **Types of SQL Joins**

1.  **INNER JOIN**

2.  **LEFT JOIN (or LEFT OUTER JOIN)**

3.  **RIGHT JOIN (or RIGHT OUTER JOIN)**

4.  **FULL JOIN (or FULL OUTER JOIN)**

5.  **CROSS JOIN**

6.  **SELF JOIN**

* * * * *

### **1\. INNER JOIN**

An **INNER JOIN** returns only the rows that have matching values in both tables. If there is no match, the row is excluded from the result set.

#### **Syntax**:

```
SELECT columns
FROM table1
INNER JOIN table2
ON table1.column = table2.column;

```

#### **Example**: Inner Join between `employees` and `departments`

Let's assume you have two tables:

-   **`employees`** (with columns `employee_id`, `first_name`, `department_id`)

-   **`departments`** (with columns `department_id`, `department_name`)

```
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id;

```

#### **Explanation**:

-   This query returns all employees along with their department names, but only those employees who have a valid `department_id` in both tables will be included.

-   If an employee's `department_id` doesn't match any `department_id` in the `departments` table, they will be excluded from the result.

#### **Sample Result**:

```
 employee_id | first_name | department_name
-------------+------------+-----------------
 1           | John       | Sales
 2           | Jane       | Engineering
 3           | Bob        | Marketing

```

* * * * *

### **2\. LEFT JOIN (or LEFT OUTER JOIN)**

A **LEFT JOIN** (also known as **LEFT OUTER JOIN**) returns all rows from the **left table** (table1), and the matched rows from the **right table** (table2). If there is no match, the result will contain `NULL` values for columns from the **right table**.

#### **Syntax**:

```
SELECT columns
FROM table1
LEFT JOIN table2
ON table1.column = table2.column;

```

#### **Example**: Left Join between `employees` and `departments`

```
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e
LEFT JOIN departments d
ON e.department_id = d.department_id;

```

#### **Explanation**:

-   This query returns all employees. If an employee does not belong to any department (i.e., no matching `department_id` in the `departments` table), the `department_name` will be `NULL` for that employee.

-   Employees who don't have a department assigned will still be listed.

#### **Sample Result**:

```
 employee_id | first_name | department_name
-------------+------------+-----------------
 1           | John       | Sales
 2           | Jane       | Engineering
 3           | Bob        | NULL

```

In this case, **Bob** has no department, so the `department_name` is `NULL`.

* * * * *

### **3\. RIGHT JOIN (or RIGHT OUTER JOIN)**

A **RIGHT JOIN** (also known as **RIGHT OUTER JOIN**) returns all rows from the **right table** (table2), and the matched rows from the **left table** (table1). If there is no match, the result will contain `NULL` values for columns from the **left table**.

#### **Syntax**:

```
SELECT columns
FROM table1
RIGHT JOIN table2
ON table1.column = table2.column;

```

#### **Example**: Right Join between `employees` and `departments`

```
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e
RIGHT JOIN departments d
ON e.department_id = d.department_id;

```

#### **Explanation**:

-   This query returns all departments. If a department does not have any employees (i.e., no matching `department_id` in the `employees` table), the `employee_id` and `first_name` will be `NULL` for that department.

-   If some departments have no employees, they will still appear in the result.

#### **Sample Result**:

```
 employee_id | first_name | department_name
-------------+------------+-----------------
 1           | John       | Sales
 2           | Jane       | Engineering
 NULL        | NULL       | HR

```

In this case, the **`HR`** department has no employees, so the corresponding `employee_id` and `first_name` are `NULL`.

* * * * *

### **4\. FULL JOIN (or FULL OUTER JOIN)**

A **FULL JOIN** (also known as **FULL OUTER JOIN**) returns all rows from both the **left** and **right** tables. If there is no match, the result will contain `NULL` values for the columns of the table that does not have a match.

#### **Syntax**:

```
SELECT columns
FROM table1
FULL JOIN table2
ON table1.column = table2.column;

```

#### **Example**: Full Join between `employees` and `departments`

```
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e
FULL JOIN departments d
ON e.department_id = d.department_id;

```

#### **Explanation**:

-   This query returns all employees and all departments.

-   If an employee is not assigned to a department, the `department_name` will be `NULL`.

-   If a department has no employees, the `employee_id` and `first_name` will be `NULL`.

#### **Sample Result**:

```
 employee_id | first_name | department_name
-------------+------------+-----------------
 1           | John       | Sales
 2           | Jane       | Engineering
 3           | Bob        | NULL
 NULL        | NULL       | HR

```

In this case, **Bob** has no department, and **HR** has no employees, so both have `NULL` values in their respective columns.

* * * * *

### **5\. CROSS JOIN**

A **CROSS JOIN** returns the **Cartesian product** of the two tables, meaning it will return all possible combinations of rows from both tables. There is no **`ON`** condition, and every row in the left table is combined with every row in the right table.

#### **Syntax**:

```
SELECT columns
FROM table1
CROSS JOIN table2;

```

#### **Example**: Cross Join between `employees` and `departments`

```
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e
CROSS JOIN departments d;

```

#### **Explanation**:

-   This query will return every possible combination of employees and departments.

-   If there are `3` employees and `3` departments, the result will contain `3 * 3 = 9` rows.

#### **Sample Result**:

```
 employee_id | first_name | department_name
-------------+------------+-----------------
 1           | John       | Sales
 1           | John       | Engineering
 1           | John       | Marketing
 2           | Jane       | Sales
 2           | Jane       | Engineering
 2           | Jane       | Marketing
 3           | Bob        | Sales
 3           | Bob        | Engineering
 3           | Bob        | Marketing

```

* * * * *

### **6\. SELF JOIN**

A **SELF JOIN** is a join where a table is joined with itself. This is useful when you want to compare rows within the same table, for example, comparing managers and their subordinates within the same `employees` table.

#### **Syntax**:

```
SELECT a.column1, b.column2
FROM table_name a, table_name b
WHERE a.common_column = b.common_column;

```

#### **Example**: Self Join on the `employees` Table (to find managers and their employees)

```
SELECT e1.first_name AS manager, e2.first_name AS employee
FROM employees e1
JOIN employees e2
ON e1.employee_id = e2.manager_id;

```

#### **Explanation**:

-   This query joins the `employees` table with itself to find pairs of managers and their employees.

-   Each employee has a `manager_id`, which refers to their manager's `employee_id`.

* * * * *

### **Summary of Key Joins**

1.  **INNER JOIN**: Returns only rows that have matching values in both tables.

2.  **LEFT JOIN**: Returns all rows from the left table and matched rows from the right table. If no match, the result contains `NULL` values for the right table.

3.  **RIGHT JOIN**: Returns all rows from the right table and matched rows from the left table. If no match, the result contains `NULL` values for the left table.

4.  **FULL JOIN**: Returns rows when there is a match in either table. If no match, the result will contain `NULL` values for the table without a match.

5.  **CROSS JOIN**: Returns the Cartesian product of both tables (all possible combinations of rows).

6.  **SELF JOIN**: A table is joined with itself to compare rows.

* * * * *

### **Exam Power-ups for Memory**:

-   **INNER JOIN**: It's the most common join and returns **only matching rows** between tables.

-   **OUTER JOINS**: **LEFT**, **RIGHT**, and **FULL** join provide results even if one of the tables does not have a match. Think of them as **inclusive joins**.

-   **CROSS JOIN**: Remember, this join gives the **Cartesian product** and can result in a large number of rows.

-   **SELF JOIN**: Use it when you need to compare data **within the same table**.

* * * * *

---

### **1. Deleting Records With Foreign Keys (3:40:53)**

When dealing with **foreign keys** in relational databases, deleting records becomes a bit more complex due to the relationship between the tables. A foreign key enforces **referential integrity** by ensuring that the value in a foreign key column corresponds to an existing record in the referenced table.

There are two important things to consider:
1. **Cascading Deletes**: You can configure **cascading deletes** so that when a record is deleted from the parent table, all the related records in the child table are also deleted automatically.
2. **Restricting Deletes**: You can restrict the deletion of records that are being referenced by foreign keys in other tables.

#### **Deleting Records with Foreign Keys:**
When trying to delete a record that is referenced by a foreign key, you have several options:
- **Restrict**: Prevents the deletion of a record if it's being referenced.
- **Cascade**: Deletes the record in the parent table and automatically deletes all referencing rows in the child table.
- **Set Null**: Sets the foreign key in the child table to `NULL` when the referenced record is deleted.

---

#### **Syntax to Add a Foreign Key with Cascading Options**:
You can define cascading behavior when creating or altering a foreign key constraint.

**`ON DELETE CASCADE`**: Automatically deletes child rows when the parent row is deleted.

```sql
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id) ON DELETE CASCADE
);
```

**`ON DELETE SET NULL`**: Sets the foreign key value to `NULL` in the child table when the parent row is deleted.

```sql
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id) ON DELETE SET NULL
);
```

**`ON DELETE RESTRICT`**: Prevents deletion of a row if it is being referenced by a foreign key in another table.

```sql
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id) ON DELETE RESTRICT
);
```

---

#### **Example: Deleting with Foreign Keys**

Let’s consider the following tables:
- **`departments`**: Stores department information.
- **`employees`**: Stores employee information, with a foreign key to `departments`.

```sql
-- Create departments table
CREATE TABLE departments (
    department_id SERIAL PRIMARY KEY,
    department_name VARCHAR(100)
);

-- Create employees table with a foreign key
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id) ON DELETE CASCADE
);
```

Now, when you **delete a department**, the **employees** in that department will also be deleted automatically due to **`ON DELETE CASCADE`**.

```sql
-- Deleting a department and all employees in that department
DELETE FROM departments WHERE department_id = 1;
```

#### Sample Result:
- All employees in the department with `department_id = 1` will be deleted along with the department.

---

### **2. Exporting Query Results to CSV (3:47:27)**

Exporting data to a **CSV** (Comma Separated Values) file is a common operation when working with databases. PostgreSQL provides a built-in function to export query results directly to a CSV file.

#### **Syntax to Export Data to CSV**:

```sql
COPY (SELECT * FROM table_name) TO '/path/to/file.csv' DELIMITER ',' CSV HEADER;
```

- **`COPY`**: The command to copy data into or out of a table.
- **`TO`**: Specifies the output file path.
- **`DELIMITER ','`**: Defines the separator used in the CSV file (commas for CSV).
- **`CSV HEADER`**: Includes the column names in the first row.

---

#### **Example 1: Exporting Query Results to CSV**

Let’s assume we have an `employees` table, and we want to export the employee data to a CSV file.

```sql
COPY (SELECT employee_id, first_name, last_name FROM employees) 
TO '/tmp/employees.csv' DELIMITER ',' CSV HEADER;
```

This will export the employee data to a CSV file located at **`/tmp/employees.csv`**, and the first row will contain the column headers (`employee_id`, `first_name`, `last_name`).

#### **Sample CSV Output**:
```
employee_id,first_name,last_name
1,John,Doe
2,Jane,Smith
3,Bob,Brown
```

---

### **3. Serial & Sequences (3:50:42)**

**Serial** and **sequences** are essential concepts in PostgreSQL for generating unique values, typically used for **primary keys**.

- **`SERIAL`**: A pseudo-type in PostgreSQL that automatically generates unique integer values for a column. It is often used for auto-incrementing primary keys.
- **Sequences**: An object in PostgreSQL that generates a series of unique numbers. The `SERIAL` type automatically uses sequences behind the scenes.

---

#### **1. `SERIAL` Data Type**

The **`SERIAL`** type is shorthand for creating an auto-incrementing integer column. It is commonly used for primary keys.

- **`SERIAL`** creates an integer column that automatically increments with each insert.
- PostgreSQL internally uses a sequence to generate the next number.

#### **Syntax for Serial Column**:
```sql
CREATE TABLE table_name (
    column_name SERIAL PRIMARY KEY
);
```

#### **Example**: Creating a Table with `SERIAL`

```sql
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    first_name VARCHAR(100),
    last_name VARCHAR(100)
);
```

Here:
- **`employee_id`** will automatically be assigned a unique value whenever a new row is inserted into the `employees` table.

---

#### **2. Sequences in PostgreSQL**

A **sequence** is an object that generates a series of unique numbers. Sequences are often used behind the scenes for columns defined as `SERIAL`, but they can also be used directly.

#### **Creating a Sequence**:
```sql
CREATE SEQUENCE sequence_name
START WITH 1
INCREMENT BY 1
NO MINVALUE
NO MAXVALUE
CACHE 1;
```

- **`START WITH`**: The initial value of the sequence.
- **`INCREMENT BY`**: The value by which the sequence increases each time.
- **`CACHE`**: The number of sequence numbers to cache in memory for performance.

#### **Example: Using a Sequence Directly**

Let’s create a sequence and use it to insert a new record into a table:

```sql
CREATE SEQUENCE employee_id_seq START WITH 1 INCREMENT BY 1;

CREATE TABLE employees (
    employee_id INT DEFAULT nextval('employee_id_seq'),
    first_name VARCHAR(100),
    last_name VARCHAR(100)
);
```

In this case:
- **`nextval('employee_id_seq')`** is used to automatically generate a new value for the `employee_id` column from the `employee_id_seq` sequence.

#### **Inserting Data Using the Sequence**:

```sql
INSERT INTO employees (first_name, last_name)
VALUES ('John', 'Doe');
```

- The `employee_id` will automatically be filled with the next value from the `employee_id_seq`.

---

### **Summary of Key Concepts**

- **Deleting Records with Foreign Keys**: When deleting records in a table with foreign key relationships, you can use cascading or restrict options to ensure referential integrity.
- **Exporting to CSV**: Use the `COPY` command to export query results to a CSV file for further processing or analysis.
- **Serial and Sequences**: **`SERIAL`** is used for auto-incrementing columns, and **sequences** provide a way to generate unique numbers for primary keys.

---

### **Exam Power-ups for Memory**:

- **Foreign Keys**: Always understand how cascading options (e.g., `ON DELETE CASCADE`) affect data when deleting records.
- **`COPY` for CSV**: Remember the `COPY` command for exporting data to CSV. It’s a powerful way to quickly move data out of PostgreSQL.
- **Sequences**: **`SERIAL`** automatically uses sequences. If you need more control over number generation, you can define and use **sequences** directly.

---


>IMPORTANT NOTES

THIS IS WRONG :
```sql
SELECT winner from nobel where yr BETWEEN 2000 and MAX(yr)
```
- because MAX(yr) means an aggregation function on the whole column sql cannot perform an aggregation of whole column every time filtering a select on a row

- select works row by row

- you can do something like this Select under a select

- when using select under select the subsequent select must return only one result

**Answer Explanation**  
Using an aggregate function like `MAX(yr)` in a `WHERE` clause is not valid in standard SQL because an aggregate function must be computed over a result set—either in a `SELECT` list or in a `HAVING` clause (after grouping). It cannot be evaluated directly as part of a simple `WHERE` filter for row-by-row selection. 

In other words, the database engine cannot simultaneously evaluate `MAX(yr)` (which depends on having read all relevant rows) and also use that value at the same time to filter rows in the exact same step. If you need to compare to `MAX(yr)`, you must do so in a subselect or a `HAVING` clause. For example:

```sql
SELECT winner
FROM nobel
WHERE subject = 'Peace'
  AND yr BETWEEN 2000 AND (
      SELECT MAX(yr) FROM nobel
  );
```

This way, the database first obtains the maximum year for the entire table (from the subselect) and then applies the `BETWEEN` condition. However, if your intention was simply to get all Peace winners since 2000, it is easier to write `AND yr >= 2000` instead.





### **Understanding `SELECT` under `SELECT` (Subqueries) in SQL**

A **subquery** is a query that is nested inside another query. Subqueries can be used in several parts of a SQL statement, including the `SELECT` clause, `FROM` clause, `WHERE` clause, and more. When a **`SELECT`** statement is nested within another **`SELECT`**, it's commonly referred to as a **subquery** or **nested query**.

Subqueries are useful for performing operations that need to retrieve data from another query, making your SQL queries more powerful and flexible.

* * * * *

### **Types of Subqueries**

1.  **Subquery in the SELECT Clause**

2.  **Subquery in the WHERE Clause**

3.  **Subquery in the FROM Clause**

4.  **Correlated Subqueries**

* * * * *

### **1\. Subquery in the SELECT Clause**

A subquery in the **`SELECT`** clause allows you to use the result of a query as part of the column selection in the outer query. It's often used to calculate values for each row.

#### **Syntax**:

```
SELECT column1,
       (SELECT aggregate_function(column2)
        FROM table2
        WHERE table2.column = table1.column) AS subquery_result
FROM table1;

```

#### **Example: Subquery in the SELECT Clause**

Let's say we have two tables:

-   **`orders`**: Contains order details (`order_id`, `customer_id`, `total_amount`).

-   **`customers`**: Contains customer details (`customer_id`, `customer_name`).

We want to retrieve the **total amount** for each customer's order, along with their **name**.

```
SELECT customer_name,
       (SELECT SUM(total_amount)
        FROM orders
        WHERE orders.customer_id = customers.customer_id) AS total_spent
FROM customers;

```

#### **Explanation**:

-   The **subquery** calculates the sum of the `total_amount` for each customer by matching the `customer_id` from the `orders` table with the `customer_id` in the `customers` table.

-   The result of the subquery will be displayed as `total_spent` alongside the customer's name.

#### **Sample Result**:

```
 customer_name | total_spent
---------------+-------------
 John Doe      | 250.50
 Jane Smith    | 320.75
 Bob Brown     | 150.20

```

In this case, the total amount spent by each customer is calculated dynamically within the `SELECT` clause.

* * * * *

### **2\. Subquery in the WHERE Clause**

A subquery in the **`WHERE`** clause is used to filter the results of the main query based on the results of the subquery. It's commonly used to compare values or check if a condition is met.

#### **Syntax**:

```
SELECT column1
FROM table1
WHERE column1 IN (SELECT column2 FROM table2 WHERE condition);

```

#### **Example: Subquery in the WHERE Clause**

We have two tables:

-   **`employees`**: Contains `employee_id`, `employee_name`, and `salary`.

-   **`departments`**: Contains `department_id` and `department_name`.

Now, we want to find the employees who work in the "Sales" department. The `department_id` of the "Sales" department can be found in the `departments` table.

```
SELECT employee_name
FROM employees
WHERE department_id IN (SELECT department_id
                         FROM departments
                         WHERE department_name = 'Sales');

```

#### **Explanation**:

-   The **subquery** retrieves the `department_id` for the "Sales" department.

-   The main query then returns the names of employees whose `department_id` matches the result of the subquery.

#### **Sample Result**:

```
 employee_name
---------------
 John Doe
 Alice Smith

```

In this case, only employees who belong to the "Sales" department will be included in the result.

* * * * *

### **3\. Subquery in the FROM Clause**

A subquery in the **`FROM`** clause is used to treat the result of a subquery as a derived table (also called a "subselect"). This is useful when you need to perform further operations on the result set of a subquery.

#### **Syntax**:

```
SELECT column1
FROM (SELECT column1, column2
      FROM table2
      WHERE condition) AS derived_table
WHERE condition;

```

#### **Example: Subquery in the FROM Clause**

Let's say we have an **`employees`** table and want to find employees who have an average salary higher than the overall average salary.

```
SELECT employee_name
FROM (SELECT employee_name, salary
      FROM employees
      WHERE salary > (SELECT AVG(salary) FROM employees)) AS high_earners;

```

#### **Explanation**:

-   The **subquery** inside the **`FROM`** clause selects employees whose salary is greater than the average salary in the company.

-   The main query then returns the names of these employees.

#### **Sample Result**:

```
 employee_name
---------------
 John Doe
 Jane Smith

```

Here, the derived table `high_earners` contains only employees with a salary above the company average, and the outer query retrieves their names.

* * * * *

### **4\. Correlated Subqueries**

A **correlated subquery** is a subquery that **depends on** the outer query. In other words, for each row in the outer query, the subquery is executed again. This is different from a regular subquery, which is executed only once and can be used independently.

#### **Syntax**:

```
SELECT column1
FROM table1 t1
WHERE column1 = (SELECT column2
                 FROM table2 t2
                 WHERE t1.column1 = t2.column2);

```

#### **Example: Correlated Subquery**

Let's consider two tables:

-   **`employees`**: Contains employee information, including `employee_id`, `name`, and `salary`.

-   **`departments`**: Contains department information, including `department_id` and `department_name`.

We want to find all employees whose salary is above the average salary of their respective department.

```
SELECT e.employee_name
FROM employees e
WHERE e.salary > (SELECT AVG(salary)
                  FROM employees
                  WHERE department_id = e.department_id);

```

#### **Explanation**:

-   The subquery calculates the **average salary** for each employee's **department**. The subquery references `e.department_id` from the outer query.

-   The outer query then selects employees whose salary is higher than the average salary of their department.

#### **Sample Result**:

```
 employee_name
---------------
 John Doe
 Jane Smith

```

In this case, **John Doe** and **Jane Smith** earn more than the average salary in their respective departments.

* * * * *

### **Summary of Key Concepts**

-   **Subqueries** allow you to embed a query inside another query. They are commonly used in the **`SELECT`**, **`WHERE`**, and **`FROM`** clauses.

-   **Types of Subqueries**:

    1.  **Subquery in the SELECT Clause**: Retrieves computed values for each row.

    2.  **Subquery in the WHERE Clause**: Filters records based on the result of the subquery.

    3.  **Subquery in the FROM Clause**: Treats the result of the subquery as a derived table.

    4.  **Correlated Subquery**: A subquery that depends on the outer query and is executed for each row of the outer query.

* * * * *

### **Exam Power-ups for Memory**:

-   **Subqueries in SELECT**: Useful for performing calculations on a per-row basis.

-   **Subqueries in WHERE**: Help filter records based on conditions evaluated from another query.

-   **Correlated Subqueries**: Think of correlated subqueries as **dynamic**, where the inner query is executed for each row in the outer query.

-   **Performance**: Always consider performance when using subqueries, especially correlated subqueries, as they can be slower due to multiple executions.

* * * * *

### **The `ALL` Keyword in SQL**

In SQL, the **`ALL`** keyword is used to compare a value to **all** values in another result set or subquery. It is often used in conjunction with comparison operators like `=`, `>`, `<`, `>=`, `<=`, and `<>` to ensure that the condition is true for **all** the rows returned by the subquery.

When using `ALL`, the query checks whether the condition is true for **every** row returned by the subquery. 

### **How `ALL` Works:**

- **`ALL`** is typically used with **comparison operators** like `>`, `<`, `=`, `>=`, `<=`, or `<>` to compare the value against a set of results.
- The condition must be true for **all** values in the subquery for the result to be returned.

---

### **1. Syntax of `ALL`**:

```sql
SELECT column
FROM table_name
WHERE column operator ALL (SELECT column FROM table_name);
```

- **`operator`**: The comparison operator (like `=`, `>`, `<`, etc.).
- **`ALL`**: Compares the value against **all** the values returned by the subquery.

---

### **2. Example Use Cases for `ALL`**:

Let's go through several examples to understand how **`ALL`** can be used effectively in SQL queries.

---

### **Example 1: Using `ALL` with `=` Operator**

Let's say you have two tables:
- **`employees`**: Contains employee information (`employee_id`, `name`, `salary`).
- **`departments`**: Contains department information (`department_id`, `department_name`).

You want to find all employees who earn a salary **equal to** the salary of **all employees** in a given department (e.g., the `Engineering` department).

#### **Query**:

```sql
SELECT name, salary
FROM employees
WHERE salary = ALL (SELECT salary FROM employees WHERE department_id = 2);
```

#### **Explanation**:
- This query will return the employees whose **salary** is equal to the salary of **all employees** in the `Engineering` department (which has `department_id = 2`).
- If the department has multiple employees with different salaries, no employee will be selected unless their salary matches **all** salaries in that department.

---

### **Example 2: Using `ALL` with `>` Operator**

You have an **`orders`** table with the following columns: `order_id`, `customer_id`, `order_total`.

Suppose you want to find customers whose `order_total` is **greater than** **all** of the orders from a particular customer (e.g., customer with `customer_id = 5`).

#### **Query**:

```sql
SELECT customer_id, order_total
FROM orders
WHERE order_total > ALL (SELECT order_total FROM orders WHERE customer_id = 5);
```

#### **Explanation**:
- This query returns the customers whose **order_total** is greater than **every** order made by the customer with `customer_id = 5`.
- It compares each row’s `order_total` against all the `order_total` values for that specific customer.

---

### **Example 3: Using `ALL` with `<` Operator**

Suppose you want to find all employees who have a **salary lower than** the **lowest salary** in the `Engineering` department.

#### **Query**:

```sql
SELECT name, salary
FROM employees
WHERE salary < ALL (SELECT salary FROM employees WHERE department_id = 2);
```

#### **Explanation**:
- This query finds all employees whose **salary** is less than **all** the salaries in the `Engineering` department (`department_id = 2`).
- If the `Engineering` department has employees with a salary of `50,000`, `55,000`, and `60,000`, then the query will return employees whose salary is lower than `50,000`.

---

### **Example 4: Using `ALL` with `<>` (Not Equal) Operator**

You want to find all employees whose salary is **not equal to** **any** of the salaries of employees in the `Marketing` department.

#### **Query**:

```sql
SELECT name, salary
FROM employees
WHERE salary <> ALL (SELECT salary FROM employees WHERE department_id = 3);
```

#### **Explanation**:
- This query finds all employees whose **salary** is **not equal to** any salary in the `Marketing` department (`department_id = 3`).
- The query compares each employee’s salary against **all** salaries in the `Marketing` department and returns those whose salary is different from all of them.

---

### **3. Key Points to Remember**

- **`ALL`** works with **comparison operators** like `=`, `>`, `<`, `>=`, `<=`, `<>`.
- The condition must be true for **every** row returned by the subquery.
- You typically use **`ALL`** when you want to compare a value to a **set** of values rather than just a single value.

---

### **4. Performance Considerations**

- **Subqueries and Performance**: Subqueries can be slow, especially when using **`ALL`**, since the subquery is executed for each row of the outer query. Consider using **joins** for better performance in some cases.
- **Indexes**: If you're using **subqueries** frequently, ensure that the columns being referenced (such as those in the subquery's **`WHERE`** clause) are indexed, as this can significantly improve performance.

---

### **5. Exam Power-ups for Memory**

- **`ALL` Keyword**: Think of **`ALL`** as saying "this condition must be true for **every** row in the result set." It's great for comparing a value to a set.
- **Use with Comparison Operators**: You can pair **`ALL`** with **`=`, `>`, `<`, `>=`, `<=`, `<>`** to create powerful conditions.
- **Subqueries**: When using **`ALL`**, the subquery is essential. Make sure the subquery returns a valid set of values to avoid errors.

---
