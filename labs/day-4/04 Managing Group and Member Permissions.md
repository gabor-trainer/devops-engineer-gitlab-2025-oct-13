# **Lab: Managing Group and Member Permissions**

**Module:** Organizing Teams & Managing Permissions
**Time:** Approx. 35 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Create a new subgroup to organize projects.
*   Invite a team member to a project with a specific role (`Developer`).
*   Experience the limitations of the `Developer` role when interacting with a protected branch.
*   Promote a team member's role to `Maintainer`.
*   Verify the expanded permissions of the `Maintainer` role.

### **2. Scenario**

Your team is growing, and your project structure needs to scale with it. To better organize your work, your team lead has asked you to create a new **subgroup** for a new set of microservices. Additionally, a new developer has joined the team, and you need to grant them the standard `Developer` role on a specific project.

In this lab, you will first act as a team lead to create the new organizational structure. You will then invite a peer to your project and guide them as they attempt an action that their `Developer` role forbids. Finally, you will promote them to `Maintainer` and have them repeat the action, demonstrating the direct impact of GitLab's role-based permission model.

### **3. Prerequisites**

*   You have a GitLab.com account and are a member of the `gabors-gitlab-training-20251013` group with the `Maintainer` role.
*   You will be paired with another student in the class.

### **4. Steps**

_**Note:** This is an interactive lab. One person will act as the **Project Maintainer**, and the other will act as the **New Developer**. You will then switch roles._

#### **Part 1: Acting as the Project Maintainer**

1.  **Create a New Project and Subgroup**
    First, you will set up the project and organizational structure.
    *   **[WHERE: GitLab UI - Browser]**
        *   Navigate to the `gabors-gitlab-training-20251013` group dashboard.
        *   Click **"New project"** and create a new blank project named `student-<firstname>-<lastname>-lab-3.5`. Initialize it with a `README.md`.
        *   Navigate back to the group dashboard. Click **"New subgroup"**.
        *   **Subgroup name:** `My Team's Services`.
        *   **Visibility:** `Private`.
        *   Click **"Create subgroup"**.

2.  **Invite Your Teammate as a Developer**
    Now, you will onboard your partner to your new project with limited permissions.
    *   **[WHERE: GitLab UI - Browser]**
        *   Navigate to your `student-<firstname>-<lastname>-lab-3.5` project.
        *   In the left-hand navigation, go to `Manage -> Members`.
        *   Click **"Invite members"**.
        *   Enter your partner's GitLab username.
        *   Under "Select a role," choose **`Developer`**.
        *   Click **"Invite"**.

#### **Part 2: Acting as the New Developer**

Your partner is now the "New Developer." They will attempt an action that their role should prevent.

3.  **Attempt to Commit to the `main` Branch**
    *   **[WHERE: GitLab UI - Browser - As the New Developer]**
        *   Navigate to the project you were just invited to.
        *   On the project's main page, click on the `README.md` file.
        *   Click the **"Edit"** dropdown and select **"Edit single file"**.
        *   Make a small change to the text.
        *   Scroll down to the "Commit message" section. **Observe:** The target branch is `main`.
        *   Click **"Commit changes"**.

4.  **Observe the Permission Error**
    *   **[WHERE: GitLab UI - Browser - As the New Developer]**
        *   **Expected Result:** GitLab will block the action with an error message: **"You are not allowed to push into this branch."** This is because the `main` branch is protected by default, and the `Developer` role does not have permission to push directly to it.

#### **Part 3: Promoting Roles and Verifying Permissions**

Now, you will switch back to the "Project Maintainer" role to grant your partner elevated permissions.

5.  **Promote Your Teammate to Maintainer**
    *   **[WHERE: GitLab UI - Browser - As the Project Maintainer]**
        *   Navigate back to your project's `Manage -> Members` page.
        *   Find your partner in the member list. In the "Max role" column, click the dropdown (which currently says `Developer`).
        *   Select **`Maintainer`**. The change is saved automatically.

6.  **Re-attempt to Commit to the `main` Branch**
    *   **[WHERE: GitLab UI - Browser - As the New Developer]**
        *   Repeat the exact same steps from Step 3: edit the `README.md` file and attempt to commit the changes directly to the `main` branch.

### **5. Verification**

1.  **Verification for Step 4:** The "New Developer" **must** receive the "You are not allowed to push into this branch" error. This confirms the `Developer` role is correctly restricted.
2.  **Verification for Step 6:** The "New Developer" (who is now a `Maintainer`) **must** be able to successfully commit the change directly to the `main` branch. This confirms the `Maintainer` role has elevated permissions.
3.  **Check the Commit History:** As the Project Maintainer, navigate to `Code -> Commits`. You should see the new commit on the `main` branch, authored by your partner.

### **6. Discussion**

This lab provided a practical, hands-on demonstration of GitLab's **Role-Based Access Control (RBAC)**. You experienced firsthand the different permission levels associated with two of the most common roles: `Developer` and `Maintainer`.

The `Developer` role is designed for day-to-day contribution following the standard Merge Request workflow. For safety and quality control, it is restricted from making direct changes to critical, **protected branches** like `main`.

The `Maintainer` role is designed for team leads or senior developers who are responsible for the overall health and integrity of the repository. They have elevated permissions, including the ability to push directly to protected branches, manage project settings, and approve merge requests from other developers. This lab clearly illustrated that permissions in GitLab are not just suggestions; they are strictly enforced by the platform to ensure a secure and orderly development process.

### **7. Questions**

1.  What is the primary reason for protecting the `main` branch in a project?
2.  Besides being able to push to a protected branch, what is another key permission that a `Maintainer` has but a `Developer` does not? (Hint: Think about project settings).
3.  In our setup, the `Developer` could not create new branches in the project. What specific project setting did the instructor have to configure to allow this?
4.  If a user is a `Developer` at the group level but is invited to a specific project as a `Maintainer`, what is their effective role within that project?
5.  Why is it a good security practice to give most team members the `Developer` role by default, rather than making everyone a `Maintainer`?

---

### **8. Solution**

This lab is process-oriented. The key artifact is the final state of the project and its members.

#### **8.1. Final State**
*   **Subgroup:** A new subgroup named `My Team's Services` exists within the main training group.
*   **Project Members:** The `student-<firstname>-<lastname>-lab-3.5` project has two members: you (as `Owner` or `Maintainer`) and your partner (promoted to `Maintainer`).
*   **Commit History:** The `main` branch has a commit authored by your partner.

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **Why protect `main`?**
    To ensure the `main` branch is always stable, high-quality, and deployable. It forces all changes to go through a controlled process (the Merge Request), which includes automated CI checks and peer code review, before being integrated.

2.  **Another `Maintainer` permission?**
    A `Maintainer` can edit project settings (`Settings -> General`), manage project members (`Manage -> Members`), and configure protected branches and other repository rules. A `Developer` cannot access these sensitive settings.

3.  **What setting allowed `Developers` to create branches?**
    In `Settings -> Repository -> Protected branches`, the instructor had to add a wildcard (`*`) rule that explicitly granted `Developers + Maintainers` the permission to "push and create" branches.

4.  **What is the effective role?**
    The user's effective role in that specific project would be **`Maintainer`**. GitLab always applies the *highest* permission level a user has been granted for a specific resource.

5.  **Why default to the `Developer` role?**
    This follows the **Principle of Least Privilege**. By granting users only the permissions they need for their day-to-day work, you reduce the risk of accidental or malicious damage. A developer doesn't need to change project settings or force-push to `main` to do their job, so those permissions should not be granted by default.