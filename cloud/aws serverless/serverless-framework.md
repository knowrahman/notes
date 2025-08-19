Perfect Rahman! Let’s now **learn the Serverless Framework step-by-step with Node.js** — in a hands-on way — and build production-grade APIs using AWS Lambda, API Gateway, and DynamoDB.

We’ll go from **zero to advanced**, including:

---

## 🔥 Full Roadmap: Serverless Framework with Node.js

### ✅ Part 1: Getting Started

* [ ] What is the Serverless Framework?
* [ ] Setup and installation (npm, pnpm, npx)
* [ ] Project structure and `serverless.yml`
* [ ] Writing your first `hello` Lambda
* [ ] Deploying to AWS and invoking via HTTP

---

### ✅ Part 2: HTTP API Routes

* [ ] Handling query strings, path params, and body
* [ ] Logging and environment variables
* [ ] Status codes and CORS
* [ ] Validating inputs (basic + schema-based)

---

### ✅ Part 3: DynamoDB Integration

* [ ] Define a DynamoDB table in `serverless.yml`
* [ ] Use AWS SDK v3 to perform CRUD in Node.js
* [ ] Pagination and filtering
* [ ] IAM permissions for DB access

---

### ✅ Part 4: Authentication & Authorization

* [ ] Add Cognito user pool
* [ ] Secure routes using `authorizer`
* [ ] Extract user info from `event.requestContext`

---

### ✅ Part 5: Advanced Features

* [ ] Organize into services & stages
* [ ] Use layers for reusable code
* [ ] Write integration tests for Lambdas
* [ ] Deploy using CI/CD (GitHub Actions)

---

### ✅ Part 6: Monitoring & Observability

* [ ] CloudWatch Logs
* [ ] AWS X-Ray tracing
* [ ] Alerts on failures and cold starts

---

### ✅ Final Project: Full CRUD Product API

* Node.js + Serverless Framework
* Cognito-protected endpoints
* Full `Create`, `Read`, `Update`, `Delete` on products
* DynamoDB schema + paginated GET
* Deployment ready for resume/portfolio

---
Awesome, Rahman. Let’s dive into the **Serverless Framework with Node.js** roadmap you confirmed — from zero to advanced.

We’ll now begin with:

---

## ✅ Part 1: Getting Started (REST API Version)

---

### 🔹 1. What is the Serverless Framework?

The **Serverless Framework (SLS)** is a CLI and configuration-based tool to:

* Deploy **Lambda functions** easily to AWS
* Wire them to **API Gateway (REST or HTTP)** with just YAML
* Provision **resources** like DynamoDB, S3, Cognito
* Manage your **infrastructure as code** (IaC)

It helps you focus on **code**, not cloud plumbing.

---

### 🔹 2. Setup & Installation

You’ll need:

* Node.js (v18+ is perfect)
* An AWS account
* IAM user with programmatic access and permissions to use Lambda, API Gateway, etc.
* `pnpm` or `npm`

Then in terminal:

```bash
pnpm add -g serverless
# OR use local project setup:
pnpx serverless create --template aws-nodejs
```

If you're having install issues: we’ll use `pnpx` for now.

---

### 🔹 3. Project Structure & `serverless.yml`

Let’s create a REST API skeleton:

```bash
mkdir rest-api && cd rest-api
pnpx serverless create --template aws-nodejs
pnpm init -y
```

**Directory structure:**

```
rest-api/
├── handler.js         # Your Lambda function code
├── serverless.yml     # Your app config
└── package.json
```

---

### 🔹 4. Writing Your First Lambda (REST API)

Update `handler.js`:

```js
module.exports.hello = async (event) => {
  return {
    statusCode: 200,
    body: JSON.stringify({
      message: "Hello from Serverless REST API!",
      input: event,
    }),
  };
};
```

Update `serverless.yml`:

```yaml
service: rest-api

frameworkVersion: "3"

provider:
  name: aws
  runtime: nodejs20.x
  region: ap-southeast-2
  stage: dev
  apiGateway:
    restApi: true  # Force use of REST API instead of HTTP API

functions:
  hello:
    handler: handler.hello
    events:
      - http:
          path: hello
          method: get
          cors: true
```

---

### 🔹 5. Deploy to AWS

```bash
pnpx serverless deploy
```

You’ll see:

```text
endpoints:
  GET - https://abc123.execute-api.ap-southeast-2.amazonaws.com/dev/hello
```

