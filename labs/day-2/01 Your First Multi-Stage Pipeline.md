# **Your First Multi-Stage Pipeline**

**Module:** 2.1: Introduction to GitLab CI/CD
**Time:** Approx. 30 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Create a new, standalone GitLab project for a lab exercise.
*   Define a multi-stage CI/CD pipeline using the `.gitlab-ci.yml` file.
*   Create jobs and assign them to specific stages.
*   Commit and push a new pipeline configuration to trigger its first run.
*   Navigate the GitLab UI to visualize the pipeline's execution and inspect job logs.

### **2. Scenario**

As a developer on a new project, your first task is to establish a basic automation workflow. The team has decided that every change should go through a consistent, three-step process: **Build**, **Test**, and **Deploy**. Before writing any application code, you will create the foundational CI/CD pipeline that defines this sequence.

Your goal is to create a `.gitlab-ci.yml` file that instructs any GitLab Runner on how to execute this workflow, ensuring that the `test` stage only runs after the `build` stage succeeds, and `deploy` only runs after `test` succeeds. For this initial setup, the jobs will simply print messages to the console to simulate real work.

### **3. Prerequisites**

*   You have a GitLab.com account and are a member of the `gabors-gitlab-training-20251013` group.
*   You have a fully configured local environment with Git and Visual Studio Code.
*   You are comfortable with the basic `git` commands from Day 1.

### **4. Steps**

_**Note:** This lab involves creating a new project in the GitLab UI, working locally in VS Code, and then returning to the GitLab UI to observe the results._

1.  **Create a New Project for the Lab**
    Each lab will be in its own repository to ensure a clean workspace.
    *   **[WHERE: GitLab UI - Browser]**
        *   Navigate to the `gabors-gitlab-training-20251013` group dashboard.
        *   Click **"New project"** and select **"Create blank project"**.
        *   **Project name:** Use the convention `student-<firstname>-<lastname>-lab-2.1`.
        *   Ensure the **Project URL** is within our training group.
        *   Check the box to **"Initialize repository with a README"**.
        *   Click **"Create project"**.

2.  **Clone the Project and Create a Branch**
    Now, set up your local workspace for the new pipeline.
    *   **[WHERE: Local Terminal / VS Code]**
        *   Clone your new project to your local machine using its SSH URL.
        *   Open the cloned project folder in VS Code.
        *   Create a new branch named `feature/initial-pipeline`.

3.  **Define the Pipeline Stages**
    The first step in any `.gitlab-ci.yml` file is to define the sequence of stages.
    *   **[WHERE: Visual Studio Code]**
        *   In the root of your project, create a new file named **`.gitlab-ci.yml`**.
        *   Add the following code to define the three stages of our workflow. The order is critical.
    ```yaml
    # filename: .gitlab-ci.yml
    stages:
      - build
      - test
      - deploy
    ```

4.  **Create the Build Job**
    Next, define the first job and assign it to the `build` stage.
    *   **[WHERE: Visual Studio Code]**
        *   In `.gitlab-ci.yml`, add the following code block. The job name (`build-job`) can be anything, but the `stage` must match one of the stages defined above.
    ```yaml
    # ... (stages definition)

    build-job:
      stage: build
      script:
        - echo "This is the build job. Simulating a build process..."
        - sleep 10 # Simulate a 10-second build time
        - echo "Build complete."
    ```

5.  **Create the Test and Deploy Jobs**
    Complete the pipeline by adding jobs for the remaining stages.
    *   **[WHERE: Visual Studio Code]**
        *   In `.gitlab-ci.yml`, add the final two job definitions.
    ```yaml
    # ... (build-job definition)

    test-job:
      stage: test
      script:
        - echo "This is the test job. Running automated tests..."
        - sleep 15 # Simulate a 15-second test run
        - echo "Tests passed."

    deploy-job:
      stage: deploy
      script:
        - echo "This is the deploy job. Deploying to a staging environment..."
        - sleep 5 # Simulate a quick deployment
        - echo "Deployment complete."
    ```

6.  **Commit and Push Your Pipeline**
    Now, you will push your `.gitlab-ci.yml` file to trigger the pipeline run.
    *   **[WHERE: Visual Studio Code]**
        *   Use the Source Control panel to stage your `.gitlab-ci.yml` file.
        *   Commit the change with the message `feat: add initial three-stage pipeline`.
        *   Push your `feature/initial-pipeline` branch to GitLab.

### **5. Verification**

