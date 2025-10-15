# **Caching Dependencies for Faster Pipelines**

**Module:** Optimizing Pipelines with Caching & Artifacts  
**Time:** Approx. 60 minutes

---

## **1. Objective(s)**

By the end of this lab, you will be able to:

*   Set up a complete Java and Maven development environment on Windows 10.
*   Create a Maven project from scratch using the command line.
*   Understand why dependency caching is critical for pipeline performance.
*   Configure a CI/CD pipeline to cache Maven dependencies using the `cache` keyword.
*   Use a `key:files` directive to create a smart cache that automatically invalidates when dependencies change.
*   Override a build tool's default directory to make it cacheable by GitLab.
*   Verify and measure the performance improvement of a cached pipeline run.

---

## **2. Scenario**

Your team's CI/CD pipeline for your Java/Maven project is working, but it's slow. Every time a pipeline runs—even for a small documentation change—the `build` job takes several minutes to re-download all of the project's dependencies from the internet. This slow feedback loop is killing productivity and wasting valuable CI/CD resources.

Your task is to build a Maven project from scratch and implement a caching strategy to optimize its pipeline. You will modify your `.gitlab-ci.yml` file to save the downloaded Maven dependencies (`.m2/repository`) after the first successful pipeline run. Subsequent runs will then be able to download a single compressed cache file from GitLab, dramatically speeding up the build process.

---

## **3. Prerequisites**

### **3.1. Required Pre-Installed Software**

Before starting this lab, ensure you have:
*   Windows 10 machine
*   Chocolatey package manager installed
*   Git installed
*   Visual Studio Code installed
*   A GitLab.com account and membership in the `gabors-gitlab-training-20251013` group

### **3.2. Install Java Development Kit (JDK) 17**

**[WHERE: Windows Command Prompt or PowerShell - Run as Administrator]**

1.  Open Command Prompt or PowerShell **as Administrator**.

2.  Install OpenJDK 17 using Chocolatey:
    ```cmd
    choco install openjdk17 -y
    ```

3.  Wait for the installation to complete. This may take a few minutes.

4.  Close and reopen your terminal to refresh environment variables.

5.  **Smoke Test - Verify Java Installation:**
    ```cmd
    java -version
    ```
    
    **Expected Output:**
    ```
    openjdk version "17.0.x" 2024-xx-xx
    OpenJDK Runtime Environment (build 17.0.x+x)
    OpenJDK 64-Bit Server VM (build 17.0.x+x, mixed mode, sharing)
    ```
    
    If you see version information starting with "17", Java is correctly installed.

6.  Verify the Java compiler:
    ```cmd
    javac -version
    ```
    
    **Expected Output:**
    ```
    javac 17.0.x
    ```

### **3.3. Install Apache Maven**

**[WHERE: Windows Command Prompt or PowerShell - Run as Administrator]**

1.  In your Administrator terminal, install Maven using Chocolatey:
    ```cmd
    choco install maven -y
    ```

2.  Wait for the installation to complete.

3.  Close and reopen your terminal to refresh environment variables.

4.  **Smoke Test - Verify Maven Installation:**
    ```cmd
    mvn -version
    ```
    
    **Expected Output:**
    ```
    Apache Maven 3.9.x (xxxxxxxxx)
    Maven home: C:\ProgramData\chocolatey\lib\maven\apache-maven-3.9.x
    Java version: 17.0.x, vendor: Oracle Corporation, runtime: C:\Program Files\OpenJDK\jdk-17.0.x
    Default locale: en_US, platform encoding: Cp1252
    OS name: "windows 10", version: "10.0", arch: "amd64", family: "windows"
    ```
    
    Verify that:
    *   Maven version is 3.9.x or higher
    *   Java version shown is 17.0.x
    *   Maven successfully detects your Java installation

### **3.4. Configure VS Code Extensions**

**[WHERE: Visual Studio Code]**

1.  Open Visual Studio Code.

2.  Click on the **Extensions** icon in the left sidebar (or press `Ctrl+Shift+X`).