Call it with:

```bash
curl https://abc123.execute-api.ap-southeast-2.amazonaws.com/dev/hello
```

✅ This confirms you deployed a working Lambda behind a REST API Gateway!

---

### ✅ Recap of Part 1

| Step                        | Summary                      |
| --------------------------- | ---------------------------- |
| ✅ Installed SLS with `pnpx` | Avoids global conflicts      |
| ✅ Created `serverless.yml`  | Defined your first route     |
| ✅ Wrote a simple Lambda     | Returns a JSON response      |
| ✅ Deployed to AWS           | Got a real REST API endpoint |

---



Great question, Rahman.

In Node.js, **ES Modules (ECMAScript Modules)** are the standard JavaScript module system — using `import` and `export` instead of `require()` and `module.exports`.

Let me show you a full example, the syntax, and how to use it in a Serverless Framework project.

---

## ✅ ES Module Example in Node.js

### 1. `utils.js`

```js
// Named export
export function sayHello(name) {
  return `Hello, ${name}!`;
}

// Default export
export default function greet() {
  return "Welcome from default export!";
}
```

---

### 2. `main.js`

```js
// Import named export
import { sayHello } from './utils.js';

// Import default export
import greet from './utils.js';

console.log(sayHello("Rahman")); // Hello, Rahman!
console.log(greet());            // Welcome from default export!
```

---

## ✅ Enabling ES Modules in Node.js

To use ES Modules, do **one of the following**:

### Option 1: Add `"type": "module"` to `package.json`

```json
{
  "type": "module"
}
```

Then you can use `import`/`export`.

> Note: File extensions like `.js` must be specified in imports (`import x from './x.js'`).

---

### Option 2: Use `.mjs` file extensions

Instead of `.js`, rename files as `.mjs` and no need for `"type": "module"`.

```bash
mv main.js main.mjs
```

---

## ✅ ES Module Example in a Serverless Lambda

If you're using **ES Modules in Serverless Framework**, you must:

1. Add `"type": "module"` in `package.json`
2. Use `import/export` syntax in your handler files

### `handler.js`

```js
export const hello = async (event) => {
  return {
    statusCode: 200,
    body: JSON.stringify({ message: "Hello from ES module!" }),
  };
};
```

### `serverless.yml`

```yaml
functions:
  hello:
    handler: handler.hello
```

It will correctly find `handler.js` and call the exported `hello` function.

---

## 🧠 Key Differences Between CommonJS and ES Modules

| Feature           | CommonJS         | ES Modules                            |
| ----------------- | ---------------- | ------------------------------------- |
| Import syntax     | `require()`      | `import`                              |
| Export syntax     | `module.exports` | `export`                              |
| Default support   | Built-in         | Must use `"type": "module"` or `.mjs` |
| Tree shaking      | ❌ No             | ✅ Yes (used in bundlers)              |
| Top-level `await` | ❌ No             | ✅ Yes                                 |

---

Great question, Rahman. Let’s break down the **three core parameters** of an AWS Lambda handler in Node.js:

```js
exports.myHandler = async (event, context, callback) => {
  // your logic here
};
```

Even though you often only use `event`, it's important to know what `context` and `callback` do — especially for debugging, performance, or using non-async handlers.

---

## ✅ 1. `event` — the Input Trigger (Always Used)

The `event` object contains **all the information sent to the Lambda** — for example, from:

* API Gateway (query params, headers, body, path)
* DynamoDB Streams (record changes)
* S3 events (bucket name, object key)
* SQS messages

### Example: `event` from API Gateway (REST)

```json
{
  "resource": "/hello",
  "path": "/hello",
  "httpMethod": "GET",
  "headers": { "Content-Type": "application/json" },
  "queryStringParameters": { "name": "Rahman" },
  "body": null,
  "isBase64Encoded": false
}
```

You extract data from it like this:

```js
const name = event.queryStringParameters?.name || "Guest";
```

---

## ✅ 2. `context` — Metadata About the Lambda Execution

`context` contains information about the Lambda itself and the execution environment.

### Key Properties:

| Property                     | Description                      |
| ---------------------------- | -------------------------------- |
| `functionName`               | The name of the function         |
| `awsRequestId`               | Unique request ID for tracing    |
| `memoryLimitInMB`            | Memory assigned to the function  |
| `functionVersion`            | Version of the Lambda            |
| `getRemainingTimeInMillis()` | Time (in ms) left before timeout |

