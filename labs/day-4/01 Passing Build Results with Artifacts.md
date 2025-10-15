# **Passing Build Results with Artifacts**

**Module:** Optimizing Pipelines with Caching & Artifacts
**Time:** Approx. 40 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Understand the purpose of CI/CD artifacts and how they differ from the cache.
*   Configure a job to create and upload an artifact using the `artifacts:paths` keyword.
*   Observe how subsequent stages automatically download artifacts.
*   Implement a pipeline that follows the "build once, test many" principle for build integrity.
*   Set an expiration policy for an artifact to manage storage.

### **2. Scenario**

Your team's pipeline now uses caching, which has significantly sped up dependency resolution. However, you've identified a major flaw in your workflow: the `build` job compiles the code, and then the `test` job *also* compiles the code again before running the tests. This is not only inefficient, but it violates a core DevOps principle: you must test the *exact* thing you built.

Your task is to re-architect the pipeline to enforce this principle. You will modify the `build` job to produce a compiled `.jar` file and save it as an **artifact**. You will then modify the `test` job to download this artifact and run its tests against that specific, immutable build product, ensuring that what you test is precisely what you will later deploy.

### **3. Prerequisites**

*   You have a GitLab.com account and are a member of the `gabors-gitlab-training-20251013` group.
*   You have a fully configured local environment with Git, Java 17, Maven, and VS Code.
*   We will use the same Java/Maven starter project from the previous lab.

### **4. Steps**

_**Note:** This lab will guide you through creating a new project, establishing a baseline "flawed" pipeline, and then correcting it with artifacts to see the difference in the workflow._

1.  **Create and Clone the Lab Project**
    *   **[WHERE: GitLab UI - Browser & Local Terminal]**
        *   In the `gabors-gitlab-training-20251013` group, create a new project named `student-<firstname>-<lastname>-lab-4.1`.
        *   Select **"Import project"** and use the **"Repository by URL"** option.
        *   For the "Git repository URL," use the same starter project URL from the previous lab: `https://gitlab.com/gabor-szabo-iqsoft/java-maven-starter.git`.
        *   Click **"Create project"**.
        *   Clone your new project to your local machine and open it in VS Code.

2.  **Create the Initial "Flawed" Pipeline**
    This pipeline inefficiently rebuilds the code in the `test` stage.
    *   **[WHERE: Visual Studio Code]**
        *   Create a new branch named `feature/implement-artifacts`.
        *   In the root of the project, create a **`.gitlab-ci.yml`** file with the caching configuration from the previous lab, but now with both `build` and `test` stages.
    ```yaml
    # filename: .gitlab-ci.yml
    image: maven:3.9-eclipse-temurin-17

    variables:
      MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository"

    cache:
      key: { files: [pom.xml] }
      paths: [".m2/repository"]

    stages:
      - build
      - test

    build-job:
      stage: build
      script:
        - echo "Compiling the code..."
        - mvn compile

    test-job:
      stage: test
      script:
        - echo "Testing the code... This will re-compile everything."
        - mvn test # mvn test automatically runs the compile phase again
    ```

3.  **Implement Artifacts in the Build Job**
    Now, you will modify the `build-job` to create a package and declare it as an artifact.
    *   **[WHERE: Visual Studio Code]**
        *   Modify the `build-job` in your `.gitlab-ci.yml` file. Change `mvn compile` to `mvn package` to create a `.jar` file, and add the `artifacts` block.
    ```yaml
    # ... inside .gitlab-ci.yml

    build-job:
      stage: build
      script:
        - echo "Packaging the application..."
        # 'package' compiles the code AND creates a .jar file in the 'target' directory
        - mvn package
      artifacts:
        paths:
          - target/ # This directory contains the compiled .jar file
        expire_in: 1 hour # Keep the artifact for 1 hour
    ```

4.  **Modify the Test Job to Use the Artifact**
    Next, you will simplify the `test-job` so that it no longer rebuilds the code. It will automatically receive the `target/` directory artifact from the `build-job`.
    *   **[WHERE: Visual Studio Code]**
        *   Modify the `test-job`'s `script` section. The `mvn test` command will now find the already-compiled classes in the downloaded artifact and proceed directly to running tests.
    ```yaml
    # ... inside .gitlab-ci.yml

    test-job:
      stage: test
      script:
        - echo "Testing the packaged artifact..."
        - mvn test # This now runs tests against the classes in the 'target' artifact
    ```

5.  **Commit and Push Your Corrected Pipeline**
    *   **[WHERE: Visual Studio Code]**
        *   Use the Source Control panel to stage your `.gitlab-ci.yml` file.
        *   Commit the change with the message `ci: use artifacts to pass build to test stage`.
        *   Push your `feature/implement-artifacts` branch to GitLab.