3.  Install the following extensions by searching for each name and clicking **Install**:

    *   **Extension Pack for Java** (by Microsoft)
        *   This pack includes: Language Support for Java, Debugger for Java, Test Runner for Java, Maven for Java, Project Manager for Java, and Visual Studio IntelliCode
    
    *   **GitLab Workflow** (by GitLab)
        *   Provides GitLab integration for VS Code

4.  After installing, you may be prompted to reload VS Code. Click **Reload** if prompted.

5.  **Verify Extensions:**
    *   Open the Command Palette (`Ctrl+Shift+P`)
    *   Type "Java: Configure Java Runtime"
    *   If you see this command, the Java extensions are properly installed
    *   You should see JDK 17 listed in the runtime configuration

---

## **4. Steps**

### **Step 1: Create an Empty GitLab Repository**

**[WHERE: GitLab UI - Browser]**

1.  Navigate to the `gabors-gitlab-training-20251013` group on GitLab.com.

2.  Click **"New project"**.

3.  Select **"Create blank project"**.

4.  Configure your project:
    *   **Project name:** `student-<firstname>-<lastname>-lab-3.1`
    *   **Project URL:** Ensure it's under the `gabors-gitlab-training-20251013` group
    *   **Visibility Level:** Private (or as instructed)
    *   **Initialize repository with a README:** ✅ **Check this box**
    *   Leave other options as default

5.  Click **"Create project"**.

6.  Once created, copy the **clone URL** (HTTPS) from your project page.

---

### **Step 2: Clone the Repository Locally**

**[WHERE: Windows Command Prompt or PowerShell & Visual Studio Code]**

1.  Open a terminal (Command Prompt or PowerShell).

