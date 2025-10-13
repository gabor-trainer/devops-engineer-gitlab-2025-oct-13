### **Enterprise Merge Request**

#### **1. The Principle: An MR is a Professional Proposal, Not Just a Code Dump**

The book shows you the mechanics of creating a Merge Request. Now, we will teach you the art. In a professional environment, an MR is the primary artifact of your work. It is a formal proposal to change the official codebase. A poorly written MR wastes your teammates' time, slows down the review process, and creates a confusing historical record.

An **enterprise-grade MR** tells a complete story. It explains **what** changed, **why** it changed, and **how** the reviewer can verify its correctness. It is a tool for communication, not just a technical process.

#### **2. The Anatomy of a High-Quality Merge Request**

A professional MR is composed of several key elements, each with a specific purpose.

##### **2.1. The Title: Clear, Concise, and Conventional**
The title is the first thing your team sees. It must be immediately understandable.

*   **Best Practice:** Follow the **Conventional Commits** format. This makes the title machine-readable and instantly communicates the nature of the change.
*   **Good Example:** `feat(api): add pagination to the /users endpoint`
*   **Bad Example:** `Update code` or `Fix bug`
*   **Pro Tip:** If your MR consists of a single commit, GitLab will automatically use that commit's subject line as the MR title. By writing good commit messages, you get good MR titles for free.

##### **2.2. The Description: The Story of Your Change**
This is the most critical part of the MR. Do not leave it blank. A good description provides the context that a reviewer needs to understand your work without having to ask you questions.

*   **Best Practice:** Use a **Merge Request Template**. Most enterprise projects configure a default template that pre-populates the description with key sections.
*   **A Standard Template Structure:**

    ```markdown
    ## What does this MR do and why?

    <!--
    Provide a clear, high-level description of the problem this MR solves
    and the approach you took.
    -->

    ## How to set up and validate locally

    <!--
    Provide a step-by-step guide for the reviewer to test your changes
    on their local machine. Be specific.
    1. `git checkout <this-branch>`
    2. `npm install`
    3. `npm run test`
    4. To see the UI change, navigate to http://localhost:3000/settings and verify...
    -->

    ## Related Issue(s)

    <!--
    Use a closing keyword to automatically link and close the relevant issue.
    This provides a clear link between the planned work and its implementation.
    -->
    Closes #42
    ```

##### **2.3. The Commits: A Clean and Logical History**
The commit history within your MR should tell a clear, step-by-step story of how you built the feature. It should not be a messy diary of every mistake and correction.

*   **Best Practice:** Use **interactive rebase** (`git rebase -i`) before pushing your branch to clean up your commit history.
    *   **Squash** small, trivial commits (e.g., "fix typo," "oops") into larger, more meaningful commits.
    *   **Reword** commit messages to be clear and consistent.
    *   **Reorder** commits to tell a more logical story.
*   **Good Commit History:**
    ```
    feat(api): add initial data model for user profiles
    feat(api): implement GET /users/:id endpoint
    test(api): add unit tests for user profile endpoint
    docs(api): update OpenAPI spec for new user profile endpoint
    ```
*   **Bad Commit History:**
    ```
    wip
    fix typo
    forgot to add file
    it works now
    final changes
    ```

##### **2.4. The Pipeline: The Automated Quality Gate**
Before you even assign a reviewer, your CI/CD pipeline must be **green**.

*   **Best Practice:** Never assign a reviewer to an MR with a failing pipeline. A failing pipeline means the code is, by definition, not ready for review. It is a sign of disrespect for the reviewer's time.
*   **Workflow:**
    1.  Push your branch and create the MR.
    2.  Wait for the pipeline to complete.
    3.  If it fails, fix the issues, commit, and push again.
    4.  Only when the pipeline is green should you assign a reviewer.

##### **2.5. The Reviewer and Other Metadata**
Use GitLab's UI features to provide clear signals about the MR's status.

*   **Reviewers:** Formally assign one or more teammates as reviewers. This sends them a notification and adds the MR to their to-do list. Do not rely on just posting a link in a chat.
*   **Draft Status:** If your MR is not yet ready for review, mark it as a **Draft** (by starting the title with `Draft:`). This prevents it from being accidentally merged and clearly signals that it is a work-in-progress.
*   **Labels:** Apply relevant labels (e.g., `feature`, `bug`, `backend`, `security`) to help categorize the MR and make it easier for others to find and understand.

#### **3. Example: A Tale of Two Merge Requests**

##### **The "Bad" MR**
*   **Title:** `fixes`
*   **Description:** (empty)
*   **Commits:** `wip`, `fix`, `final`
*   **Pipeline:** Failing
*   **Reviewer:** Unassigned

**Result:** A reviewer has no idea what this change does, why it was made, or how to test it. They must either reject it or spend significant time trying to understand the code from scratch. This is slow and inefficient.

##### **The "Enterprise" MR**
*   **Title:** `fix(auth): resolve incorrect password validation logic`
*   **Description:**
    > This MR fixes a critical bug where the password validation was not correctly enforcing the minimum character length.
    >
    > **Validation:**
    > 1. Run `npm run test:auth`. All tests should pass.
    > 2. Manually try to sign up with a 5-character password. The UI should now correctly display an error message.
    >
    > Closes #123
*   **Commits:** `fix(auth): enforce MIN_PASSWORD_LENGTH in validation service`, `test(auth): add test case for short password failure`
*   **Pipeline:** Green
*   **Reviewer:** `@gabor`

**Result:** The reviewer immediately understands the problem, the solution, and how to verify it. The review is fast, efficient, and focused. The historical record is perfect.

---

Crafting a high-quality Merge Request is a fundamental skill of a senior developer. It is an act of professional empathy for your teammates. By treating every MR as a clear, complete, and verifiable proposal, you contribute to a culture of quality, speed, and effective collaboration.