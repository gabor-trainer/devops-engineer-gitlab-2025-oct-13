### **The Enterprise Git Cheat Sheet**

#### **1. Getting Started & Setup**
*Initialize a project or get a copy of an existing one.*

| Command                                            | Description                                                      |
| :------------------------------------------------- | :--------------------------------------------------------------- |
| `git init`                                         | Initialize a new, empty Git repository in the current directory. |
| `git clone <url>`                                  | Create a local copy of a remote repository.                      |
| `git config --global user.name "Your Name"`        | Set the name that will be attached to your commits.              |
| `git config --global user.email "you@example.com"` | Set the email address that will be attached to your commits.     |
| `git remote add <name> <url>`                      | Add a new remote repository (e.g., `origin`).                    |

#### **2. The Core Workflow: Making Changes**
*The fundamental cycle of creating, staging, and saving your work.*

| Command                   | Description                                                                                 |
| :------------------------ | :------------------------------------------------------------------------------------------ |
| `git status`              | Check the current status of your working directory (modified, staged, and untracked files). |
| `git add <file>`          | Add a specific file's changes to the staging area for the next commit.                      |
| `git add .`               | Add all new and modified files' changes to the staging area.                                |
| `git commit -m "message"` | Create a new commit with all staged changes and a short message.                            |
| `git commit`              | Open a text editor to write a more detailed, multi-line commit message.                     |
| `git reset <file>`        | Unstage a specific file, but keep the changes in your working directory.                    |
| `git restore <file>`      | Discard all unstaged changes in a specific file, reverting it to its last committed state.  |

#### **3. Branching & Navigation**
*Isolate your work and switch between different lines of development.*

| Command                  | Description                                                                   |
| :----------------------- | :---------------------------------------------------------------------------- |
| `git branch`             | List all local branches. The current branch is marked with an asterisk (`*`). |
| `git branch <name>`      | Create a new branch.                                                          |
| `git switch <name>`      | Switch your working directory to an existing branch.                          |
| `git switch -c <name>`   | **(Modern Best Practice)** Create a new branch and immediately switch to it.  |
| `git checkout -b <name>` | (Legacy) Create a new branch and immediately switch to it.                    |
| `git branch -d <name>`   | Delete a local branch that has been merged.                                   |
| `git branch -D <name>`   | Force-delete a local branch, even if it has unmerged changes.                 |

#### **4. Syncing with the Remote Repository**
*Share your work with your team and get their latest updates.*

| Command                         | Description                                                                                                    |
| :------------------------------ | :------------------------------------------------------------------------------------------------------------- |
| `git fetch origin`              | Download all changes from the `origin` remote but do not apply them to your local branches.                    |
| `git pull origin main`          | **(Merge Strategy)** Fetch changes from `origin` and merge the `main` branch into your current branch.         |
| `git pull --rebase origin main` | **(Rebase Strategy)** Fetch changes from `origin` and re-apply your current branch's commits on top of `main`. |
| `git push`                      | Push your committed changes from your current branch to its remote tracking branch.                            |
| `git push -u origin <name>`     | Push a new local branch to the `origin` remote for the first time, setting it as the tracking branch.          |

#### **5. Inspecting History (Code Archaeology)**
*Understand the evolution of the project.*

| Command                     | Description                                                     |
| :-------------------------- | :-------------------------------------------------------------- |
| `git log`                   | Show the commit history for the current branch.                 |
| `git log --oneline --graph` | Show a condensed, graphical view of the commit history.         |
| `git log <file>`            | Show the history of commits that modified a specific file.      |
| `git diff`                  | Show all unstaged changes in your working directory.            |
| `git diff --staged`         | Show changes that are staged but not yet committed.             |
| `git show <commit>`         | Show the metadata and changes for a specific commit.            |
| `git blame <file>`          | Show who last modified each line of a file and in which commit. |

#### **6. Undoing & Modifying Commits**
*Correct mistakes and refine your commit history.*

| Command                   | Description                                                                                                       |
| :------------------------ | :---------------------------------------------------------------------------------------------------------------- |
| `git commit --amend`      | Modify the most recent commit. You can change its message or add newly staged files to it.                        |
| `git reset HEAD~1`        | **(Soft Undo)** Un-commit the last commit but keep all its changes in your working directory as unstaged changes. |
| `git reset --hard HEAD~1` | **(Hard Undo - DANGEROUS)** Permanently discard the last commit and all of its associated changes.                |
| `git rebase -i HEAD~N`    | Start an interactive rebase to edit the last `N` commits (e.g., squash, reword, reorder).                         |
| `git stash`               | Temporarily save all your uncommitted (staged and unstaged) changes so you can switch branches.                   |
| `git stash pop`           | Re-apply the most recently stashed changes to your working directory.                                             |

#### **7. Professional Workflow: Merge Request Preparation**
*The standard process taught in our training.*

| Step                 | Command                                                       | Description                                                               |
| :------------------- | :------------------------------------------------------------ | :------------------------------------------------------------------------ |
| **1. Sync `main`**   | `git switch main`<br>`git pull`                               | Ensure your local `main` branch is up-to-date with the remote repository. |
| **2. Create Branch** | `git switch -c feature/my-new-idea`                           | Create a new, descriptively named branch for your work.                   |
| **3. Do Work**       | `...`                                                         | Edit, create, and delete files.                                           |
| **4. Commit**        | `git add .`<br>`git commit -m "feat(scope): describe change"` | Stage and commit your changes using the Conventional Commits format.      |
| **5. Push**          | `git push -u origin feature/my-new-idea`                      | Push your new branch and its commits to GitLab.                           |
| **6. Create MR**     | (In GitLab UI)                                                | Go to GitLab and create a Merge Request from your pushed branch.          |