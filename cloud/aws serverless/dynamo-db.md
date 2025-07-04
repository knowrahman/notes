Great — let’s get started.

## Part 1: What is DynamoDB?

### Definition

**Amazon DynamoDB** is a **fully managed NoSQL database service** provided by AWS. It delivers single-digit millisecond performance at any scale, with built-in security, backup and restore, and in-memory caching.

It is **serverless**, meaning:

* No need to provision or manage servers.
* Scales automatically.
* You pay for what you use (on-demand or provisioned capacity).

---

### Key Characteristics

| Feature                          | Description                                                                         |
| -------------------------------- | ----------------------------------------------------------------------------------- |
| **NoSQL (Key-Value & Document)** | Data is stored as items (rows), which consist of key-value pairs. It’s schema-less. |
| **Fully managed**                | No need to patch, back up, scale, or worry about hardware.                          |
| **Highly available & durable**   | Data is replicated across multiple AZs in a region.                                 |
| **Fast performance**             | Single-digit millisecond reads/writes.                                              |
| **Scalable**                     | Scales horizontally to handle virtually any traffic.                                |
| **Secure**                       | Integrated with IAM, KMS (encryption), VPC, CloudTrail, etc.                        |
| **Event-driven**                 | Integrated with DynamoDB Streams, AWS Lambda.                                       |

---

### Use Cases

Use DynamoDB when:

* You need **high scalability** and **performance** for read/write-heavy applications.
* You have **simple access patterns** (e.g., lookup by ID).
* You're building **serverless applications**.
* You want **automatic scaling** with no management overhead.
* You store **semi-structured or unstructured data** like JSON.

Example Applications:

* E-commerce product catalog
* Gaming leaderboards
* IoT device data ingestion
* Real-time analytics and dashboards
* User session tracking
* Serverless APIs

---

### DynamoDB vs SQL (RDS)

| Feature            | DynamoDB (NoSQL)                             | RDS (SQL)                      |
| ------------------ | -------------------------------------------- | ------------------------------ |
| **Data model**     | Key-Value, Document                          | Relational (Tables, Rows)      |
| **Schema**         | Flexible                                     | Fixed schema                   |
| **Scaling**        | Horizontal                                   | Vertical (usually)             |
| **Transactions**   | Supported (with limitations)                 | Fully supported                |
| **Query language** | DynamoDB API                                 | SQL                            |
| **Join support**   | No joins                                     | Supports joins                 |
| **Best for**       | Predictable access patterns, high throughput | Complex queries, relationships |

---

### Real-world Example

Let’s say you’re building a **serverless to-do list app**. Every task has:

* A `taskId` (UUID),
* A `userId` (the owner),
* A `status` (pending, completed),
* A `description`.

DynamoDB would store each task as an *item* with flexible attributes. You can quickly retrieve all tasks for a user or filter by status without needing joins or complex queries.

---

### Exam Power-Up 💡

> DynamoDB is a **fully managed NoSQL database** that supports **key-value** and **document** data models. It is ideal for **high-throughput, low-latency applications** that require **auto-scaling**, **serverless architecture**, and **flexible schemas**.

---

### Sample AWS Developer Associate Question (with explanation):

**Q1:**
Which of the following best describes Amazon DynamoDB?

A. A relational database service that supports SQL joins and complex queries
B. A key-value and document-based NoSQL database designed for low-latency and high scalability
C. A file storage service for large unstructured data
D. A time-series database for logging metrics

**Answer:** B

**Explanation:**

* A is incorrect: That's describing RDS or Aurora.
* B is correct: DynamoDB is a NoSQL service supporting key-value and document types.
* C is incorrect: This describes Amazon S3.
* D is incorrect: Amazon Timestream is the time-series database.

---
Awesome! Let’s dive into:

## Part 2: DynamoDB Core Concepts

To effectively design and use DynamoDB, you need to understand these core components:

---

### 1. **Table**

A table in DynamoDB is where all your data is stored — like a collection of items (rows). Unlike relational DBs, there's **no fixed schema**, except for the **primary key**.

Example:

```json
{
  "TableName": "Tasks"
}
```

---

### 2. **Item**

An item is like a row in SQL. It's a collection of attributes uniquely identified by a **primary key**.

Example:

```json
{
  "userId": "user123",
  "taskId": "task789",
  "description": "Buy milk",
  "status": "pending"
}
```

---

### 3. **Attribute**

An attribute is a key-value pair. Each item can have different attributes.

Types: `String`, `Number`, `Boolean`, `Binary`, `Null`, `List`, `Map`, `Set`

Example:

```json
"description": "Buy milk"
```

---

### 4. **Primary Key**

Every item must have a **unique primary key**.

Two types:

#### a) Partition Key (Hash Key):

* Uniquely identifies an item.
* All data with the same key go to the same partition.

```json
"taskId": "task123"
```

#### b) Partition Key + Sort Key (Composite Key):

* Enables sorting within a partition.
* Useful for fetching multiple items with the same partition key.

```json
"userId": "user123",  
"taskId": "task001"
```

Use Case: Get all tasks for a user, ordered by creation date or task ID.

---

### 5. **Capacity Modes**

| Mode            | Description                                                               |
| --------------- | ------------------------------------------------------------------------- |
| **On-Demand**   | Auto-scales; pay per request; best for unpredictable traffic              |
| **Provisioned** | Specify Read/Write Capacity Units (RCUs/WCUs); cheaper for steady traffic |

---

### 6. **Indexes**

Indexes let you query data efficiently using different keys.

#### a) Local Secondary Index (LSI):

* Same Partition Key as the base table
* Different Sort Key
* Max 5 per table
* Created **at table creation**

#### b) Global Secondary Index (GSI):

* Different Partition Key and Sort Key
* Can be added **after table creation**
* Supports different access patterns

**Example GSI:**

```json
{
  "IndexName": "statusIndex",
  "KeySchema": [
    { "AttributeName": "status", "KeyType": "HASH" }
  ]
}
```

---

### 7. **DynamoDB Streams**

* Captures table activity (create, update, delete)
* Can trigger **AWS Lambda** for real-time processing

Use Case: Audit logs, real-time sync to Elasticsearch, or triggering workflows.

---

### 8. **TTL (Time to Live)**

* Automatically deletes expired items
* Useful for session data, temporary records

```json
"ttl": 1728000000  // Unix epoch timestamp
```

---

### 9. **Consistency Models**

| Type                      | Description                                       |
| ------------------------- | ------------------------------------------------- |
| **Eventually Consistent** | Fast, default, may not show latest data instantly |
| **Strongly Consistent**   | Guarantees latest data, slightly higher latency   |

---

### 10. **Partitioning and Throughput**

* Data is split across partitions based on the hash of the Partition Key
* Choose Partition Keys wisely to avoid **hot partitions**

---

### Real Example (Tasks Table):

| userId  | taskId  | description | status  |
| ------- | ------- | ----------- | ------- |
| user123 | task001 | Buy milk    | pending |
| user123 | task002 | Read book   | done    |
| user456 | task101 | Study AWS   | pending |

Primary Key: `userId` (Partition Key), `taskId` (Sort Key)

---

### Exam Power-Up 💡

> DynamoDB organizes data in **tables** composed of **items** (rows) and **attributes** (columns). The uniqueness of an item is determined by its **primary key**, which can be a **Partition Key** or **Partition + Sort Key**. Use **GSI** to support multiple access patterns and **Streams** to enable real-time data processing.

---

### Sample AWS Developer Associate Question:

**Q2:**
What does a Local Secondary Index (LSI) allow you to do in DynamoDB?

A. Use a different partition key
B. Use the same partition key but different sort key
C. Use different partition and sort keys
D. Automatically replicate data to other regions

**Answer:** B

**Explanation:**

* A is incorrect: LSI must use the same partition key.
* B is correct: LSI changes only the sort key.
* C is for GSI, not LSI.
* D refers to DynamoDB Global Tables, not indexes.

---

Excellent question!

### Can Two Items Have the Same Partition Key *Without* a Sort Key?

**No**, they **cannot**.

In DynamoDB, if your table uses **only a Partition Key (no Sort Key)**, then the **Partition Key must be unique** for each item. That means:

* You **cannot** insert two items with the same partition key.
* The second insert with the same key will **overwrite** the existing item.

---

### Example: Table with Only a Partition Key

Let’s say your table `Users` has this schema:

* **Partition Key:** `userId` (String)
* **No Sort Key**

#### ✅ Allowed:

```json
{ "userId": "user1", "name": "Rahman" }
{ "userId": "user2", "name": "Amina" }
```

#### ❌ Not Allowed (Duplicate key):

```json
{ "userId": "user1", "name": "New Rahman" }
```

This will **replace** the first `user1` record instead of adding a new one.

---

### Example of Insert Overwriting

Initial Insert:

```json
{
  "userId": "user1",
  "name": "Rahman",
  "email": "rahman@example.com"
}
```

Second Insert (same `userId`):

```json
{
  "userId": "user1",
  "name": "Rahman Updated",
  "phone": "1234567890"
}
```

**Resulting item in table:**

```json
{
  "userId": "user1",
  "name": "Rahman Updated",
  "phone": "1234567890"
}
```

The original `email` attribute is **gone** unless you explicitly include it again — because DynamoDB **fully replaces the item** unless you use `UpdateItem`.

---

### So When Do You Need a Sort Key?

You use a **Sort Key** when:

* You want **multiple items** with the same Partition Key.
* You want to **query** or **sort** those items efficiently.

---

### Example with Partition + Sort Key

Table: `Tasks`

* Partition Key: `userId`
* Sort Key: `taskId`

Now you can have:

```json
{ "userId": "user1", "taskId": "task1", "description": "Buy milk" }
{ "userId": "user1", "taskId": "task2", "description": "Read book" }
{ "userId": "user2", "taskId": "task1", "description": "Write essay" }
```

Here, each `(PartitionKey, SortKey)` pair must be **unique**, but multiple items can share the same Partition Key.

---

### Exam Power-Up 💡

> In a DynamoDB table **without a Sort Key**, the Partition Key must be unique. If you try to insert another item with the same Partition Key, it will **overwrite** the existing one. To store multiple items under the same Partition Key, use a **composite key** (Partition Key + Sort Key).

---

### Sample Question:

**Q3:**
You create a DynamoDB table with only a Partition Key `userId`. What happens if you try to insert two items with the same `userId`?

A. DynamoDB throws an error
B. The second insert is ignored
C. The second item replaces the first
D. Both items are inserted with a generated sort key

**Answer:** C

**Explanation:** Without a Sort Key, the Partition Key must be unique. A second item with the same Partition Key will overwrite the first.

---
Absolutely! Let's break down three important advanced DynamoDB concepts: **Indexes**, **Streams**, and **Consistency Models** — each is critical for both **real-world apps** and the **AWS Developer Associate exam**.

