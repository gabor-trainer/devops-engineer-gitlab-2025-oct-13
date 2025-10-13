# **Lab: Conditional Job Execution with `rules`**

**Module:** Advanced Pipeline Control with `rules`
**Time:** Approx. 30 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Write a CI/CD `rules` block to control job execution.
*   Use the `$CI_COMMIT_BRANCH` predefined variable to create a branch-specific rule.
*   Configure a job to run *only* on the `main` branch.
*   Verify that a job is correctly skipped on a feature branch.
*   Understand how `rules` create dynamic pipelines that adapt to their context.

### **2. Scenario**

Your team has a pipeline that builds and tests your application, but it also includes a `deploy` job that runs on every single commit, on every single branch. This is inefficient and dangerous; you must never deploy code from a feature branch directly to a production or staging environment. Deployments should be a controlled process that only happens from a stable, protected branch like `main`.

Your task is to modify the `.gitlab-ci.yml` file to make the pipeline "smarter." You will implement a `rules` block that adds a conditional gate to the `deploy-job`, ensuring it is **only** included in the pipeline when a change is committed to the `main` branch.

### **3. Prerequisites**

*   You have a GitLab.com account and are a member of the `gabors-gitlab-training-20251013` group.
*   You have a fully configured local environment with Git and Visual Studio Code.
*   You are comfortable with the basic Git workflow and `.gitlab-ci.yml` syntax from previous labs.

### **4. Steps**

_**Note:** This lab will require you to run two separate pipelines—one on a feature branch and one after merging to `main`—to observe the different behaviors._

1.  **Create and Clone a New Project for the Lab**
    *   **[WHERE: GitLab UI - Browser & Local Terminal]**
        *   In the `gabors-gitlab-training-20251013` group, create a new blank project named `student-<firstname>-<lastname>-lab-2.4`.
        *   Initialize it with a `README.md` file.
        *   Clone this new project to your local machine and open it in VS Code.

2.  **Create the Initial "Flawed" Pipeline**
    First, you will create a simple pipeline where the `deploy-job` runs on every branch.
    *   **[WHERE: Visual Studio Code]**
        *   In the root of your project, create a new file named **`.gitlab-ci.yml`**.
        *   Add the following configuration. Note that there are no `rules` yet.
    ```yaml
    # filename: .gitlab-ci.yml
    stages:
      - test
      - deploy

    test-job:
      stage: test
      script:
        - echo "Running tests on branch: $CI_COMMIT_BRANCH"

    deploy-job:
      stage: deploy
      script:
        - echo "Deploying to production from branch: $CI_COMMIT_BRANCH"
    ```

3.  **Add the `rules` Block to the Deploy Job**
    Now, you will add the conditional logic to restrict the `deploy-job`.
    *   **[WHERE: Visual Studio Code]**
        *   Modify the `deploy-job` in your `.gitlab-ci.yml` file to include a `rules` block. This rule tells GitLab to only include this job if the commit is on the `main` branch.
    ```yaml
    # filename: .gitlab-ci.yml
    # ... (test-job definition)

    deploy-job:
      stage: deploy
      rules:
        - if: '$CI_COMMIT_BRANCH == "main"' # This is the critical change
      script:
        - echo "Deploying to production from branch: $CI_COMMIT_BRANCH"
    ```

4.  **Test the Rule on a Feature Branch**
    You will now commit and push this change on a new branch to verify that the `deploy-job` is correctly skipped.
    *   **[WHERE: Visual Studio Code]**
        *   Create a new branch named `feature/restrict-deploy-job`.
        *   Use the Source Control panel to stage and commit your updated `.gitlab-ci.yml` file with the message `ci: restrict deploy job to main branch`.
        *   Push your `feature/restrict-deploy-job` branch to GitLab.

5.  **Verify the Feature Branch Pipeline**
    *   **[WHERE: GitLab UI - Browser]**
        *   Create a Merge Request for your new branch.
        *   Navigate to the pipeline that was triggered on the `feature/restrict-deploy-job` branch.