### Example:

```js
console.log(context.functionName);
console.log(context.getRemainingTimeInMillis());
```

This is useful when you want to **log execution time left** or handle retries.

---

## ✅ 3. `callback` — Manual Completion for Non-Async Lambdas

When not using `async`, you **must call** `callback(error, response)` to finish execution.

```js
exports.handler = (event, context, callback) => {
  const response = {
    statusCode: 200,
    body: JSON.stringify({ message: "Hello without async!" }),
  };
  callback(null, response); // No error
};
```

If there's an error:

```js
callback(new Error("Something went wrong"));
```

---

## ✅ Which One Should You Use?

| Use Case            | Recommendation                             |
| ------------------- | ------------------------------------------ |
| Modern Lambda       | Use `async` and `event` only               |
| Advanced metadata   | Use `context` for tracing, logs, or limits |
| Legacy or sync code | Use `callback` style                       |

---

## 🧠 Recap

| Parameter  | Purpose                            | Use When        |
| ---------- | ---------------------------------- | --------------- |
| `event`    | Input data (path, body, trigger)   | ✅ Always        |
| `context`  | Info about the function/execution  | Optional        |
| `callback` | Finish execution in non-async mode | Rarely (legacy) |

---

Excellent question, Rahman. Let’s clarify the difference between an **async Lambda handler** and a **non-async (callback-based) Lambda handler** in Node.js — with side-by-side examples and when to use which.

---

## ✅ Key Difference: Return Style

| Feature            | **Async Lambda** (`async`) | **Non-Async Lambda** (`callback`) |
| ------------------ | -------------------------- | --------------------------------- |
| Return value       | Uses `return` (a Promise)  | Uses `callback(err, result)`      |
| Simpler syntax     | ✅ Yes                      | ❌ More verbose                    |
| Works with `await` | ✅ Yes                      | ❌ No                              |
| Preferred style    | ✅ Yes (modern standard)    | ❌ Legacy support only             |

---

## 🔹 1. Async Lambda (Modern & Preferred)

```js
export const hello = async (event) => {
  return {
    statusCode: 200,
    body: JSON.stringify({ message: "Hello from async Lambda!" }),
  };
};
```

* Automatically returns a Promise
* Can use `await` inside
* Cleaner code and easier error handling

### Example with `await`

```js
export const fetchData = async () => {
  const data = await getFromDynamoDB();
  return {
    statusCode: 200,
    body: JSON.stringify(data),
  };
};
```

---

## 🔹 2. Non-Async Lambda (Callback-based)

```js
export const hello = (event, context, callback) => {
  const response = {
    statusCode: 200,
    body: JSON.stringify({ message: "Hello from callback Lambda!" }),
  };
  callback(null, response); // Pass error as first argument if needed
};
```

* Does not use `await`
* You must **explicitly call** the `callback()` function
* Slightly harder to debug
* Rarely used in modern setups

---

## ✅ Error Handling: Async vs Non-Async

**Async Example (recommended):**

```js
export const handler = async () => {
  try {
    const data = await fetchData();
    return {
      statusCode: 200,
      body: JSON.stringify(data),
    };
  } catch (err) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: err.message }),
    };
  }
};
```

**Non-Async Example:**

```js
export const handler = (event, context, callback) => {
  try {
    const data = fetchDataSync(); // no await
    callback(null, {
      statusCode: 200,
      body: JSON.stringify(data),
    });
  } catch (err) {
    callback(err); // must pass error manually
  }
};
```

---

## ✅ Which One Should You Use?

| If You...                                   | Use                    |
| ------------------------------------------- | ---------------------- |
| Are writing new Lambda functions            | `async`                |
| Need `await` (calls to DB, S3, etc.)        | `async`                |
| Are working in a modern Node.js environment | `async`                |
| Are maintaining very old legacy code        | `callback` (non-async) |

**→ TL;DR:** Always go with `async` unless you have a specific reason to use the `callback` style.

---


Sure, Rahman! Let’s walk through a **full example of a Lambda function using a Lambda Layer** — with files placed in `/opt` — using the **Serverless Framework (SLS)**.

We’ll cover:

1. What Lambda Layers are
2. Folder structure
3. Layer creation and deployment
4. Attaching the layer to a function
5. Accessing `/opt` in your function

---

## ✅ 1. What Is a Lambda Layer?

A **Lambda Layer** lets you include **shared code or binaries** that can be reused across multiple functions.