---

## Part 3: Indexes, Streams & Consistency Models (Detailed)

---

### **I. Indexes in DynamoDB**

Indexes allow you to **query data using alternate keys** (other than the table’s primary key). There are two types:

---

#### 1. **Local Secondary Index (LSI)**

* **Same Partition Key**, different **Sort Key**
* Defined **at table creation only**
* Max **5 LSIs per table**
* Strongly consistent reads supported

**Use Case**: Query tasks for a user, sorted by priority or due date.

##### Example

**Base Table:**

* `userId` (PK)
* `taskId` (SK)

**LSI:**

* Partition Key: `userId`
* Sort Key: `dueDate`

Now you can do:

```js
// Get all tasks for user123 sorted by due date
KeyConditionExpression: 'userId = :uid AND dueDate >= :today'
```

---

#### 2. **Global Secondary Index (GSI)**

* **Different Partition and Sort Keys**
* Can be added **any time**
* Supports **eventual consistency only**
* Max **20 GSIs per table** (soft limit)

**Use Case**: Query tasks by `status` or all tasks with `priority = high`

##### Example

**GSI:**

* Partition Key: `status`
* Sort Key: `createdAt`

Now you can do:

```js
// Get all "pending" tasks, ordered by creation time
KeyConditionExpression: 'status = :s AND createdAt > :date'
```

##### GSI Projection Types:

* `ALL`: All attributes copied into GSI
* `KEYS_ONLY`: Only PK and SK
* `INCLUDE`: Specific attributes

---

### **II. DynamoDB Streams**

**Streams** capture **real-time data changes** in a DynamoDB table.

For each insert, update, or delete, the stream records an **event** that can be processed by AWS Lambda.

---

#### Features

* Records stored for **24 hours**
* **Event types**: `INSERT`, `MODIFY`, `REMOVE`
* Can be **enabled/disabled** on a table
* Used to trigger **Lambda**, replicate data, log changes

##### Example Use Cases

* Real-time analytics
* Sending notifications
* Triggering workflows
* Data replication (to S3, Redshift, etc.)

##### Sample Stream Record:

```json
{
  "eventName": "INSERT",
  "dynamodb": {
    "Keys": {
      "userId": { "S": "user1" },
      "taskId": { "S": "task1" }
    },
    "NewImage": {
      "description": { "S": "Buy milk" },
      "status": { "S": "pending" }
    }
  }
}
```

##### Setting Up a Stream Lambda:

1. Enable stream on table.
2. Create a Lambda function.
3. Set DynamoDB Stream as the event source.
4. Process `event.Records[]` inside Lambda.

---

### **III. Consistency Models in DynamoDB**

When you read from DynamoDB, you choose between two models:

---

#### 1. **Eventually Consistent Reads (Default)**

* Data **might be stale** for a short time
* **Higher throughput**
* **Cheaper** read capacity
* Good for most use cases

---

#### 2. **Strongly Consistent Reads**

* **Guaranteed to reflect latest writes**
* Slightly **slower**
* **Costs more** (1 read = 1 RCU vs 0.5 RCU for eventual)

##### Note:

* GSIs do **not support strong consistency**
* Use **strongly consistent reads** for read-after-write operations if your logic requires immediate freshness

---

### Table: Summary of Differences

| Feature            | LSI                       | GSI                 |
| ------------------ | ------------------------- | ------------------- |
| Partition Key      | Same as table             | Can be different    |
| Sort Key           | Must differ               | Optional            |
| Add after creation | ❌                         | ✅                   |
| Strong consistency | ✅                         | ❌                   |
| Use cases          | Alternate sort on same PK | New access patterns |

| Feature    | Eventually Consistent | Strongly Consistent |
| ---------- | --------------------- | ------------------- |
| Speed      | Faster                | Slightly Slower     |
| Freshness  | May be stale          | Always up to date   |
| Cost (RCU) | 0.5                   | 1                   |
| Default?   | ✅                     | ❌                   |

---

### Exam Power-Up 💡

* **LSI** = Same Partition Key, new Sort Key, must be created with table.
* **GSI** = New Partition + Sort Key, can be added anytime.
* **Streams** = Trigger Lambda on INSERT/MODIFY/REMOVE, great for real-time workflows.
* **Eventually Consistent Reads** = Fast and cheap, not always up to date.
* **Strongly Consistent Reads** = Slower, up-to-date, not supported on GSIs.

---

### Sample AWS Developer Associate Questions

**Q4:**
What is a key difference between a Local Secondary Index and a Global Secondary Index?

A. LSI can be added after table creation
B. GSI requires the same partition key as the base table
C. GSI allows different partition keys and can be added anytime
D. LSI supports only eventually consistent reads

**Answer:** C
**Explanation:** GSI allows different PK and SK, and can be added after table creation. LSIs must be created up front and use the same PK.

---

**Q5:**
What does DynamoDB Streams allow you to do?

A. Archive deleted records automatically
B. Trigger Lambda functions based on table changes
C. Query the table using SQL
D. Perform joins across multiple tables

**Answer:** B
**Explanation:** Streams capture table changes, and can invoke Lambda for processing. Options C and D are invalid for NoSQL.

---

**Q6:**
Which consistency model provides the most up-to-date data in DynamoDB?

A. Eventually consistent
B. Weakly consistent
C. Strongly consistent
D. Global consistency

**Answer:** C
**Explanation:** Strongly consistent reads always reflect the latest committed write.

---
Perfect! Let’s move on to:

## Part 4 (was Part 3 in your original roadmap): Schema Design in a NoSQL World

Unlike relational databases, DynamoDB doesn't rely on table joins or normalized schemas. Instead, **you model your data based on access patterns** — this is one of the most critical mindset shifts when working with DynamoDB.

---

### Key Principles of NoSQL Schema Design

1. **Denormalization is OK**

   * You store related data together, even if it causes some duplication.
   * Prioritize read efficiency over storage efficiency.

2. **Query-first design**

   * Design your schema based on **how you want to query the data**, not what the data looks like.
   * Think of "what questions will I ask?" and model accordingly.

3. **Single Table Design (often preferred)**

   * Use **one table** for your whole application, with **different item types**.
   * Identify them using a `type` attribute (e.g., `user`, `order`, `product`)
   * Use **composite keys** to store multiple types of relationships.

---

### Common Access Patterns

Let’s say you're building an **e-commerce backend**.

Typical questions (access patterns):

* Get all orders for a user
* Get all products in a category
* Get a single order by ID
* Get reviews for a product

Relational design would use many tables and joins.
In DynamoDB, we model these as **items with common keys** and **clever key structuring**.

---

### 1. Modeling One-to-Many Relationships

**Example: A user has many orders**

#### Relational Way:

```sql
SELECT * FROM orders WHERE user_id = 'user123'
```

#### DynamoDB Way:

Use:

* `PK = user#user123`
* `SK = order#<orderId>`

#### Items:

```json
{ "PK": "user#user123", "SK": "user#user123", "name": "Rahman", "type": "user" }
{ "PK": "user#user123", "SK": "order#001", "orderDate": "2024-01-01", "total": 100 }
{ "PK": "user#user123", "SK": "order#002", "orderDate": "2024-01-10", "total": 50 }
```

Now you can `Query` all items with `PK = user#user123` to get both the user and their orders.

---

### 2. Modeling Many-to-Many Relationships

**Example: Products belong to multiple categories**

Use an **association item** pattern.

#### Items:

```json
{ "PK": "product#123", "SK": "product#123", "name": "MacBook Pro" }
{ "PK": "category#electronics", "SK": "product#123" }
{ "PK": "category#computers", "SK": "product#123" }
```

Now query:

* `PK = category#electronics` → get all products in "electronics"

---

### 3. Modeling Hierarchical Data

**Example: A company has departments, departments have employees**

Use sort keys to encode hierarchy.

#### Items:

```json
{ "PK": "company#piction", "SK": "company#piction", "type": "company" }
{ "PK": "company#piction", "SK": "department#engineering" }
{ "PK": "company#piction", "SK": "department#engineering#employee#rahman" }
{ "PK": "company#piction", "SK": "department#sales" }
```

Then query:

* `PK = company#piction AND begins_with(SK, "department#engineering")`

---

### Single Table Design Best Practices

| Tip                                   | Why It Matters                                                |
| ------------------------------------- | ------------------------------------------------------------- |
| Use prefixes like `user#123`          | Prevents key collision across entity types                    |
| Design for read operations            | Write once, read many – so optimize reads                     |
| Use GSIs for alternate queries        | Avoid scans, support secondary access patterns                |
| Store all related data in a partition | Enables efficient queries with 1 read                         |
| Add metadata fields                   | Helps interpret different item types (e.g., `type = 'order'`) |

---

### Query vs Scan

* **Query**: Efficient; uses partition key (and optionally sort key)
* **Scan**: Reads **every item** in the table; very expensive

> You want **every query to be a `Query`**, never a `Scan`.

---

### Exam Power-Up 💡

> In DynamoDB, you design schemas based on **access patterns**, not entity relationships. Use **composite keys** and **prefixes** to model one-to-many, many-to-many, and hierarchical relationships in a single table. Always **avoid scans**, and use **GSIs** for alternate query paths.

---

### Sample Question:

**Q7:**
You're designing a DynamoDB table where a user can have multiple orders. What's the best key schema design?

A. Use userId as Partition Key and no Sort Key
B. Use orderId as Partition Key and userId as Sort Key
C. Use `user#<id>` as Partition Key and `order#<id>` as Sort Key
D. Use `order#<id>` as Partition Key and `user#<id>` as Sort Key

**Answer:** C

**Explanation:**
C enables querying all orders for a user using `PK = user#<id>` and sorting by order ID.

---

Great — now we’ll cover:

## Part 5: Querying and Scanning Data in DynamoDB

This section will show how to **efficiently retrieve data** using DynamoDB’s API — especially using **Query**, **Scan**, and **expression syntax**. You’ll need this both for real projects and the AWS Developer Associate exam.

---

### 1. **Query Operation**

**Purpose**: Efficiently retrieve items by **Partition Key** and (optionally) **Sort Key condition**.

> You must **know the Partition Key value** to use a `Query`.

---

#### Basic Query Example:

Let's say your table has:

* Partition Key: `userId`
* Sort Key: `taskId`

To get all tasks for `user123`:

```js
const { QueryCommand } = require("@aws-sdk/client-dynamodb");

const command = new QueryCommand({
  TableName: "Tasks",
  KeyConditionExpression: "userId = :uid",
  ExpressionAttributeValues: {
    ":uid": { S: "user123" }
  }
});
```

---

### 2. **Sort Key Conditions**

You can use:

