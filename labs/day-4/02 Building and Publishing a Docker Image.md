# **Building and Publishing a Docker Image**

**Module:** Building & Sharing Assets with GitLab Registries
**Time:** Approx. 50 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Write a CI/CD pipeline that can build a Docker image.
*   Understand and use a "Docker-in-Docker" (dind) service in a GitLab CI job.
*   Authenticate to the GitLab Container Registry using predefined CI/CD variables.
*   Build a `Dockerfile` and push the resulting image to your project's private registry.
*   Use your newly published custom image in a subsequent CI/CD job.

### **2. Scenario**

Your team is standardizing its build environments. Instead of each CI/CD job pulling a generic `maven` image and re-installing tools, the platform team has mandated that each project should build and maintain its own custom Docker image containing all necessary dependencies. This improves speed, consistency, and security.

Your task is to create a CI/CD pipeline that automates this process. You will write a `Dockerfile` for a simple Java/Maven environment. You will then create a pipeline with a dedicated `build-image` job that builds this `Dockerfile` and publishes the resulting image to your project's private GitLab Container Registry. Finally, you will prove that it works by creating a second job that uses your newly published image as its execution environment.

### **3. Prerequisites**

*   You have a GitLab.com account and are a member of the `gabors-gitlab-training-20251013` group.
*   You have a fully configured local environment with Git and Visual Studio Code.

### **4. Steps**

_**Note:** This lab is advanced and combines several concepts. Pay close attention to the variable names and YAML syntax._

1.  **Create and Clone the Lab Project**
    *   **[WHERE: GitLab UI - Browser & Local Terminal]**
        *   In the `gabors-gitlab-training-20251013` group, create a new blank project named `student-<firstname>-<lastname>-lab-4.2`.
        *   Initialize it with a `README.md` file.
        *   Clone the project to your local machine and open it in VS Code.

2.  **Create the `Dockerfile` for Your Custom Environment**
    This file defines the custom build environment you want to package.
    *   **[WHERE: Visual Studio Code]**
        *   Create a new branch named `feature/docker-image-pipeline`.
        *   In the root of the project, create a new file named **`Dockerfile`**.
        *   Add the following content. This creates a simple image based on Alpine Linux with `git` and `curl` installed.
    ```Dockerfile
    # filename: Dockerfile
    # Start from a minimal base image
    FROM alpine:3.18

    # Install essential tools needed for many CI jobs
    RUN apk add --no-cache git curl
    ```

3.  **Create the CI/CD Pipeline to Build and Push the Image**
    This is the core of the lab. You will create a two-stage pipeline: one to build the image, and one to use it.
    *   **[WHERE: Visual Studio Code]**
        *   In the root of the project, create a new file named **`.gitlab-ci.yml`**.
        *   Add the following complete configuration:
    ```yaml
    # filename: .gitlab-ci.yml
    stages:
      - build_image
      - test_image

    # This job builds the Dockerfile and pushes it to this project's Container Registry.
    build-the-image:
      stage: build_image
      # We need a special image that contains the Docker client.
      image: docker:24.0.5
      # We also need to run the Docker daemon (engine) as a background service.
      services:
        - docker:24.0.5-dind
      # These variables are required for Docker-in-Docker to work correctly.
      variables:
        DOCKER_TLS_CERTDIR: "/certs"
      # The script logs into the GitLab registry, builds the image, and pushes it.
      script:
        - echo "Logging into GitLab Container Registry..."
        - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY
        - echo "Building the Docker image..."
        # $CI_REGISTRY_IMAGE is a predefined variable that points to your project's registry.
        # $CI_COMMIT_REF_SLUG creates a unique tag for your branch (e.g., feature-docker-image-pipeline).
        - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG .
        - echo "Pushing the Docker image to the registry..."
        - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG

    # This job will USE the image we just built.
    test-the-image:
      stage: test_image
      # Here, we tell GitLab to use the image we just pushed to our registry.
      image: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
      script:
        - echo "Hello from inside our custom-built image!"
        - echo "Verifying that our tools are installed..."
        - git --version
        - curl --version
    ```

4.  **Commit, Push, and Run the Pipeline**
    *   **[WHERE: Visual Studio Code]**
        *   Use the Source Control panel to stage both your `Dockerfile` and `.gitlab-ci.yml` files.
        *   Commit the changes with the message `feat: add pipeline to build and publish docker image`.
        *   Push your `feature/docker-image-pipeline` branch to GitLab.

### **5. Verification**

1.  **Observe the Pipeline:**
    *   **[WHERE: GitLab UI - Browser]**
        *   Create a Merge Request for your branch.
        *   Navigate to the running pipeline. It will have two stages: `build_image` and `test_image`.