Layers are **mounted at `/opt`** in the Lambda runtime.

---

## ✅ 2. Folder Structure

Let’s set up a simple project that:

* Uses a shared utility function in a layer.
* Calls that utility in the main Lambda.

```bash
lambda-layer-example/
├── handler.js
├── serverless.yml
└── layer/
    └── my-layer/
        └── nodejs/
            └── utils.js
```

> `nodejs/` is required when creating Node.js layers. AWS expects `node_modules` or `.js` files to be in this directory.

---

## ✅ 3. The Layer Code (`utils.js`)

```js
// layer/my-layer/nodejs/utils.js
module.exports.greet = (name) => {
  return `Hello, ${name}!`;
};
```

---

## ✅ 4. The Lambda Function (`handler.js`)

```js
// handler.js
const { greet } = require('/opt/nodejs/utils');

module.exports.hello = async () => {
  const message = greet("Rahman");

  return {
    statusCode: 200,
    body: JSON.stringify({ message }),
  };
};
```

---

## ✅ 5. `serverless.yml` Configuration

```yaml
service: lambda-layer-demo

frameworkVersion: '3'

provider:
  name: aws
  runtime: nodejs20.x
  region: ap-southeast-2

functions:
  hello:
    handler: handler.hello
    layers:
      - { Ref: MyLambdaLayer }

layers:
  myLayer:
    path: layer/my-layer
    name: my-shared-layer
    compatibleRuntimes:
      - nodejs20.x

resources:
  Resources:
    MyLambdaLayer:
      Type: AWS::Lambda::LayerVersion
      Properties:
        LayerName: my-shared-layer
        CompatibleRuntimes:
          - nodejs20.x
        Content:
          S3Bucket: 
            Ref: ServerlessDeploymentBucket
          S3Key: 
            Fn::Join:
              - '/'
              - - serverless
                - ${sls:stage}
                - ${sls:service}
                - layers
                - myLayer.zip
```

> ✅ Note: Serverless Framework will **automatically upload the layer to S3** and reference it properly.

---

## ✅ 6. Deploy

```bash
pnpx serverless deploy
```

Your Lambda will now have access to the code in `/opt/nodejs/utils.js`.

---

## 🧠 Recap

| Concept      | Purpose                                        |
| ------------ | ---------------------------------------------- |
| Lambda Layer | Shared code, dependencies, or binaries         |
| `/opt`       | Mount point inside Lambda to access layer code |
| `layers:`    | Used in `serverless.yml` to attach layers      |

---

## ✅ Sample Output (Function Response)

```json
{
  "message": "Hello, Rahman!"
}
```
Great follow-up, Rahman.

Let me now walk you through **how a Lambda Layer is deployed** step-by-step using the **Serverless Framework**, and what happens behind the scenes.

---

## ✅ How a Lambda Layer Is Deployed

When you define a layer in `serverless.yml`, the Serverless Framework does the following during `deploy`:

---

### 🔹 Step-by-Step Deployment Flow

```bash
pnpx serverless deploy
```

Behind the scenes, this is what happens:

---

### ✅ Step 1: **Zip the Layer Code**

Serverless zips your layer content at:

```
layer/my-layer/
└── nodejs/
    └── utils.js
```

It creates: `myLayer.zip` containing:

```
nodejs/utils.js
```

This ensures AWS will mount it at `/opt/nodejs/utils.js`.

---

### ✅ Step 2: **Upload to S3**

Serverless uploads the zipped layer to the **Serverless-managed S3 bucket**, under a path like:

```
serverless/dev/lambda-layer-demo/layers/myLayer.zip
```

---

### ✅ Step 3: **Create the Layer in CloudFormation**

The following resource is created (as defined in `resources:` or automatically):

```yaml
Type: AWS::Lambda::LayerVersion
Properties:
  LayerName: my-shared-layer
  CompatibleRuntimes:
    - nodejs20.x
  Content:
    S3Bucket: <auto-generated>
    S3Key: <path to zip>
```

This creates the Layer in your AWS account and **assigns it a Layer Version ARN** like:

```text
arn:aws:lambda:ap-southeast-2:123456789012:layer:my-shared-layer:1
```

---

### ✅ Step 4: **Attach Layer to Lambda Function**

In your `functions.hello.layers` config:

```yaml
functions:
  hello:
    handler: handler.hello
    layers:
      - { Ref: MyLambdaLayer }
```

