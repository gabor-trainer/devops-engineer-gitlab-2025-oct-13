# **Lab 1.4: Managing Merge Conflicts**

**Module:** 1.3: Professional Local Git Workflow
**Time:** Approx. 40 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Understand the conditions that lead to a merge conflict.
*   Intentionally create a merge conflict in a controlled scenario.
*   Identify and interpret merge conflict markers (e.g., `<<<<<<<`, `=======`, `>>>>>>>`).
*   Resolve a merge conflict using the Visual Studio Code merge editor.
*   Push a conflict resolution to an open Merge Request in GitLab.

### **2. Scenario**

Merge conflicts are a natural and unavoidable part of collaborative software development. They occur when two developers make competing changes to the same lines of a file on different branches. As a senior developer, you must be proficient at identifying, understanding, and safely resolving these conflicts.

In this lab, you will intentionally create a merge conflict to simulate a common real-world scenario. You will create a feature branch and open a Merge Request. While your MR is open for review, a teammate (simulated by you) will merge another change into the `main` branch that directly conflicts with your work. Your task is to pull these new changes from `main`, resolve the conflict on your local machine using the powerful tools in VS Code, and update your Merge Request so it can be merged safely.

### **3. Prerequisites**

*   You have successfully completed **Lab 1.3** and have a local clone of your `My First Project`.
*   You have opened the project folder in Visual Studio Code.
*   Your local repository is clean, and you are on the `main` branch.

### **4. Steps**

_**Note:** This lab requires careful execution of steps to create the conflict correctly. You will switch between acting as "yourself" on a feature branch and "a teammate" on the `main` branch._

1.  **Ensure Your Local `main` is Up-to-Date**
    Before starting any new work, always ensure your local `main` branch is synchronized with the remote GitLab repository.
    *   **[WHERE: Visual Studio Code]**
        *   Ensure you are on the `main` branch (check the bottom-left corner).
        *   Click the "Sync Changes" button (circular arrows icon) in the bottom-left corner or run `git pull` in the terminal.

2.  **Create Your Feature Branch**
    You will now create a branch to work on a new feature.
    *   **[WHERE: Visual Studio Code]**
        *   Create a new branch named `feature/add-contact-info`.
        *   In the `README.md` file, add the following lines at the very **end** of the file:
        ```markdown
        ## Contact Information

        For questions, please contact the development team.
        ```
        *   Stage and commit this change with the message `feat: add contact section`.
        *   Push the new branch to GitLab by clicking **"Publish Branch"**.

3.  **Create a Merge Request for Your Feature**
    *   **[WHERE: GitLab UI - Browser]**
        *   Navigate to your project in GitLab. A notification banner will appear for your new branch.
        *   Click **"Create merge request"** and then **"Create merge request"** again to open the MR. **Leave this MR open.**

4.  **Simulate a Conflicting Change from a Teammate**
    Now, you will switch roles and act as a teammate who is unaware of your work and merges a conflicting change directly into `main`.
    *   **[WHERE: Visual Studio Code]**
        *   Switch back to your `main` branch.
        *   In the `README.md` file, add the following *different* lines at the very **end** of the file:
        ```markdown
        ## Support

        For support, please file an issue in the project tracker.
        ```
        *   Stage and commit this change directly to `main` with the message `docs: add support section`.
        *   Push this commit to `main` by clicking the **"Sync Changes"** button.

5.  **Identify the Conflict**
    Your Merge Request in GitLab is now out of date and contains a conflict.
    *   **[WHERE: GitLab UI - Browser]**
        *   Refresh your open Merge Request page. You will now see a message: **"Merge blocked: merge conflicts must be resolved."** This confirms the conflict exists.

6.  **Resolve the Conflict Locally**
    The professional way to resolve conflicts is on your local machine.
    *   **[WHERE: Visual Studio Code]**
        *   Switch back to your feature branch, `feature/add-contact-info`.
        *   To bring the conflicting change from `main` into your branch, run the following command in the VS Code terminal:
        ```bash
        git pull origin main
        ```
        *   Git will immediately fail with a `CONFLICT (content): Merge conflict in README.md` error.
        *   In the VS Code Source Control panel, `README.md` will now appear under **"Merge Changes."** Click on it.

7.  **Use the VS Code Merge Editor**
    VS Code provides a powerful UI for resolving conflicts.
    *   **[WHERE: Visual Studio Code]**
        *   The editor will open showing the conflict markers. You will see your "Current Change" (`## Contact Information...`) and the "Incoming Change" (`## Support...`).
        *   Your task is to create a final version that includes **both** sections. You must do this by manually editing the code.
        *   Delete the conflict markers (`<<<<<<< HEAD`, `=======`, `>>>>>>> main`).
        *   Arrange the text so both new sections are present, for example:
        ```markdown
        ## Support

        For support, please file an issue in the project tracker.

        ## Contact Information

        For questions, please contact the development team.
        ```
        *   Save the `README.md` file.