6.  **Test the Rule on the `main` Branch**
    The final step is to merge your change and verify that the `deploy-job` runs correctly on `main`.
    *   **[WHERE: GitLab UI - Browser]**
        *   On your open Merge Request, click the **"Merge"** button.
        *   This action will trigger a new pipeline on the `main` branch.
        *   Navigate to this new pipeline (you can find it under `Build -> Pipelines`).

### **5. Verification**

1.  **Verify the Feature Branch Pipeline:**
    *   The pipeline graph for the `feature/restrict-deploy-job` branch **must only show the `test` stage**. The `deploy` stage should be completely absent, and the `deploy-job` should not appear in the "Jobs" tab. The pipeline should pass after `test-job` succeeds.

2.  **Verify the `main` Branch Pipeline:**
    *   The pipeline graph for the `main` branch **must show both the `test` and `deploy` stages**.
    *   Inspect the log for the `deploy-job`. It should have executed successfully and printed the message: `Deploying to production from branch: main`.

### **6. Discussion**

This lab demonstrated one of the most powerful and essential features of GitLab CI/CD: the `rules` keyword. By adding a simple rule, you transformed a static, "dumb" pipeline into a dynamic, context-aware workflow.

The core of the solution was the conditional statement `if: '$CI_COMMIT_BRANCH == "main"'`. This tells the GitLab Runner to evaluate the predefined `$CI_COMMIT_BRANCH` variable at the start of a pipeline. If the condition is true, the job is added to the pipeline. If it is false (as it was on our feature branch), the job is completely excluded.

This is the fundamental pattern for implementing **Continuous Delivery**. It allows developers to get rapid feedback from test jobs on their feature branches, while strictly gating the high-stakes deployment jobs to a single, protected source of truth—the `main` branch.

### **7. Questions**

1.  What would you need to change in the `rules` block to make the `deploy-job` run on *every* branch *except* `main`?
2.  Imagine you want a `review-job` to run only when a change is part of a Merge Request. Which predefined variable would you check in a `rules:if` statement?
3.  A `rules` block can also contain a `when` keyword (e.g., `when: manual`). What is the difference between a rule that is not met (and the job is skipped) versus a rule with `when: manual`?
4.  If a job has no `rules` block at all, under what conditions does it run?
5.  How could you combine multiple conditions in a single `if` statement to create a rule that runs a job only on the `main` branch *and* only if the pipeline was triggered manually by a user?

---

### **8. Solution**

This section contains the completed artifacts for this lab.

#### **8.1. Final `.gitlab-ci.yml` File**
```yaml
# filename: .gitlab-ci.yml
stages:
  - test
  - deploy

test-job:
  stage: test
  script:
    - echo "Running tests on branch: $CI_COMMIT_BRANCH"

deploy-job:
  stage: deploy
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
  script:
    - echo "Deploying to production from branch: $CI_COMMIT_BRANCH"
```

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **How to run on every branch *except* `main`?**
    You would change the equality operator `==` to the inequality operator `!=`. The rule would be: `if: '$CI_COMMIT_BRANCH != "main"'`.

2.  **Which variable detects a Merge Request?**
    You would use `$CI_PIPELINE_SOURCE`. The rule would be: `if: '$CI_PIPELINE_SOURCE == "merge_request_event"'`.

3.  **`skipped` vs. `when: manual`?**
    When a rule is not met, the job is **skipped** and does not appear in the pipeline graph at all. When a rule with `when: manual` is met, the job **appears** in the pipeline graph but is in a "paused" state, requiring a user to click a "play" button to run it.

4.  **If a job has no `rules` block, when does it run?**
    By default (without a `workflow` keyword), a job with no `rules` runs on every push to every branch, but it does **not** run for merge request pipelines. This default behavior is often a source of confusion, which is why using explicit `rules` is a best practice.

5.  **How to combine multiple conditions?**
    You can use the `&&` (AND) operator inside the `if` statement. The rule would be: `if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "web"'`.