### **5. Verification**

1.  **Observe the Pipeline in GitLab:**
    *   **[WHERE: GitLab UI - Browser]**
        *   Create a Merge Request for your branch.
        *   Navigate to the latest running pipeline.

2.  **Verify Artifact Upload:**
    *   Click on the `build-job` in the pipeline graph.
    *   In the job log, scroll to the bottom. **Expected Result:** You must see a message like `Uploading artifacts...` followed by a list of the files being uploaded, including your `.jar` file.

3.  **Verify Artifact Download:**
    *   Navigate back to the pipeline graph and click on the `test-job`.
    *   In the job log, scroll to the top. **Expected Result:** You must see a message like `Downloading artifacts...`. This proves the job is receiving the build product from the previous stage.

4.  **Verify No Re-compilation:**
    *   In the `test-job` log, examine the output from the `mvn test` command. **Expected Result:** You should see output related to running tests, but you should **not** see the `[INFO] Compiling ...` messages that were present in your initial "flawed" pipeline run.

### **6. Discussion**

This lab demonstrated the critical difference between a `cache` and an `artifact`. The **cache** is for temporary, reusable data that speeds up your jobs, like downloaded dependencies. It is an optimization. The **artifact**, on the other hand, is the **intended output of your job**—the valuable product you have created, such as a compiled binary, a Docker image, or a test report.

By defining the `target` directory as an artifact in the `build-job`, you created a persistent payload. GitLab automatically uploaded this artifact to its storage. When the `test-job` in the next stage began, GitLab automatically downloaded and unpacked the artifact into the job's workspace. This powerful mechanism ensures **build integrity**: the `test-job` is guaranteed to be testing the *exact same binary code* that the `build-job` produced. This prevents a whole class of "works on my machine" errors and is a fundamental principle of reliable CI/CD pipelines.

### **7. Questions**

1.  What is the primary difference between the purpose of a `cache` and the purpose of an `artifact`?
2.  In our `build-job`, we set `expire_in: 1 hour`. What happens to the artifact after one hour? Why is this a good practice?
3.  The book mentions `artifacts:reports`. What is a common use case for `artifacts:reports:junit` in a Java/Maven project?
4.  If you wanted the `test-job` to run immediately after the `build-job` finished, without waiting for any other jobs in the `build` stage, what keyword would you use?
5.  Imagine a `deploy` stage after `test`. How would the `deploy-job` get access to the compiled `.jar` file created in the `build-job`?

---

### **8. Solution**

This section contains the completed artifacts for this lab.

#### **8.1. Final `.gitlab-ci.yml` File**
```yaml
# filename: .gitlab-ci.yml
image: maven:3.9-eclipse-temurin-17

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository"

cache:
  key: { files: [pom.xml] }
  paths: [".m2/repository"]

stages:
  - build
  - test

build-job:
  stage: build
  script:
    - echo "Packaging the application..."
    - mvn package
  artifacts:
    paths:
      - target/
    expire_in: 1 hour

test-job:
  stage: test
  script:
    - echo "Testing the packaged artifact..."
    - mvn test
```

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **`cache` vs. `artifact`?**
    A **cache** is for speeding up subsequent pipeline runs (e.g., dependencies you download). It is an optimization and not guaranteed to exist. An **artifact** is the valuable output of a job *within the current pipeline* (e.g., a binary you build). It is used to pass results between stages and is guaranteed to be available for the duration of the pipeline.

2.  **What does `expire_in` do?**
    It tells GitLab to automatically delete the artifact from storage after the specified time has passed. This is a critical best practice for managing storage costs and preventing old, temporary build products from accumulating indefinitely.

3.  **Use case for `artifacts:reports:junit`?**
    If your `mvn test` command is configured to produce a JUnit XML test report, you can declare it using `artifacts:reports:junit`. GitLab will then parse this report and display a summary of test results directly in the Merge Request UI, providing a quick, at-a-glance view of test successes and failures.

4.  **How to run a job immediately?**
    You would use the `needs` keyword in the `test-job`: `needs: [build-job]`. This would create a direct dependency, allowing `test-job` to start as soon as `build-job` finishes, without waiting for the entire `build` stage to complete.

5.  **How would the `deploy-job` get the `.jar` file?**
    Exactly the same way the `test-job` did. By default, artifacts created in an earlier stage are downloaded by **all jobs in all subsequent stages**. The `deploy-job` would automatically receive the `target/` directory and could then contain a `script` to upload the `.jar` file from that directory to a production environment.