2.  **Verify the Image Build and Push:**
    *   Inspect the log for the `build-the-image` job.
    *   **Expected Result:** You should see a successful `docker login`, the output from the `docker build` process (including `RUN apk add...`), and a final confirmation that the image was pushed to your registry.

3.  **Verify the Container Registry:**
    *   In your project's left-hand navigation, go to `Deploy -> Container registry`.
    *   **Expected Result:** You will see a new image repository listed. Clicking on it will show you a new image tag that matches your branch name (e.g., `feature-docker-image-pipeline`).

4.  **Verify the Image Usage:**
    *   Inspect the log for the `test-the-image` job.
    *   **Expected Result:** At the top of the log, you will see a message like `Pulling docker image gabors-gitlab-training-20251013/student-gabor-szabo-lab-3.3:feature-docker-image-pipeline...`.
    *   The script output must show the version numbers for `git` and `curl`, proving the job ran inside your custom-built container.

### **6. Discussion**

This lab demonstrated a powerful, advanced CI/CD pattern: using a pipeline to build its own execution environment. The `build-the-image` job is a special case that requires a "Docker-in-Docker" (`dind`) setup. We used the official `docker` image, which contains the Docker client, and then added `docker:dind` as a **service**. This created a sandboxed Docker engine that our job could use to build and push images.

Authentication was handled automatically and securely using GitLab's **predefined CI/CD variables**. We used `$CI_REGISTRY_USER`, `$CI_REGISTRY_PASSWORD`, and `$CI_REGISTRY` to log in without ever hardcoding a password.

The most important concept was the "dogfooding" in the `test-the-image` job. By setting its `image` to `$CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG`, we instructed the GitLab Runner to pull and use the exact image that was created just one stage earlier. This creates a self-sufficient, verifiable loop and is a common pattern for building and testing custom base images for a development team.

### **7. Questions**

1.  What is the purpose of the `services` keyword in the `build-the-image` job? Why couldn't we just use the `docker:dind` image as the main `image`?
2.  What is the value of the predefined variable `$CI_REGISTRY_IMAGE`? Why is it better than hardcoding the image path?
3.  The `build-the-image` job uses `$CI_COMMIT_REF_SLUG` as the image tag. What is a potential long-term problem with this tagging strategy for a project with many branches?
4.  If the `test-the-image` job failed, would the `build-the-image` job be re-run automatically when you retry the failed job? Why or why not?
5.  What change would you make to the `build-the-image` job to *also* tag the image with `latest` but only when merging to the `main` branch?

---

### **8. Solution**

This section contains the completed artifacts for this lab.

#### **8.1. Final `.gitlab-ci.yml` File**
```yaml
# filename: .gitlab-ci.yml
stages:
  - build_image
  - test_image

build-the-image:
  stage: build_image
  image: docker:24.0.5
  services:
    - docker:24.0.5-dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  script:
    - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG

test-the-image:
  stage: test_image
  image: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
  script:
    - echo "Hello from inside our custom-built image!"
    - echo "Verifying that our tools are installed..."
    - git --version
    - curl --version
```

#### **8.2. Final `Dockerfile`**
```Dockerfile
# filename: Dockerfile
FROM alpine:3.18

RUN apk add --no-cache git curl
```

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **Purpose of `services`?**
    The `services` keyword launches a linked container alongside your main job container. Your `script` runs in the main `image` (`docker`), which has the client tools, and it communicates with the `service` container (`docker:dind`), which runs the Docker engine (daemon). You cannot just use `dind` as the main image because it is not configured to be a client.

2.  **Value of `$CI_REGISTRY_IMAGE`?**
    It is a predefined variable that automatically resolves to the correct path for your project's container registry (e.g., `gitlab.com/gabors-gitlab-training-20251013/student-gabor-szabo-lab-3.3`). Using the variable is better than hardcoding because it makes your `.gitlab-ci.yml` portable; you can reuse the same file in another project, and it will automatically point to that new project's registry.

3.  **Problem with branch-name tags?**
    Over time, this will fill your registry with hundreds of old, abandoned feature-branch images that are no longer needed. This consumes storage and makes it hard to find relevant images. A professional workflow would include a **Container Registry cleanup policy** (configured in the UI) to automatically delete tags that are old and not associated with a protected branch.

4.  **Would `build-the-image` re-run?**
    No. When you retry a failed job, GitLab only re-runs that specific job and any subsequent jobs in the pipeline. It does not go back and re-run jobs from already successfully completed stages. The `test-the-image` job would simply re-download the existing artifact (the image) created by the original `build-the-image` run.

5.  **How to add a `latest` tag on `main`?**
    You would add a `rules` block to the `build-the-image` job. Then, you would add an `after_script` section with its own `rules` that contains the `docker tag` and `docker push` commands, which would only execute when `$CI_COMMIT_BRANCH == "main"`.