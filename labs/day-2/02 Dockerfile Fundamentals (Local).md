# **Dockerfile Fundamentals (Local)**

**Module:** Docker Essentials for DevOps
**Time:** Approx. 30 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Write a simple, multi-instruction `Dockerfile`.
*   Build a `Dockerfile` into a custom Docker image using the `docker build` command.
*   Run your custom image as a container using the `docker run` command.
*   Inspect your local Docker images and running containers.

### **2. Scenario**

As a DevOps engineer, one of your core responsibilities is to create consistent, reproducible application environments. Docker is the industry-standard tool for this. Instead of manually installing software on a server, you define the entire environment as code in a special file called a `Dockerfile`. This allows you to build a portable "snapshot" (an image) of your application and its dependencies, which can then be run identically on any machine, from a developer's laptop to a production server.

In this lab, you will create your first Docker image. You will write a `Dockerfile` for a simple "Hello World" Python application, build it into an image, and run it as a container on your local Windows machine using Docker Desktop.

### **3. Prerequisites**

*   You are on a Windows machine.
*   Docker Desktop is installed and running.
*   You have Visual Studio Code installed.

### **4. Steps**

_**Note:** All commands in this lab should be executed in a standard Windows **PowerShell** or **Command Prompt (cmd)** terminal, not Git Bash._

1.  **Prepare Your Application Workspace**
    First, you need a simple application to containerize.
    *   **[WHERE: Windows File Explorer & VS Code]**
        *   Create a new folder on your machine, for example, `C:\gitlab-training\docker-lab`.
        *   Open this empty folder in Visual Studio Code.
        *   Inside VS Code, create a new file named `app.py`.
        *   Add the following Python code to `app.py`. This is our simple "Hello World" application.
    ```python
    # filename: app.py
    import sys

    # Get the name from the command-line argument, or default to "World"
    name = sys.argv[1] if len(sys.argv) > 1 else "World"

    print(f"Hello, {name} from inside a Docker container!")
    ```

2.  **Start Docker Desktop**
    Before using any `docker` commands, you must ensure the Docker engine is running.
    *   **[WHERE: Windows Start Menu]**
        *   Open the Windows Start Menu, search for "Docker Desktop," and launch the application.
        *   Wait for the Docker icon in your system tray to become stable (no longer animating). This indicates the Docker engine is running and ready.

3.  **Create the `Dockerfile`**
    Now, you will write the "recipe" for your Docker image.
    *   **[WHERE: Visual Studio Code]**
        *   In the root of your `docker-lab` folder, create a new file named `Dockerfile` (no file extension).
        *   Add the following instructions. Each line is a step in the image-building process.
    ```Dockerfile
    # filename: Dockerfile
    # 1. Start from a lightweight, official Python base image
    FROM python:3.9-slim

    # 2. Set the working directory inside the container
    WORKDIR /app

    # 3. Copy our application script into the container's working directory
    COPY app.py .

    # 4. Define the default command to run when the container starts
    ENTRYPOINT ["python", "app.py"]
    ```

4.  **Build Your Docker Image**
    Use the `docker build` command to turn your `Dockerfile` into an image.
    *   **[WHERE: Windows PowerShell / CMD in VS Code]**
        *   Open a new terminal inside VS Code (`Ctrl+`` ` or `Terminal > New Terminal`).
        *   Execute the following command. The `-t` flag lets you "tag" (name) your image. The `.` at the end tells Docker to look for the `Dockerfile` in the current directory.
    ```powershell
    docker build -t hello-world-app .
    ```
    *   **Observe:** Docker will execute each step from your `Dockerfile`, starting by downloading the `python:3.9-slim` base image.

5.  **Run Your Application as a Container**
    With the image built, you can now run it as a container.
    *   **[WHERE: Windows PowerShell / CMD in VS Code]**
        *   Execute the `docker run` command to start a container from your image.
    ```powershell
    docker run --rm hello-world-app
    ```
    *   Now, run it again, but this time pass a command-line argument to your Python script. Anything after the image name is passed to the `ENTRYPOINT`.
    ```powershell
    docker run --rm hello-world-app Gabor
    ```

### **5. Verification**

