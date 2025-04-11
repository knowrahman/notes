# Docker


Docker is a platform that enables developers to **build, package, and run applications in containers**. A container is a lightweight, standalone, and executable software package that includes everything needed to run an application—**code, runtime, libraries, dependencies, and system tools**.

### Why Docker is Useful
Traditionally, running software across different systems would cause issues like "it works on my machine." Docker solves this by packaging the app with its environment so it behaves the same everywhere.

### Real-World Example
Imagine you're developing a web app that runs on Node.js and uses MongoDB. Instead of asking every team member to install Node.js, MongoDB, and all dependencies, you can just:
- Create a **Dockerfile** (a script that defines the environment)
- Run `docker build` and `docker run` to start the app
- Ship the container to anyone or deploy it to production, and it will work the same.

### Key Components of Docker:
1. **Dockerfile** – a text file with instructions on how to build a Docker image.
2. **Docker Image** – a snapshot of your app and its environment.
3. **Docker Container** – a running instance of the Docker image.
4. **Docker Hub** – like GitHub, but for Docker images (public/private registries).
5. **Docker Engine** – the software that runs and manages containers.

### Summary
Docker allows you to:
- Eliminate "works on my machine" problems
- Speed up development and deployment
- Run multiple isolated apps on the same host

---


A common problem scenario that Docker solves is **dependency and environment mismatch** between development, testing, and production environments. Here's an example:

### Scenario: "It works on my machine" Problem

#### Problem:
You are working on a Node.js application that relies on MongoDB for its database. Your development environment is a local machine with specific versions of Node.js, MongoDB, and various libraries installed. You test everything on your local machine, and everything works fine.

However, when you deploy the application to production, or send it to a teammate for testing, it fails. The production environment has a different version of MongoDB or Node.js, or perhaps it's missing some required dependencies. You and your teammates struggle to set up the exact environment as your local machine, and the application behaves inconsistently between environments.

#### How Docker Solves This:
With Docker, you can package your entire application and its dependencies into a **Docker container**. This container includes:
- The **Node.js version** you’re using
- The **MongoDB version** you depend on
- Any libraries or dependencies that are required to run your app

You can then create a **Dockerfile** to define the application environment and build a **Docker image**. This image can be shared and run anywhere (on your teammate’s machine, in staging, or on production servers) without worrying about mismatched dependencies or environments.

Since the container runs in an isolated environment that includes everything needed, it works the same on **every machine**. The app will behave exactly the same in development, testing, and production environments, eliminating the "it works on my machine" problem.

### Benefits:
- **Consistency**: Docker ensures that your app runs consistently across different environments.
- **Portability**: You can run Docker containers on any machine that has Docker installed (Windows, Linux, macOS).
- **Isolation**: The app runs in a container with all dependencies bundled together, so you don't worry about conflicts with other apps on the same machine.

---


To get started with Docker, you'll need to install both the Docker CLI (Command Line Interface) and Docker Desktop. Below are the steps to install and run Docker on your machine.

### 1. Installing Docker CLI and Desktop