* `=`
* `<`, `<=`, `>`, `>=`
* `BETWEEN`
* `begins_with`

Example:

```js
KeyConditionExpression: "userId = :uid AND begins_with(taskId, :prefix)"
```

---

### 3. **FilterExpression**

Filters the **results after the query/scan**, not part of key lookup. **Less efficient**.

```js
FilterExpression: "status = :stat",
ExpressionAttributeValues: {
  ":stat": { S: "completed" }
}
```

> Use filters **only** when necessary. Try to use key conditions + GSIs instead.

---

### 4. **ProjectionExpression**

Controls which attributes to return.

Example:

```js
ProjectionExpression: "taskId, description"
```

---

### 5. **ExpressionAttributeNames and Values**

Use `ExpressionAttributeNames` to avoid reserved words.

Example:

```js
ProjectionExpression: "#n, age",
ExpressionAttributeNames: { "#n": "name" }
```

---

### 6. **Scan Operation**

* Reads **every item** in the table or index
* Use only for **small datasets** or **one-time tools**
* You can use `FilterExpression` and `ProjectionExpression`

Example:

```js
const command = new ScanCommand({
  TableName: "Tasks",
  FilterExpression: "status = :s",
  ExpressionAttributeValues: { ":s": { S: "pending" } }
});
```

---

### 7. **Pagination**

Both `Query` and `Scan` may return partial results.

Look for:

```js
LastEvaluatedKey
```

Use it to request the next page:

```js
ExclusiveStartKey: lastKey
```

Loop until `LastEvaluatedKey` is `undefined`.

---

### 8. **Batch Operations**

DynamoDB supports **batch read and write**:

* `BatchGetItem`: Up to 100 items by key
* `BatchWriteItem`: Up to 25 `PutItem` or `DeleteItem` requests

---

### Exam Power-Up 💡

> Use `Query` when you know the Partition Key and want to efficiently retrieve sorted or filtered items. Use `Scan` only as a last resort. Filters reduce returned data but not the read cost. Always **page** through results with `LastEvaluatedKey`.

---

### Sample AWS Developer Associate Questions

**Q8:**
What is the key difference between a Query and a Scan in DynamoDB?

A. Query supports complex joins; Scan does not
B. Query retrieves items by Partition Key; Scan reads all items
C. Scan is faster than Query
D. Query works only with GSIs

**Answer:** B

**Explanation:** Query fetches items using Partition Key; Scan reads all items, so it's slower and more expensive.

---

**Q9:**
Which expression type lets you limit returned attributes in a Query?

A. FilterExpression
B. ConditionExpression
C. ProjectionExpression
D. KeyConditionExpression

**Answer:** C

**Explanation:** `ProjectionExpression` is used to specify which attributes you want to retrieve.

---
Let’s take a **deep dive into the `Query` command** in DynamoDB and how to use it effectively with **`KeyConditionExpression`** — which is the heart of most read operations in DynamoDB.

---

## DynamoDB `QueryCommand` (Detailed)

### What does `QueryCommand` do?

* It **retrieves items from a table or index** using the **Partition Key** and an **optional Sort Key condition**.
* It's **efficient** and uses the table’s indexing capabilities (not a full scan).
* Returns **paginated results** if there are many items.

---

### Syntax (AWS SDK v3 — Node.js)

```js
import { DynamoDBClient, QueryCommand } from "@aws-sdk/client-dynamodb";

const client = new DynamoDBClient({ region: "your-region" });

const command = new QueryCommand({
  TableName: "Tasks",
  KeyConditionExpression: "userId = :uid",
  ExpressionAttributeValues: {
    ":uid": { S: "user123" }
  }
});

const response = await client.send(command);
console.log(response.Items);
```

---

## Understanding `KeyConditionExpression`

The `KeyConditionExpression` defines the condition for:

1. The **Partition Key** (must be exact match)
2. The **Sort Key** (optional filters)

### Partition Key condition (mandatory):

* Must be **equality only**
* Format:
  `"partitionKey = :value"`

### Sort Key conditions (optional):

You can use:

* `=`
* `<`, `<=`, `>`, `>=`
* `BETWEEN :val1 AND :val2`
* `begins_with(sortKey, :prefix)`

---

### Example 1: Query by Partition Key only

```js
KeyConditionExpression: "userId = :uid",
ExpressionAttributeValues: {
  ":uid": { S: "user123" }
}
```

> This fetches **all items** where `userId = 'user123'`

---

### Example 2: Query by Partition + Sort Key range

```js
KeyConditionExpression: "userId = :uid AND taskId BETWEEN :start AND :end",
ExpressionAttributeValues: {
  ":uid": { S: "user123" },
  ":start": { S: "task100" },
  ":end": { S: "task200" }
}
```

> Fetches tasks between two IDs.

---

### Example 3: begins\_with on Sort Key

```js
KeyConditionExpression: "userId = :uid AND begins_with(taskId, :prefix)",
ExpressionAttributeValues: {
  ":uid": { S: "user123" },
  ":prefix": { S: "task" }
}
```

---

### Example 4: Combine with FilterExpression

```js
KeyConditionExpression: "userId = :uid",
FilterExpression: "status = :status",
ExpressionAttributeValues: {
  ":uid": { S: "user123" },
  ":status": { S: "completed" }
}
```

> This retrieves all tasks for `user123` **then filters** them in-memory to return only completed ones.

> ⚠️ FilterExpression **does not reduce read capacity cost** — only the response.

---

### Real-World Use Case

**Table:** `Tasks`

* Partition Key: `userId`
* Sort Key: `createdAt`

**Query:** Get all tasks for `user123` created in the last 7 days

```js
KeyConditionExpression: "userId = :uid AND createdAt >= :since",
ExpressionAttributeValues: {
  ":uid": { S: "user123" },
  ":since": { S: "2025-06-28T00:00:00Z" }
}
```

---

### Optional Parameters with Query

* `Limit`: Max number of items returned
* `ScanIndexForward: false`: To **sort descending**
* `ProjectionExpression`: Return only selected fields
* `ExclusiveStartKey`: For pagination

---

### Query Result

```json
{
  "Items": [
    {
      "userId": { "S": "user123" },
      "taskId": { "S": "task001" },
      "description": { "S": "Buy milk" }
    },
    ...
  ],
  "Count": 2,
  "ScannedCount": 2,
  "LastEvaluatedKey": null
}
```

---

## Common Pitfalls

| Mistake                                 | Why It Fails                                  |
| --------------------------------------- | --------------------------------------------- |
| Using `>` or `BETWEEN` on Partition Key | Only `=` is allowed for Partition Key         |
| Omitting ExpressionAttributeValues      | Required for every placeholder                |
| Using `FilterExpression` instead of GSI | Filters are not efficient for large datasets  |
| Forgetting pagination with large data   | Query only returns up to 1MB of data per call |

---

### Exam Power-Up 💡

> `KeyConditionExpression` is used in DynamoDB `QueryCommand` to filter by **Partition Key (mandatory)** and **Sort Key (optional)**. You can use conditions like `BETWEEN`, `begins_with`, or comparison operators only on the Sort Key.

---

### Sample Exam Question

**Q10:**
You want to get all items for `userId = 'user123'` and sort them by `createdAt`. What should your KeyConditionExpression look like?

A. `createdAt = :createdAt`
B. `userId = :uid AND begins_with(createdAt, :prefix)`
C. `userId = :uid AND createdAt > :startDate`
D. `userId = :uid`

**Answer:** C

**Explanation:**

* A is wrong: No Partition Key.
* B is wrong: `begins_with` is okay, but doesn’t retrieve a date range.
* C is correct: Partition + Sort Key condition.
* D is valid if you only want all items, not sorted.

---

Great question!

### What does `"S"` mean in `ExpressionAttributeValues`?

In DynamoDB's **low-level API**, each value must be **typed explicitly** using Amazon's internal data types.

`"S"` stands for **String**.

---

### Supported DynamoDB Data Types

| Code   | Meaning      | Example                                       |
| ------ | ------------ | --------------------------------------------- |
| `S`    | String       | `{ "S": "Rahman" }`                           |
| `N`    | Number       | `{ "N": "100" }` (as a string)                |
| `BOOL` | Boolean      | `{ "BOOL": true }`                            |
| `L`    | List (array) | `{ "L": [{ "S": "tag1" }, { "S": "tag2" }] }` |
| `M`    | Map (object) | `{ "M": { "city": { "S": "Sydney" } } }`      |
| `NULL` | Null         | `{ "NULL": true }`                            |
| `SS`   | String Set   | `{ "SS": ["a", "b"] }`                        |
| `NS`   | Number Set   | `{ "NS": ["1", "2"] }`                        |
| `BS`   | Binary Set   | `{ "BS": ["base64value"] }`                   |
| `B`    | Binary       | `{ "B": "base64encodedstring" }`              |

> All numbers (`N`, `NS`) are passed as **strings**, even though they represent numeric values.

---

### Example in QueryCommand

```js
ExpressionAttributeValues: {
  ":uid": { S: "user123" },    // String
  ":limit": { N: "5" }         // Number
}
```

If you use AWS SDK v3 with **DocumentClient** (`@aws-sdk/lib-dynamodb`), it **automatically handles data types** for you.

---

### SDK Shortcut (Using DocumentClient)

```js
import { DynamoDBDocumentClient } from "@aws-sdk/lib-dynamodb";

const docClient = DynamoDBDocumentClient.from(client); // wraps client
```

Then you can just pass native JS types:

```js
ExpressionAttributeValues: {
  ":uid": "user123",     // no need for { S: "user123" }
  ":limit": 5
}
```

---

### Exam Power-Up 💡

> In low-level DynamoDB operations, `"S"`, `"N"`, `"BOOL"` etc. are type indicators required for each attribute value. `"S"` represents a **String**, and `"N"` a **Number**, passed as a string.

---

Perfect! Let’s first wrap up the **remaining DynamoDB operations** — `PutItem`, `UpdateItem`, `DeleteItem`, and **pagination with `QueryCommand`** — and then move straight into the **Node.js + DynamoDB CRUD mini project**.

---

## Part 6: DynamoDB Commands + Pagination (with SDK v3)

---

### 1. **PutItemCommand**

Used to insert or replace an item.

```js
import { PutItemCommand } from "@aws-sdk/client-dynamodb";

const command = new PutItemCommand({
  TableName: "Tasks",
  Item: {
    userId: { S: "user123" },
    taskId: { S: "task001" },
    description: { S: "Buy milk" },
    status: { S: "pending" }
  }
});

await client.send(command);
```

> If the item already exists with the same PK, it **overwrites** it.

---

### 2. **UpdateItemCommand**

Used to modify attributes of an existing item.

