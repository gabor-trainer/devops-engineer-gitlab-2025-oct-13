# **The Professional Git Cycle (local, vs code)**

**Module:** 1.3: Professional Local Git Workflow
**Time:** Approx. 50 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Clone a GitLab repository to your local machine using SSH.
*   Use the Visual Studio Code UI to create a new branch.
*   Edit code locally and use the VS Code Source Control panel to stage, commit, and push changes.
*   Initiate a Merge Request (MR) from a pushed branch in the GitLab UI.
*   Perform a local merge to resolve a conflict and push the resolution to GitLab.

### **2. Scenario**

You have successfully completed a small task using the GitLab Web IDE, but for most development work, you will use a professional, local Integrated Development Environment (IDE). Your task is to repeat the process from the previous lab—addressing the "Update project documentation" issue—but this time using a fully local workflow with Visual Studio Code.

This represents the standard day-to-day workflow for a professional developer: clone the code, create a branch, write code locally, commit and push changes, and then use GitLab's web interface for the collaborative parts of the process, like creating and managing the Merge Request.

### **3. Prerequisites**

*   You have successfully completed **Lab 1.1** and have a working SSH key configured with your GitLab account.
*   You have the full local development stack installed: Git, Java 17, Maven, and Visual Studio Code with the recommended extensions.
*   Your `My First Project` exists in the `gabors-gitlab-training-20251013` group.

### **4. Steps**

_**Note:** This lab involves switching between your local VS Code application and the GitLab web UI in your browser. Pay close attention to the **[WHERE]** indicator for each step._

1.  **Clone the Repository to Your Local Machine**
    First, you need to get a local copy of the project's repository.
    *   **[WHERE: GitLab UI - Browser]**
        *   Navigate to your `My First Project`.
        *   Click the blue **"Code"** button in the top right.
        *   Select **"Clone with SSH"** and copy the provided URL (it will look like `git@gitlab.com:gabors-gitlab-training-20251013/my-first-project.git`).
    *   **[WHERE: Visual Studio Code]**
        *   Open VS Code.
        *   Open the Command Palette (`Ctrl+Shift+P` or `Cmd+Shift+P`).
        *   Type `Git: Clone` and press Enter.
        *   Paste the SSH URL you copied and press Enter.
        *   Select a location on your local drive to save the project.
        *   Once cloned, click **"Open"** in the notification pop-up to open the project folder.

2.  **Create a New Branch in VS Code**
    Before making changes, create a new branch to isolate your work.
    *   **[WHERE: Visual Studio Code]**
        *   In the bottom-left corner of the VS Code window, click on the current branch name (which should be `main`).
        *   A menu will appear at the top. Select **"+ Create new branch..."**.
        *   Enter a branch name, for example `feature/update-readme-local`, and press Enter. You are now on your new branch.

3.  **Edit the File Locally**
    Make the required documentation changes in the `README.md` file.
    *   **[WHERE: Visual Studio Code]**
        *   In the VS Code Explorer panel on the left, click on `README.md` to open it.
        *   Add a new section to the file:
        ```markdown
        ## Local Development Setup

        This project requires a local installation of Java 17 and Maven.
        ```
        *   Save the file (`Ctrl+S` or `Cmd+S`).

4.  **Commit the Changes Using VS Code's Source Control**
    Use VS Code's integrated Git panel to commit your work.
    *   **[WHERE: Visual Studio Code]**
        *   Click the **Source Control** icon (the branching icon) in the left-hand sidebar.
        *   You will see `README.md` listed under "Changes."
        *   In the **Message** box at the top, enter a descriptive commit message, like `docs: describe local dev setup`.
        *   Click the **Commit** button (the checkmark icon).

5.  **Push the Branch to GitLab**
    Your commit exists only on your local machine. You now need to push it to the remote GitLab repository.
    *   **[WHERE: Visual Studio Code]**
        *   In the bottom-left corner, you will see a "Publish Branch" button (an icon of a cloud with an up arrow). Click this button.
        *   This action pushes your new branch and its commit to GitLab.

6.  **Create the Merge Request in GitLab**
    The code is now on GitLab, so you can propose to merge it.
    *   **[WHERE: GitLab UI - Browser]**
        *   Navigate to your `My First Project` main page. A notification banner will appear for your newly pushed branch.
        *   Click the **"Create merge request"** button in this banner.
        *   On the "New merge request" page, review the title and description, then click **"Create merge request"**.

7.  **Simulate and Resolve a Conflicting Change**
    *   **[WHERE: Instructor-led]** The instructor will again merge a conflicting change into the `main` branch.
    *   **[WHERE: Visual Studio Code]** Now, you must resolve this conflict locally.
        1.  First, switch back to your `main` branch in VS Code (click the branch name in the bottom-left corner and select `main`).
        2.  Pull the latest changes, which include the conflict: click the "Sync Changes" button (the circular arrows icon) in the bottom-left corner, or run `git pull` in the VS Code terminal.
        3.  Switch back to your feature branch (`feature/update-readme-local`).
        4.  Attempt to merge the updated `main` branch into your feature branch. In the VS Code terminal, run: `git merge main`.
        5.  Git will report a merge conflict. In the Source Control panel, `README.md` will now appear under "Merge Changes." Click on it.
        6.  VS Code will open a special three-way merge editor. You will see "Incoming" changes (from `main`) and "Current" changes (from your branch). Above the conflicting block, click the **"Accept Incoming"** button.
        7.  Once resolved, save the `README.md` file. In the Source Control panel, stage the resolved file by clicking the `+` icon next to it.
        8.  Enter a commit message like `fix: merge main and resolve conflicts` and click the **Commit** button.

