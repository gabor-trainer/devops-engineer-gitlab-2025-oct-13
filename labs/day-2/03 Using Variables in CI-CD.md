# **Lab: Using Variables in CI/CD**

**Module:** CI/CD Variables & Environment Management
**Time:** Approx. 30 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Define and use custom CI/CD variables in your `.gitlab-ci.yml` file.
*   Utilize GitLab's predefined variables to access contextual information about the pipeline run.
*   Understand the syntax for using variables within the `script` section of a job.
*   Override a globally defined variable at the job level.

### **2. Scenario**

Your team's basic three-stage pipeline is working, but it's completely static. To make it more useful, you need to introduce configuration parameters that can be easily changed without modifying the core script logic. Additionally, for debugging and traceability, you want every job log to clearly state the context in which it was run, such as the branch name and the specific commit being processed.

Your task is to modify your existing `.gitlab-ci.yml` file to use CI/CD variables. You will define a custom variable to control a greeting message and then use several of GitLab's powerful, built-in predefined variables to print a dynamic, contextual header at the start of each job.

### **3. Prerequisites**

*   You have completed **Lab 2.1** and have a project with a working, three-stage `.gitlab-ci.yml` file.
*   You are comfortable with the basic Git workflow (clone, branch, commit, push) from Day 1.

### **4. Steps**

_**Note:** This lab involves creating a new project, cloning it, and then making a series of incremental changes to the `.gitlab-ci.yml` file._

1.  **Create and Clone a New Project for the Lab**
    *   **[WHERE: GitLab UI - Browser & Local Terminal]**
        *   In the `gabors-gitlab-training-20251013` group, create a new blank project named `student-<firstname>-<lastname>-lab-2.3`.
        *   Initialize it with a `README.md` file.
        *   Clone this new project to your local machine and open it in VS Code.
        *   In VS Code, create a new branch named `feature/pipeline-variables`.

2.  **Create the Initial Pipeline**
    To start, you will add the three-stage pipeline from Lab 2.1.
    *   **[WHERE: Visual Studio Code]**
        *   Create a new file named `.gitlab-ci.yml`.
        *   Add the following basic pipeline configuration:
    ```yaml
    # filename: .gitlab-ci.yml
    stages:
      - build
      - test

    build-job:
      stage: build
      script:
        - echo "Running the build job..."

    test-job:
      stage: test
      script:
        - echo "Running the test job..."
    ```

3.  **Define and Use a Global Custom Variable**
    Now, you will add a `variables` block at the top level to define a variable that will be available to all jobs.
    *   **[WHERE: Visual Studio Code]**
        *   Modify your `.gitlab-ci.yml` file to add a `variables` section and use the new variable in your jobs.
    ```yaml
    # filename: .gitlab-ci.yml
    variables:
      GREETING: "Hello from a global variable!"

    stages:
      - build
      - test

    build-job:
      stage: build
      script:
        - echo "This is the build job."
        - echo $GREETING # Use the variable

    test-job:
      stage: test
      script:
        - echo "This is the test job."
        - echo $GREETING # Use the same variable
    ```

4.  **Use Predefined Variables for Context**
    Next, you will enhance the jobs to print a dynamic header using GitLab's powerful, built-in variables.
    *   **[WHERE: Visual Studio Code]**
        *   Modify the `script` section of both `build-job` and `test-job`.
    ```yaml
    # ... inside build-job:
    script:
      - echo "--- Job Information ---"
      - echo "Job running on branch: $CI_COMMIT_BRANCH"
      - echo "Commit SHA: $CI_COMMIT_SHORT_SHA"
      - echo "Triggered by: $GITLAB_USER_NAME"
      - echo "-----------------------"
      - echo "This is the build job."
      - echo $GREETING

    # ... inside test-job:
    script:
      - echo "--- Job Information ---"
      - echo "Job running on branch: $CI_COMMIT_BRANCH"
      - echo "Commit SHA: $CI_COMMIT_SHORT_SHA"
      - echo "Triggered by: $GITLAB_USER_NAME"
      - echo "-----------------------"
      - echo "This is the test job."
      - echo $GREETING
    ```

5.  **Override a Variable at the Job Level**
    Finally, you will demonstrate variable precedence by overriding the global `GREETING` variable specifically for the `test-job`.
    *   **[WHERE: Visual Studio Code]**
        *   Add a `variables` block directly inside the `test-job`. This local variable will take precedence over the global one.
    ```yaml
    # ... (build-job definition)

    test-job:
      stage: test
      variables:
        GREETING: "Hello from a job-specific override!" # This will override the global value
      script:
        - echo "--- Job Information ---"
        # ... (rest of the script is unchanged)
        - echo "This is the test job."
        - echo $GREETING # This will now print the new message
    ```