8.  **Commit and Push the Resolution**
    Finalize the merge and update your Merge Request.
    *   **[WHERE: Visual Studio Code]**
        *   In the Source Control panel, stage the now-resolved `README.md` file by clicking the `+` icon next to it.
        *   The commit message box will be pre-filled with "Merge branch 'main' into...". You can leave this as is.
        *   Click the **Commit** button. This creates a merge commit on your local feature branch.
        *   Finally, push the resolution to GitLab by clicking the **"Sync Changes"** button.

### **5. Verification**

1.  **Verify the Merge Request:** In the GitLab UI, refresh your Merge Request page. The "Merge blocked" error will be gone, and the "Merge" button will be active.
2.  **Inspect the Changes:** Click the "Changes" tab in the MR. You will see the final, correctly resolved content of `README.md`.
3.  **Check the Commits:** Click the "Commits" tab. You will see your original commit (`feat: add contact section`) followed by your new merge commit (`Merge branch 'main' into...`).

### **6. Discussion**

This lab demonstrated a critical, real-world scenario in collaborative development. A **merge conflict** occurs when Git cannot automatically decide how to combine two competing changes made to the same lines of a file. It is not an error, but a signal that Git needs a human to make the final decision.

You resolved the conflict using the professional **local-first** approach. By pulling the latest `main` into your feature branch, you initiated the merge on your own machine. This allowed you to use the powerful merge editor in VS Code to see both sets of changes and manually craft the correct, final version. Committing this resolution and pushing it updated your remote branch, which automatically resolved the conflict in the Merge Request, making it ready for final review and integration.

### **7. Questions**

1.  What is the purpose of the `<<<<<<< HEAD`, `=======`, and `>>>>>>> [branch-name]` markers that Git adds to a file during a conflict?
2.  In this lab, you resolved the conflict by creating a *merge commit*. What is an alternative Git strategy you could have used to update your branch with changes from `main` that results in a cleaner, linear history?
3.  What is one proactive action a developer can take on a long-running feature branch to minimize the chance of large merge conflicts?
4.  Why did the conflict *not* appear when you first created the Merge Request, but only after the "teammate's" change was pushed to `main`?
5.  If the conflict had been in a binary file (like an image) instead of a text file, how would the resolution process have been different?

---

### **8. Solution**

This lab is process-oriented. The key artifacts are the commands and the final state of the repository.

#### **8.1. Final `README.md`**
The final `README.md` file on the `main` branch (after the MR is eventually merged) will contain both the "Support" and "Contact Information" sections.

#### **8.2. Command Summary**
```bash
# On your feature branch
git push -u origin feature/add-contact-info

# On main branch (simulating teammate)
git checkout main
# ...edit file...
git add README.md
git commit -m "docs: add support section"
git push

# Back on your feature branch to resolve
git checkout feature/add-contact-info
git pull origin main
# (Manual resolution in VS Code)
git add README.md
git commit
git push
```

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **What is the purpose of the conflict markers?**
    These markers are Git's way of showing you the two conflicting blocks of code directly in the file. The code between `<<<<<<< HEAD` and `=======` is *your* change (the "HEAD" of your current branch). The code between `=======` and `>>>>>>> [branch-name]` is the *incoming* change from the branch you are merging. Your job is to edit this block to create the final correct version and remove the markers.

2.  **What is an alternative to a merge commit?**
    You could have used `git rebase main` instead of `git merge main`. Rebasing rewrites your branch's history by re-applying your commits on top of the latest `main`. You would still resolve the conflict, but the end result would be a clean, linear history without an extra merge commit, which many teams prefer.

3.  **How to proactively minimize conflicts?**
    On a long-running feature branch, you should frequently pull or rebase from `main` (e.g., `git pull origin main` or `git rebase origin/main`). This allows you to integrate changes from the rest of the team in small, manageable chunks and resolve minor conflicts as they happen, rather than facing one giant conflict at the end.

4.  **Why did the conflict appear later?**
    When the Merge Request was first created, the feature branch could be cleanly merged into `main` because `main` had not changed since the feature branch was created. The conflict was only introduced *after* the "teammate" pushed a competing change to `main`. GitLab continuously re-evaluates the mergeability of an MR every time its source or target branch is updated.

5.  **How would resolving a binary file conflict be different?**
    Git cannot merge binary files, so it cannot show you `<<<<<<<` markers. When a conflict occurs in a binary file, Git will save both versions (e.g., `image.png.ours`, `image.png.theirs`). You would have to use an external tool (like an image editor) to inspect both versions and then manually tell Git which version to keep for the final commit using `git checkout --ours image.png` or `git checkout --theirs image.png`.