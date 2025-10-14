# **Lab: The Full Planning-to-Execution Workflow**

**Module:** Project Planning & Issue Management
**Time:** Approx. 35 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Create a detailed issue to define a piece of work.
*   Organize issues by assigning them labels for categorization and milestones for time-boxing.
*   Generate a Merge Request directly from an issue for seamless traceability.
*   Understand and use GitLab's "closing pattern" to automatically close an issue upon merging.
*   Verify the end-to-end link from a closed issue to the code that resolved it.

### **2. Scenario**

Your team is adopting a more structured approach to project management. All new development work must now begin with an **issue**, which serves as the single source of truth for a task's requirements and discussion. To improve planning, issues must also be categorized with **labels** and assigned to a **milestone** for the current work cycle.

Your task is to follow this new process from start to finish. You will create an issue for a new feature, organize it with the appropriate metadata, and then use GitLab's integrated tools to create a Merge Request that is directly linked to it. Finally, you will merge your code and verify that GitLab automatically closes the original issue, completing the traceability loop.

### **3. Prerequisites**

*   You have a GitLab.com account and are a member of the `gabors-gitlab-training-20251013` group.
*   You have a fully configured local environment with Git and Visual Studio Code.

### **4. Steps**

_**Note:** This lab primarily uses the GitLab UI to demonstrate the project management workflow, with a final step performed locally in VS Code._

1.  **Create and Clone the Lab Project**
    *   **[WHERE: GitLab UI - Browser & Local Terminal]**
        *   In the `gabors-gitlab-training-20251013` group, create a new blank project named `student-<firstname>-<lastname>-lab-3.4`.
        *   Initialize it with a `README.md` file.
        *   Clone the project to your local machine and open it in VS Code.

