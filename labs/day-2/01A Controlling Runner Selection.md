# **Controlling Runner Selection**

**Module:** Advanced Pipeline Control with `rules`
**Time:** Approx. 25 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Inspect a project's settings to see which runners are available to it.
*   Understand the hierarchy and precedence of GitLab runners (Instance, Group, Specific).
*   Use `tags` in your `.gitlab-ci.yml` file to explicitly direct a job to a specific runner.
*   Disable shared runners at the project level to enforce the use of group runners.

### **2. Scenario**

Your team has just run its first CI/CD pipeline, but you've noticed from the job log that it was executed by one of GitLab's general-purpose "Instance Runners" and not the dedicated, high-performance "Group Runner" that your platform team (us) has provided. While the job succeeded, this is not the desired behavior. The Group Runner has specialized tools and a consistent configuration that your project depends on.

Your task is to investigate which runners are available to your project and then modify your `.gitlab-ci.yml` file to **explicitly demand** that your jobs run only on the correct Group Runner. Finally, you will enforce this policy at the project level to prevent accidental use of the wrong runners in the future.

### **3. Prerequisites**

*   You have a project from a previous lab with a working `.gitlab-ci.yml` file (e.g., `student-<firstname>-<lastname>-lab-2.1`).
*   The instructor has (for the purpose of this lab) temporarily re-enabled "Instance runners" for our main training group, making both types of runners available to your project.

### **4. Steps**

_**Note:** This lab involves navigating settings in the GitLab UI and making a small but critical change to your CI/CD configuration file._

1.  **Investigate Available Runners (The "In Advance" Check)**
    Before you can select a runner, you must know what your options are.
    *   **[WHERE: GitLab UI - Browser]**
        *   Navigate to your lab project.
        *   In the left-hand navigation pane, go to **`Settings -> CI/CD`**.
        *   Expand the **"Runners"** section.
        *   **Observe:** You will see three distinct sections:
            1.  **Specific runners:** Runners locked to this project only (should be 0).
            2.  **Group runners:** Our dedicated "Azure Group Runner for Gabor's Training" should be listed here. **Note its tags (e.g., `docker`, `azure`, `linux`).**
            3.  **Instance runners:** A list of GitLab.com's general-purpose runners.
        *   This screen gives you a complete inventory of every runner that is currently eligible to run jobs for your project.

2.  **Control Runner Selection Using Tags in `.gitlab-ci.yml`**
    Now, you will modify your pipeline to explicitly request a runner that has the specific tags of our Group Runner.
    *   **[WHERE: Visual Studio Code]**
        *   Open your `.gitlab-ci.yml` file.
        *   Add a `tags` keyword to your `build-job`. Use one of the unique tags you observed in the previous step, for example, `azure`.
    ```yaml
    # filename: .gitlab-ci.yml
    # ... (stages definition)

    build-job:
      stage: build
      tags:
        - azure # This is the critical change
      script:
        - echo "This is the build job. Forcing it to run on the Azure runner..."
        - sleep 5
        - echo "Build complete."
    ```
    *   Apply the same `tags` section to your `test-job` and `deploy-job` as well.

3.  **Commit, Push, and Verify the Change**
    *   **[WHERE: Visual Studio Code]**
        *   Stage and commit your changes with the message `ci: add tags to control runner selection`.
        *   Push the changes to your feature branch.
    *   **[WHERE: GitLab UI - Browser]**
        *   Go to your open Merge Request and view the new pipeline that has started.
        *   Click on the `build-job` to inspect its log.

4.  **Enforce Runner Policy at the Project Level (Outside YAML)**
    Tags are a great way to select a runner, but what if you want to prevent your project from *ever* using the Instance runners? You can enforce this in the project settings.
    *   **[WHERE: GitLab UI - Browser]**
        *   Navigate back to `Settings -> CI/CD` and expand the **"Runners"** section.
        *   Find the **"Instance runners"** section.
        *   Locate the toggle switch labeled **"Turn on instance runners for this project"**. It will be enabled.
        *   **Disable this toggle.**

5.  **Test the Enforced Policy**
    Now, remove the tags from your YAML file. The Group Runner should be the only one left, so it should be selected automatically.
    *   **[WHERE: Visual Studio Code]**
        *   Remove the `tags:` section from all three of your jobs in `.gitlab-ci.yml`.
        *   Commit this change with the message `ci: remove tags to test project-level policy`.
        *   Push the change.
    *   **[WHERE: GitLab UI - Browser]**
        *   Observe the new pipeline and inspect the job log for `build-job`.