1.  **Observe the Pipeline Graph:**
    *   **[WHERE: GitLab UI - Browser]**
        *   Navigate to your project. After pushing, create a Merge Request for your branch.
        *   In the MR or by navigating to `Build -> Pipelines`, click on the latest pipeline.
        *   **Expected Result:** You will see a visual graph with three columns: `build`, `test`, and `deploy`. The `build` stage will run first. Only after it succeeds will the `test` stage begin. Only after `test` succeeds will the `deploy` stage begin.

2.  **Inspect a Job Log:**
    *   In the pipeline graph view, click on the **`test-job`** bubble.
    *   **Expected Result:** You will see the job log, which includes the output of your `echo` and `sleep` commands. At the top of the log, you will see `Running with gitlab-runner ... on Azure Group Runner for Gabor's Training`, confirming it ran on our dedicated runner.

### **6. Discussion**

In this lab, you created your first piece of Infrastructure as Code: the `.gitlab-ci.yml` file. This file acts as a blueprint for your automation workflow. When you pushed your commit, GitLab detected this file and a **GitLab Runner** (our Azure VM) executed the instructions within it.

The most critical concept demonstrated here is the role of **stages**. They enforce a strict, sequential order of execution. The pipeline will automatically stop if any job in a stage fails, preventing a broken build from being tested or a failed test from being deployed. This creates a powerful, automated quality gate at every step of your delivery process.

### **7. Questions**

1.  In the pipeline graph, what do the different columns represent, and why is their order significant?
2.  What would happen to the pipeline's execution if the `sleep 10` command in the `build-job` were to fail (e.g., if the command was invalid)?
3.  If you added a second job, `code-analysis-job`, and also assigned it to the `build` stage, how would it run in relation to `build-job`?
4.  Looking at the job log for `test-job`, what was the first thing the runner did before executing your `script` section? (Hint: Look for `Pulling docker image...`).
5.  Why is it a best practice to use a tool like `sleep` to simulate long-running tasks in a simple pipeline like this, rather than just using `echo` commands?

---

### **8. Solution**

This section contains the completed artifacts for this lab.

#### **8.1. Final `.gitlab-ci.yml` File**
```yaml
# filename: .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

build-job:
  stage: build
  script:
    - echo "This is the build job. Simulating a build process..."
    - sleep 10 # Simulate a 10-second build time
    - echo "Build complete."

test-job:
  stage: test
  script:
    - echo "This is the test job. Running automated tests..."
    - sleep 15 # Simulate a 15-second test run
    - echo "Tests passed."

deploy-job:
  stage: deploy
  script:
    - echo "This is the deploy job. Deploying to a staging environment..."
    - sleep 5 # Simulate a quick deployment
    - echo "Deployment complete."
```

#### **8.2. Command Summary**
```bash
# Clone the repository (example)
git clone git@gitlab.com:gabors-gitlab-training-20251013/student-gabor-szabo-lab-2.1.git
cd student-gabor-szabo-lab-2.1

# Create branch, add pipeline file, commit, and push
git switch -c feature/initial-pipeline
# (Create and edit .gitlab-ci.yml in VS Code)
git add .gitlab-ci.yml
git commit -m "feat: add initial three-stage pipeline"
git push -u origin feature/initial-pipeline
```

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **What do the columns represent?**
    The columns represent the `stages` you defined in your `.gitlab-ci.yml` file. Their order is significant because it dictates the sequence of execution. All jobs in the `build` stage must succeed before any jobs in the `test` stage can begin.

2.  **What if the `build-job` failed?**
    The `build-job` would be marked as "Failed." The entire pipeline would immediately stop and be marked as "Failed" as well. The jobs in the `test` and `deploy` stages would be marked as "skipped" and would never run.

3.  **How would a second job in the same stage run?**
    Jobs within the same stage run **in parallel**, assuming there are enough available runners. GitLab would attempt to execute `build-job` and `code-analysis-job` at the same time to complete the `build` stage as quickly as possible.

4.  **What was the first thing the runner did?**
    Before running the script, the runner's first major action was to **pull a Docker image** (e.g., `Using Docker executor with image ruby:3.1...`). Every job runs inside a clean, ephemeral container, and the first step is always to prepare that containerized environment.

5.  **Why use `sleep`?**
    Using `sleep` simulates the **passage of time**, which is a crucial aspect of real-world pipelines. It makes the sequential execution of stages tangible; you can physically see the `test` stage waiting for the `build` stage to finish. This provides a much more realistic and intuitive demonstration of the pipeline's flow than instantaneous `echo` commands.