This adds the layer’s ARN to the Lambda’s configuration. Now, AWS automatically **mounts your layer at `/opt`** inside the Lambda runtime.

---

### ✅ Step 5: Deploy Logs

Finally, you’ll see this in the console:

```
✔ Service deployed to stage dev (region: ap-southeast-2)

functions:
  hello: lambda-layer-demo-dev-hello
layers:
  myLayer: arn:aws:lambda:ap-southeast-2:123456789012:layer:my-shared-layer:1
```

You can now **invoke your function** and it will load code from `/opt/nodejs/`.

---

## 📁 In Summary:

| Stage               | Action                               |
| ------------------- | ------------------------------------ |
| `serverless deploy` | Zips layer + uploads to S3           |
| CloudFormation      | Creates `LayerVersion` resource      |
| Lambda Function     | Is configured to include that Layer  |
| Runtime             | `/opt` is available during execution |

---

Would you like to test this now by adding **`node-fetch` as a dependency in the layer**, so you can make HTTP requests inside multiple Lambdas without reinstalling the package each time?




Great idea, Rahman! Let's build a **secure AWS Lambda function** with:

* **API Gateway REST API**
* A **Lambda authorizer (formerly called custom authorizer)**
* A protected **GET route**
* The authorizer checks if the incoming request has a valid token and authorizes it.

---

## ✅ Goal

We’ll create:

1. A **Lambda Authorizer** that checks an `Authorization` header (e.g., token equals `'allow-token'`)
2. A **Lambda function** that’s only called if the token is valid
3. Full setup in **Serverless Framework** with step-by-step explanation

---

## ✅ Project Structure

```bash
lambda-auth-project/
├── authorizer.js       # Lambda authorizer
├── handler.js          # Protected business logic
└── serverless.yml      # Serverless config
```

---

## ✅ Step 1: `authorizer.js` – Custom Lambda Authorizer

```js
// authorizer.js
module.exports.auth = async (event) => {
  const token = event.authorizationToken;

  // Simulate token verification
  if (token === "allow-token") {
    return {
      principalId: "user123",
      policyDocument: {
        Version: "2012-10-17",
        Statement: [
          {
            Action: "execute-api:Invoke",
            Effect: "Allow",
            Resource: event.methodArn,
          },
        ],
      },
      context: {
        userId: "user123",
      },
    };
  }

  throw new Error("Unauthorized");
};
```

---

## ✅ Step 2: `handler.js` – Protected Lambda

```js
// handler.js
module.exports.hello = async (event) => {
  return {
    statusCode: 200,
    body: JSON.stringify({
      message: "Authorized access!",
      userId: event.requestContext.authorizer.userId,
    }),
  };
};
```

---

## ✅ Step 3: `serverless.yml`

```yaml
service: lambda-auth-demo

frameworkVersion: "3"

provider:
  name: aws
  runtime: nodejs20.x
  region: ap-southeast-2

functions:
  authorizer:
    handler: authorizer.auth

  hello:
    handler: handler.hello
    events:
      - http:
          path: hello
          method: get
          cors: true
          authorizer:
            name: authorizer
            type: TOKEN
```

---

## ✅ Step 4: Deploy

```bash
pnpx serverless deploy
```

---

## ✅ Step 5: Test with `curl`

### 🔹 ✅ Authorized:

```bash
curl -H "Authorization: allow-token" https://<your-api-id>.execute-api.ap-southeast-2.amazonaws.com/dev/hello
```

Output:

```json
{
  "message": "Authorized access!",
  "userId": "user123"
}
```

---

### 🔹 ❌ Unauthorized:

```bash
curl -H "Authorization: deny-token" https://<your-api-id>.execute-api.ap-southeast-2.amazonaws.com/dev/hello
```

Output:

```json
{"message": "Unauthorized"}
```

---

## ✅ Recap

| Component              | Purpose                                      |
| ---------------------- | -------------------------------------------- |
| Lambda Authorizer      | Authenticates and returns IAM policy         |
| API Gateway            | Triggers authorizer first before main Lambda |
| Main Lambda            | Runs only if authorized                      |
| `Authorization` header | Contains the token (e.g., `allow-token`)     |

---

Perfect, Rahman! Let’s walk through **how to deploy a Lambda function using a Docker container image** with **Amazon ECR** and the **Serverless Framework** step-by-step.

---

## ✅ Goal