8.  **Push the Conflict Resolution**
    *   **[WHERE: Visual Studio Code]**
        *   Click the **"Sync Changes"** button in the bottom-left corner to push your new merge commit to GitLab.

9.  **Merge the Final Changes**
    *   **[WHERE: GitLab UI - Browser]**
        *   Refresh your Merge Request page. It will now be free of conflicts and ready to merge.
        *   Click the **"Merge"** button.

### **5. Verification**

1.  **Verify the Final Content:** Check the `README.md` on the `main` branch in the GitLab UI. It should contain the correctly merged content.
2.  **Verify the Commit History:** In your local VS Code, open the terminal and run `git log --graph --oneline main`. You should see the merge commit from your branch, showing how the histories were combined.
3.  **Verify the MR:** The Merge Request in GitLab should now be in the "Merged" state.

### **6. Discussion**

This lab demonstrated the standard, professional workflow for a developer using local tools. By cloning the repository, you created a complete, independent copy of the project on your machine. All coding and initial commits happened locally, giving you speed and control without needing constant network access.

The core of the professional workflow is the **`pull -> code -> commit -> push` cycle**. You used VS Code's powerful integrated Git tooling to manage this entire process. When a merge conflict occurred, you handled it locally using a sophisticated merge editor, which provides far more context and control than a simple web UI. The final step—the Merge Request—remained in GitLab, reinforcing its role as the central platform for **collaboration, review, and integration**, while your local machine remains the primary environment for **creation**.

### **7. Questions**

1.  In the VS Code Source Control panel, what is the purpose of the "stage" (`+`) icon? How does it differ from the "Commit" button?
2.  In step 7, we used `git merge main` to resolve the conflict. An alternative command is `git rebase main`. What is the primary difference in the resulting Git history between a merge and a rebase?
3.  What does the "Sync Changes" button in VS Code actually do? What two Git commands does it combine?
4.  If you had multiple commits on your feature branch before pushing, what would the "Publish Branch" button have done?
5.  Why is resolving a complex merge conflict often easier in a local IDE like VS Code compared to the GitLab web UI?

---

### **8. Solution**

This lab involves local configuration and UI interactions. The key artifacts are the commands and the final code.

#### **8.1. Final `README.md`**
The `README.md` file on the `main` branch will contain a combination of the changes you added and the conflicting changes introduced by the instructor, as resolved by you.

#### **8.2. Command Summary**
While most actions were performed in the VS Code UI, these are the underlying Git commands that were executed:
```bash
# 1. Clone the repository
git clone git@gitlab.com:gabors-gitlab-training-20251013/my-first-project.git

# 2. Create and switch to a new branch
git checkout -b feature/update-readme-local

# 4. Stage and commit changes
git add README.md
git commit -m "docs: describe local dev setup"

# 5. Push the new branch to the remote
git push -u origin feature/update-readme-local

# 7. Update local main and resolve conflict
git checkout main
git pull
git checkout feature/update-readme-local
git merge main
# (Manual conflict resolution in editor)
git add README.md
git commit -m "fix: merge main and resolve conflicts"

# 8. Push the resolution
git push
```

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **Staging vs. Committing in VS Code?**
    The `+` (stage) icon performs a `git add` on a file. This moves the file from your "working changes" to the "staged changes" area, marking it for inclusion in the next commit. The "Commit" button performs a `git commit`, taking everything in the "staged changes" area and creating a permanent snapshot. This two-step process allows you to build a commit out of a subset of your modified files.

2.  **`merge` vs. `rebase`?**
    `git merge` creates a new "merge commit" that has two parents, tying the two histories together. This preserves the exact history of your feature branch but can make the overall log look complex. `git rebase` rewrites your feature branch's history by replaying your commits one by one on top of the latest `main` branch, resulting in a clean, linear history but altering your original commit SHAs.

3.  **What does "Sync Changes" do?**
    The "Sync Changes" button in VS Code typically performs a `git pull` followed by a `git push`. It first fetches and merges changes from the remote tracking branch (pull) and then uploads your local commits to the remote (push), synchronizing the state in both directions.

4.  **What would "Publish Branch" have done with multiple commits?**
    It would have pushed the new branch to GitLab along with **all** of the commits you had made on that branch locally. The remote repository would then have the complete history of your work on that branch.

5.  **Why is local conflict resolution often easier?**
    A local IDE like VS Code has access to the entire project's context. Its merge editor provides a three-way comparison (base, incoming, current), syntax highlighting, and allows you to run local tests to verify your resolution is correct *before* you finalize the merge commit. The web UI is more limited and is best for simple, line-based conflicts.