6.  **Commit and Push Your Changes**
    *   **[WHERE: Visual Studio Code]**
        *   Use the Source Control panel to stage your `.gitlab-ci.yml` file.
        *   Commit the change with the message `feat: add and use CI/CD variables`.
        *   Push your `feature/pipeline-variables` branch to GitLab.

### **5. Verification**

1.  **Observe the Pipeline:**
    *   **[WHERE: GitLab UI - Browser]**
        *   Create a Merge Request for your new branch.
        *   Navigate to the running pipeline and inspect the job logs.

2.  **Verify `build-job` Log:**
    *   **Expected Result:** The log for `build-job` should display the contextual header with your branch name, commit SHA, and username. It must show the **global** greeting message: `Hello from a global variable!`.

3.  **Verify `test-job` Log:**
    *   **Expected Result:** The log for `test-job` should display the same contextual header. However, it must show the **overridden** greeting message: `Hello from a job-specific override!`.

### **6. Discussion**

This lab demonstrated the power and flexibility of CI/CD variables in GitLab. Variables are the primary mechanism for parameterizing your pipelines, making them configurable and dynamic.

We explored two key types:
1.  **Custom Variables:** You defined a `GREETING` variable and saw how a value set at the top level (`variables:`) is inherited by all jobs. You also demonstrated **variable precedence**, where a variable defined at the job level overrides a global variable with the same name.
2.  **Predefined Variables:** GitLab automatically injects a rich set of contextual variables into every job. By using variables like `$CI_COMMIT_BRANCH` and `$GITLAB_USER_NAME`, you can create scripts that adapt to their environment, providing invaluable information for logging, debugging, and traceability.

Mastering variables is essential for moving beyond simple, static pipelines and creating the sophisticated, dynamic automation required in an enterprise DevOps environment.

### **7. Questions**

1.  What is the primary difference between a predefined variable (like `$CI_COMMIT_BRANCH`) and a custom variable you define yourself?
2.  In the `script` block, we used `$GREETING`. What would have been printed if we had used `'$GREETING'` (with single quotes) instead?
3.  The book mentions "masked" variables. What is the purpose of masking a variable, and where would you configure it?
4.  How could you define a CI/CD variable that is available to all jobs in your project but is **not** written in the `.gitlab-ci.yml` file?
5.  Imagine you want a job to behave differently in a Merge Request pipeline versus a regular branch pipeline. Which predefined variable would be most useful for a `rules` block to detect this?

---

### **8. Solution**

This section contains the completed artifacts for this lab.

#### **8.1. Final `.gitlab-ci.yml` File**```yaml
# filename: .gitlab-ci.yml
variables:
  GREETING: "Hello from a global variable!"

stages:
  - build
  - test

build-job:
  stage: build
  script:
    - echo "--- Job Information ---"
    - echo "Job running on branch: $CI_COMMIT_BRANCH"
    - echo "Commit SHA: $CI_COMMIT_SHORT_SHA"
    - echo "Triggered by: $GITLAB_USER_NAME"
    - echo "-----------------------"
    - echo "This is the build job."
    - echo $GREETING

test-job:
  stage: test
  variables:
    GREETING: "Hello from a job-specific override!"
  script:
    - echo "--- Job Information ---"
    - echo "Job running on branch: $CI_COMMIT_BRANCH"
    - echo "Commit SHA: $CI_COMMIT_SHORT_SHA"
    - echo "Triggered by: $GITLAB_USER_NAME"
    - echo "-----------------------"
    - echo "This is the test job."
    - echo $GREETING
```

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **Predefined vs. Custom Variables?**
    Predefined variables are automatically injected and populated by the GitLab Runner with contextual information about the job's environment (e.g., branch name, user). Custom variables are user-defined and are used to pass static configuration values or parameters into the pipeline.

2.  **`$GREETING` vs. `'$GREETING'`?**
    In a shell script, double quotes (`"`) allow for variable expansion, while single quotes (`'`) do not. If you used `echo '$GREETING'`, the shell would literally print the string `$GREETING` to the log instead of its value.

3.  **What is a "masked" variable?**
    A masked variable is a security feature for protecting secrets. When you define a variable as "masked" in the GitLab UI, its value will be automatically redacted and replaced with `[MASKED]` in all job logs. You configure this in the project or group's `Settings -> CI/CD -> Variables` section.

4.  **How to define a variable outside of YAML?**
    You can define a CI/CD variable in the GitLab UI at either the **Project** or **Group** level (`Settings -> CI/CD -> Variables`). Variables defined here are injected into every pipeline and are the standard way to store secrets and environment-specific configuration that should not be committed to the repository.

5.  **Which variable detects an MR pipeline?**
    The `$CI_PIPELINE_SOURCE` predefined variable is the most useful. It will have the value `merge_request_event` for a pipeline running in the context of an MR, and a value like `push` for a standard branch pipeline.