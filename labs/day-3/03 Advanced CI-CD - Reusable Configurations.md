# **Advanced CI/CD: Reusable Configurations**

**Module:** Advanced Pipeline Control with `include`, `extends`, and `!reference`
**Time:** Approx. 50 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Organize a complex pipeline by splitting it into multiple files using `include:local`.
*   Reduce duplication within a file by creating reusable templates with hidden jobs and `extends`.
*   Compose jobs from granular, reusable script blocks using `!reference` tags.
*   Use the Pipeline Editor's "Full configuration" view to validate your refactoring.
*   Understand the trade-offs between `include`, `extends`, and `!reference` for different scenarios.

### **2. Scenario**

Your team's `.gitlab-ci.yml` file is growing rapidly. It has become a single, monolithic file that is difficult to read, maintain, and navigate. Different developers are editing the same file to manage build, test, and deployment jobs, leading to confusion and merge conflicts. Furthermore, you've noticed that many jobs share large blocks of identical configuration, violating the DRY (Don't Repeat Yourself) principle.

Your task is to refactor this unwieldy pipeline into a clean, modular, and maintainable structure. You will first split the configuration into logical files, then use `extends` to create a common job template, and finally use `!reference` tags to share common script blocks.

### **3. Prerequisites**

*   You have a GitLab.com account and are a member of the `gabors-gitlab-training-20251013` group.
*   You have a fully configured local environment with Git and Visual Studio Code.

### **4. Steps**

_**Note:** This lab consists of five progressive exercises. You will start with a single large file and refactor it step-by-step._

1.  **Create and Clone the Lab Project**
    *   **[WHERE: GitLab UI - Browser & Local Terminal]**
        *   Create a new blank project in our group named `student-<firstname>-<lastname>-lab-3.6`.
        *   Initialize it with a `README.md`.
        *   Clone the project and open it in VS Code.

2.  **Create the Initial Monolithic Pipeline**
    First, create the large, "bad" `.gitlab-ci.yml` file that we will refactor.
    *   **[WHERE: Visual Studio Code]**
        *   Create a new branch named `feature/refactor-pipeline`.
        *   In the root of the project, create a new file named **`.gitlab-ci.yml`**.
        *   Add the following complete configuration. Notice the significant duplication.
    ```yaml
    # filename: .gitlab-ci.yml
    stages:
      - build
      - test

    build-job:
      stage: build
      image: alpine:latest
      rules:
        - if: '$CI_COMMIT_BRANCH == "main"'
      before_script:
        - echo "--- Preparing Build Environment ---"
        - apk add --no-cache git
      script:
        - echo "Building the application..."
        - git --version
      after_script:
        - echo "--- Cleaning Up Build Environment ---"

    unit-test-job:
      stage: test
      image: alpine:latest
      rules:
        - if: '$CI_COMMIT_BRANCH == "main"'
      before_script:
        - echo "--- Preparing Test Environment ---"
        - apk add --no-cache curl
      script:
        - echo "Running unit tests..."
        - curl --version
      after_script:
        - echo "--- Cleaning Up Test Environment ---"
    ```
    *   Commit and push this change. Create a Merge Request and verify that both jobs run on the `main` branch after you merge (you don't have to merge yet, just have the MR ready).

---
#### **Exercise 1: Refactoring with `include`**
*Goal: Split the monolithic file into logical parts.*

1.  **Create Separate CI Files**
    *   **[WHERE: Visual Studio Code]**
        *   Create a new file named `.gitlab-ci-build.yml` and move the entire `build-job` block into it.
        *   Create a new file named `.gitlab-ci-test.yml` and move the entire `unit-test-job` block into it.

2.  **Update the Main `.gitlab-ci.yml`**
    *   Modify the main `.gitlab-ci.yml` file to include the new files. The `stages` block should remain here.
    ```yaml
    # filename: .gitlab-ci.yml
    stages:
      - build
      - test

    include:
      - local: '.gitlab-ci-build.yml'
      - local: '.gitlab-ci-test.yml'
    ```
    *   Commit and push this change. Verify in the MR that the pipeline still runs with two jobs.

---
#### **Exercise 2: Refactoring with `extends` (Common Properties)**
*Goal: Remove duplication of `stage`, `image`, and `rules`.*

1.  **Create a Hidden Template Job**
    *   **[WHERE: Visual Studio Code]**
        *   In the main `.gitlab-ci.yml` file, create a "hidden" job (starts with a `.`) to act as a template.
    ```yaml
    # filename: .gitlab-ci.yml
    .base_job_template:
      image: alpine:latest
      rules:
        - if: '$CI_COMMIT_BRANCH == "main"'

    stages:
      # ...
    ```

2.  **Update Jobs to Use `extends`**
    *   In `.gitlab-ci-build.yml`, modify `build-job` to use `extends` and remove the duplicated lines.
    ```yaml
    # filename: .gitlab-ci-build.yml
    build-job:
      stage: build
      extends: .base_job_template
      before_script:
        # ...
    ```
    *   Do the same for `unit-test-job` in `.gitlab-ci-test.yml`.

---
#### **Exercise 3: Refactoring with `extends` (Overriding Properties)**
*Goal: Understand how `extends` can be overridden.*

1.  **Override the `image` in a Job**
    *   **[WHERE: Visual Studio Code]**
        *   In `.gitlab-ci-test.yml`, modify `unit-test-job` to specify a different image. This `image` keyword will override the one inherited from the template.
    ```yaml
    # filename: .gitlab-ci-test.yml
    unit-test-job:
      stage: test
      extends: .base_job_template
      image: ubuntu:latest # Override the image
      before_script:
        # ...
    ```

---
#### **Exercise 4: Refactoring with `!reference` (Reusing Script Blocks)**
*Goal: Share a common script block between jobs.*

1.  **Create a Hidden Job with a Reusable Script**
    *   **[WHERE: Visual Studio Code]**
        *   In the main `.gitlab-ci.yml` file, create a new hidden job that contains a common cleanup script. Use a YAML anchor (`&cleanup_script`).
    ```yaml
    # filename: .gitlab-ci.yml
    .script_templates:
      cleanup: &cleanup_script
        - echo "--- Cleaning Up Generic Environment ---"

    # ... (rest of the file)
    ```

2.  **Use `!reference` to Inject the Script**
    *   Modify both `build-job` and `unit-test-job` to replace their `after_script` with a `!reference` tag.
    ```yaml
    # In .gitlab-ci-build.yml
    build-job:
      # ...
      after_script:
        - !reference [.script_templates, cleanup]

    # In .gitlab-ci-test.yml
    unit-test-job:
      # ...
      after_script:
        - !reference [.script_templates, cleanup]
    ```

---
#### **Exercise 5: Refactoring with `!reference` (Composing Scripts)**
*Goal: Combine a shared script block with a job-specific script.*

1.  **Modify a Job to Compose its `after_script`**
    *   **[WHERE: Visual Studio Code]**
        *   Modify `build-job` to have both its own specific cleanup command *and* the shared command.
    ```yaml
    # filename: .gitlab-ci-build.yml
    build-job:
      # ...
      after_script:
        - echo "--- Cleaning Up Build-Specific Cache ---"
        - !reference [.script_templates, cleanup] # Also run the shared script
    ```
    *   Commit and push all your changes.

### **5. Verification**

1.  **Verify the Final Pipeline:**
    *   **[WHERE: GitLab UI - Browser]**
        *   Go to your Merge Request and view the latest pipeline. It should still have two jobs.
        *   Inspect the log for `build-job`. It should run on `alpine`, and its `after_script` must show **both** the "Build-Specific" and "Generic Environment" echo messages.
        *   Inspect the log for `unit-test-job`. It must show that it ran on an `ubuntu` image, and its `after_script` must show only the "Generic Environment" message.

2.  **Verify with the Full Configuration Viewer:**
    *   Navigate to `Build -> Pipeline editor`.
    *   Click the **"Full configuration"** tab.
    *   **Expected Result:** You will see a single, large YAML file that GitLab has automatically compiled from all your `include`, `extends`, and `!reference` directives. This is the "ground truth" of what GitLab will execute.

### **6. Discussion**

This lab demonstrated the three primary techniques for creating maintainable, DRY (Don't Repeat Yourself) CI/CD configurations in GitLab.

*   **`include`** is for **organization**. It allows you to split your pipeline into logical files, making the configuration easier to navigate and reducing merge conflicts by separating concerns.
*   **`extends`** is for **inheritance**. It allows you to create a base template and have multiple jobs inherit its properties. This is perfect for enforcing consistency across a set of similar jobs (e.g., all test jobs must use the same `rules`).
*   **`!reference`** is for **composition**. It provides a more granular way to reuse specific snippets, like a block of script commands, allowing you to build up a job's script from multiple reusable parts.

By combining these techniques, you can move from simple, monolithic pipelines to a sophisticated, scalable, and highly maintainable automation architecture, which is a requirement for any enterprise-scale project.

### **7. Questions**

1.  What is the primary difference in purpose between `include` and `extends`?
2.  In Exercise 3, the `image: ubuntu:latest` in `unit-test-job` overrode the `image: alpine:latest` from the template. What would have happened to the `rules` block if you had also added a `rules` block to `unit-test-job`?
3.  Why was it necessary to use `!reference` to reuse the `after_script`, instead of just putting the `after_script` in the `.base_job_template` and using `extends`?
4.  If you have a configuration that is shared by 20 different projects in your group, which reusability keyword would be the most appropriate to use?
5.  What is a potential risk of using YAML anchors and aliases (`&` and `*`) directly instead of the `!reference` tag that GitLab provides?

---

### **8. Solution**

This section contains the final, completed state of all files for this lab.

#### **8.1. Final `.gitlab-ci.yml`**
```yaml
# filename: .gitlab-ci.yml
stages:
  - build
  - test

.base_job_template:
  image: alpine:latest
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'

.script_templates:
  cleanup: &cleanup_script
    - echo "--- Cleaning Up Generic Environment ---"

include:
  - local: '.gitlab-ci-build.yml'
  - local: '.gitlab-ci-test.yml'
```

#### **8.2. Final `.gitlab-ci-build.yml`**
```yaml
# filename: .gitlab-ci-build.yml
build-job:
  stage: build
  extends: .base_job_template
  before_script:
    - echo "--- Preparing Build Environment ---"
    - apk add --no-cache git
  script:
    - echo "Building the application..."
    - git --version
  after_script:
    - echo "--- Cleaning Up Build-Specific Cache ---"
    - !reference [.script_templates, cleanup]
```

#### **8.3. Final `.gitlab-ci-test.yml`**
```yaml
# filename: .gitlab-ci-test.yml
unit-test-job:
  stage: test
  extends: .base_job_template
  image: ubuntu:latest
  before_script:
    - echo "--- Preparing Test Environment ---"
    - apt-get update && apt-get install -y curl
  script:
    - echo "Running unit tests..."
    - curl --version
  after_script:
    - !reference [.script_templates, cleanup]
```

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **`include` vs. `extends`?**
    `include` is for combining **entire files** into a single pipeline configuration. Its primary purpose is file organization. `extends` is for sharing **properties between jobs** within the final, merged configuration. Its purpose is to reduce duplication and create reusable templates.

2.  **What if you added a `rules` block to the job?**
    The `rules` block in the `unit-test-job` would have **completely replaced** the `rules` block inherited from the template. Unlike dictionaries like `variables` which are merged, list-based properties like `rules`, `script`, and `before_script` are overwritten when using `extends`.

3.  **Why use `!reference` instead of putting `after_script` in the template?**
    Because in Exercise 5, the `build-job` needed to **compose** its `after_script`. It needed its own specific command *in addition to* the shared command. If the shared command was in the base template, `build-job` would have had to overwrite the entire `after_script`, losing the shared command. `!reference` allows for this granular, compositional reuse.

4.  **Which keyword for sharing across 20 projects?**
    You would use `include:project`. You would create a central "CI/CD Templates" project, place your shared configuration file in it, and then each of the 20 projects would `include` that file using a path to that central project.

5.  **Risk of using native YAML anchors directly?**
    While they work, GitLab's `!reference` tag provides better validation and error messaging within the GitLab ecosystem. More importantly, native YAML anchors (`&` and `*`) only work if the anchor and the alias are in the **same file**. The `!reference` tag is designed to work across files brought in via `include`, which is essential for a modular pipeline architecture.