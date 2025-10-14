# **Caching Dependencies for Faster Pipelines**

**Module:** Optimizing Pipelines with Caching & Artifacts
**Time:** Approx. 40 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Understand why dependency caching is critical for pipeline performance.
*   Configure a CI/CD pipeline to cache Maven dependencies using the `cache` keyword.
*   Use a `key:files` directive to create a smart cache that automatically invalidates when dependencies change.
*   Override a build tool's default directory to make it cacheable by GitLab.
*   Verify and measure the performance improvement of a cached pipeline run.

### **2. Scenario**

Your team's CI/CD pipeline for your Java/Maven project is working, but it's slow. Every time a pipeline runs—even for a small documentation change—the `build` job takes several minutes to re-download all of the project's dependencies from the internet. This slow feedback loop is killing productivity and wasting valuable CI/CD resources.

Your task is to implement a caching strategy to fix this. You will modify your `.gitlab-ci.yml` file to save the downloaded Maven dependencies (`.m2/repository`) after the first successful pipeline run. Subsequent runs will then be able to download a single compressed cache file from GitLab, dramatically speeding up the build process.

### **3. Prerequisites**

*   A GitLab.com account and membership in the `gabors-gitlab-training-20251013` group.
*   A fully configured local environment with Git, Java 17, Maven, and VS Code.
*   We will provide you with a starter project containing a standard Maven boilerplate application.

### **4. Steps**

_**Note:** This lab will guide you through creating a new project from a provided starter, running a slow uncached pipeline, and then running a fast cached pipeline to observe the difference._

1.  **Create and Clone the Lab Project**
    *   **[WHERE: GitLab UI - Browser & Local Terminal]**
        *   In the `gabors-gitlab-training-20251013` group, create a new project named `student-<firstname>-<lastname>-lab-3.1`.
        *   **Crucially, select "Import project"** and use the **"Repository by URL"** option.
        *   For the "Git repository URL," use the URL for our starter project that we will provide: `https://gitlab.com/gabor-szabo-iqsoft/java-maven-starter.git`.
        *   Click **"Create project"**.
        *   Clone your new project to your local machine and open it in VS Code.

2.  **Create the Initial (Uncached) Pipeline**
    First, you will create a simple build pipeline without any caching to establish a performance baseline.
    *   **[WHERE: Visual Studio Code]**
        *   Create a new branch named `feature/implement-caching`.
        *   In the root of the project, create a new file named **`.gitlab-ci.yml`**.
        *   Add the following configuration:
    ```yaml
    # filename: .gitlab-ci.yml
    image: maven:3.9-eclipse-temurin-17 # Use a Docker image with Maven and Java 17

    build-job:
      stage: build
      script:
        - echo "Compiling the project... This will download all dependencies."
        - mvn compile
    ```
    *   Commit this file with the message `ci: add initial uncached pipeline` and push your branch.

3.  **Run the Uncached Pipeline and Record the Time**
    *   **[WHERE: GitLab UI - Browser]**
        *   Create a Merge Request for your branch.
        *   Navigate to the pipeline and click on the `build-job`.
        *   **Observe:** Watch the job log as Maven downloads numerous `.jar` files from the central repository.
        *   **Record:** Once the job succeeds, note the **Job duration** displayed in the GitLab UI (e.g., "2 minutes 30 seconds"). This is your baseline.

4.  **Implement the Cache Configuration**
    Now, you will add the `cache` block to your pipeline. This involves two key parts: telling Maven where to put the dependencies and telling GitLab what to cache.
    *   **[WHERE: Visual Studio Code]**
        *   Modify your `.gitlab-ci.yml` file to add the `variables` and `cache` sections.
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
    *   Commit this change with the message `ci: add maven dependency cache` and push to your branch.

5.  **Run the "Cache Creation" Pipeline**
    This second pipeline run will be slow again, but this time it will create and upload the cache.
    *   **[WHERE: GitLab UI - Browser]**
        *   Go to your Merge Request and view the new pipeline.
        *   Inspect the log for the `build-job`. At the **end of the log**, you will see a new section: **"Saving cache"**. This confirms GitLab has successfully created and uploaded your `.m2/repository` cache.
        *   Note the duration of this job. It should be similar to the first run.