### **5. Verification**

1.  **Verification for Step 3:** The job log for your pipeline with `tags` **must** show the runner description: `Running with gitlab-runner ... on Azure Group Runner for Gabor's Training`. This confirms your `tags` keyword successfully directed the job.
2.  **Verification for Step 5:** The job log for your pipeline *without* `tags` **must also** show the runner description: `Running with gitlab-runner ... on Azure Group Runner for Gabor's Training`. This confirms that by disabling the instance runners, you have forced all jobs in your project to use the only remaining eligible runner: our Group Runner.

### **6. Discussion**

This lab demonstrated a fundamental concept in enterprise CI/CD: **explicit runner selection and governance**. When a project has access to multiple runner fleets (e.g., a shared company fleet and a dedicated team fleet), you cannot rely on luck. You must have a strategy.

We explored two methods of control:
1.  **Inside YAML (`tags`):** This is a flexible, per-job method. It allows you to direct specific jobs to specific runners. For example, a `build-job` might run on a general-purpose Linux runner (`tag: linux`), while an `ios-build` job must run on a macOS runner (`tag: macos`). The `tags` keyword is the primary mechanism for routing jobs to the correct infrastructure.

2.  **Outside YAML (Project Settings):** This is a powerful, project-wide governance tool. By disabling "Instance runners" at the project or group level, a manager or team lead can enforce a policy that guarantees no jobs will ever run on the general-purpose, shared fleet. This is often done for security, compliance, or cost-control reasons.

### **7. Questions**

1.  What would happen if you added a tag to your job (e.g., `tags: [non-existent-tag]`) that did not match any available runner?
2.  Imagine our Group Runner was tagged with `[docker, azure]` and a GitLab Instance Runner was tagged with `[docker, linux]`. If your job was configured with `tags: [docker]`, which runner would be eligible to run it?
3.  The Instructor disabled Instance Runners at the *Group* level in our main setup. In this lab, you disabled them at the *Project* level. Which setting has higher precedence?
4.  If a job has no `tags` and both Group and Instance runners are enabled, how does GitLab's coordinator decide which runner gets the job?
5.  Why is it a good security practice for a "deployment" job that uses production secrets to be tagged with a very specific tag (e.g., `production-deployer`) that only exists on a highly secured, protected runner?

---

### **8. Solution**

This section contains the completed artifacts for this lab.

#### **8.1. Final `.gitlab-ci.yml` (After Step 2)**
```yaml
# filename: .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

build-job:
  stage: build
  tags:
    - azure
  script:
    - echo "This is the build job. Forcing it to run on the Azure runner..."
    - sleep 5
    - echo "Build complete."

test-job:
  stage: test
  tags:
    - azure
  script:
    - echo "This is the test job..."

deploy-job:
  stage: deploy
  tags:
    - azure
  script:
    - echo "This is the deploy job..."
```

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **What if a tag doesn't match?**
    The job would get stuck in a "pending" state. The UI would show a warning: "This job is stuck. Check that you have runners available with the following tags: `non-existent-tag`." It would eventually time out and fail.

2.  **If a job has a common tag (`docker`), which runner is eligible?**
    **Both** runners would be eligible. The job only requires a runner that has *at least one* of the specified tags. GitLab's coordinator would then assign the job to whichever of the two runners becomes available first. This is why having unique, specific tags is important for precise control.

3.  **Which setting has higher precedence?**
    Settings at a more specific level (the Project) **override** settings at a broader level (the Group). If the instructor disables instance runners at the group level, a project owner cannot re-enable them for their project. Conversely, if the group allows instance runners, a project owner can choose to disable them for their specific project.

4.  **How does the coordinator choose a runner?**
    The exact scheduling algorithm is complex, but it generally prioritizes runner availability and recent usage. For a simple job, it will likely be assigned to the first available runner that is polling for jobs. This is why you cannot rely on chance and must use tags or project settings for control.

5.  **Why use specific tags for deployment jobs?**
    This is a core security principle. It ensures that a job with access to sensitive credentials (like production database passwords or cloud deployment keys) can **only** run on a machine that has been specifically hardened and secured for that purpose. It prevents a sensitive deployment job from accidentally running on a general-purpose, less secure build runner.