```js
import { UpdateItemCommand } from "@aws-sdk/client-dynamodb";

const command = new UpdateItemCommand({
  TableName: "Tasks",
  Key: {
    userId: { S: "user123" },
    taskId: { S: "task001" }
  },
  UpdateExpression: "SET #s = :newStatus",
  ExpressionAttributeNames: { "#s": "status" },
  ExpressionAttributeValues: {
    ":newStatus": { S: "completed" }
  }
});

await client.send(command);
```

---

### 3. **DeleteItemCommand**

Removes a specific item by its key.

```js
import { DeleteItemCommand } from "@aws-sdk/client-dynamodb";

const command = new DeleteItemCommand({
  TableName: "Tasks",
  Key: {
    userId: { S: "user123" },
    taskId: { S: "task001" }
  }
});

await client.send(command);
```

---

### 4. **Pagination with QueryCommand**

DynamoDB returns up to **1MB** of data per call.

If more results are available, you get:

```js
LastEvaluatedKey: { ... }
```

You use this in the **next request** as `ExclusiveStartKey`.

#### Looping Through Pages:

```js
let lastKey;
let allResults = [];

do {
  const command = new QueryCommand({
    TableName: "Tasks",
    KeyConditionExpression: "userId = :uid",
    ExpressionAttributeValues: { ":uid": { S: "user123" } },
    ExclusiveStartKey: lastKey
  });

  const result = await client.send(command);
  allResults.push(...result.Items);
  lastKey = result.LastEvaluatedKey;

} while (lastKey);
```

> Always check `LastEvaluatedKey` to know if more results are available.

---

### Exam Power-Up 💡

> Use `PutItem` to insert or replace items, `UpdateItem` for partial updates, and `DeleteItem` to remove by key. `QueryCommand` returns up to 1MB — use `LastEvaluatedKey` + `ExclusiveStartKey` for pagination.

---

## Now: Mini Project Setup — Task Manager API

We’ll build a mini serverless app:

> **Task Manager API** with Node.js + AWS SDK v3 + DynamoDB

### Features:

* Add a task
* Get tasks by user
* Update task status
* Delete a task
* Optional: Paginate task results

Would you like to:

1. **Start with DynamoDB table design + schema**
2. Or jump directly into the **CRUD code setup in Node.js**?

Let me know how you'd like to start!


Great question — and a very common one when you're starting with DynamoDB.

Let’s clarify **why you must use `ExpressionAttributeNames` and `ExpressionAttributeValues`** instead of directly using attribute names and values in expressions.

---

## ❓Why Not Just Write This?

You might wonder why you can’t just write:

```js
UpdateExpression: "SET status = 'completed'"
```

Instead of:

```js
UpdateExpression: "SET #s = :val",
ExpressionAttributeNames: { "#s": "status" },
ExpressionAttributeValues: { ":val": { S: "completed" } }
```

The answer has **two main reasons**:

---

## ✅ 1. **Reserved Words in DynamoDB**

DynamoDB has a long list of **reserved keywords** that **cannot be used directly** in expressions — such as `status`, `name`, `date`, `size`, `type`, `order`, `timestamp`, etc.

So this will cause an error:

```js
UpdateExpression: "SET status = :val"
```

Because `status` is a reserved word.

**Solution**: Use `ExpressionAttributeNames` to safely reference any attribute:

```js
UpdateExpression: "SET #s = :val",
ExpressionAttributeNames: { "#s": "status" }
```

---

## ✅ 2. **Security and Injection Protection**

Using `ExpressionAttributeValues` helps avoid **injection vulnerabilities** and handles **dynamic value substitution** safely.

This is similar to using prepared statements in SQL:

```sql
SELECT * FROM users WHERE email = ?
```

In DynamoDB:

```js
KeyConditionExpression: "userId = :uid",
ExpressionAttributeValues: {
  ":uid": { S: "user123" }
}
```

This ensures:

* The value is **typed explicitly** (`S`, `N`, etc.)
* It's safely substituted into the expression

---

## Summary Table

| Reason               | Description                                                        |
| -------------------- | ------------------------------------------------------------------ |
| Reserved Words       | Many attribute names are reserved and cannot be used directly.     |
| Dynamic Placeholders | Lets you inject values safely and reuse expressions.               |
| Security             | Prevents injection, just like parameterized SQL.                   |
| Type Safety          | You must explicitly define types (`S`, `N`, etc.) for every value. |

---

## ✅ Can I Avoid This?

Yes — if you use **`@aws-sdk/lib-dynamodb`'s DocumentClient**, you can skip this:

```js
import { DynamoDBDocumentClient } from "@aws-sdk/lib-dynamodb";

const docClient = DynamoDBDocumentClient.from(client);

// Then you can write:
ExpressionAttributeValues: {
  ":val": "completed"  // No { S: "completed" } needed
}
```

But you'll **still need ExpressionAttributeNames** for reserved words like `name`, `status`, etc.

---

### Exam Power-Up 💡

> Use `ExpressionAttributeNames` to **avoid reserved word conflicts**, and use `ExpressionAttributeValues` to **inject typed, safe values**. These are required for all expressions involving updates, filters, or conditions.

---

### Sample Question

**Q11:**
Why do you need `ExpressionAttributeNames` in DynamoDB?

A. To avoid writing ProjectionExpressions
B. To define alternative table names
C. To prevent errors caused by reserved keywords
D. To create nested indexes

**Answer:** C

**Explanation:** Attribute names like `name`, `status`, and `size` are reserved. You must alias them using `ExpressionAttributeNames`.

---

Perfect! Let’s first wrap up the **remaining DynamoDB operations** — `PutItem`, `UpdateItem`, `DeleteItem`, and **pagination with `QueryCommand`** — and then move straight into the **Node.js + DynamoDB CRUD mini project**.

---

## Part 6: DynamoDB Commands + Pagination (with SDK v3)

---

### 1. **PutItemCommand**

Used to insert or replace an item.

```js
import { PutItemCommand } from "@aws-sdk/client-dynamodb";

const command = new PutItemCommand({
  TableName: "Tasks",
  Item: {
    userId: { S: "user123" },
    taskId: { S: "task001" },
    description: { S: "Buy milk" },
    status: { S: "pending" }
  }
});

await client.send(command);
```

> If the item already exists with the same PK, it **overwrites** it.

---

### 2. **UpdateItemCommand**

Used to modify attributes of an existing item.

```js
import { UpdateItemCommand } from "@aws-sdk/client-dynamodb";

const command = new UpdateItemCommand({
  TableName: "Tasks",
  Key: {
    userId: { S: "user123" },
    taskId: { S: "task001" }
  },
  UpdateExpression: "SET #s = :newStatus",
  ExpressionAttributeNames: { "#s": "status" },
  ExpressionAttributeValues: {
    ":newStatus": { S: "completed" }
  }
});

await client.send(command);
```

---

### 3. **DeleteItemCommand**

Removes a specific item by its key.

```js
import { DeleteItemCommand } from "@aws-sdk/client-dynamodb";

const command = new DeleteItemCommand({
  TableName: "Tasks",
  Key: {
    userId: { S: "user123" },
    taskId: { S: "task001" }
  }
});

await client.send(command);
```

---

### 4. **Pagination with QueryCommand**

DynamoDB returns up to **1MB** of data per call.

If more results are available, you get:

```js
LastEvaluatedKey: { ... }
```

You use this in the **next request** as `ExclusiveStartKey`.

#### Looping Through Pages:

```js
let lastKey;
let allResults = [];

do {
  const command = new QueryCommand({
    TableName: "Tasks",
    KeyConditionExpression: "userId = :uid",
    ExpressionAttributeValues: { ":uid": { S: "user123" } },
    ExclusiveStartKey: lastKey
  });

  const result = await client.send(command);
  allResults.push(...result.Items);
  lastKey = result.LastEvaluatedKey;

} while (lastKey);
```

> Always check `LastEvaluatedKey` to know if more results are available.

---

### Exam Power-Up 💡

> Use `PutItem` to insert or replace items, `UpdateItem` for partial updates, and `DeleteItem` to remove by key. `QueryCommand` returns up to 1MB — use `LastEvaluatedKey` + `ExclusiveStartKey` for pagination.

---
Awesome — moving forward in your roadmap, the **next topic** is:

## Part 7: DynamoDB with AWS Lambda (Serverless Architecture)

This section is critical for building **real-world, scalable, event-driven apps** using DynamoDB, especially in a serverless environment. It ties together everything we’ve learned so far with **Lambda + API Gateway + DynamoDB**.

---

### What You'll Learn in This Part

1. **How to trigger Lambda functions using DynamoDB Streams**
2. **How to use Lambda functions to perform CRUD operations on DynamoDB**
3. **How to expose those Lambdas as APIs using API Gateway**
4. **Deploy a complete serverless CRUD API with DynamoDB and Lambda**

---

## I. Triggering Lambda from DynamoDB Streams

### What are DynamoDB Streams?

* A stream of real-time changes (INSERT, MODIFY, REMOVE)
* Can trigger AWS Lambda
* Great for **real-time workflows**, **event sourcing**, **replication**, or **notifications**

### Steps:

1. Enable **Stream** on your DynamoDB table.
2. Choose `NEW_AND_OLD_IMAGES` for full context.
3. Create a Lambda function.
4. Add DynamoDB Stream as an **event source** for that Lambda.

### Stream Payload Example

When a new item is added:

```json
{
  "Records": [
    {
      "eventName": "INSERT",
      "dynamodb": {
        "NewImage": {
          "userId": { "S": "user123" },
          "taskId": { "S": "task001" },
          "description": { "S": "Buy milk" }
        }
      }
    }
  ]
}
```

---

## II. Lambda as the CRUD Layer for DynamoDB

### Why Use Lambda?

* Fully managed, scalable compute
* No need to manage infrastructure
* Perfect for building **microservices or APIs**

### Basic Lambda CRUD Handlers:

Each Lambda can handle a different operation:

| Operation          | Description         |
| ------------------ | ------------------- |
| `createTask`       | `PutItemCommand`    |
| `getTasks`         | `QueryCommand`      |
| `updateTaskStatus` | `UpdateItemCommand` |
| `deleteTask`       | `DeleteItemCommand` |

We’ll write these using **AWS SDK v3** with native or DocumentClient.

---

## III. Connecting Lambda to API Gateway

To expose Lambda functions as HTTP endpoints:

1. Create a REST API in API Gateway.
2. Define routes:

   * `POST /tasks` → createTask Lambda
   * `GET /tasks/{userId}` → getTasks Lambda
   * `PUT /tasks/{userId}/{taskId}` → updateTask Lambda
   * `DELETE /tasks/{userId}/{taskId}` → deleteTask Lambda
3. Use **Lambda proxy integration** for full control over request/response.

---

## IV. Diagram: Serverless Task API Flow