6.  **Trigger the Cached Pipeline Run**
    The final step is to run the pipeline one more time to see the cache in action.
    *   **[WHERE: GitLab UI - Browser]**
        *   In your pipeline view, click the **"Run pipeline"** button in the top right, and then **"Run pipeline"** again on the next page to manually trigger a new run on your feature branch.
    *   Navigate to the `build-job` for this new, third pipeline.

### **5. Verification**

1.  **Verify Cache is Being Used:** In the job log of your **third** pipeline run, you will see a new section at the **very top** of the log: **"Restoring cache"**. This confirms the job is downloading and using the cache.
2.  **Verify No Downloads:** Observe the output of the `mvn compile` command. It should **not** show any "Downloading from central..." messages. It will find all dependencies in the restored cache.
3.  **Measure the Performance Improvement:** Note the **Job duration** for this third pipeline run. It should be dramatically shorter than your first two runs (e.g., under 30 seconds instead of over 2 minutes).

### **6. Discussion**

This lab demonstrated one of the most important techniques for optimizing CI/CD pipelines: **dependency caching**. The first pipeline run was slow because the GitLab Runner, running in a clean Docker container, had to download every required `.jar` file from the internet.

By implementing the `cache` keyword, you instructed the runner to save the dependency directory (`.m2/repository`) upon job completion. The critical step was using the `MAVEN_OPTS` variable to force Maven to use a project-local directory that GitLab could access.

On the final run, the runner downloaded the single, compressed cache file, which is significantly faster than hundreds of individual HTTP requests. The `key:files` directive is a powerful best practice that ensures your cache is "smart." It will be automatically invalidated and rebuilt if and only if you change your `pom.xml`, ensuring you always have the correct dependencies without slowing down every single run.

### **7. Questions**

1.  Why is it necessary to use the `MAVEN_OPTS` variable to override `maven.repo.local`? What would happen if you tried to cache the default `~/.m2/repository` directory?
2.  What is the purpose of the `key:files: - pom.xml` directive? What would be the downside of using a simpler, static key like `key: maven-dependencies`?
3.  If you change a single line in a `.java` source file (but not `pom.xml`) and push the change, will the pipeline use the cache or rebuild it? Why?
4.  The book mentions a `policy` for caches. What is the difference between a `pull-push` policy (the default) and a `pull` policy? In what kind of job might you use a `pull` policy?
5.  How is the GitLab `cache` different from a GitLab `artifact`? When would you choose one over the other?

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

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **Why override `maven.repo.local`?**
    GitLab Runners can only cache files and directories that are within the project's working directory (`$CI_PROJECT_DIR`). The default Maven repository (`~/.m2/repository`) is located in the user's home directory, which is outside the project's scope and therefore cannot be cached by GitLab. Overriding the location is a mandatory step.

2.  **Purpose of `key:files:`?**
    It creates a "smart" cache key. GitLab calculates a hash of the contents of `pom.xml`. The cache is only uploaded/downloaded if a cache with that specific hash key exists. If you change a dependency in `pom.xml`, the hash changes, a new cache is created, and the old one is ignored. A static key like `maven-dependencies` would never invalidate, and you would have to manually clear the cache every time you changed a dependency.

3.  **If you change a `.java` file, will the cache be used?**
    Yes, the cache **will be used**. The cache key is based only on `pom.xml`. Since the `.java` file change does not alter `pom.xml`, the cache key remains the same, and the runner will successfully download and use the existing dependency cache, resulting in a fast build.

4.  **`pull-push` vs. `pull` policy?**
    `pull-push` (the default) means the job will download the cache at the start and upload any changes to it at the end. `pull` means the job will only download the cache; it will not re-upload it, even if the contents change. You would use a `pull` policy in a job that *consumes* dependencies but does not *download* any new ones, such as a `test` or `deploy` job.

5.  **`cache` vs. `artifact`?**
    A **cache** is for temporary data used to speed up jobs (e.g., dependencies). It is an optimization and may be deleted by GitLab. An **artifact** is the permanent, intended output of a job (e.g., a compiled binary, a test report). Artifacts are used to pass results between stages and are guaranteed to be available. You use a cache for things you download and an artifact for things you build.