2.  Navigate to your workspace directory:
    ```cmd
    cd C:\Users\<YourUsername>\workspace
    ```
    *(Create this directory if it doesn't exist: `mkdir workspace`)*

3.  Clone your repository:
    ```cmd
    git clone <your-repository-url>
    ```
    
    Example:
    ```cmd
    git clone https://gitlab.com/gabors-gitlab-training-20251013/student-john-doe-lab-3.1.git
    ```

4.  Navigate into the cloned directory:
    ```cmd
    cd student-<firstname>-<lastname>-lab-3.1
    ```

5.  Open the project in VS Code:
    ```cmd
    code .
    ```

---

### **Step 3: Create a Maven Project Structure**

**[WHERE: Windows Command Prompt or PowerShell - In your project directory]**

1.  In your terminal, ensure you're in your project root directory.

2.  Create a Maven project using the quickstart archetype:
    ```cmd
    mvn archetype:generate -DgroupId=com.gitlab.training -DartifactId=cache-demo -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
    ```
    
    **What this does:**
    *   Creates a standard Maven project structure
    *   Sets the group ID to `com.gitlab.training`
    *   Sets the artifact ID to `cache-demo`
    *   Uses the quickstart archetype (simple Java application)
    *   Runs non-interactively with default settings

3.  Wait for Maven to download the archetype and create the project structure.

4.  Move the contents of the `cache-demo` subdirectory to your project root:
    ```cmd
    move cache-demo\* .
    move cache-demo\.gitignore . 2>nul
    rmdir /s /q cache-demo
    ```

5.  **Verify the project structure** in VS Code. You should now see:
    ```
    student-<firstname>-<lastname>-lab-3.1/
    ├── README.md
    ├── pom.xml
    └── src/
        ├── main/
        │   └── java/
        │       └── com/
        │           └── gitlab/
        │               └── training/
        │                   └── App.java
        └── test/
            └── java/
                └── com/
                    └── gitlab/
                        └── training/
                            └── AppTest.java
    ```

---

### **Step 4: Test the Project Locally**

**[WHERE: Windows Command Prompt or PowerShell]**

1.  Compile the project:
    ```cmd
    mvn compile
    ```
    
    **Expected Output:**
    *   Maven will download many dependencies (this will take 1-3 minutes on first run)
    *   You should see "BUILD SUCCESS" at the end
    *   Dependencies are downloaded to `C:\Users\<YourUsername>\.m2\repository`

2.  Run the tests:
    ```cmd
    mvn test
    ```
    
    **Expected Output:**
    *   Tests should run successfully
    *   You should see "BUILD SUCCESS"

3.  Run the application:
    ```cmd
    mvn exec:java -Dexec.mainClass="com.gitlab.training.App"
    ```
    
    **Expected Output:**
    ```
    Hello World!
    ```

**Congratulations!** You've successfully created and tested a Maven project locally.

---

### **Step 5: Push Your Project to GitLab**

**[WHERE: Windows Command Prompt or PowerShell & VS Code]**

1.  Create a `.gitignore` file in your project root with the following content:
    
    **[WHERE: Visual Studio Code]**
    
    Create `.gitignore`:
    ```gitignore
    # Maven
    target/
    pom.xml.tag
    pom.xml.releaseBackup
    pom.xml.versionsBackup
    pom.xml.next
    release.properties
    dependency-reduced-pom.xml
    buildNumber.properties
    .mvn/timing.properties
    .mvn/wrapper/maven-wrapper.jar
    
    # IDE
    .idea/
    .vscode/
    *.iml
    *.iws
    *.ipr
    
    # OS
    .DS_Store
    Thumbs.db
    ```

2.  **[WHERE: Terminal]** Add, commit, and push your changes:
    ```cmd
    git add .
    git commit -m "Initial Maven project setup"
    git push origin main
    ```

3.  Verify in the GitLab UI that your project files are now visible in the repository.

---

### **Step 6: Create the Initial (Uncached) Pipeline**

First, you will create a simple build pipeline without any caching to establish a performance baseline.

**[WHERE: Visual Studio Code]**

1.  Create a new branch:
    ```cmd
    git checkout -b feature/implement-caching
    ```

2.  In the root of the project, create a new file named **`.gitlab-ci.yml`**.

3.  Add the following configuration:
    ```yaml
    # filename: .gitlab-ci.yml
    image: maven:3.9-eclipse-temurin-17 # Use a Docker image with Maven and Java 17
    
    build-job:
      stage: build
      script:
        - echo "Compiling the project... This will download all dependencies."
        - mvn compile
    ```

4.  Save the file.

5.  Commit and push:
    ```cmd
    git add .gitlab-ci.yml
    git commit -m "ci: add initial uncached pipeline"
    git push origin feature/implement-caching
    ```

---

### **Step 7: Run the Uncached Pipeline and Record the Time**

**[WHERE: GitLab UI - Browser]**

1.  Navigate to your project on GitLab.

2.  Click **"Merge requests"** in the left sidebar.

3.  Click **"Create merge request"** for your `feature/implement-caching` branch.

4.  Click **"Create merge request"** again (you can leave the default title and description).

5.  Navigate to the **Pipelines** tab in your merge request.

6.  Click on the pipeline to view it, then click on the **`build-job`**.

7.  **Observe:** Watch the job log as Maven downloads numerous `.jar` files from Maven Central:
    ```
    Downloading from central: https://repo.maven.apache.org/maven2/...
    Downloading from central: https://repo.maven.apache.org/maven2/...
    ...
    ```

8.  **Record:** Once the job succeeds, note the **Job duration** displayed in the top right of the job page (e.g., "2 minutes 30 seconds"). Write this down—this is your baseline.

    **Baseline Duration: _____ minutes _____ seconds**

---

### **Step 8: Implement the Cache Configuration**

Now, you will add the `cache` block to your pipeline. This involves two key parts: telling Maven where to put the dependencies and telling GitLab what to cache.

**[WHERE: Visual Studio Code]**

1.  Open your `.gitlab-ci.yml` file.

2.  Modify it to add the `variables` and `cache` sections:
    ```yaml
    # filename: .gitlab-ci.yml
    image: maven:3.9-eclipse-temurin-17
    
    variables:
      # Tell Maven to use a project-local directory for its repository.
      # This is CRITICAL for GitLab's cache to work.
      MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository"
    
    cache:
      # Define a smart cache key. The cache will be invalidated and rebuilt
      # only if the pom.xml file changes.
      key:
        files:
          - pom.xml
      # Tell GitLab which directory to save.
      paths:
        - .m2/repository
    
    build-job:
      stage: build
      script:
        - echo "Compiling the project... This run should create the cache."
        - mvn compile
    ```

3.  Save the file.

4.  Commit and push:
    ```cmd
    git add .gitlab-ci.yml
    git commit -m "ci: add maven dependency cache"
    git push origin feature/implement-caching
    ```

---

### **Step 9: Run the "Cache Creation" Pipeline**

This second pipeline run will be slow again, but this time it will create and upload the cache.

**[WHERE: GitLab UI - Browser]**

1.  Go to your Merge Request and view the new pipeline that was automatically triggered.

2.  Click on the `build-job`.

3.  Inspect the log. You'll still see Maven downloading dependencies (because the cache doesn't exist yet).