1.  **Verify the Build:** The `docker build` command should complete with a message like `Successfully tagged hello-world-app:latest`.
2.  **Verify Your Image Exists:** Run the following command. You should see `hello-world-app` listed at the top of your local images.
    ```powershell
    docker images
    ```
3.  **Verify the Container Runs:** The output from your `docker run` commands should be:
    ```
    Hello, World from inside a Docker container!
    ```
    And for the second command:
    ```
    Hello, <yourname> from inside a Docker container!
    ```
    The `--rm` flag automatically cleans up and removes the container after it exits, which is a good practice for simple, one-off tasks.

### **6. Discussion**

This lab demonstrated the fundamental Docker workflow, which is the engine of modern CI/CD. You created a **`Dockerfile`**, which is a piece of Infrastructure as Code that declaratively defines your application's environment. The `FROM` instruction specified your base layer, `WORKDIR` and `COPY` configured the environment, and `ENTRYPOINT` defined its runtime behavior.

The `docker build` command executed this recipe, creating a portable, self-contained **image**. This image is a static, immutable snapshot of your application and all its dependencies (in this case, the entire Python 3.9 runtime).

Finally, the `docker run` command created a live, running **container** from that image. The container is the ephemeral, executable instance of your image. This separation of a static image from a running container is the core concept that enables consistent and reproducible deployments.

### **7. Questions**

1.  In the `Dockerfile`, what is the purpose of the `WORKDIR /app` instruction? What would happen if you removed it?
2.  What is the difference between the `COPY app.py .` command in your `Dockerfile` and running `COPY . .`?
3.  The `ENTRYPOINT` instruction defined the *default* command. How could you override this and run a *different* command inside the container (e.g., to get an interactive shell)?
4.  If you were to change the `print` statement in `app.py`, would you need to run `docker build` again, `docker run` again, or both? Why?
5.  Our base image was `python:3.9-slim`. What is the benefit of using a `-slim` tag versus a full tag like `python:3.9` in a production environment?

---

### **8. Solution**

This section contains the completed artifacts for this lab.

#### **8.1. Final Code Artifacts**

**`app.py`:**
```python
# filename: app.py
import sys

name = sys.argv[1] if len(sys.argv) > 1 else "World"

print(f"Hello, {name} from inside a Docker container!")
```

**`Dockerfile`:**
```Dockerfile
# filename: Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY app.py .

ENTRYPOINT ["python", "app.py"]
```

#### **8.2. Command Summary**
```powershell
# Build the Docker image from the Dockerfile in the current directory
docker build -t hello-world-app .

# Run the container with the default command
docker run --rm hello-world-app

# Run the container and pass a command-line argument
docker run --rm hello-world-app Gabor

# List all local Docker images
docker images
```

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **What is `WORKDIR`?**
    The `WORKDIR` instruction sets the default directory for all subsequent commands in the `Dockerfile` (`COPY`, `RUN`, `CMD`, `ENTRYPOINT`). If you removed it, the `COPY app.py .` command would copy `app.py` to the root directory (`/`), which is considered bad practice. It's essential for organizing the container's filesystem.

2.  **`COPY app.py .` vs. `COPY . .`?**
    `COPY app.py .` is very specific; it only copies the `app.py` file into the container. `COPY . .` is a broad command that copies **everything** in the current build context (the `docker-lab` folder) into the container. This is often used, but can be dangerous if you don't have a `.dockerignore` file, as it could copy in temporary files, build artifacts, or even your `.git` directory.

3.  **How to override the `ENTRYPOINT`?**
    You can run an interactive shell (`sh`) by specifying it as the command after the image name: `docker run --rm -it hello-world-app sh`. The `-it` flags are for "interactive" and "tty," which are necessary for a shell.

4.  **If you change `app.py`, what commands must you run?**
    You must run **both**. First, you need to run `docker build` to create a new version of the image that contains your updated `app.py` file. Then, you need to run `docker run` to start a new container from that *new* image. Containers are created from a static image and do not automatically update if the source code changes.

5.  **Benefit of a `-slim` tag?**
    `-slim` images are significantly smaller than their full counterparts because they omit many common packages and development tools that are not strictly necessary to run the application. Using a smaller base image results in a smaller final image, which is faster to pull from a registry, consumes less disk space, and has a smaller security attack surface. It is a best practice for production deployments.