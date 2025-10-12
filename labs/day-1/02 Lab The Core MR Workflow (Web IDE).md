# **Lab: The Professional MR Workflow (vs code)**

**Module:** The Core Collaboration Workflow
**Time:** Approx. 45 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Create a new branch directly from an issue.
*   Use the GitLab Web IDE to stage and commit file changes.
*   Open, review, and manage the lifecycle of a Merge Request (MR).
*   Visualize branch changes using the repository graph.
*   Intentionally create and resolve a merge conflict within the GitLab UI.

### **2. Scenario**

A project manager has created an issue requesting an update to the project's main documentation. As a developer on the team, your task is to address this issue by following the standard GitLab workflow. You will create a dedicated branch for your changes, update the requested file, and then use a Merge Request to have your changes reviewed and integrated back into the main codebase.

During this process, another developer will have merged a conflicting change, and you will be responsible for resolving this conflict before your work can be merged. This entire lab will be performed using only the GitLab web interface, demonstrating its power as a complete development environment for many tasks.

### **3. Prerequisites**

*   You have successfully completed **Lab 1.1** and have an active GitLab.com account.
*   A project named `My First Project` has been created for you within our training group, initialized with a `README.md` file.
*   An issue titled "Update project documentation" has been created in this project.

### **4. Steps**

_**Note:** This section provides the key UI actions. The "Discussion" section will explain the underlying Git concepts._

1.  **Create a Branch from the Issue**
    First, we will create a dedicated branch to isolate our work, linking it directly to the issue for traceability.
    *   Navigate to your `My First Project`.
    *   Go to `Plan -> Issues` and click on the issue titled **"Update project documentation"**.
    *   Locate the **"Create merge request"** button. Click the dropdown arrow next to it and select **"Create branch"**.
    *   In the "Branch name" field, accept the default name generated from the issue title.
    *   Click the **"Create branch"** button. You will be automatically redirected to the repository view for your new branch.

2.  **Edit Files with the Web IDE**
    Now, you will make the required documentation changes using GitLab's integrated Web IDE.
    *   On your new branch's repository page, click the **"Edit"** dropdown button and select **"Web IDE"**.
    *   In the Web IDE's file explorer on the left, click on `README.md`.
    *   Modify the content of the `README.md` file. For example, add a new section:
    ```markdown
    ## Project Usage

    To use this project, clone the repository and follow the setup instructions.
    ```

3.  **Commit the Changes**
    Next, you will commit your changes, creating a permanent snapshot in the branch's history.
    *   In the Web IDE's left-hand sidebar, click the **"Source Control"** icon. You will see `README.md` listed under "Changes."
    *   In the **"Commit message"** text box, enter a descriptive message, such as `docs: add project usage section`.
    *   Click the **"Commit to '[your-branch-name]'"** button.

4.  **Create the Merge Request**
    With your changes committed, you will now create a Merge Request to propose integrating your branch back into `main`.
    *   Navigate back to your project's main page. A notification banner will appear at the top: "You pushed to [your-branch-name]...".
    *   Click the **"Create merge request"** button in this banner.
    *   On the "New merge request" page, observe that the title and description are pre-filled. The description will automatically include "Closes #[issue-number]".
    *   Click the **"Create merge request"** button.

5.  **Simulate a Conflicting Change**
    *For this step, the instructor will simulate another developer's work by merging a change that conflicts with yours.* This will update the `main` branch while your MR is still open.

6.  **Identify and Resolve the Merge Conflict**
    Now, you will see that your Merge Request is blocked. You must resolve the conflict.
    *   Navigate back to your open Merge Request.
    *   You will see an error message: "Merge blocked: merge conflicts must be resolved."
    *   Click the **"Resolve conflicts"** button.
    *   GitLab will show you the conflicting sections. For each conflict, choose which version to keep by clicking **"Use ours"** or **"Use theirs"**. For this lab, choose **"Use theirs"** to accept the changes from the `main` branch.
    *   Once all conflicts are resolved, enter a commit message at the bottom of the page, such as `fix: resolve merge conflict with main`.
    *   Click **"Commit to source branch"**.

7.  **Merge the Final Changes**
    With the conflicts resolved, your Merge Request is now ready to be merged.
    *   On your Merge Request page, the status will now be "Ready to merge."
    *   Click the **"Merge"** button.