#### **For Windows:**
1. **Download Docker Desktop**:
   - Go to the [Docker Desktop download page](https://www.docker.com/products/docker-desktop) and download the installer for Windows.
   
2. **Run the Installer**:
   - After downloading the installer, run it. Follow the installation prompts to install Docker Desktop.
   
3. **Enable WSL 2 (Windows Subsystem for Linux 2)**:
   - Docker Desktop on Windows uses WSL 2 as its default backend. If you don't have it installed, Docker will guide you through the installation process. Follow the instructions to enable WSL 2.

4. **Start Docker Desktop**:
   - After installation, launch Docker Desktop. It will take a minute to start, and you’ll see the Docker whale icon in your taskbar when it’s ready.

5. **Check Docker Version**:
   - Open PowerShell or Command Prompt and run the following command to check if Docker is installed properly:
     ```bash
     docker --version
     ```
   - You should see the Docker version displayed, confirming that Docker CLI is installed.

#### **For macOS:**
1. **Download Docker Desktop**:
   - Go to the [Docker Desktop download page](https://www.docker.com/products/docker-desktop) and download the installer for macOS.

2. **Run the Installer**:
   - Open the downloaded `.dmg` file and drag the Docker application into your Applications folder.

3. **Launch Docker Desktop**:
   - After the installation is complete, launch Docker Desktop from your Applications folder.
   - Docker will start up, and you will see the Docker whale icon in your top menu bar when it's ready.

4. **Check Docker Version**:
   - Open Terminal and run:
     ```bash
     docker --version
     ```
   - You should see the Docker version displayed.

#### **For Linux:**
Docker can be installed directly using your system’s package manager. Here’s how you can install Docker on various Linux distributions.

- **For Ubuntu/Debian**:
  1. Update the package database:
     ```bash
     sudo apt-get update
     ```
  2. Install the necessary dependencies:
     ```bash
     sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
     ```
  3. Add Docker’s official GPG key:
     ```bash
     curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
     ```
  4. Set up the stable Docker repository:
     ```bash
     sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
     ```
  5. Update the package database again:
     ```bash
     sudo apt-get update
     ```
  6. Install Docker:
     ```bash
     sudo apt-get install docker-ce
     ```
  7. Verify that Docker is installed:
     ```bash
     docker --version
     ```

- **For CentOS**:
  1. Install Docker:
     ```bash
     sudo yum install docker
     ```
  2. Start Docker:
     ```bash
     sudo systemctl start docker
     ```
  3. Enable Docker to start on boot:
     ```bash
     sudo systemctl enable docker
     ```
  4. Verify Docker installation:
     ```bash
     docker --version
     ```

### 2. Running the Docker Daemon

The Docker daemon (or `dockerd`) is the core service that runs in the background and manages Docker containers. Usually, Docker Desktop on Windows and macOS automatically runs the Docker daemon when you launch the Docker Desktop application. On Linux, you might need to start the Docker service manually.

#### **For Windows and macOS**:
- Docker Desktop automatically starts the Docker daemon when you launch the application.
- You don’t need to manually start the Docker daemon as it runs in the background.

#### **For Linux**:
1. **Start Docker Daemon**:
   If Docker is not running already, start the service with:
   ```bash
   sudo systemctl start docker
   ```

2. **Enable Docker Daemon to Start on Boot**:
   To ensure Docker starts automatically when the system boots, run:
   ```bash
   sudo systemctl enable docker
   ```

3. **Verify Docker Daemon is Running**:
   Check the Docker daemon status:
   ```bash
   sudo systemctl status docker
   ```
   If it’s running, you should see an active status.

### 3. Running a Test Docker Container

Once Docker is installed and the daemon is running, you can test it by running a simple Docker container.

1. Open your terminal or command prompt.

2. Run the following command to download and run the official "hello-world" image from Docker Hub:
   ```bash
   docker run hello-world
   ```

   This command does the following:
   - **docker run**: Downloads the image and starts a container from it.
   - **hello-world**: This is the name of the image you are running. Docker will pull it from the Docker Hub if it’s not already available on your machine.

3. **Expected Output**:
   You should see a message that says something like:
   ```
   Hello from Docker!
   This message shows that your installation appears to be working correctly.
   ```

This confirms that Docker is up and running on your system.


---

## *docker run -it ubuntu*



The command `docker run -it ubuntu` is used to run a Docker container based on the **Ubuntu** image interactively. Here's a breakdown of what each part of the command does:

### Command Breakdown:
1. **`docker run`**: 
   - This is the base Docker command used to create and start a container from a Docker image.

2. **`-i`** (short for `--interactive`): 
   - This option tells Docker to keep the container's standard input (stdin) open, allowing you to interact with the container. Without `-i`, the container would run in the background, and you wouldn't be able to interact with it.

3. **`-t`** (short for `--tty`):
   - This option allocates a pseudo-TTY (terminal) for the container, giving it an interactive terminal interface. It ensures that you get a proper terminal session where you can run commands.

4. **`ubuntu`**:
   - This is the name of the Docker image you want to use. In this case, it's the official Ubuntu image from Docker Hub. If the Ubuntu image is not already available locally, Docker will pull it from Docker Hub.

### What Happens When You Run This Command:
- When you execute `docker run -it ubuntu`, Docker will:
  1. **Check if the Ubuntu image is available locally**. If it's not, Docker will download it from Docker Hub.
  2. **Start a new container** from the Ubuntu image.
  3. **Allocate an interactive terminal session** within the container, allowing you to run commands inside the container as if you were inside an actual Ubuntu system.
  4. **Give you access to a shell** inside the container (typically the default `bash` shell for Ubuntu).

### Example Usage:
1. **Start a Container**:
   ```bash
   docker run -it ubuntu
   ```

2. **You will get a shell prompt inside the Ubuntu container**:
   ```
   root@<container_id>:/#
   ```

3. **Run Commands**:
   From here, you can run any commands available within the Ubuntu container. For example:
   ```bash
   apt-get update
   apt-get install curl
   ```

4. **Exit the Container**:
   To exit the container, simply type `exit`:
   ```bash
   exit
   ```

### Summary:
- The `docker run -it ubuntu` command launches an interactive Ubuntu container, allowing you to work inside the container with a terminal interface.
- It’s useful for exploring or testing within an isolated Ubuntu environment without needing to install Ubuntu directly on your machine.


---

In Docker, **images** and **containers** are two fundamental concepts, but they serve different purposes. Here's a detailed explanation of their differences:

### 1. **Docker Images**

A **Docker image** is a lightweight, standalone, and executable package that includes everything needed to run a software application:
- The **code** itself.
- **Libraries**, dependencies, and frameworks required to run the app.
- **System tools** and settings (OS, configuration, etc.).

#### Characteristics of Docker Images:
- **Read-only**: Images are static and cannot be changed. Once an image is created, it remains the same unless a new image is built.
- **Layers**: Docker images are built in layers. Each layer represents a change in the file system (such as adding a file, installing a library, or changing a configuration). This layered structure allows Docker to optimize image storage by reusing layers across images.
- **Portable**: Since they contain everything required to run an app, images are portable and can be shared and run across different environments, such as development, testing, and production.
- **Versioning**: Images can have different versions, usually indicated by a tag (e.g., `ubuntu:18.04`).

#### How Docker Images Are Created:
1. **Dockerfile**: A text file that contains the instructions for creating a Docker image. It defines what the image will contain and how it will be built.
2. **`docker build`**: The command used to create an image from a Dockerfile.

Example: 
```Dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y curl
CMD ["echo", "Hello from Docker!"]
```
Once this Dockerfile is written, running `docker build` will generate an image that includes Ubuntu 20.04, the `curl` package, and a command to echo a message.

#### Key Points:
- Docker images are **immutable** and **reusable**.
- They are stored in Docker registries (e.g., Docker Hub).
  
### 2. **Docker Containers**

A **Docker container** is a running instance of a Docker image. Containers are the actual environment where applications are executed. While an image is static, a container is dynamic, created from the image when it is run.

#### Characteristics of Docker Containers:
- **Writable**: When you run a container from an image, it starts as a read-only copy of the image. Any changes (file creation, modifications, deletions) made inside the container during its execution are **written to a separate layer**. The changes are lost when the container is stopped and removed, unless explicitly saved (via a new image).
- **Ephemeral**: Containers can be started and stopped. They can also be removed, and when they are removed, any changes that were made to the container are lost unless you commit the container to a new image.
- **Isolated Environment**: Containers run in isolated environments. They can share the host's OS kernel but operate as if they are separate, with their own filesystem and network interfaces.

#### Key Points:
- Containers are **mutable** and represent the actual running environment of an application.
- Containers **execute** the application or service defined by the image.
  
### How Images and Containers Work Together:

1. **Images as Templates**:
   - Docker images are like blueprints or templates that define the base environment for running your application.

2. **Containers as Running Instances**:
   - Containers are created from images and represent the actual execution of the application in an isolated environment. When you run a container, you are launching an instance of the image with the associated environment.

#### Example:
1. You create a Docker image for a Node.js application using a `Dockerfile`.
2. You run the command `docker run -it node-app`, which creates a **container** from the `node-app` image.
3. The container runs the application and allows you to interact with it.
4. Once the container is stopped, it can be removed, but the image remains intact and reusable to create new containers.

### Summary of Differences:

| **Feature**                | **Image**                                      | **Container**                                    |
|----------------------------|------------------------------------------------|--------------------------------------------------|
| **Definition**              | A static, read-only blueprint for creating containers | A running instance of a Docker image             |
| **Mutability**              | Immutable (cannot be changed once created)     | Mutable (can make changes during runtime)        |
| **Lifecycle**               | Exists as long as you keep it in the registry  | Exists as long as it is running (can be stopped/removed) |
| **Storage**                 | Stored in Docker registries (e.g., Docker Hub) | Stored on the host machine in a separate layer   |
| **Usage**                   | Used to create containers                     | Used to run applications inside an isolated environment |

### Example to Illustrate:
1. **Create an Image**: You write a `Dockerfile` to create an image for a simple web server. You build the image with `docker build`.
2. **Run a Container**: You run `docker run -it <image_name>` to start a container from the image. The container is now running your web server.
3. **Stop the Container**: You stop the container with `docker stop <container_id>`. The container is no longer running, but the image is still there and can be reused to create new containers.

---

Here are some of the most **common Docker commands** you will use frequently when working with Docker:

### 1. **docker --version**
- **Purpose**: Check the version of Docker installed on your machine.
- **Example**:
  ```bash
  docker --version
  ```

### 2. **docker pull <image>**
- **Purpose**: Download a Docker image from a registry (like Docker Hub).
- **Example**: 
  ```bash
  docker pull ubuntu:20.04
  ```

### 3. **docker build -t <image_name> <path>**
- **Purpose**: Build a Docker image from a `Dockerfile`. The `-t` flag assigns a name (tag) to the image.
- **Example**:
  ```bash
  docker build -t my-node-app .
  ```

### 4. **docker run <options> <image>**
- **Purpose**: Create and start a container from a specified image.
  - `-it`: Runs the container interactively (with a terminal).
  - `--name <container_name>`: Names the container for easy reference.
  - `-p <host_port>:<container_port>`: Maps the host port to the container port.
- **Example**:
  ```bash
  docker run -it -p 8080:80 --name web-container nginx
  ```
  This runs an Nginx web server container and maps port 80 inside the container to port 8080 on the host machine.

### 5. **docker ps**
- **Purpose**: List all running containers.
- **Example**:
  ```bash
  docker ps
  ```

### 6. **docker ps -a**
- **Purpose**: List all containers, including stopped ones.
- **Example**:
  ```bash
  docker ps -a
  ```

### 7. **docker stop <container_name_or_id>**
- **Purpose**: Stop a running container.
- **Example**:
  ```bash
  docker stop web-container
  ```

### 8. **docker start <container_name_or_id>**
- **Purpose**: Start a stopped container.
- **Example**:
  ```bash
  docker start web-container
  ```

### 9. **docker restart <container_name_or_id>**
- **Purpose**: Restart a running or stopped container.
- **Example**:
  ```bash
  docker restart web-container
  ```

### 10. **docker rm <container_name_or_id>**
- **Purpose**: Remove a container (it must be stopped first).
- **Example**:
  ```bash
  docker rm web-container
  ```

### 11. **docker rmi <image_name_or_id>**
- **Purpose**: Remove a Docker image.
- **Example**:
  ```bash
  docker rmi my-node-app
  ```

### 12. **docker logs <container_name_or_id>**
- **Purpose**: View the logs of a running or stopped container.
- **Example**:
  ```bash
  docker logs web-container
  ```

### 13. **docker exec -it <container_name_or_id> <command>**
- **Purpose**: Run a command inside a running container. The `-it` flag allows for interactive input.
- **Example**: 
  ```bash
  docker exec -it web-container bash
  ```
  This opens an interactive bash shell inside the `web-container`.

### 14. **docker images**
- **Purpose**: List all available Docker images on your system.
- **Example**:
  ```bash
  docker images
  ```

### 15. **docker network ls**
- **Purpose**: List all networks in Docker.
- **Example**:
  ```bash
  docker network ls
  ```

### 16. **docker volume ls**
- **Purpose**: List all Docker volumes on your system.
- **Example**:
  ```bash
  docker volume ls
  ```

### 17. **docker-compose up**
- **Purpose**: Start up services defined in a `docker-compose.yml` file.
- **Example**:
  ```bash
  docker-compose up
  ```
  This will start the services (containers) defined in your `docker-compose.yml`.

### 18. **docker-compose down**
- **Purpose**: Stop and remove all containers defined in the `docker-compose.yml` file.
- **Example**:
  ```bash
  docker-compose down
  ```

### 19. **docker exec -it <container_name> /bin/bash**
- **Purpose**: Access the terminal of a running container.
- **Example**:
  ```bash
  docker exec -it web-container /bin/bash
  ```

### 20. **docker inspect <container_name_or_id>**
- **Purpose**: View detailed information about a container or image (in JSON format).
- **Example**:
  ```bash
  docker inspect web-container
  ```

### 21. **docker stats**
- **Purpose**: View real-time stats (like CPU, memory usage) of running containers.
- **Example**:
  ```bash
  docker stats
  ```

### 22. **docker commit <container_name_or_id> <new_image_name>**
- **Purpose**: Create a new image from the changes made in a running container.
- **Example**:
  ```bash
  docker commit web-container new-image-name
  ```

### 23. **docker save -o <file_name>.tar <image_name>**
- **Purpose**: Save a Docker image to a tar archive file.
- **Example**:
  ```bash
  docker save -o my-image.tar my-image-name
  ```

### 24. **docker load -i <file_name>.tar**
- **Purpose**: Load a Docker image from a tar archive.
- **Example**:
  ```bash
  docker load -i my-image.tar
  ```

These are some of the most commonly used Docker commands. They will help you manage images, containers, networks, and volumes as you work with Docker.
