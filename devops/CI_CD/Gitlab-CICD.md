
# 1. What is GitLab?

GitLab is a web-based DevOps lifecycle tool. It provides:
- Source code management (like GitHub, Bitbucket)
- Continuous Integration and Continuous Deployment (CI/CD)
- Issue tracking
- Code review
- Security scans
- DevOps Automation (Infrastructure as Code, Kubernetes management)

The special thing about GitLab is **everything is built-in** — you don’t need to install extra CI/CD tools separately.

---

# 2. What is GitLab CI/CD?

**CI/CD** stands for:
- **Continuous Integration (CI)**: Automatically building and testing code every time someone pushes to GitLab.
- **Continuous Deployment (CD)**: Automatically deploying the new version when it passes tests.

In GitLab, you define your CI/CD steps in a file called **`.gitlab-ci.yml`** placed at the root of your project.

Each time you push code, GitLab reads this file and runs the steps (called **Jobs**) using **Runners**.

---

# 3. What is a GitLab Runner?

This is a **very important concept**.

A **GitLab Runner** is a lightweight, agent (program) installed on a server, VM, or even your own machine, that **executes the CI/CD jobs** you define.

- It listens to GitLab for jobs.
- When a job is available (like build, test, deploy), the runner picks it up and runs it.

Types of Runners:
1. **Shared Runners**: Provided by GitLab itself, shared between many users/projects.
2. **Specific Runners**: Set up by you (or your company) and assigned to specific projects.
3. **Group Runners**: Available to every project inside a GitLab group.

