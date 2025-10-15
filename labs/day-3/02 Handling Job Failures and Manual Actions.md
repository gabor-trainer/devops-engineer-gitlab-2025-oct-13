# **Lab: Handling Job Failures and Manual Actions**

**Module:** Advanced Pipeline Control with `rules`
**Time:** Approx. 40 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Configure a CI/CD job to be resilient to failure using `allow_failure: true`.
*   Understand the visual difference between a failed job and a job that failed but was allowed to.
*   Implement a manual deployment gate using `when: manual` within a `rules` block.
*   Manually trigger a job from the GitLab pipeline graph UI.
*   Observe how these features affect the overall status of a pipeline.

### **2. Scenario**

Your team's CI/CD pipeline is maturing, but you've encountered two new challenges. First, a newly added code quality ("linting") job is proving to be overly sensitive and sometimes fails for non-critical style issues. These failures are blocking the entire pipeline and preventing developers from getting feedback on their builds. Second, the business team has requested that deployments to the production environment no longer happen automatically; they require a manual approval step before any new code goes live.

Your task is to update the `.gitlab-ci.yml` file to address both requirements. You will make the flaky linting job "non-blocking" and transform the automatic deployment job into a controlled, manual gate that can only be triggered by a human.

### **3. Prerequisites**

*   You have a GitLab.com account and are a member of the `gabors-gitlab-training-20251013` group.
*   You have a fully configured local environment with Git and Visual Studio Code.
*   You are comfortable with the basic `.gitlab-ci.yml` syntax from previous labs.

### **4. Steps**

_**Note:** This lab will guide you through three pipeline runs to observe the behavior at each stage of the configuration._

1.  **Create and Clone the Lab Project**
    *   **[WHERE: GitLab UI - Browser & Local Terminal]**
        *   In the `gabors-gitlab-training-20251013` group, create a new blank project named `student-<firstname>-<lastname>-lab-2.5`.
        *   Initialize it with a `README.md` file.
        *   Clone the project to your local machine and open it in VS Code.

2.  **Create the Initial Pipeline with a Failing Job**
    First, create a pipeline that includes a linting job designed to fail.
    *   **[WHERE: Visual Studio Code]**
        *   Create a new branch named `feature/pipeline-controls`.
        *   Create a `.gitlab-ci.yml` file with the following content. The `exit 1` command in `lint-job` simulates a failure.
    ```yaml
    # filename: .gitlab-ci.yml
    stages:
      - test
      - deploy

    lint-job:
      stage: test
      script:
        - echo "Running linter... Found style issues."
        - exit 1 # This command forces the job to fail

    deploy-job:
      stage: deploy
      script:
        - echo "Deploying to production..."
    ```
    *   Commit and push this change to your branch.

3.  **Observe the Pipeline Failure**
    *   **[WHERE: GitLab UI - Browser]**
        *   Create a Merge Request for your branch.
        *   Navigate to the running pipeline.
        *   **Observe:** The `lint-job` will fail (red 'X' icon). The pipeline will stop immediately, and the `deploy-job` will be marked as "skipped." This is the default, blocking behavior.

4.  **Implement `allow_failure`**
    Now, make the `lint-job` non-blocking.
    *   **[WHERE: Visual Studio Code]**
        *   Modify the `lint-job` by adding `allow_failure: true`.
    ```yaml
    # ... inside .gitlab-ci.yml
    lint-job:
      stage: test
      allow_failure: true # This is the critical change
      script:
        - echo "Running linter... Found style issues."
        - exit 1
    ```
    *   Commit and push this change to your feature branch.

5.  **Observe the Non-Blocking Failure**
    *   **[WHERE: GitLab UI - Browser]**
        *   Navigate to your Merge Request and view the new pipeline.
        *   **Observe:** The `lint-job` will fail again, but this time with a **yellow warning icon**. Critically, the pipeline will **continue**, and the `deploy-job` will now run and succeed. The overall pipeline will be marked as "Passed with warnings."

6.  **Implement the Manual Deployment Gate**
    Finally, add the rule to make the `deploy-job` a manual action on the `main` branch.
    *   **[WHERE: Visual Studio Code]**
        *   Modify the `deploy-job` to include a `rules` block.
    ```yaml
    # ... inside .gitlab-ci.yml
    deploy-job:
      stage: deploy
      rules:
        - if: '$CI_COMMIT_BRANCH == "main"'
          when: manual # This is the critical change
      script:
        - echo "Deploying to production..."
    ```
    *   Commit and push this final change.