```
Client (e.g. React App)
       ↓ REST API
API Gateway (HTTP endpoint)
       ↓
    Lambda
       ↓
  DynamoDB Table
       ↑
DynamoDB Streams (optional)
       ↓
    Stream Processor Lambda
```

---

## V. Best Practices for Lambda + DynamoDB

| Practice                             | Reason                                  |
| ------------------------------------ | --------------------------------------- |
| Use DocumentClient                   | Easier type handling                    |
| Separate logic per Lambda            | Easier scaling and debugging            |
| Use environment variables            | Store table names, regions              |
| Use API Gateway validation           | Validate payloads before hitting Lambda |
| Use pagination in queries            | Avoid 1MB limit errors                  |
| Keep Lambda functions small and fast | AWS bills per ms                        |

---

### Exam Power-Up 💡

> You can trigger Lambda from **DynamoDB Streams** or use Lambda to **perform CRUD operations** on DynamoDB. Expose Lambda using **API Gateway** to build a **fully serverless REST API**.

---

### Sample Question

**Q12:**
Which of the following allows you to respond in real-time to changes in a DynamoDB table?

A. Amazon S3 Events
B. DynamoDB Streams with AWS Lambda
C. AWS Step Functions
D. Amazon CloudWatch Logs

**Answer:** B
**Explanation:** DynamoDB Streams can be used to trigger Lambda functions in response to data changes.

---

Great! Let’s begin building the **end-to-end serverless project**:

---

## **Mini Project: Serverless Task Manager API**

We’ll build a backend using:

* **DynamoDB** for storage
* **AWS Lambda** for business logic
* **API Gateway** for HTTP interface
* **Node.js (AWS SDK v3)** for interacting with DynamoDB

---

## 📦 Features to Build

| Feature            | HTTP Method | Endpoint                   | Lambda Function    |
| ------------------ | ----------- | -------------------------- | ------------------ |
| Create Task        | POST        | `/tasks`                   | `createTask`       |
| Get All Tasks      | GET         | `/tasks/{userId}`          | `getTasksByUser`   |
| Update Task Status | PUT         | `/tasks/{userId}/{taskId}` | `updateTaskStatus` |
| Delete Task        | DELETE      | `/tasks/{userId}/{taskId}` | `deleteTask`       |

---

## 1. Create DynamoDB Table

| Field                                | Type          | Notes  |
| ------------------------------------ | ------------- | ------ |
| `userId`                             | Partition Key | String |
| `taskId`                             | Sort Key      | String |
| `status`, `description`, `createdAt` | Attributes    |        |

Name the table: `Tasks`

In the AWS Console:

* Go to DynamoDB > Create Table
* Partition Key: `userId` (String)
* Sort Key: `taskId` (String)
* Capacity Mode: On-demand (for simplicity)

---

## 2. Create Lambda Functions

### Common Setup

Install dependencies:

```bash
npm install @aws-sdk/client-dynamodb @aws-sdk/lib-dynamodb
```

Then create a `dynamoClient.js`:

```js
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient } from "@aws-sdk/lib-dynamodb";

const client = new DynamoDBClient({ region: "us-east-1" });
export const docClient = DynamoDBDocumentClient.from(client);
```

---

### a. `createTask.js`

```js
import { PutCommand } from "@aws-sdk/lib-dynamodb";
import { docClient } from "./dynamoClient.js";
import { v4 as uuidv4 } from "uuid";

export const handler = async (event) => {
  const { userId, description } = JSON.parse(event.body);

  const task = {
    userId,
    taskId: uuidv4(),
    description,
    status: "pending",
    createdAt: new Date().toISOString(),
  };

  await docClient.send(new PutCommand({
    TableName: "Tasks",
    Item: task
  }));

  return {
    statusCode: 201,
    body: JSON.stringify(task)
  };
};
```

---

### b. `getTasksByUser.js`

```js
import { QueryCommand } from "@aws-sdk/lib-dynamodb";
import { docClient } from "./dynamoClient.js";

export const handler = async (event) => {
  const userId = event.pathParameters.userId;

  const result = await docClient.send(new QueryCommand({
    TableName: "Tasks",
    KeyConditionExpression: "userId = :uid",
    ExpressionAttributeValues: { ":uid": userId }
  }));

  return {
    statusCode: 200,
    body: JSON.stringify(result.Items)
  };
};
```

---

### c. `updateTaskStatus.js`

```js
import { UpdateCommand } from "@aws-sdk/lib-dynamodb";
import { docClient } from "./dynamoClient.js";

export const handler = async (event) => {
  const { userId, taskId } = event.pathParameters;
  const { status } = JSON.parse(event.body);

  await docClient.send(new UpdateCommand({
    TableName: "Tasks",
    Key: { userId, taskId },
    UpdateExpression: "SET #s = :s",
    ExpressionAttributeNames: { "#s": "status" },
    ExpressionAttributeValues: { ":s": status }
  }));

  return {
    statusCode: 200,
    body: JSON.stringify({ message: "Task updated." })
  };
};
```

---

### d. `deleteTask.js`

```js
import { DeleteCommand } from "@aws-sdk/lib-dynamodb";
import { docClient } from "./dynamoClient.js";

export const handler = async (event) => {
  const { userId, taskId } = event.pathParameters;

  await docClient.send(new DeleteCommand({
    TableName: "Tasks",
    Key: { userId, taskId }
  }));

  return {
    statusCode: 200,
    body: JSON.stringify({ message: "Task deleted." })
  };
};
```

---

## 3. Connect to API Gateway

In AWS Console:

* Go to **API Gateway**
* Create a new REST API
* Add resources:

  * `/tasks` → POST (create)
  * `/tasks/{userId}` → GET (get all)
  * `/tasks/{userId}/{taskId}` → PUT (update), DELETE (delete)
* Link each method to its respective Lambda function
* Deploy the API

---

## 4. Test the API

Use Postman or curl to:

* Create a task:

  ```bash
  curl -X POST https://<api>/tasks -d '{"userId": "user123", "description": "Buy milk"}'
  ```
* Get tasks:

  ```bash
  curl https://<api>/tasks/user123
  ```
* Update task:

  ```bash
  curl -X PUT https://<api>/tasks/user123/task001 -d '{"status": "completed"}'
  ```
* Delete task:

  ```bash
  curl -X DELETE https://<api>/tasks/user123/task001
  ```

---

Would you like me to help you package this with **Serverless Framework** or **AWS SAM**, or continue with the next roadmap topic: **Performance tuning, indexing, and best practices**?

Great! Since you said "yes", I’ll assume you want to **continue to the next topic in the roadmap**.

---

## Part 8: Indexing and Performance in DynamoDB

This part will help you scale your app properly by understanding how to use **Global Secondary Indexes (GSIs)**, **Local Secondary Indexes (LSIs)**, and how to avoid common **performance bottlenecks** like **hot partitions**.

---

### 1. **Global Secondary Index (GSI)** – Review & Use

**Use when:** You want to query the table using **different keys** (partition/sort key) than the base table.

#### Example Use Case:

In your `Tasks` table, you want to query all tasks with:

* status = "pending"
* ordered by `createdAt`

#### Create GSI:

* Partition Key: `status`
* Sort Key: `createdAt`
* Index Name: `status-createdAt-index`

Now, you can do:

```js
KeyConditionExpression: "status = :s AND createdAt > :date"
```

---

### 2. **Local Secondary Index (LSI)** – Use With Caution

**Use when:** You want an **alternative sort key** for the same partition key.

#### Example:

Partition Key: `userId`
LSI Sort Key: `dueDate`

You can now query:

```js
KeyConditionExpression: "userId = :uid AND dueDate < :today"
```

**Limitations:**

* Must be created **with the table**
* Max **5 LSIs per table**

---

### 3. **Avoiding Hot Partitions**

Hot partitions occur when:

* Too many requests hit the same partition key
* Results in throttling or degraded performance

#### Solutions:

* Use high-cardinality partition keys
* Add random suffixes/prefixes (e.g., `user#A`, `user#B`)
* Use **write sharding**

---

### 4. **Read/Write Capacity Modes**

| Mode            | Description                                            |
| --------------- | ------------------------------------------------------ |
| **On-Demand**   | Auto-scales. Best for unpredictable traffic.           |
| **Provisioned** | You manually set Read/Write Capacity Units (RCUs/WCUs) |

---

### 5. **Efficient Key Design**

Tips:

* Choose partition keys with **even distribution**
* Keep partition+sort key combos **unique**
* Use **shorter key names and values** (DynamoDB charges by size)

---

### 6. **Query vs Scan (again)**

* **Query** uses indexes, fast and cheap
* **Scan** reads the entire table, avoid for production use
* Always prefer **GSI** over using **Scan + FilterExpression**

---

### 7. **Monitoring & Metrics**

Use **Amazon CloudWatch** to monitor:

* ThrottledRequests
* ConsumedReadCapacityUnits
* UserErrors (e.g., invalid expression syntax)
* Latency

---

### Exam Power-Up 💡

> Use **GSIs** for alternative query patterns, but avoid **hot partitions** by choosing high-cardinality partition keys. Always use **Query** over Scan, and monitor usage via **CloudWatch**. Choose **On-Demand** capacity for unpredictable traffic patterns.

---

### Sample Exam Questions

**Q13:**
You have a DynamoDB table with a large number of items sharing the same partition key. What is a likely result?

A. Faster queries due to better caching
B. Items are automatically balanced
C. Hot partition and throttling
D. DynamoDB rejects the inserts

**Answer:** C
**Explanation:** Too many operations on the same partition key = hot partition = throttling.

---

**Q14:**
Which statement about GSIs is true?

A. A GSI must have the same partition key as the base table
B. A GSI cannot be added after table creation
C. A GSI supports different partition and sort keys
D. A GSI can store unlimited items per partition

**Answer:** C

---
Great follow-up — and the answer is:

### ❌ No, **Scan is not a GSI**.

They are completely different concepts:

---

## 🔍 `Scan` is an **operation**

* It’s a **way to read all the data** from a table or index.
* It **does not require any key**.
* It can be used on:

  * A **table**
  * Or a **GSI** (yes, you can scan a GSI — but you still shouldn't unless it's small)

---

## 📦 GSI (Global Secondary Index) is a **structure**

* A GSI is like a **copy of your table**, with **different keys** (partition/sort).
* You **define it** to support different access patterns.
* Then you can run **Query operations** on it efficiently.

---

### ✅ You can **Scan a GSI**, but that doesn’t make Scan = GSI.

---

## Quick Comparison

| Term   | Type      | Purpose                                                      |
| ------ | --------- | ------------------------------------------------------------ |
| `Scan` | Operation | Read **everything** from table or index                      |
| `GSI`  | Index     | Create a **new key structure** to support efficient querying |

---

### Example: Scanning a GSI