**Executors**: A runner needs an *executor* — this is how it actually runs the job. Executors can be:
- `docker` (inside Docker containers) — Most common
- `shell` (directly on machine's OS)
- `virtualbox`
- `kubernetes`
- `ssh`

**Example**: If you choose a `docker` executor, the runner will spin up a Docker container to run your job.

---

# 4. Important Components

**Job**:  
One unit of work (like install packages, run tests, build docker image).

```yaml
build_job:
  stage: build
  script:
    - npm install
    - npm run build
```

**Stages**:  
Jobs are grouped into stages. E.g., `build`, `test`, `deploy`. Jobs in one stage must complete before moving to the next stage.

```yaml
stages:
  - build
  - test
  - deploy
```

**Pipeline**:  
A collection of stages and jobs defined in `.gitlab-ci.yml`.

**Artifacts**:  
Files generated during jobs that you can pass between stages (example: built files, binaries).

```yaml
artifacts:
  paths:
    - dist/
```

**Cache**:  
Store dependencies like `node_modules`, so jobs run faster next time.

```yaml
cache:
  paths:
    - node_modules/
```

**Environment Variables**:  
Use variables to keep secrets or dynamic values.

Example:
```yaml
variables:
  NODE_ENV: production
```
Or define them securely in GitLab's **Settings > CI/CD > Variables** section.

---

# 5. How GitLab CI/CD Works Step-by-Step

1. You push code to GitLab.
2. GitLab detects `.gitlab-ci.yml` in the repo.
3. GitLab creates a **Pipeline** and splits it into **Stages** and **Jobs**.
4. A **Runner** picks up a job:
   - Runs the script.
   - Saves artifacts and caches.
   - Moves to the next job.
5. If all jobs succeed, the pipeline passes.
6. Optional: A job can deploy your application automatically.

---

# 6. How to Install Your Own Runner (Self-Hosted)

You can install a Runner if you don't want to use GitLab shared runners (recommended for private projects or heavy builds).

Basic steps:
1. Install Docker (optional but recommended).
2. Install GitLab Runner:
   ```bash
   sudo apt install gitlab-runner
   ```
3. Register the Runner:
   ```bash
   sudo gitlab-runner register
   ```
   - Enter your GitLab project URL.
   - Enter your GitLab runner token (from Settings > CI/CD > Runners).
   - Choose an executor (e.g., docker).
4. Start the Runner:
   ```bash
   sudo gitlab-runner start
   ```

Now, your project will use **your runner** instead of GitLab's shared runner.

---

# 7. Example Real-World GitLab CI/CD with Docker for Node.js

Suppose you are building a Node.js API.

**Files:**
- `Dockerfile` - How to build the app.
- `.gitlab-ci.yml` - How to automate everything.

**Dockerfile**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

**.gitlab-ci.yml**
```yaml
image: docker:latest

services:
  - docker:dind

variables:
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

stages:
  - build
  - deploy

build_image:
  stage: build
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG

deploy:
  stage: deploy
  script:
    - ssh $SSH_USER@$SSH_HOST "docker pull $IMAGE_TAG && docker stop my-app || true && docker rm my-app || true && docker run -d -p 3000:3000 --name my-app $IMAGE_TAG"
```

---

# 8. Key GitLab CI/CD Features You Should Know

- **Manual Jobs**: Trigger certain jobs manually.
- **Scheduled Pipelines**: Run pipelines daily, weekly automatically.
- **Multi-Project Pipelines**: Chain pipelines between projects.
- **Parent-Child Pipelines**: Break huge pipelines into small independent pipelines.
- **Security Scanning**: GitLab can scan code for vulnerabilities automatically.
- **Review Apps**: Temporary apps that spin up for every Merge Request for preview.
- **Dynamic Environments**: Create an environment dynamically during a pipeline.
- **Auto DevOps**: GitLab's default way to automate full build-test-deploy (good for basic apps).

---

# 9. Common Mistakes Beginners Make

- **Forgetting to register their runners** properly.
- **Wrong indentation** in `.gitlab-ci.yml`. (YAML files are super sensitive to spaces.)
- **Not caching node_modules** leading to slow builds.
- **Not using artifacts** when needed.
- **Mixing up variables** between `.gitlab-ci.yml` and GitLab Settings.

---

# 10. Real World Example How Companies Use GitLab CI/CD

- **GitLab's own website** is built with GitLab CI/CD.
- **NASA**, **Siemens**, **Sony**, **Ticketmaster**, etc., use GitLab to automate their deployments.
- In startups: 
  - Developers push code.
  - Pipelines build images.
  - Images get deployed to Kubernetes clusters.
  - All using GitLab with minimal human effort.

---

# Summary

You should now know:
- What GitLab is.
- What GitLab CI/CD means.
- What a Runner is (and types of runners).
- How a `.gitlab-ci.yml` file controls your pipeline.
- How jobs, stages, artifacts, cache, variables work.
- How to write a real CI/CD pipeline for a Node.js Docker project.
- Extra GitLab features that companies use.

---

# Exam Power-up

**Important Keywords to Remember:**
- Runner
- Executor (Docker/Shell/Kubernetes)
- Jobs
- Stages
- Pipelines
- Artifacts
- Cache
- GitLab CI/CD variables

---

let’s go very deep into how the `.gitlab-ci.yml` file works, **line-by-line**, **concept-by-concept**, so you really master it.

This is the single most important file for controlling **CI/CD in GitLab**.

---

# What is `.gitlab-ci.yml`?

- It is a **YAML** file (`.yml`) placed at the **root** of your repository.
- It defines **how your project will be built, tested, and deployed**.
- GitLab reads it **every time you push your code**.
- It tells GitLab **what to do, when to do it, how to do it, and under what conditions**.

---

# Basic Structure of `.gitlab-ci.yml`

At the highest level, you define:

1. **Global settings** (optional) - for the whole pipeline.
2. **Stages** - big groups of jobs (like build, test, deploy).
3. **Jobs** - tasks inside stages (install dependencies, run tests, etc).
4. **Variables** - reusable data for jobs.
5. **Services** - extra containers you need (like a database).
6. **Artifacts** and **Caches** - save and reuse files across jobs.
7. **Rules/only/except** - control when jobs run.
8. **Before_script / After_script** - run extra commands before or after all jobs.

---

# Deep Dive into Sections

## 1. `image`

This defines the Docker image your job runs inside.

```yaml
image: node:18
```
- All commands in jobs will run inside a container based on `node:18`.
- Useful for consistent environments (same Node.js version everywhere).

**Real world**: Always pick an official Docker image matching your app (node, python, golang, alpine, etc.)

---

## 2. `services`

Services are **side containers** that jobs can use.

Example: If your tests need a database:

```yaml
services:
  - postgres:latest
```
Now during your test job, Postgres will be running automatically.

---

## 3. `stages`

Define the **order** of big blocks (build → test → deploy).

```yaml
stages:
  - build
  - test
  - deploy
```
GitLab will:
- Run all `build` jobs first.
- Then run all `test` jobs.
- Then all `deploy` jobs.

**Rule**: Jobs of the same stage run in parallel. Different stages run sequentially.

---

## 4. `variables`

Define environment variables used across jobs.

```yaml
variables:
  NODE_ENV: production
  DOCKER_DRIVER: overlay2
```
You can also define them in GitLab UI under Project → Settings → CI/CD → Variables (safer for secrets).

---

## 5. `before_script` and `after_script`

Commands to run **before or after** each job automatically.

```yaml
before_script:
  - npm ci

after_script:
  - echo "Cleaning up..."
```
**Example**: Install dependencies once before running every job.

---

## 6. `job` Definitions

A **job** has:
- A name (any valid string).
- A stage.
- A script (list of commands).

Example:
```yaml
install_dependencies:
  stage: build
  script:
    - npm install
```

**More options inside a job**:
- `script`: main commands.
- `only` / `except` / `rules`: when to run.
- `tags`: specify a runner.
- `artifacts`: upload files to be used later.
- `cache`: reuse files like node_modules.
- `dependencies`: define job dependencies.
- `retry`: retry if it fails.
- `timeout`: max job run time.

---

## 7. `script`

The actual commands the runner will execute inside the job.

```yaml
script:
  - echo "Hello World"
  - npm install
  - npm test
```
Think of it like a mini shell script inside your pipeline.

---

## 8. `artifacts`

Files or folders you **save after a job**, and download in the next stages.

```yaml
artifacts:
  paths:
    - dist/
```
Example: Save your `build` folder after building, so `deploy` stage can use it.

---

## 9. `cache`

Caches files or folders between **different pipelines** or **different jobs**.

```yaml
cache:
  paths:
    - node_modules/
```
Example: Save `node_modules/` so you don’t have to run `npm install` every time.

---

## 10. `only`, `except`, and `rules`

Control **when** jobs run.

Old way (`only/except`):
```yaml
only:
  - main
except:
  - dev
```
Meaning: Run only when pushing to `main`, NOT `dev`.

Modern way (`rules`):
```yaml
rules:
  - if: '$CI_COMMIT_BRANCH == "main"'
    when: always
```
More flexible. You can define multiple conditions.

---

# Full Example Explained

```yaml
image: node:18

services:
  - postgres:latest

variables:
  DB_HOST: postgres
  DB_PORT: 5432
  DB_USER: postgres
  DB_PASSWORD: password

stages:
  - build
  - test
  - deploy

before_script:
  - npm ci

build_app:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/

test_app:
  stage: test
  script:
    - npm run test

deploy_app:
  stage: deploy
  script:
    - echo "Deploying to Production Server"
    - scp -r dist/ user@server:/var/www/html
  only:
    - main
```

**Explanation:**
- Runs in a `node:18` docker container.
- Also spins up a Postgres database.
- Defines 3 stages: build → test → deploy.
- Installs npm packages before every job.
- Builds the app and saves the `dist/` folder.
- Runs tests.
- Deploys only if pushing to `main` branch.

---

# Visual Representation

```
Push code →
  GitLab sees .gitlab-ci.yml →
    Runner spins up docker container →
      Stage 1: Build → npm ci, npm build → Save dist/
      Stage 2: Test  → npm test
      Stage 3: Deploy → Copy dist/ to server
```

---

# Important GitLab CI/CD Environment Variables

Some predefined variables you automatically get:
- `CI_COMMIT_BRANCH`: Current branch name.
- `CI_PIPELINE_ID`: Unique pipeline ID.
- `CI_COMMIT_SHA`: Commit ID.
- `CI_REGISTRY`: GitLab Container Registry URL.
- `CI_PROJECT_DIR`: Full path where code is cloned.
- `CI_JOB_STAGE`: Current stage name.

You can use them anywhere in your YAML!

Example:
```yaml
script:
  - echo "Deploying branch $CI_COMMIT_BRANCH"
```

---

# Tips for Writing `.gitlab-ci.yml`

- Indentation is very important! Always use **2 spaces**.
- Validate YAML file using GitLab CI Lint tool (in GitLab UI).
- Keep pipelines fast by caching dependencies.
- Secure secrets in GitLab Variables, never hardcode them.
- Use `rules` instead of old `only/except` for more control.

---

# Summary

You should now understand:
- What `.gitlab-ci.yml` is
- What parts it has: image, services, stages, variables, jobs
- How jobs and stages work
- What artifacts and cache mean
- How to use rules, scripts, and environment variables
- Best practices for writing efficient GitLab pipelines

---


I'll create a **full professional GitLab `.gitlab-ci.yml` pipeline** for you for a **Node.js** application, where:

- We **test** the app
- We **build** a Docker image
- We **login to Docker Hub (Registry)**
- We **push the image** to Docker Hub
- We **SSH into a DigitalOcean server** and **deploy** the new image
- I'll explain **every line and every concept** in detail.

Let's go step by step.

---

# Assumptions:
- Node.js app (like Express.js) lives inside a GitLab repo.
- You have a **DigitalOcean droplet** (server) running (Ubuntu ideally).
- You have access to **Docker Hub** account (or GitLab registry — we will use Docker Hub for now).
- You have **SSH key** access to your server.
- Your server already has **Docker** installed.
- Your app listens on **port 3000** (you can change it easily).

---

# Important Environment Variables (to setup in GitLab)

In GitLab > Project > Settings > CI/CD > Variables, add these:

| Key | Value | Purpose |
|:----|:------|:--------|
| DOCKER_USER | your-dockerhub-username | Login to Docker Hub |
| DOCKER_PASSWORD | your-dockerhub-password | Login to Docker Hub |
| SSH_PRIVATE_KEY | your private SSH key | For ssh into DigitalOcean server |
| SSH_SERVER_IP | your DigitalOcean server IP address | |
| SSH_USER | usually `root` or another user with SSH access | |
| DOCKER_IMAGE_NAME | your-dockerhub-username/your-repo-name | Full docker image name |

---

# GitLab `.gitlab-ci.yml`

Here is the full file:

```yaml
image: docker:20.10.16

services:
  - docker:dind

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: ""  # disable TLS to make docker:dind simpler
  IMAGE_TAG: $DOCKER_IMAGE_NAME:$CI_COMMIT_SHORT_SHA

stages:
  - test
  - build
  - push
  - deploy

before_script:
  - echo "Setting up environment..."

test:
  stage: test
  image: node:18
  script:
    - echo "Running tests..."
    - npm install
    - npm test

build:
  stage: build
  script:
    - echo "Building docker image..."
    - docker login -u "$DOCKER_USER" -p "$DOCKER_PASSWORD"
    - docker build -t "$IMAGE_TAG" .

push:
  stage: push
  script:
    - echo "Pushing docker image to Docker Hub..."
    - docker push "$IMAGE_TAG"

deploy:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache openssh
    - mkdir -p ~/.ssh
    - echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
    - chmod 600 ~/.ssh/id_rsa
    - ssh-keyscan -H "$SSH_SERVER_IP" >> ~/.ssh/known_hosts
  script:
    - echo "Deploying on server..."
    - ssh $SSH_USER@$SSH_SERVER_IP "
        docker login -u '$DOCKER_USER' -p '$DOCKER_PASSWORD' && \
        docker pull '$IMAGE_TAG' && \
        docker stop my-node-app || true && \
        docker rm my-node-app || true && \
        docker run -d --name my-node-app -p 3000:3000 '$IMAGE_TAG'
      "
  only:
    - main
```

---

# Full Explanation Line by Line

### `image: docker:20.10.16`
- The base docker image we use to run our CI jobs.
- Includes `docker` CLI.

### `services: docker:dind`
- **docker:dind** means "**Docker-in-Docker**".
- It runs a **Docker daemon** inside the container.
- This is required because **building/pushing images** needs Docker engine access.
- **dind** = Docker **D**aemon **in**side Docker.

**In simple words**:  
The GitLab job is inside a container, but it needs Docker installed to build other containers. So `docker:dind` provides that service.

---

### `variables`
- `DOCKER_DRIVER: overlay2`: Best Docker storage driver (required by docker:dind).
- `DOCKER_TLS_CERTDIR: ""`: Disable TLS, making it simpler to connect inside CI.
- `IMAGE_TAG`: Defines the image name **with commit ID**. Example: `username/myapp:abcdef1`.

---

### `stages`
Defines the **order**:
1. `test`
2. `build`
3. `push`
4. `deploy`

---

### `before_script`
Common script that runs **before each job**.

---

## Job: `test`
```yaml
test:
  stage: test
  image: node:18
  script:
    - npm install
    - npm test
```
- Runs tests using a Node.js environment.
- It **does not need Docker**.
- If tests fail, pipeline stops here.

---

## Job: `build`
```yaml
build:
  stage: build
  script:
    - docker login
    - docker build
```
- Logs into Docker Hub.
- Builds the Docker image.
- Tags it with `IMAGE_TAG` (commit-specific tag).

---

## Job: `push`
```yaml
push:
  stage: push
  script:
    - docker push
```
- Pushes the built image to Docker Hub.
- Now your image is available online.

---

## Job: `deploy`
```yaml
deploy:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add openssh
    - Setup SSH key
    - Setup known_hosts
  script:
    - ssh into server
    - login to Docker
    - pull latest image
    - stop old container
    - remove old container
    - run new container
```
- This job uses **Alpine Linux** (lightweight) to SSH into your server.
- Installs `openssh`.
- Adds private SSH key.
- Connects securely without password.
- Pulls the latest Docker image.
- Stops and removes the old running container.
- Runs the new container with `-d` (detached mode) and `-p 3000:3000` (maps server’s port).

**Important**:
- We use `|| true` after stop and remove because if the container doesn't exist, it shouldn't break the script.

---

# In Summary

**GitLab Pipeline Flow**:

```
Push to GitLab →
  GitLab Runner starts →
    1. Test Node.js code inside node:18
    2. Build Docker image inside docker:dind
    3. Push Docker image to Docker Hub
    4. SSH into DigitalOcean server
    5. Pull the image
    6. Stop and remove old container
    7. Run the new container
```

---

# Bonus Tips:

- If you want to **deploy only on tags** (example: only when you create a version like v1.0.0), you can change:

```yaml
only:
  - tags
```
instead of `- main`.

- You can **rollback easily** by running the older image tag manually.

- Always monitor server's Docker disk space to avoid filling it up.

---

# Important Commands You Should Know:

| Command | Meaning |
|:--------|:--------|
| `docker build -t myimage .` | Build a docker image |
| `docker login` | Login to docker registry |
| `docker push myimage` | Push image to registry |
| `docker pull myimage` | Pull image from registry |
| `docker run -d -p 3000:3000 myimage` | Run container |
| `docker stop container_name` | Stop running container |
| `docker rm container_name` | Remove container |

---

# Final Tips:
- **Use `CI/CD Variables`** for anything sensitive.
- **Separate `build` and `deploy` stages** to avoid mistakes.
- **Use commit SHA tags** for unique image versions.
- **Use docker:dind carefully** — it can consume a lot of memory if pipelines are heavy.

---

# Summary

Now you know:
- How `.gitlab-ci.yml` is written
- How to run tests
- How to build a Docker image in Docker-in-Docker
- How to push to Docker Hub
- How to deploy securely via SSH into a DigitalOcean server

---

Same **Node.js GitLab CI/CD**, but this time **deploying to Amazon ECS Fargate** instead of DigitalOcean.

This setup is **very professional**.  
Big companies use ECS + Fargate because:
- **No servers to manage** (serverless containers)
- **Scalable**, **secure**, and **integrated with AWS**.

I’ll explain **every step**: GitLab YAML, AWS setup (Task Definition, ECS Service, Fargate), and what exactly happens behind the scenes.

---

# 1. Big Picture

Here’s the flow:

```
Push Code to GitLab →
   GitLab Runner starts →
     1. Test Node.js App
     2. Build Docker Image
     3. Push Docker Image to Amazon ECR
     4. Update Task Definition
     5. Deploy updated Task to ECS Service (Fargate)
```

---

# 2. AWS Services You Need

You must prepare these on AWS before starting:

| AWS Service | Purpose |
|:------------|:--------|
| Amazon **ECR** | (Elastic Container Registry) to store Docker images |
| Amazon **ECS** | (Elastic Container Service) to run containers |
| ECS **Task Definition** | Defines how container runs (CPU, Memory, Image, Ports) |
| ECS **Cluster** | Logical grouping of tasks |
| ECS **Service** | Runs and manages copies of your Task |
| **Fargate Launch Type** | Serverless — AWS manages EC2 instances for you |
| **IAM Roles** | Secure permissions for ECS and GitLab |

---

# 3. AWS Preparation (One-Time Setup)

**Step 1: Create an ECR Repository**

- Go to ECR → Repositories → Create.
- Example repo name: `my-node-app`

**Step 2: Create ECS Cluster**

- Go to ECS → Clusters → Create Cluster.
- Select "Networking only" (Fargate).

**Step 3: Create Task Definition**

- Go to ECS → Task Definitions → Create new.
- Launch Type: **Fargate**
- Container definitions:
  - Container name: `node-app`
  - Image URI: Leave empty (GitLab will update it during pipeline)
  - Port mapping: 3000 → 3000
- Task CPU: 256
- Task Memory: 512
- IAM Role: Create a task role if needed (permissions to access ECR).

**Step 4: Create ECS Service**

- Go to ECS → Services → Create.
- Launch Type: Fargate
- Task Definition: Choose the one you just created.
- Cluster: Choose the one you created.
- Load Balancer: Optional. You can expose directly through a Public IP for now.

---

# 4. GitLab CI/CD Variables (Set in GitLab Settings > CI/CD > Variables)

| Key | Purpose |
|:----|:--------|
| AWS_ACCESS_KEY_ID | IAM user’s Access Key |
| AWS_SECRET_ACCESS_KEY | IAM user’s Secret Key |
| AWS_DEFAULT_REGION | Your region (example: `us-east-1`) |
| ECR_REPOSITORY | Your ECR repo URL |
| ECS_CLUSTER | Your ECS Cluster name |
| ECS_SERVICE | Your ECS Service name |
| ECS_TASK_DEFINITION_FAMILY | Your Task definition family name |
| CONTAINER_NAME | Your container name (like `node-app`) |

**Important**: The IAM user must have permission to:
- Push to ECR
- Update ECS services
- Register new task definitions

---

# 5. Final `.gitlab-ci.yml` (Deploy to ECS Fargate)

```yaml
image: amazon/aws-cli:2.13.6

services:
  - docker:dind

variables:
  DOCKER_DRIVER: overlay2
  AWS_REGION: $AWS_DEFAULT_REGION
  IMAGE_TAG: $ECR_REPOSITORY:$CI_COMMIT_SHORT_SHA

stages:
  - test
  - build
  - push
  - deploy

before_script:
  - echo "Setting up AWS CLI..."
  - aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
  - aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
  - aws configure set default.region $AWS_REGION

test:
  stage: test
  image: node:18
  script:
    - echo "Running tests..."
    - npm install
    - npm test

build:
  stage: build
  script:
    - echo "Logging into Amazon ECR..."
    - aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPOSITORY
    - echo "Building docker image..."
    - docker build -t $IMAGE_TAG .

push:
  stage: push
  script:
    - echo "Pushing docker image to ECR..."
    - docker push $IMAGE_TAG

deploy:
  stage: deploy
  script:
    - echo "Deploying to ECS Fargate..."
    # Fetch existing task definition
    - |
      export TASK_DEFINITION=$(aws ecs describe-task-definition \
        --task-definition $ECS_TASK_DEFINITION_FAMILY)
    - |
      export NEW_TASK_DEF=$(echo $TASK_DEFINITION | jq --arg IMAGE "$IMAGE_TAG" \
        '.taskDefinition | {containerDefinitions, cpu, executionRoleArn, family, memory, networkMode, requiresCompatibilities, taskRoleArn} | .containerDefinitions[0].image = $IMAGE | .')
    # Register new task definition
    - |
      echo $NEW_TASK_DEF > new-task-def.json
      aws ecs register-task-definition --cli-input-json file://new-task-def.json
    # Update ECS service to use new task
    - |
      aws ecs update-service --cluster $ECS_CLUSTER --service $ECS_SERVICE --force-new-deployment
```

---

# 6. Detailed Explanation Line by Line

## **image: amazon/aws-cli**
- We use **AWS CLI Docker Image** because we need `aws ecr`, `aws ecs` commands.

## **services: docker:dind**
- `docker:dind` again provides Docker-in-Docker — we need it to build Docker images inside CI.

## **variables**
- Set defaults for AWS region and image tagging.
- Every commit gets a unique image tag.

---

## **before_script**
- Configure AWS CLI inside the runner using GitLab environment variables.

---

## **test job**
- Node.js environment to install packages and run tests.

---

## **build job**
- Authenticate to ECR using AWS CLI and Docker.
- Build the image using `docker build -t`.

---

## **push job**
- Push the Docker image to the Amazon Elastic Container Registry (ECR).

---

## **deploy job**
Biggest job, here’s what happens:
1. **Fetch current task definition** using `aws ecs describe-task-definition`.
2. **Modify the container image** inside the task definition JSON using `jq`.
3. **Register new task definition** using `aws ecs register-task-definition`.
4. **Update ECS service** to use the new task (with the new image).

AWS ECS will then:
- Pull the new image.
- Replace the old container without downtime (blue/green deployment).

---

# 7. Task Definition Update: Behind the Scenes

- In AWS ECS, you **can't modify** a running service directly.
- Instead, you must **register a new task definition version**.
- ECS Service will automatically detect the new task and **replace old tasks with new ones**.

Each task definition version is immutable:
- v1: old version
- v2: new one (after deployment)

---

# 8. Docker-in-Docker: Why Needed Again?

- GitLab runners don't have Docker engine installed by default.
- `docker:dind` service spins up a Docker daemon **inside** the GitLab job container.
- This lets us:
  - Build images.
  - Push images.
  - Login to Docker Registries (like ECR).

---

# 9. Visual Workflow

```
Push Code →
  GitLab Runner →
    Test App →
    Build Docker Image →
    Push to Amazon ECR →
    Register New Task Definition →
    Update ECS Service →
    Fargate Deploys New Container →
    Live Website Updated
```

---

# Final Notes and Best Practices

- **Never expose AWS keys** — always use GitLab CI/CD Variables.
- **Use commit SHA for tagging** — easy rollback and traceability.
- **Use Blue/Green deployments** (optional advanced) — safer upgrades.
- **Use `jq`** to modify task definitions dynamically without hardcoding anything.
- **Monitor ECS Service** health after deploy — ECS will show task status.

---

# Summary

You now learned:
- How to build a GitLab CI/CD pipeline for Node.js apps with Docker
- How to push Docker images to Amazon ECR
- How ECS Task Definitions work
- How to deploy updated containers to Fargate
- How Docker-in-Docker (`docker:dind`) allows building inside CI pipelines

---

Would you like me next to show you an even **better and more professional GitLab pipeline** that:
- **Detects if deployment failed**
- **Automatically rolls back to the previous Task Definition**
- **Sends Slack notification if deployment succeeded or failed**?

It’s a real-world setup used in top companies!  
Would you like to see it? I can prepare it step-by-step for you.

---


Let’s now build a **very professional GitLab pipeline** that includes:

- Full **CI/CD** for Node.js app
- **Deploys to AWS ECS Fargate**
- **Detects Deployment Failure**
- **Rolls back** to the **previous Task Definition** automatically if deployment fails
- **Sends Slack Notifications** on Success or Failure

This is **how real companies do production deployments** — high safety, automated rollback, and team alerts.

---

# 1. Flow Overview

Here’s the advanced flow:

```
Push to GitLab →
  Test Code →
  Build Docker Image →
  Push to ECR →
  Deploy new Task Definition →
    - If deploy succeeds: send "Success" message to Slack
    - If deploy fails: Rollback to previous Task Definition + send "Failure" message to Slack
```

---

# 2. Additional Requirements

Besides what we discussed earlier, you also need:

| GitLab Variable | Purpose |
|:----------------|:--------|
| SLACK_WEBHOOK_URL | Your Slack Incoming Webhook URL |
| (Optional) ECS_SERVICE_ROLE_ARN | For advanced permissions (if using complex setups) |

**Slack Setup**:
- Go to Slack → Create **Incoming Webhook**.
- Set it up in a channel (like `#deployments`).
- Copy the Webhook URL and add it in GitLab Variables.

---

# 3. GitLab `.gitlab-ci.yml`

Here’s the full pipeline for you:

```yaml
image: amazon/aws-cli:2.13.6

services:
  - docker:dind

variables:
  DOCKER_DRIVER: overlay2
  AWS_REGION: $AWS_DEFAULT_REGION
  IMAGE_TAG: $ECR_REPOSITORY:$CI_COMMIT_SHORT_SHA

stages:
  - test
  - build
  - push
  - deploy
  - notify

before_script:
  - echo "Setting up AWS CLI..."
  - aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
  - aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
  - aws configure set default.region $AWS_REGION

test:
  stage: test
  image: node:18
  script:
    - echo "Running tests..."
    - npm install
    - npm test

build:
  stage: build
  script:
    - echo "Logging into ECR..."
    - aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPOSITORY
    - echo "Building docker image..."
    - docker build -t $IMAGE_TAG .

push:
  stage: push
  script:
    - echo "Pushing docker image to ECR..."
    - docker push $IMAGE_TAG

deploy:
  stage: deploy
  script:
    - echo "Deploying to ECS Fargate..."
    - |
      export CURRENT_TASK_ARN=$(aws ecs describe-services --cluster $ECS_CLUSTER --services $ECS_SERVICE --query "service.taskDefinition" --output text)
    - |
      export TASK_DEF_JSON=$(aws ecs describe-task-definition --task-definition $ECS_TASK_DEFINITION_FAMILY)
      export NEW_TASK_DEF=$(echo $TASK_DEF_JSON | jq --arg IMAGE "$IMAGE_TAG" '.taskDefinition | {containerDefinitions, cpu, executionRoleArn, family, memory, networkMode, requiresCompatibilities, taskRoleArn} | .containerDefinitions[0].image = $IMAGE')
      echo $NEW_TASK_DEF > new-task-def.json
      aws ecs register-task-definition --cli-input-json file://new-task-def.json
    - |
      aws ecs update-service --cluster $ECS_CLUSTER --service $ECS_SERVICE --force-new-deployment
  when: on_success
  allow_failure: true

notify_success:
  stage: notify
  script:
    - curl -X POST -H 'Content-type: application/json' --data '{"text":"Deployment Succeeded for '$CI_PROJECT_NAME' :tada:"}' $SLACK_WEBHOOK_URL
  when: on_success
  needs: ["deploy"]

notify_failure:
  stage: notify
  script:
    - echo "Deployment failed. Rolling back..."
    - |
      aws ecs update-service --cluster $ECS_CLUSTER --service $ECS_SERVICE --task-definition $CURRENT_TASK_ARN --force-new-deployment
    - curl -X POST -H 'Content-type: application/json' --data '{"text":"Deployment Failed for '$CI_PROJECT_NAME'. Rolled back to previous version :warning:"}' $SLACK_WEBHOOK_URL
  when: on_failure
  needs: ["deploy"]
```

---

# 4. Deep Detailed Explanation

## `image: amazon/aws-cli`
- We use AWS CLI because we need aws commands.

## `services: docker:dind`
- We need Docker daemon for building/pushing Docker images inside the GitLab runner.

## `variables`
- Define your AWS region and Image Tag based on the commit.

---

# Jobs Breakdown

## `test`
- Node.js testing with npm.
- Fast feedback if anything is broken.

## `build`
- Login to AWS ECR.
- Build the Docker Image.

## `push`
- Push Docker Image to AWS ECR.

---

## `deploy`
This job:
- Fetches the **current running task definition ARN**.
- Creates a **new task definition** with the updated Docker image.
- Registers the new Task Definition.
- Tells ECS Service to **start new deployment**.

If deployment **fails** (detected by GitLab), `allow_failure: true` allows pipeline to continue to `notify_failure`.

---

## `notify_success`
- Sends a success message to Slack with project name and happy emoji.

Example Slack Message:
> Deployment Succeeded for my-node-app 🎉

---

## `notify_failure`
- If `deploy` job fails:
  - Rollback ECS service to the **previous stable task definition**.
  - Send a failure message to Slack with warning emoji.

Example Slack Message:
> Deployment Failed for my-node-app. Rolled back to previous version ⚠️

---

# 5. Visual Pipeline Structure

```
Test →
  Build →
    Push →
      Deploy →
        Notify (Success/Failure + Rollback if needed)
```

---

# 6. Important Concepts

| Concept | Explanation |
|:--------|:------------|
| `needs: ["deploy"]` | Makes `notify` jobs depend on `deploy` result |
| `allow_failure: true` | Even if deploy fails, continue pipeline |
| `rollback` | Uses `aws ecs update-service` to rollback to old task definition |
| `Slack notifications` | Inform your team immediately |
| `docker:dind` | Docker-in-Docker to build/push images |

---

# 7. What Happens If Deployment Fails?

1. ECS update service fails (maybe image bad, container crash, etc).
2. GitLab marks `deploy` as failed.
3. GitLab automatically triggers `notify_failure`.
4. It runs rollback command using saved `CURRENT_TASK_ARN`.
5. Old app is redeployed automatically.
6. Slack gets a message saying "Rolled back".

This ensures **zero downtime** and **instant team communication**.

---

# Summary

You now know how to:

- Build a very **robust production-ready GitLab pipeline**.
- **Build, Push, Deploy**, and **Rollback** automatically.
- **Send Slack notifications** on status.
- Use **docker:dind**, **AWS CLI**, **ECR**, **ECS**, and **Fargate** like a pro.

---

# Final Advice

Real companies use setups like this because:

- You can safely deploy 10-15 times per day.
- Rollbacks happen in minutes automatically.
- DevOps engineers sleep peacefully knowing even failed deployments are safe.

---