7.  **Verify and Trigger the Manual Job**
    *   **[WHERE: GitLab UI - Browser]**
        *   Merge your Merge Request. This will trigger a new pipeline on the `main` branch.
        *   Navigate to this new pipeline on the `main` branch.
        *   **Observe:** The `lint-job` will show its warning, but the `deploy-job` will now appear with a "play" icon, in a "manual" state. The pipeline is considered "blocked" until you take action.
        *   Click the "play" icon for `deploy-job` and then click **"Run job"** to manually trigger the deployment.

### **5. Verification**

1.  **Verify Non-Blocking Failure:** The second pipeline run (on your feature branch) must show the `lint-job` with a yellow warning icon and the `deploy-job` as successfully completed.
2.  **Verify Manual Gate:** The final pipeline run (on the `main` branch) must show the `deploy-job` with a "play" icon. After you manually trigger it, its status should change to "passed."
3.  **Verify Job Skipped on Feature Branch:** In the final pipeline on your feature branch, the `deploy-job` should have been skipped, because the rule `if: '$CI_COMMIT_BRANCH == "main"'` was not met.

### **6. Discussion**

This lab demonstrated two essential techniques for building mature and practical CI/CD pipelines.

The **`allow_failure: true`** attribute is a powerful tool for jobs that provide valuable information but are not critical enough to block the entire delivery process. This is commonly used for code quality scans, test coverage reports, or "flaky" end-to-end tests that are being stabilized. It allows the pipeline to succeed while still notifying the team that an issue requires attention.

The **`when: manual`** keyword is the cornerstone of a **Continuous Delivery** workflow. It creates a deliberate separation between the automated CI phase (build and test) and the business decision to deploy. By placing this manual gate on your production deployment job, you ensure that code is only released to users after an explicit, auditable human action, providing a final layer of control and safety.

### **7. Questions**

1.  What is the difference in the overall pipeline status between a job that fails normally and a job that fails with `allow_failure: true`?
2.  In our lab, `allow_failure` was set at the job level. The book mentions you can also set it within a `rules` block. Why might you want to allow a job to fail only under certain conditions (e.g., only on feature branches)?
3.  Besides `manual`, what are two other possible values for the `when` keyword in a `rules` block?
4.  If a pipeline is in a "blocked" state waiting for a manual job, can other jobs in later stages run?
5.  What is a real-world business reason for using `when: manual` for a production deployment instead of letting it run automatically?

---

### **8. Solution**

This section contains the completed artifacts for this lab.

#### **8.1. Final `.gitlab-ci.yml` File**
```yaml
# filename: .gitlab-ci.yml
stages:
  - test
  - deploy

lint-job:
  stage: test
  allow_failure: true
  script:
    - echo "Running linter... Found style issues."
    - exit 1

deploy-job:
  stage: deploy
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual
  script:
    - echo "Deploying to production..."
```

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **Pipeline Status Difference:** A normal job failure causes the entire pipeline to be marked as **"Failed"** (red). A job that fails with `allow_failure: true` allows the pipeline to continue, and if all other critical jobs succeed, the pipeline is marked as **"Passed with warnings"** (yellow).

2.  **Conditional `allow_failure`:** You might want to enforce strict quality on the `main` branch but be more lenient on feature branches. A rule could specify `allow_failure: true` for all branches *except* `main`, ensuring that code cannot be merged if the linting job fails, but developers are not blocked during their development process.

3.  **Other `when` values:** Two other common values are `on_success` (the default, runs when previous stages succeed) and `always` (runs regardless of whether previous stages failed).

4.  **Can later stages run?** No. A pipeline in a "blocked" state waiting for a manual job will not proceed to any subsequent stages. The manual job acts as a hard gate for the rest of the sequential workflow.

5.  **Business reason for `when: manual`?**
    A business may want to coordinate a software release with a marketing announcement, a customer communication, or a specific time of low user traffic (like a weekend). A manual gate allows the code to be fully tested and ready, with the final "go" decision being a business or operational one, not a purely technical one.