Let’s say you have a GSI called `status-createdAt-index`:

```js
const command = new ScanCommand({
  TableName: "Tasks",
  IndexName: "status-createdAt-index",
  FilterExpression: "status = :s",
  ExpressionAttributeValues: {
    ":s": "pending"
  }
});
```

This performs a **Scan on the GSI**, but it’s still inefficient if the GSI has lots of data.

---

## So Remember:

* **Scan** = Operation (slow, non-key-based)
* **GSI** = Index structure (used to support fast queries)

---

### Exam Power-Up 💡

> `Scan` is an operation that reads all data, optionally with a filter. A `GSI` is an index that allows fast `Query` operations using alternate keys. You can scan a GSI, but it’s still a scan — not an efficient access method.

---
✅ **Yes, you are absolutely correct!**

In **DynamoDB**, you **can define a GSI (Global Secondary Index) after the table has been created** — and this is one of the key benefits of GSIs.

---

## 🛠️ Global Secondary Index (GSI) – Post-Table Creation

### What you can do:

* ✅ Add new GSIs **after** table creation.
* ✅ Add up to **20 GSIs per table** (soft limit).
* ✅ Choose **any attributes** as the GSI’s partition and sort key — even ones not in the table’s primary key.

---

## ❌ What You Cannot Do:

* ❌ You **cannot modify or delete** the **Partition Key** or **Sort Key** of the base table.
* ❌ You **cannot create LSIs** after table creation (Local Secondary Indexes must be created **at table creation time**).

---

### Example: Add GSI After Table Creation

You already have a `Tasks` table with:

* Partition Key: `userId`
* Sort Key: `taskId`

Now, you want to **query tasks by `status` and sort by `createdAt`**, regardless of user.

You can add a GSI:

* **Index Name**: `status-createdAt-index`
* **Partition Key**: `status`
* **Sort Key**: `createdAt`

In the console or via SDK/CloudFormation, just define:

```json
{
  "IndexName": "status-createdAt-index",
  "KeySchema": [
    { "AttributeName": "status", "KeyType": "HASH" },
    { "AttributeName": "createdAt", "KeyType": "RANGE" }
  ],
  "Projection": {
    "ProjectionType": "ALL"
  }
}
```

Then you can `Query` the GSI like this:

```js
KeyConditionExpression: "status = :s AND createdAt > :date"
```

---

### Exam Power-Up 💡

> **GSIs can be added after table creation**, and allow querying data using **alternate partition and sort keys**. In contrast, **LSIs must be defined at table creation time** and always share the base table’s partition key.

---

### Sample Question:

**Q16:**
Which of the following is true about GSIs in DynamoDB?

A. GSIs must use the same partition key as the base table
B. GSIs must be created at the same time as the table
C. GSIs allow alternate partition/sort keys and can be added anytime
D. GSIs support strong consistency by default

**Answer:** C

---
Awesome! Let's enhance our **Task Manager API** by adding a **Global Secondary Index (GSI)** so we can **query tasks by status** — regardless of user.

---

## 🎯 Goal

We want to:

* Add a GSI: `status-createdAt-index`
* Query all tasks with `status = "pending"` (e.g., for an admin dashboard)
* Sort them by `createdAt`

---

## ✅ Step 1: Modify the DynamoDB Table

If you're using the AWS Console:

1. Go to **DynamoDB > Tables > Tasks**
2. Click **Indexes → Create index**
3. Enter:

   * **Index name**: `status-createdAt-index`
   * **Partition Key**: `status` (String)
   * **Sort Key**: `createdAt` (String)
   * Projection: `All attributes`
4. Click **Create index**

Wait for the index to become **ACTIVE** before using it.

---

## ✅ Step 2: Query the GSI from Lambda

### Create a new Lambda: `getTasksByStatus.js`

```js
import { QueryCommand } from "@aws-sdk/lib-dynamodb";
import { docClient } from "./dynamoClient.js";

export const handler = async (event) => {
  const { status } = event.pathParameters;

  const command = new QueryCommand({
    TableName: "Tasks",
    IndexName: "status-createdAt-index",
    KeyConditionExpression: "status = :s",
    ExpressionAttributeValues: {
      ":s": status
    },
    ScanIndexForward: false  // to sort newest first
  });

  const result = await docClient.send(command);

  return {
    statusCode: 200,
    body: JSON.stringify(result.Items)
  };
};
```

---

## ✅ Step 3: Add a Route in API Gateway

Create a new REST route:

* **GET** `/tasks/status/{status}`
* Integration: Lambda function `getTasksByStatus`

Test with:

```bash
curl https://your-api-url/tasks/status/pending
```

---

## ✅ Step 4: Add Sample Items

If you're testing with the console or Postman, use your existing POST endpoint to create tasks like:

```json
{
  "userId": "user123",
  "description": "Write blog post",
  "status": "pending"
}
```

Now query `/tasks/status/pending` and it should return all matching tasks across all users.

---

## 📘 Optional Enhancements

* Add pagination to support large lists using `LastEvaluatedKey`
* Add a GSI on `createdAt` alone (to get the newest tasks globally)
* Add filtering by date using sort key ranges (e.g., `createdAt BETWEEN`)

---

### Exam Power-Up 💡

> GSIs let you support **alternate access patterns** like querying by `status` or `createdAt`. You can **query a GSI** just like a table — use `IndexName` and `KeyConditionExpression`.

---
Perfect — moving on to:

## Part 9: DynamoDB Best Practices

These best practices are essential for building efficient, scalable, and cost-effective applications using DynamoDB, and they are frequently tested in the **AWS Developer Associate exam**.

---

### 1. **Design Your Schema Based on Access Patterns**

Unlike relational databases:

* DynamoDB is **query-first**
* You must **model your data around how you access it**

✅ Before designing your table, ask:

* What will I query?
* How often will I access it?
* Will I need to sort it?

---

### 2. **Use Single-Table Design Where Appropriate**

Single-table design:

* Stores **all entity types** (e.g., users, tasks, products) in **one table**
* Differentiates using a `type` field or by formatting keys (`PK = user#123`, `PK = order#456`)

✅ Pros:

* Fewer queries
* All related data in a partition
* Faster joins (via access pattern)

❌ Cons:

* More complex logic
* Requires **thoughtful key design**

---

### 3. **Avoid Scans in Production**

* `Scan` is **inefficient**, especially for large tables.
* Use **Query**, **GSI**, or **BatchGetItem** instead.

✅ Scan is acceptable for:

* Admin tools
* One-time data cleanups
* Small tables only

---

### 4. **Choose High-Cardinality Partition Keys**

High-cardinality = lots of unique values.

✅ Why?

* Avoids **hot partitions**
* Ensures **even traffic distribution**

❌ Avoid low-cardinality keys like:

* `status` (e.g., "pending", "done") — too many items in one partition

---

### 5. **Use Projections Wisely in GSIs**

You can project:

* `ALL` (default): all attributes
* `KEYS_ONLY`: just partition and sort keys
* `INCLUDE`: selected attributes

✅ Choose `INCLUDE` to reduce storage and cost when you don’t need all attributes.

---

### 6. **Use Time to Live (TTL)**

TTL is a field that lets DynamoDB **auto-delete expired items**.

✅ Great for:

* Session tokens
* Temporary orders
* One-time events

Just add a `ttl` attribute with a **Unix epoch timestamp** in seconds.

---

### 7. **Paginate Results**

* Use `LastEvaluatedKey` to handle large result sets.
* Set `Limit` to avoid overwhelming the client or hitting API limits.

---

### 8. **Use Conditional Writes**

Use `ConditionExpression` to:

* Prevent overwriting existing items
* Enforce business rules (e.g., only update if status is `pending`)

Example:

```js
ConditionExpression: "attribute_not_exists(userId)"
```

---

### 9. **Monitor Usage with CloudWatch**

Track:

* Read/Write usage
* Throttled requests
* Latency
* DynamoDB errors (4XX, 5XX)

✅ Use this to **right-size capacity** or spot bad access patterns.

---

### 10. **Choose the Right Capacity Mode**

| Mode            | When to Use                            |
| --------------- | -------------------------------------- |
| **On-Demand**   | Unpredictable or spiky workloads       |
| **Provisioned** | Consistent workloads; cheaper at scale |
| + Auto Scaling  | Dynamically adjust capacity            |

---

### 11. **Don't Abuse Updates**

Every `UpdateItem`:

* Reads the full item
* Applies change
* Writes back

❌ Avoid constant partial updates on large objects. Use **targeted attributes**, not huge objects.

---

### 12. **Don't Overuse GSIs**

While helpful:

* GSIs take up storage
* They must be updated on every write
* They **do not support strongly consistent reads**

✅ Use only for **critical access patterns**

---

### Exam Power-Up 💡

> Design DynamoDB schemas for **query efficiency**, not normalization. Use **high-cardinality keys**, avoid **Scan**, leverage **GSIs**, and monitor capacity with **CloudWatch**. Enable **TTL** for expiring data and paginate large results with `LastEvaluatedKey`.

---

### Sample Question:

**Q17:**
Which of the following is a recommended practice when choosing a partition key?

A. Use low-cardinality attributes like status
B. Use a fixed string for all items
C. Use high-cardinality, evenly distributed attributes
D. Use a random UUID always

**Answer:** C
**Explanation:** You want even access distribution across partitions to avoid throttling.

---


Awesome — here's a **real-world, industry-grade serverless project** that leverages **AWS Lambda, API Gateway, DynamoDB, Cognito, CloudWatch**, and more. This project is perfect to **showcase your full-stack AWS skills on your resume or portfolio**.

---

## 🚀 Project Title: **SkillForge – Serverless Learning Management System (LMS)**

### ✨ What It Is:

A fully serverless LMS (like Udemy, Coursera) where:

* Users can sign up, log in, and enroll in courses
* Admins can create, update, and manage courses, lessons, and users
* Learners can track their progress, complete quizzes, and leave reviews

---

## 🧱 Tech Stack

| Layer        | Technology                                |
| ------------ | ----------------------------------------- |
| Frontend     | React / Next.js (optional)                |
| Auth         | **Amazon Cognito**                        |
| API Layer    | **API Gateway + Lambda (Node.js)**        |
| Database     | **Amazon DynamoDB** (single-table design) |
| Events       | **DynamoDB Streams + Lambda**             |
| Logs/Metrics | **CloudWatch + X-Ray**                    |
| Storage      | **S3 (course images, video files)**       |
| Deployment   | **Serverless Framework** or **SAM**       |

---

## 📂 DynamoDB Table Design (Single Table)