We’ll build and deploy a simple Lambda function using:

* A **custom Dockerfile**
* **Amazon ECR** for storing the image
* **Serverless Framework** for deployment

---

## ✅ Step-by-Step Guide

### 🔹 1. Prerequisites

Make sure you have installed:

* Docker
* Serverless Framework (v3+)
* AWS CLI (configured)
* `pnpm` or `npm` (your choice)

---

### 🔹 2. Create Project Folder

```bash
mkdir lambda-container-example && cd lambda-container-example
```

---

### 🔹 3. Create a Dockerfile

```Dockerfile
# Dockerfile
FROM public.ecr.aws/lambda/nodejs20.x

# Copy code
COPY app.js ./

# Set the CMD to your handler function
CMD [ "app.handler" ]
```

---

### 🔹 4. Write the Lambda Code (`app.js`)

```js
// app.js
exports.handler = async (event) => {
  return {
    statusCode: 200,
    body: JSON.stringify({ message: "Hello from Docker Lambda!" }),
  };
};
```

---

### 🔹 5. Create `serverless.yml`

```yaml
service: docker-lambda-demo

frameworkVersion: '3'

provider:
  name: aws
  ecr:
    images:
      myfunction:
        path: .    # where the Dockerfile is

functions:
  hello:
    image:
      name: myfunction
```

---

### 🔹 6. Deploy!

```bash
pnpx serverless deploy
```

Serverless Framework will:

* Build the Docker image using your `Dockerfile`
* Push it to an auto-generated **ECR repository**
* Create the Lambda using the image
* Output the function’s ARN and status

---

### 🔹 7. Test It

```bash
pnpx serverless invoke -f hello
```

Output:

```json
{
  "statusCode": 200,
  "body": "{\"message\":\"Hello from Docker Lambda!\"}"
}
```

---

## 🧠 Pro Tip

You can now:

* Add native binaries or modules in the Dockerfile (e.g. Puppeteer, FFmpeg)
* Use `.env` files for secrets
* Add custom layers or AWS SDKs
* Standardize the image across Lambda and ECS

---


In the **Serverless Framework**, the term **"package"** refers to the **deployment artifact** that contains everything your service needs to deploy and run on AWS (or another cloud provider). This includes your:

* Lambda function code
* Dependencies (e.g., `node_modules`, Python packages)
* `serverless.yml` configuration
* Layers (if defined)
* Any custom files or folders you've included manually

---

## Why is Packaging Important?

When you deploy your Serverless application using:

```bash
serverless deploy
```

The framework **packages** your service into a **zip file** and uploads it to AWS so that it can be used by your Lambda functions.

---

## Key Concepts of `package` in Serverless Framework

### 1. **Default Packaging**

By default, Serverless:

* Zips **each Lambda function** and its dependencies separately (if you're using `package.individually: true`)
* Or, zips **all functions together** into a single artifact (`package.individually: false`)

The zip file(s) is/are:

* Stored in `.serverless/` directory
* Uploaded to S3 behind the scenes during deployment

---

### 2. **Customize Packaging with `package` block in `serverless.yml`**

You can configure the package behavior using the `package:` property in your `serverless.yml`.

#### a. **Exclude files/folders** from the package:

```yaml
package:
  exclude:
    - node_modules/**
    - test/**
    - README.md
```

#### b. **Include specific files/folders:**

```yaml
package:
  include:
    - src/**
    - config.json
```

#### c. **Use `package.individually` for function-specific packages:**

```yaml
package:
  individually: true
```

Each function is packaged and deployed **independently**, which:

* Speeds up deployment of unchanged functions
* Reduces deployment size

---

### 3. **Pre-Built Artifacts**

You can also use a **pre-packaged artifact**:

```yaml
functions:
  hello:
    handler: handler.hello
    package:
      artifact: path/to/hello.zip
```

This is helpful when:

* You build your code externally (e.g., CI/CD pipeline)
* You want **complete control** over what's zipped and deployed

---

## Summary

| Term                          | Meaning in Serverless Framework                                            |
| ----------------------------- | -------------------------------------------------------------------------- |
| `package`                     | The deployment bundle for your service or function                         |
| `.serverless/`                | Local folder where Serverless stores zipped artifacts before deployment    |
| `package.individually`        | Package each Lambda function separately (vs. one package for all)          |
| `package.include` / `exclude` | Fine-tune which files are added or ignored in the deployment package       |
| `package.artifact`            | Use your own zip file instead of letting Serverless build it automatically |

---

## Exam Power-Up 💡

* In **Serverless Framework**, a **package** is the zipped code + dependencies deployed to AWS
* Use `package.individually: true` to speed up large multi-function deployments
* Exclude unnecessary files (e.g., tests, docs) to reduce deployment size
* Use `artifact` to supply your own zip (e.g., from a CI/CD build step)

---

## Would you like a working example of a `serverless.yml` that uses custom packaging settings?

Here’s a **fully working `serverless.yml` example** that shows how to use the `package` configuration in the **Serverless Framework** — including:

* Multiple functions
* Packaging them **individually**
* Custom includes/excludes
* Pre-built artifact usage

---

### Example Directory Structure:

```
/my-service
  ├── handler.js
  ├── emailHandler.js
  ├── orderHandler.js
  ├── config/
  │     └── settings.json
  ├── utils/
  │     └── helper.js
  ├── test/
  ├── README.md
  ├── serverless.yml
```

---

### `serverless.yml`

```yaml
service: order-service

provider:
  name: aws
  runtime: nodejs18.x
  region: ap-southeast-2

package:
  individually: true       # Package each function separately
  exclude:
    - test/**
    - README.md
    - "*.log"
  include:
    - config/settings.json # Include config file globally

functions:
  createOrder:
    handler: orderHandler.create
    package:
      include:
        - utils/helper.js  # Only include this helper for this function

  sendConfirmationEmail:
    handler: emailHandler.send
    package:
      artifact: artifacts/email-handler.zip  # Use a prebuilt package

  getStatus:
    handler: handler.getStatus
    package:
      exclude:
        - utils/**         # Exclude utils for this function only
```

---

### What This Configuration Does:

| Function                | Source File       | Packaging Behavior                                                            |
| ----------------------- | ----------------- | ----------------------------------------------------------------------------- |
| `createOrder`           | `orderHandler.js` | Includes `utils/helper.js`, `config/settings.json`, excludes tests and README |
| `sendConfirmationEmail` | `emailHandler.js` | **Uses a pre-built zip** file located at `artifacts/email-handler.zip`        |
| `getStatus`             | `handler.js`      | Excludes the `utils/` folder entirely                                         |

---

### Key Notes:

* `individually: true` ensures each function is deployed separately. This:

  * **Reduces deployment size**
  * Speeds up updates (if only one function changes)
* You can **override global includes/excludes per function**
* Using `artifact` is helpful when:

  * You're compiling TypeScript or bundling with Webpack
  * You're building with GitHub Actions or CodeBuild

---

## Would you like a TypeScript version of this example or how to use Webpack to optimize your packaging as well?

Here’s a **TypeScript version** of a Serverless Framework project, showing how to configure **custom packaging** using `serverless.yml`. This includes:

* TypeScript handlers
* `serverless-esbuild` plugin for bundling
* Custom includes and excludes
* `individually: true` packaging

---

## 📁 Example Project Structure

```
/order-service
├── src/
│   ├── handlers/
│   │   ├── createOrder.ts
│   │   ├── getStatus.ts
│   │   └── emailHandler.ts
│   ├── utils/
│   │   └── logger.ts
│   └── config/
│       └── settings.json
├── test/
├── package.json
├── tsconfig.json
├── serverless.yml
```

---

## 📦 Install Required Packages

Run:

```bash
npm install --save-dev serverless-esbuild typescript
```

---

## 🧠 `serverless.yml`

```yaml
service: order-service

frameworkVersion: '3'

provider:
  name: aws
  runtime: nodejs18.x
  region: ap-southeast-2

plugins:
  - serverless-esbuild

package:
  individually: true
  exclude:
    - test/**
    - "*.md"

functions:
  createOrder:
    handler: src/handlers/createOrder.handler
    package:
      include:
        - src/utils/logger.ts
        - src/config/settings.json

  getStatus:
    handler: src/handlers/getStatus.handler
    package:
      exclude:
        - src/utils/**

  sendConfirmationEmail:
    handler: src/handlers/emailHandler.handler
    package:
      artifact: artifacts/email-handler.zip # Pre-zipped artifact from CI/CD
```

---

## ⚙️ `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "CommonJS",
    "lib": ["ES2020"],
    "rootDir": "src",
    "outDir": "dist",
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*.ts"]
}
```

---

## 📦 `package.json`

```json
{
  "name": "order-service",
  "scripts": {
    "build": "tsc",
    "deploy": "serverless deploy"
  },
  "devDependencies": {
    "serverless-esbuild": "^1.43.0",
    "typescript": "^5.3.3"
  }
}
```

---

## 🧪 Optional: Example Handler

```ts
// src/handlers/createOrder.ts
import { APIGatewayProxyHandler } from 'aws-lambda';