2.  **Create a Milestone for the Work Cycle**
    First, you will define a time-box for the upcoming work.
    *   **[WHERE: GitLab UI - Browser]**
        *   In your new project, navigate to `Plan -> Milestones`.
        *   Click **"New milestone"**.
        *   **Title:** `Sprint 1`
        *   **Start date:** (Today's date)
        *   **Due date:** (Two weeks from today)
        *   Click **"Create milestone"**.

3.  **Create and Organize the Issue**
    Now, you will create the issue that defines your task.
    *   **[WHERE: GitLab UI - Browser]**
        *   Navigate to `Plan -> Issues` and click **"New issue"**.
        *   **Title:** `Add Contribution Guidelines`
        *   **Description:**
            ```markdown
            We need to add a `CONTRIBUTING.md` file to the repository to guide new contributors.

            **Acceptance Criteria:**
            - A file named `CONTRIBUTING.md` must exist in the root of the repository.
            - The file should contain a basic welcome message and a link to the project's issue tracker.
            ```
        *   In the right-hand sidebar:
            *   **Labels:** Click "Edit", then **"Create project label"**.
                *   **Name:** `documentation`
                *   **Color:** Choose a green color.
                *   Click **"Create"**. The `documentation` label will be automatically assigned.
            *   **Milestone:** Click "Edit" and select **`Sprint 1`**.
        *   Click **"Create issue"**.

4.  **Create a Merge Request from the Issue**
    This is the key step that links your planned work to your development activity.
    *   **[WHERE: GitLab UI - Browser]**
        *   You are now on the issue detail page. Locate the **"Create merge request"** button.
        *   Click the button. GitLab will automatically pre-fill the MR with a sensible branch name and link it to the issue.
        *   **Observe:** A new Merge Request is created in **Draft** mode. The description contains the text `Closes #1`. This is the "closing pattern."

5.  **Implement the Change Locally**
    With the MR and branch created, you will now write the code.
    *   **[WHERE: Visual Studio Code]**
        *   First, pull the new branch that GitLab created for you. In the terminal, run:
        ```bash
        git fetch
        git switch add-contribution-guidelines # Or the branch name GitLab created
        ```
        *   Create a new file named **`CONTRIBUTING.md`**.
        *   Add the following content:
        ```markdown
        # Contributing

        We welcome contributions! Please file an issue in our issue tracker before starting work.
        ```
        *   Stage and commit this change with the message `docs: add initial contribution guidelines`.
        *   Push the commit to your branch: `git push`.

6.  **Finalize and Merge the Merge Request**
    Your code is complete and pushed. Now you can finalize the MR.
    *   **[WHERE: GitLab UI - Browser]**
        *   Navigate back to your open Merge Request. You will see your new commit has been added.
        *   Click the **"Mark as ready"** button to remove the "Draft" status.
        *   Click the **"Merge"** button.

### **5. Verification**

1.  **Verify the File:** On the `main` branch of your project, verify that the `CONTRIBUTING.md` file now exists and contains the correct content.
2.  **Verify the MR Status:** The Merge Request should now be in the **"Merged"** state.
3.  **Verify the Issue Status (The Critical Check):**
    *   Navigate back to `Plan -> Issues`.
    *   The issue `Add Contribution Guidelines` should **no longer be in the "Open" tab**.
    *   Click on the **"Closed"** tab. **Expected Result:** Your issue is now listed here.
    *   Click on the issue. In the activity feed, you will see a system note: "closed via merge request !1...".

### **6. Discussion**

This lab demonstrated the powerful, integrated workflow between GitLab's planning and SCM features. By starting with an **issue**, you created a single source of truth for the task. Assigning **labels** and a **milestone** provided crucial context for organization and scheduling, making the work visible on planning boards and reports.

The most important step was creating the **Merge Request directly from the issue**. This action automatically created a branch and, more importantly, embedded a "closing pattern" (`Closes #1`) in the MR's description. This simple text is not just a comment; it's a command that instructs GitLab's automation.

When you finally merged the MR, GitLab detected this pattern and automatically closed the corresponding issue. This creates a seamless, end-to-end audit trail. Anyone can start from the closed issue, find the MR that resolved it, and trace that back to the exact lines of code and commits that were introduced, providing perfect traceability.

### **7. Questions**

1.  What is the difference between a Label and a Milestone? When would you use one over the other?
2.  Besides `Closes`, what are two other keywords that can be used in an MR description to automatically close an issue?
3.  What would have happened if you had created your branch and MR manually and forgotten to add `Closes #1` to the description?
4.  If one Merge Request needs to fix three different issues, how would you format the closing pattern in the description?
5.  Why is it a good practice for GitLab to create the new MR in "Draft" mode when you generate it from an issue?

---

### **8. Solution**

This lab is process-oriented. The key artifacts are the final state of the project, the issue, and the MR.

#### **8.1. Final Artifacts**
*   **Project State:** A `CONTRIBUTING.md` file exists on the `main` branch.
*   **Issue State:** The "Add Contribution Guidelines" issue is in the "Closed" state.
*   **MR State:** The Merge Request is in the "Merged" state.
*   **Traceability:** The closed issue contains a link pointing to the MR that closed it.

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **Label vs. Milestone?**
    A **Label** is for *categorization* (what is it about?). Examples: `bug`, `feature`, `documentation`, `backend`. An issue can have many labels. A **Milestone** is for *scheduling* (when is it due?). It represents a time-box like a sprint or a release version. An issue can only belong to one milestone.

2.  **Other closing keywords?**
    Common alternatives include `Fixes #1` and `Resolves #1`. GitLab recognizes a number of variations on these keywords.

3.  **What if you forgot the closing pattern?**
    The Merge Request would be merged successfully, and the code would be updated, but the issue would **remain open**. You would then have to manually navigate back to the issue and close it, breaking the automated traceability.

4.  **How to close three issues?**
    You can list them all on separate lines or in the same line. GitLab will parse them all. For example:
    ```
    Closes #12
    Closes #15
    Resolves #23
    ```

5.  **Why start in "Draft" mode?**
    This is a safety and communication mechanism. Creating an MR from an issue signals *intent* to start work, but the work is not yet complete. The "Draft" status clearly communicates to the rest of the team that the MR is a work-in-progress, is not ready for review, and prevents it from being accidentally merged before the code has even been written.