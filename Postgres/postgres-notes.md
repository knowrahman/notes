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