### **5. Verification**

1.  **Verify the Merge:** On the `main` branch of your project, view the `README.md` file. It should contain the changes from *both* your work and the conflicting change you resolved.
2.  **Verify the Branch Graph:** Navigate to `Code -> Repository graph`. You should see a visual representation of your feature branch splitting off from `main` and then being merged back in.
3.  **Verify the Issue Status:** Navigate to `Plan -> Issues`. The "Update project documentation" issue should now be in the **"Closed"** tab.

### **6. Discussion**

This lab demonstrated the complete, end-to-end workflow for contributing a change in GitLab. We followed the **GitLab Flow** branching strategy by creating a short-lived **feature branch** to isolate our work. Using the **Web IDE**, we made changes and created a **commit**, a permanent record of our work on that branch.

The **Merge Request (MR)** was the central point of collaboration. It announced our changes were ready for review and, crucially, it was where GitLab's automated systems detected a **merge conflict**. By resolving the conflict, we created a new commit on our branch that integrated the latest changes from `main`, ensuring a clean and safe final merge. Finally, by linking the MR to the original issue, we enabled GitLab to automatically close the issue upon merging, providing seamless traceability from task to completion.

### **7. Questions**

1.  In the Web IDE, what is the difference between "Staging" a change and "Committing" a change? (Hint: The Web IDE combines these, but they are separate Git concepts).
2.  What would have happened if you had ignored the merge conflict and a Maintainer had tried to merge your MR anyway?
3.  Besides the Web IDE, what other tool could you have used to resolve the merge conflict? (Hint: See Chapter 3).
4.  Why is it a best practice to create a new branch from an issue instead of just creating a branch manually from the repository page?
5.  Imagine your feature branch was very long-lived (weeks). How could you have proactively updated it with the latest changes from `main` to *prevent* a merge conflict from happening at the end?

---

### **8. Solution**

This lab primarily involves UI interactions. The key artifacts are the repository's state and Git history.

#### **8.1. Final Code Artifact**
The `README.md` file on the `main` branch should contain a combination of your changes and the conflicting change that was introduced.

#### **8.2. Command Summary**
While this lab used the UI, the equivalent Git commands for the core actions are:
```bash
# 1. Create a branch and switch to it
git checkout -b issue-branch-name

# 2. Stage and commit changes
git add README.md
git commit -m "docs: add project usage section"

# 3. Get latest changes from the remote main branch
git fetch origin main

# 4. Rebase your branch on top of main to find and resolve conflicts
git rebase origin/main
# (Manually edit files to resolve conflicts here)
git add README.md
git rebase --continue

# 5. Push your resolved branch and merge the MR
git push --force-with-lease
```

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **Staging vs. Committing?**
    In standard Git, "staging" (using `git add`) is the act of selecting which of your modified files you want to include in the *next* commit. "Committing" (`git commit`) then takes all staged files and creates a permanent snapshot. The Web IDE simplifies this by automatically staging any file you've modified when you click the commit button.

2.  **What if the conflict was ignored?**
    GitLab would have blocked the merge entirely. The "Merge" button would have remained disabled. A merge conflict is a hard gate; Git cannot proceed with an automatic merge until a human has explicitly resolved the conflicting instructions.

3.  **What other tool could be used to resolve the conflict?**
    You could have used a local Git client. The process would involve pulling the latest changes from `main` into your local repository, merging or rebasing `main` into your feature branch, resolving the conflict in your local code editor (like VS Code), and then pushing the resolved code back up to GitLab.

4.  **Why create a branch from an issue?**
    Traceability. Creating a branch from an issue automatically names the branch based on the issue and, more importantly, establishes a link between the two. This makes it easy for anyone viewing the issue to see the branch where the work is happening, and for anyone on the branch/MR to refer back to the original requirements in the issue.

5.  **How to proactively prevent conflicts?**
    The best practice is to frequently update your long-lived feature branch with the latest changes from `main`. You would do this locally by running `git pull origin main` and then `git merge main` (or, more cleanly, `git rebase main`) into your feature branch. This allows you to resolve small, incremental conflicts regularly instead of facing one massive conflict at the end.