export const handler: APIGatewayProxyHandler = async (event) => {
  console.log("Creating order...");
  return {
    statusCode: 200,
    body: JSON.stringify({ message: "Order created!" })
  };
};
```

---

## Summary Power-Up 💡

* Use **serverless-esbuild** to bundle TypeScript automatically (no need to run `tsc` manually).
* `package.individually: true` helps reduce deployment size and speeds up CI/CD.
* Include or exclude specific files for each function using the `package` block.
* Use pre-built `.zip` files with `artifact` for production deployments via CI/CD pipelines.

---
Great question! Understanding the differences between **environment variables**, **custom variables**, and **parameters** in the **Serverless Framework** is key for building flexible, secure, and maintainable applications.

Let’s break them down clearly.

---

## 1. **Environment Variables (`environment`)**

Environment variables are:

* Injected into the **Lambda function's runtime environment**
* Accessible **in your code** using `process.env` (Node.js)

### Where to define:

```yaml
functions:
  hello:
    handler: handler.hello
    environment:
      TABLE_NAME: my-dynamo-table
      LOG_LEVEL: debug
```

Now inside your code:

```js
const table = process.env.TABLE_NAME;
const logLevel = process.env.LOG_LEVEL;
```

### Scope:

* Can be defined at **provider level** (global for all functions)
* Or at **function level** (overrides or adds to global)

---

## 2. **Custom Variables (`custom`)**

Custom variables are:

* Defined in the `custom:` block of `serverless.yml`
* **Not exposed** to your Lambda function
* Used **internally by Serverless Framework** for templating and reuse

### Where to define:

```yaml
custom:
  tableName: my-dynamo-table
  region: us-east-1
```

### Where to use:

```yaml
provider:
  environment:
    TABLE_NAME: ${self:custom.tableName}
    REGION: ${self:custom.region}
```

So you're **not duplicating strings** everywhere — and can change `tableName` in one place.

---

## 3. **Parameters (`--param`) \[Framework v3+]**

Parameters are:

* Values passed **from the CLI** or `.env` files at **deploy time**
* Used for **secure, dynamic values** like secrets, DB URLs, etc.
* Available in `serverless.yml` via `${param:...}`

### Example:

```bash
serverless deploy --param="stage=prod" --param="dbUrl=postgres://..."
```

In `serverless.yml`:

```yaml
provider:
  environment:
    DB_URL: ${param:dbUrl}
```

### Can also use `.env`:

```
PARAM_DB_URL=postgres://user:pass@host
```

And reference with:

```yaml
provider:
  environment:
    DB_URL: ${param:DB_URL}
```

---

## 🔍 Comparison Table

| Feature                  | Description                              | Scope                | Accessible in Lambda?        | Use Case                                  |
| ------------------------ | ---------------------------------------- | -------------------- | ---------------------------- | ----------------------------------------- |
| **Environment Variable** | Injected into function runtime           | provider or function | ✅ Yes                        | Accessing values like table names, tokens |
| **Custom Variable**      | Internal config for Serverless Framework | Entire file          | ❌ No                         | Templating, centralizing strings          |
| **Parameter**            | Runtime values passed via CLI/env        | Deploy-time          | ✅ If mapped to `environment` | Secrets, stage-specific values            |

---

## Example All-in-One

```yaml
service: my-service

custom:
  stage: ${opt:stage, 'dev'}
  tableName: user-table-${self:custom.stage}

provider:
  name: aws
  runtime: nodejs18.x
  environment:
    TABLE_NAME: ${self:custom.tableName}
    SECRET_TOKEN: ${param:secretToken}
    REGION: ${opt:region, 'us-east-1'}

functions:
  getUser:
    handler: src/handler.getUser
```

---

### Then deploy like:

```bash
serverless deploy --stage prod --param="secretToken=mySecret123"
```

---

## Summary Power-Up 💡

* **environment** → exposed in code as `process.env.X`
* **custom** → reusable variables within `serverless.yml`
* **param** → passed at deploy time, used for secrets/config, **not stored**

---