| Partition Key       | Sort Key                         | Type         |
| ------------------- | -------------------------------- | ------------ |
| `user#<userId>`     | `profile`                        | User Profile |
| `course#<courseId>` | `course`                         | Course Data  |
| `course#<courseId>` | `lesson#<lessonId>`              | Lesson       |
| `user#<userId>`     | `enroll#<courseId>`              | Enrollment   |
| `course#<courseId>` | `review#<reviewId>`              | Review       |
| `user#<userId>`     | `progress#<courseId>#<lessonId>` | Progress     |

Use GSIs:

* `GSI1: courseStatus-createdAt-index` → for published courses
* `GSI2: userType-email-index` → for admin vs learner
* `GSI3: reviewScore-courseId-index` → for top-rated courses

---

## 🔐 Authentication with Cognito

* Cognito User Pool: sign-up/login with email/phone
* Use **Cognito authorizer** in API Gateway
* Add **role-based access**: `admin`, `instructor`, `student`

---

## 📡 API Routes (with Pagination + Best Practices)

### ✅ Public Routes

| Method | Path                       | Description                            |
| ------ | -------------------------- | -------------------------------------- |
| GET    | `/courses`                 | List all published courses (paginated) |
| GET    | `/courses/{id}`            | Course details                         |
| GET    | `/courses/{id}/lessons`    | Get lessons in a course                |
| GET    | `/courses/search?query=js` | Full-text search (via GSI)             |

### 🔐 Authenticated User Routes

| Method | Path                              | Description                       |
| ------ | --------------------------------- | --------------------------------- |
| POST   | `/enroll/{courseId}`              | Enroll in a course                |
| GET    | `/me/enrollments`                 | List enrolled courses (paginated) |
| PUT    | `/progress/{courseId}/{lessonId}` | Update lesson progress            |
| POST   | `/reviews/{courseId}`             | Submit a review                   |

### 🔐 Admin/Instructor Routes

| Method | Path                    | Description                     |
| ------ | ----------------------- | ------------------------------- |
| POST   | `/courses`              | Create course                   |
| PUT    | `/courses/{id}`         | Update course                   |
| DELETE | `/courses/{id}`         | Delete course                   |
| POST   | `/courses/{id}/lessons` | Add a lesson to course          |
| PUT    | `/lessons/{id}`         | Update lesson                   |
| GET    | `/admin/users`          | View users (GSI for type=email) |

---

## 📜 Features to Implement

1. **Pagination**: `Limit` + `LastEvaluatedKey` on every GET list route
2. **Validation**: Use `API Gateway Request Models` or validation inside Lambda
3. **Role-based Auth**: Validate Cognito claims (`group: admin/student`)
4. **Logging**: Use CloudWatch structured logs + correlation ID
5. **Monitoring**: CloudWatch alarms for error rates and latency
6. **Throttling & Caching**: Enable throttling in API Gateway; optional caching for public routes
7. **Pre-Signed URLs for File Uploads**: Users upload lesson files or avatars to S3 using secure URLs
8. **Serverless Deployment**: Use Serverless Framework with per-stage environments

---

## 🌐 Deployment with Serverless Framework

Directory structure:

```
/lms-api
  /handlers
    - createCourse.js
    - getCourses.js
    ...
  /libs
    - dynamoClient.js
    - authUtils.js
serverless.yml
```

Sample `serverless.yml` snippet:

```yaml
functions:
  getCourses:
    handler: handlers/getCourses.handler
    events:
      - http:
          path: courses
          method: get
          cors: true
          authorizer:
            type: COGNITO_USER_POOLS
            userPoolArn: arn:aws:cognito-idp:...
```

---

## ✅ Best Practices Summary

* ✅ Single Table Design
* ✅ Query-first modeling
* ✅ Role-based Cognito authentication
* ✅ Pagination with `Limit` + `LastEvaluatedKey`
* ✅ Use GSIs for alternate access
* ✅ Conditional writes (e.g. only enroll once)
* ✅ CloudWatch structured logging + alarms
* ✅ Lambda timeouts + memory optimized
* ✅ Use environment variables for table/index names

---

## 🧠 Highlight for Resume

**Project:** *SkillForge – Serverless LMS*

**What I did:**

* Built a fully serverless backend using **Lambda, API Gateway, DynamoDB, and Cognito**
* Implemented authentication and role-based access with **Cognito**
* Designed a **single-table schema** supporting multiple entities and access patterns
* Used **GSIs and pagination** for optimized querying
* Monitored with **CloudWatch**, structured logging, and alarms
* Deployed via **Serverless Framework** with multiple environments

---
Perfect! Let’s **scaffold the SkillForge LMS backend step-by-step** using:

* **Node.js (AWS SDK v3 + modular handlers)**
* **DynamoDB (Single Table Design)**
* **Cognito Authentication**
* **API Gateway Integration**
* **Serverless Framework for deployment**
* **Full CRUD, pagination, role-based access, and CloudWatch logs**

---

## 🧱 Project Scaffolding: `skillforge-lms-api`

### 📁 Directory Structure

```
skillforge-lms-api/
├── handlers/
│   ├── auth/
│   │   ├── getUser.js
│   ├── courses/
│   │   ├── createCourse.js
│   │   ├── listCourses.js
│   │   ├── getCourseById.js
│   ├── lessons/
│   │   ├── addLesson.js
│   │   ├── getLessonsByCourse.js
│   ├── enrollments/
│   │   ├── enrollInCourse.js
│   │   ├── listEnrollments.js
│   ├── reviews/
│   │   ├── postReview.js
│   ├── users/
│   │   ├── listUsers.js
│   └── progress/
│       ├── updateProgress.js
├── libs/
│   ├── dynamoClient.js
│   ├── response.js
│   ├── authUtils.js
├── serverless.yml
├── package.json
└── README.md
```

---

## 📦 Step 1: Initialize Project

```bash
mkdir skillforge-lms-api && cd skillforge-lms-api
npm init -y
npm install @aws-sdk/client-dynamodb @aws-sdk/lib-dynamodb uuid
npm install --save-dev serverless dotenv
```

---

## 🔧 Step 2: Create `dynamoClient.js`

```js
// libs/dynamoClient.js
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient } from "@aws-sdk/lib-dynamodb";

const client = new DynamoDBClient({ region: "us-east-1" });
export const docClient = DynamoDBDocumentClient.from(client);
```

---

## 🛠️ Step 3: Create `response.js` Helper

```js
// libs/response.js
export const success = (data) => ({
  statusCode: 200,
  body: JSON.stringify(data)
});

export const failure = (err) => ({
  statusCode: 500,
  body: JSON.stringify({ error: err.message })
});
```

---

## 🔐 Step 4: Auth Util to Check Roles

```js
// libs/authUtils.js
export const isAdmin = (event) => {
  const claims = event.requestContext.authorizer.claims;
  return claims["cognito:groups"]?.includes("admin");
};

export const getUserId = (event) => {
  return event.requestContext.authorizer.claims.sub;
};
```

---

## ✍️ Step 5: Create First Lambda — `createCourse.js`

```js
// handlers/courses/createCourse.js
import { PutCommand } from "@aws-sdk/lib-dynamodb";
import { docClient } from "../../libs/dynamoClient.js";
import { success, failure } from "../../libs/response.js";
import { isAdmin } from "../../libs/authUtils.js";
import { v4 as uuidv4 } from "uuid";

export const handler = async (event) => {
  try {
    if (!isAdmin(event)) {
      return { statusCode: 403, body: "Forbidden" };
    }

    const { title, description, category } = JSON.parse(event.body);
    const courseId = uuidv4();
    const now = new Date().toISOString();

    const item = {
      PK: `course#${courseId}`,
      SK: "course",
      courseId,
      title,
      description,
      category,
      createdAt: now,
      status: "draft"
    };

    await docClient.send(new PutCommand({
      TableName: process.env.TABLE_NAME,
      Item: item
    }));

    return success(item);
  } catch (err) {
    return failure(err);
  }
};
```

---

## 🌐 Step 6: Define `serverless.yml`

```yaml
service: skillforge-lms-api
frameworkVersion: '3'

provider:
  name: aws
  runtime: nodejs20.x
  region: us-east-1
  environment:
    TABLE_NAME: SkillForgeLMS

  iam:
    role:
      statements:
        - Effect: Allow
          Action:
            - dynamodb:*
          Resource: "*"

functions:
  createCourse:
    handler: handlers/courses/createCourse.handler
    events:
      - http:
          path: courses
          method: post
          cors: true
          authorizer:
            type: COGNITO_USER_POOLS
            userPoolArn: arn:aws:cognito-idp:your-region:acct-id:userpool/your-user-pool-id
```

> Replace `userPoolArn` with your actual Cognito pool ARN.

---

## 🧪 Step 7: Deploy & Test

```bash
npx serverless deploy
```

Then call your endpoint with a valid **Cognito token**:

```bash
curl -X POST https://<your-api>.amazonaws.com/dev/courses \
  -H "Authorization: Bearer <your-cognito-jwt>" \
  -d '{"title":"Next.js 14 Mastery", "description":"..." }'
```

---

Would you like me to:

* Continue scaffolding the next endpoints (enrollments, lessons, reviews)?
* Or help you connect this to a **frontend React app with Cognito Auth + API calls**?
Excellent — let’s continue building the **SkillForge LMS API** by adding more real-world endpoints, following best practices like:

* Cognito authentication
* Role-based access
* Pagination
* Proper key design
* Reusable logic with shared utilities

---

## 🧩 Next Endpoint: `listCourses.js` (Public, Paginated)

### Goal:

Allow anyone to list **published courses**, sorted by `createdAt`, with pagination.

> We'll use a **GSI**:
>
> * `IndexName`: `status-createdAt-index`
> * `PartitionKey`: `status`
> * `SortKey`: `createdAt`

---

### ✅ DynamoDB GSI Setup

Make sure your `SkillForgeLMS` table includes:

```json
"status-createdAt-index": {
  "PartitionKey": "status",
  "SortKey": "createdAt",
  "Projection": "ALL"
}
```

In Serverless Framework or Console, add the index.

---

### 📄 `handlers/courses/listCourses.js`

```js
import { QueryCommand } from "@aws-sdk/lib-dynamodb";
import { docClient } from "../../libs/dynamoClient.js";
import { success, failure } from "../../libs/response.js";

export const handler = async (event) => {
  try {
    const limit = Number(event.queryStringParameters?.limit) || 10;
    const lastKey = event.queryStringParameters?.lastKey
      ? JSON.parse(decodeURIComponent(event.queryStringParameters.lastKey))
      : undefined;

    const result = await docClient.send(new QueryCommand({
      TableName: process.env.TABLE_NAME,
      IndexName: "status-createdAt-index",
      KeyConditionExpression: "#s = :published",
      ExpressionAttributeNames: { "#s": "status" },
      ExpressionAttributeValues: { ":published": "published" },
      Limit: limit,
      ExclusiveStartKey: lastKey,
      ScanIndexForward: false  // newest first
    }));

    return success({
      items: result.Items,
      lastKey: result.LastEvaluatedKey
        ? encodeURIComponent(JSON.stringify(result.LastEvaluatedKey))
        : null
    });
  } catch (err) {
    return failure(err);
  }
};
```

---

### 🔧 Add to `serverless.yml`

```yaml
  listCourses:
    handler: handlers/courses/listCourses.handler
    events:
      - http:
          path: courses
          method: get
          cors: true