4.  **At the end of the log**, you will see a new section: **"Saving cache for successful job"**:
    ```
    Saving cache for successful job
    Creating cache maven-dependencies...
    .m2/repository: found 1234 matching files
    Uploading cache.zip to https://storage.googleapis.com/...
    Created cache
    ```

5.  This confirms GitLab has successfully created and uploaded your `.m2/repository` cache.

6.  Note the duration of this job. It should be similar to the first run.

    **Cache Creation Duration: _____ minutes _____ seconds**

---

### **Step 10: Trigger the Cached Pipeline Run**

The final step is to run the pipeline one more time to see the cache in action.

**[WHERE: GitLab UI - Browser]**

1.  In your pipeline view (or merge request page), click the **"Run pipeline"** button (or click the **↻** icon).

2.  In the "Run pipeline" dialog, ensure the branch is `feature/implement-caching`, then click **"Run pipeline"**.

3.  Navigate to the `build-job` for this new, third pipeline.

---

## **5. Verification**

**[WHERE: GitLab UI - Browser - Viewing the third pipeline's build-job log]**

1.  **Verify Cache is Being Used:** 
    
    At the **very top** of the job log, you will see:
    ```
    Restoring cache
    Checking cache for maven-dependencies...
    Downloading cache.zip from https://storage.googleapis.com/...
    Successfully extracted cache
    ```
    
    This confirms the job is downloading and using the cache.

2.  **Verify No Downloads:** 
    
    Scroll through the output of the `mvn compile` command. It should **not** show any "Downloading from central..." messages. Maven will find all dependencies in the restored cache and simply use them.

3.  **Measure the Performance Improvement:** 
    
    Note the **Job duration** for this third pipeline run in the top right corner.
    
    **Cached Pipeline Duration: _____ seconds**
    
    It should be dramatically shorter than your first two runs (e.g., under 30 seconds instead of over 2 minutes).

4.  **Calculate Improvement:**
    ```
    Time Saved = Baseline Duration - Cached Duration
    Percentage Improvement = (Time Saved / Baseline Duration) × 100%
    ```

---

## **6. Discussion**

This lab demonstrated one of the most important techniques for optimizing CI/CD pipelines: **dependency caching**. The first pipeline run was slow because the GitLab Runner, running in a clean Docker container, had to download every required `.jar` file from the internet.

By implementing the `cache` keyword, you instructed the runner to save the dependency directory (`.m2/repository`) upon job completion. The critical step was using the `MAVEN_OPTS` variable to force Maven to use a project-local directory that GitLab could access.

On the final run, the runner downloaded the single, compressed cache file, which is significantly faster than hundreds of individual HTTP requests. The `key:files` directive is a powerful best practice that ensures your cache is "smart." It will be automatically invalidated and rebuilt if and only if you change your `pom.xml`, ensuring you always have the correct dependencies without slowing down every single run.

You also learned how to create a Maven project from scratch, understand its structure, and test it locally before pushing to GitLab—essential skills for any Java developer working in a CI/CD environment.

---

## **7. Questions**

1.  Why is it necessary to use the `MAVEN_OPTS` variable to override `maven.repo.local`? What would happen if you tried to cache the default `~/.m2/repository` directory?

2.  What is the purpose of the `key:files: - pom.xml` directive? What would be the downside of using a simpler, static key like `key: maven-dependencies`?

3.  If you change a single line in a `.java` source file (but not `pom.xml`) and push the change, will the pipeline use the cache or rebuild it? Why?

4.  The book mentions a `policy` for caches. What is the difference between a `pull-push` policy (the default) and a `pull` policy? In what kind of job might you use a `pull` policy?

5.  How is the GitLab `cache` different from a GitLab `artifact`? When would you choose one over the other?

6.  What Maven command did we use to create the initial project structure? What does the `-DarchetypeArtifactId=maven-archetype-quickstart` parameter specify?

---

## **8. Solution**

This section contains the completed artifacts for this lab.

### **8.1. Final `.gitlab-ci.yml` File**

```yaml
# filename: .gitlab-ci.yml
image: maven:3.9-eclipse-temurin-17

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository"

cache:
  key:
    files:
      - pom.xml
  paths:
    - .m2/repository

build-job:
  stage: build
  script:
    - echo "Compiling the project..."
    - mvn compile
```

### **8.2. Project Structure**

```
student-<firstname>-<lastname>-lab-3.1/
├── .gitignore
├── .gitlab-ci.yml
├── README.md
├── pom.xml
└── src/
    ├── main/
    │   └── java/
    │       └── com/
    │           └── gitlab/
    │               └── training/
    │                   └── App.java
    └── test/
        └── java/
            └── com/
                └── gitlab/
                    └── training/
                        └── AppTest.java
```

---

## **9. Answers**

This section contains the detailed answers to the questions.

### **9.1. Answers to Questions**

1.  **Why override `maven.repo.local`?**
    
    GitLab Runners can only cache files and directories that are within the project's working directory (`$CI_PROJECT_DIR`). The default Maven repository (`~/.m2/repository`) is located in the user's home directory, which is outside the project's scope and therefore cannot be cached by GitLab. Overriding the location to a project-relative path (`.m2/repository`) is a mandatory step for caching to work.

2.  **Purpose of `key:files:`?**
    
    It creates a "smart" cache key based on file content. GitLab calculates a hash of the contents of `pom.xml`. The cache is only uploaded/downloaded if a cache with that specific hash key exists. If you change a dependency in `pom.xml`, the hash changes, a new cache is created, and the old one is ignored. A static key like `maven-dependencies` would never invalidate, and you would have to manually clear the cache every time you changed a dependency, potentially causing builds to fail with outdated dependencies.

3.  **If you change a `.java` file, will the cache be used?**
    
    Yes, the cache **will be used**. The cache key is based only on `pom.xml`. Since the `.java` file change does not alter `pom.xml`, the cache key remains the same, and the runner will successfully download and use the existing dependency cache, resulting in a fast build. This is the desired behavior—you only want to re-download dependencies when they actually change.

4.  **`pull-push` vs. `pull` policy?**
    
    `pull-push` (the default) means the job will download the cache at the start and upload any changes to it at the end. `pull` means the job will only download the cache; it will not re-upload it, even if the contents change. You would use a `pull` policy in a job that *consumes* dependencies but does not *download* any new ones, such as a `test` or `deploy` job. This saves time by skipping the cache upload step when it's unnecessary.

5.  **`cache` vs. `artifact`?**
    
    A **cache** is for temporary data used to speed up jobs (e.g., dependencies). It is an optimization and may be deleted by GitLab to free up storage space. An **artifact** is the permanent, intended output of a job (e.g., a compiled binary, a test report, documentation). Artifacts are used to pass results between stages and are guaranteed to be available for download. You use a cache for things you download from external sources and an artifact for things your pipeline builds or generates.

6.  **Maven archetype command?**
    
    We used `mvn archetype:generate` with the `-DarchetypeArtifactId=maven-archetype-quickstart` parameter. This parameter specifies which project template (archetype) to use. The `maven-archetype-quickstart` is a simple archetype that creates a basic Java application with a standard Maven directory structure, a sample `App.java` class, and a sample unit test. Other archetypes exist for web applications, libraries, and other project types.