```

---

### 🧪 Test the API

```bash
curl "https://<api-url>/courses?limit=5"
```

If there are more results, the response will include:

```json
{
  "items": [...],
  "lastKey": "%7B%22PK%22%3A%22course%23id123%22%2C%22SK%22%3A%22course%22%7D"
}
```

Use `lastKey` in the next request:

```bash
curl "https://<api-url>/courses?limit=5&lastKey=<encoded-key>"
```

---

## 🔐 Up Next: Authenticated User Flow

### Next endpoint: `enrollInCourse.js`

Would you like me to continue with:

1. **Enrollment API** (user → course)
2. **Lesson listing + access control**
3. **User review + rating system**
4. **Frontend sample integration (React/Cognito)**

Let me know your preferred next step!
Great! Let’s now implement the **Enrollment API**, where a user (student) can enroll in a course. This endpoint will:

* Require authentication (via Cognito)
* Prevent duplicate enrollments
* Use **DynamoDB conditional writes**
* Store the enrollment using our **single-table design**

---

## 🧩 Endpoint: `enrollInCourse.js`

### Goal:

Allow an authenticated user to enroll in a course:

* No double enrollment
* Store:
  `PK = user#<userId>`
  `SK = enroll#<courseId>`

---

### 📄 `handlers/enrollments/enrollInCourse.js`

```js
import { PutCommand } from "@aws-sdk/lib-dynamodb";
import { docClient } from "../../libs/dynamoClient.js";
import { success, failure } from "../../libs/response.js";
import { getUserId } from "../../libs/authUtils.js";

export const handler = async (event) => {
  try {
    const userId = getUserId(event);
    const courseId = event.pathParameters.courseId;

    const now = new Date().toISOString();

    const enrollment = {
      PK: `user#${userId}`,
      SK: `enroll#${courseId}`,
      enrolledAt: now,
      progress: 0
    };

    await docClient.send(new PutCommand({
      TableName: process.env.TABLE_NAME,
      Item: enrollment,
      ConditionExpression: "attribute_not_exists(PK)"  // no overwrite
    }));

    return success({ message: "Enrollment successful." });
  } catch (err) {
    if (err.name === "ConditionalCheckFailedException") {
      return { statusCode: 400, body: JSON.stringify({ error: "Already enrolled." }) };
    }
    return failure(err);
  }
};
```

---

### 🔧 Add to `serverless.yml`

```yaml
  enrollInCourse:
    handler: handlers/enrollments/enrollInCourse.handler
    events:
      - http:
          path: enroll/{courseId}
          method: post
          cors: true
          authorizer:
            type: COGNITO_USER_POOLS
            userPoolArn: arn:aws:cognito-idp:...
```

---

### 🧪 Test the API

```bash
curl -X POST https://<api-url>/enroll/course123 \
  -H "Authorization: Bearer <cognito-jwt>"
```

---

## 📦 Next Step Suggestions:

We can continue with:

1. **GET /me/enrollments** — List all enrolled courses (paginated)
2. **POST /progress/{courseId}/{lessonId}** — Track lesson progress
3. **POST /reviews/{courseId}** — Submit a review
4. **GET /lessons/{courseId}** — List lessons only for enrolled users

Would you like to continue with **listing enrollments** or move into **lesson access control** next?

Great — let’s continue with:

## 🧩 Endpoint: `listEnrollments.js`

> Authenticated users can fetch all the courses they've enrolled in, with **pagination support**.

---

### 💡 Data Layout Reminder

When a user enrolls:

* `PK = user#<userId>`
* `SK = enroll#<courseId>`
* Also stores `enrolledAt`, `progress`, etc.

So to list enrolled courses:

> We’ll `Query` using the **Partition Key**: `user#<userId>`
> and filter where `begins_with(SK, "enroll#")`

---

### 📄 `handlers/enrollments/listEnrollments.js`

```js
import { QueryCommand } from "@aws-sdk/lib-dynamodb";
import { docClient } from "../../libs/dynamoClient.js";
import { success, failure } from "../../libs/response.js";
import { getUserId } from "../../libs/authUtils.js";

export const handler = async (event) => {
  try {
    const userId = getUserId(event);
    const limit = Number(event.queryStringParameters?.limit) || 5;
    const lastKey = event.queryStringParameters?.lastKey
      ? JSON.parse(decodeURIComponent(event.queryStringParameters.lastKey))
      : undefined;

    const result = await docClient.send(new QueryCommand({
      TableName: process.env.TABLE_NAME,
      KeyConditionExpression: "PK = :pk AND begins_with(SK, :prefix)",
      ExpressionAttributeValues: {
        ":pk": `user#${userId}`,
        ":prefix": "enroll#"
      },
      Limit: limit,
      ExclusiveStartKey: lastKey
    }));

    return success({
      items: result.Items,
      lastKey: result.LastEvaluatedKey
        ? encodeURIComponent(JSON.stringify(result.LastEvaluatedKey))
        : null
    });
  } catch (err) {
    return failure(err);
  }
};
```

---

### 🔧 Add to `serverless.yml`

```yaml
  listEnrollments:
    handler: handlers/enrollments/listEnrollments.handler
    events:
      - http:
          path: me/enrollments
          method: get
          cors: true
          authorizer:
            type: COGNITO_USER_POOLS
            userPoolArn: arn:aws:cognito-idp:...
```

---

### 🧪 Test the Endpoint

```bash
curl "https://<api-url>/me/enrollments?limit=5" \
  -H "Authorization: Bearer <cognito-token>"
```

The response:

```json
{
  "items": [
    {
      "PK": "user#abc123",
      "SK": "enroll#course456",
      "enrolledAt": "2025-07-01T10:00Z",
      "progress": 0
    }
  ],
  "lastKey": "%7B%22PK%22%3A%22user%23abc123%22%2C%22SK%22%3A%22enroll%23course456%22%7D"
}
```

Use the `lastKey` for pagination:

```bash
curl "https://<api-url>/me/enrollments?limit=5&lastKey=<encoded>"
```

---

### ✅ Summary of Best Practices Used

* Query with partition key (`PK`)
* Filter on sort key prefix with `begins_with`
* Paginate using `LastEvaluatedKey`
* Secure access via Cognito Auth
* Reusable helpers and clean modular code

---

Would you like to continue with:

1. **Lesson listing for enrolled users only** (with access control)
2. **Progress tracking API**
3. **Course review submission**
4. **CloudWatch alerting and structured logging**
5. Or start frontend integration with React + Cognito?
Excellent choice! Let’s implement **CloudWatch logging and alerting** — an essential part of making your **SkillForge LMS** truly production-grade.

---

## Part 4️⃣: CloudWatch Logging, Metrics, and Alerts

A high-quality serverless system must include:

* ✅ **Structured logs** for debugging and audits
* ✅ **Metrics** for performance monitoring
* ✅ **Alarms** for error or latency thresholds
* ✅ Optional: **AWS X-Ray** for tracing (advanced)

---

## 🧱 1. Structured Logging in Lambda

### Use CloudWatch logs with structure and correlation IDs.

### 🔧 Create Utility: `libs/logger.js`

```js
export const log = (level, message, context = {}) => {
  const logEntry = {
    level,
    timestamp: new Date().toISOString(),
    message,
    ...context
  };
  console.log(JSON.stringify(logEntry));
};
```

Use in any Lambda handler:

```js
import { log } from "../../libs/logger.js";

log("info", "User enrolled", { userId, courseId });
```

---

## 📊 2. CloudWatch Metrics (Automatic)

Lambda logs and metrics (duration, errors, invocations) are created automatically per function:

* `Invocations`
* `Errors`
* `Duration`
* `Throttles`

Go to:
**CloudWatch → Metrics → AWS/Lambda → By Function Name**

You can create custom metrics (advanced):

```js
import { CloudWatchClient, PutMetricDataCommand } from "@aws-sdk/client-cloudwatch";

const cloudwatch = new CloudWatchClient({ region: "us-east-1" });

await cloudwatch.send(new PutMetricDataCommand({
  Namespace: "SkillForgeLMS",
  MetricData: [
    {
      MetricName: "Enrollments",
      Dimensions: [{ Name: "Function", Value: "enrollInCourse" }],
      Unit: "Count",
      Value: 1
    }
  ]
}));
```

---

## 🚨 3. CloudWatch Alarms (via Console or Serverless)

Set alarms on:

* ❌ `Errors > 0` for any function
* 🕑 `Duration > 2s`
* 🧍 `Throttles > 0`
* 📈 Custom metric spikes (e.g., too many enrollments)

---

## 🔧 4. Add Alarms in `serverless.yml`

```yaml
plugins:
  - serverless-plugin-aws-alerts

custom:
  alerts:
    stages:
      - dev
    topics:
      alarm:
        topic: "SkillForgeAlarms"
        notifications:
          - protocol: email
            endpoint: your.email@example.com
    alarms:
      - functionErrors
      - functionThrottles
      - functionDuration
```

Install:

```bash
npm install --save-dev serverless-plugin-aws-alerts
```

Then:

```bash
npx serverless deploy
```

You’ll receive an email to confirm the SNS subscription.

---

## 🧠 Optional: Use AWS X-Ray (for full tracing)

Add to `provider` block:

```yaml
provider:
  tracing:
    lambda: true
```

View full invocation traces in **X-Ray console** — useful for performance profiling.

---

## ✅ Best Practices Recap

| Practice                  | Benefit                        |
| ------------------------- | ------------------------------ |
| Structured Logs (JSON)    | Easier to parse/search         |
| CloudWatch Metrics        | Auto-monitor Lambda usage      |
| Alerts (errors, duration) | Notify you of issues instantly |
| SNS Notifications         | Get alerts via email or Slack  |
| X-Ray Tracing             | Analyze latency + cold starts  |

---

### Sample Monitoring Question (Exam Style)

**Q:**
You want to be notified whenever a Lambda function errors more than once within a minute. What's the best way to do this?

A. Write a Lambda to scan logs
B. Use CloudWatch Logs Insights
C. Set a CloudWatch alarm on Errors metric
D. Configure SQS Dead Letter Queue

**Answer:** ✅ C

---

Would you like to:

1. Continue building the **lesson access + progress tracking API**?
2. Or start integrating **Cognito + frontend (React)**?
3. Or export this whole project structure into a repo or ZIP for you?

