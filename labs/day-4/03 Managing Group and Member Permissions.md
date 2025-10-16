# **Lab: Managing Group and Member Permissions**

**Module:** Organizing Teams & Managing Permissions  
**Time:** Approx. 50 minutes (including profile setup)

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

In this lab, you will act in two different roles: first as a **team lead/project maintainer** to create the organizational structure, and then as a **new developer** joining the team. You will experience firsthand how GitLab's role-based permission model directly impacts what actions each role can perform. By working through both perspectives yourself, you'll gain a deeper understanding of how permissions shape collaboration in GitLab.

### **3. Prerequisites**

#### **3.1. Required Accounts and Access**
*   You have a GitLab.com account and are a member of the `gabors-gitlab-training-20251013` group with the `Maintainer` role. (This is your existing account from Day 1.)
*   You are using a Windows computer with Microsoft Edge browser.

#### **3.2. Setting Up Two Browser Profiles**

In this lab, you will simulate two different users by creating two separate browser profiles in Microsoft Edge. This allows you to be logged into GitLab as two different users simultaneously.

**Step 1: Create the First Profile (Maintainer Profile)**

1.  **Close all Edge browser windows.**
2.  **Right-click on your Desktop** and select **New > Shortcut**.
3.  **In the location field, enter the following command** (copy and paste exactly):
    ```
    "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --user-data-dir="%LOCALAPPDATA%\EdgeProfile_Maintainer" --profile-directory="Maintainer"
    ```
4.  Click **Next**.
5.  **Name the shortcut:** `GitLab Maintainer`
6.  Click **Finish**.
7.  **Double-click the new "GitLab Maintainer" shortcut** to launch Edge with the Maintainer profile.
8.  **A fresh Edge window will open.** You may see the Edge welcome screen. Complete the initial setup if prompted (you can skip signing into Microsoft account).
9.  **Navigate to `https://gitlab.com`** in this browser window.
10. **Log in using your existing GitLab account** (the one you created on Day 1 of training that is already a member of the `gabors-gitlab-training-20251013` group).
11. **Verify your access:** Navigate to the `gabors-gitlab-training-20251013` group dashboard to confirm you can see it.
12. **Keep this browser window open** (you can minimize it for now).

**Step 2: Create the Second Profile (Developer Profile)**

1.  **Right-click on your Desktop** and select **New > Shortcut**.
2.  **In the location field, enter the following command** (copy and paste exactly):
    ```
    "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --user-data-dir="%LOCALAPPDATA%\EdgeProfile_Developer" --profile-directory="Developer"
    ```
3.  Click **Next**.
4.  **Name the shortcut:** `GitLab Developer`
5.  Click **Finish**.
6.  **Double-click the new "GitLab Developer" shortcut** to launch Edge with the Developer profile.
7.  **A second, completely separate Edge window will open.** Complete the initial Edge setup if prompted.

**Step 3: Create a New GitLab Account for the Developer Profile**

You will now create a brand new GitLab account to simulate a new team member joining your project.

**IMPORTANT: Email Requirement**

GitLab requires email verification for all accounts, and your trial account will need to send confirmation codes. You **must** use a different email address than your existing GitLab account.

**If you do not have another email address available:**
1.  In the Developer profile browser window, navigate to `https://outlook.com`
2.  Click **"Create free account"**
3.  Create a new email address (e.g., `yourname.dev@outlook.com`)
4.  **Note:** Outlook.com will NOT require a phone number for verification
5.  Complete the account creation process
6.  **Keep this email address accessible** - you will need it for GitLab verification

**Now create the GitLab account:**

1.  **In the Developer profile browser window,** navigate to `https://gitlab.com`
2.  Click **"Register now"** or **"Get free trial"**
3.  **Create a new GitLab account** using your alternate email address (e.g., your new Outlook email or another existing email)
4.  **Username suggestion:** Use a variation like `yourname-dev` or `yourname-developer` to distinguish it from your main account
5.  **Complete the registration process** exactly as you did on Day 1:
    *   Verify your email address (check your inbox for the verification code)
    *   Complete any required profile information
    *   Skip the trial signup if prompted (you only need a free account for this role)
6.  **Once logged in successfully,** keep this browser window open.

**Step 4: Verify Your Setup**

You should now have:
*    **Two desktop shortcuts:** "GitLab Maintainer" and "GitLab Developer"
*    **Two separate Edge browser windows open:**
    *   **Maintainer window:** Logged into GitLab with your original training account
    *   **Developer window:** Logged into GitLab with your newly created developer account
*    **Both accounts are fully verified and functional**

** TIP:** To easily distinguish between the two windows, you can arrange them side-by-side on your screen, or use Alt+Tab to switch between them. The browser window title will show different usernames.

---

### **4. Steps**

_**Note:** You will be switching between your two browser profiles throughout this lab. Clear visual indicators show which profile to use for each step._

#### **Part 1: Acting as the Project Maintainer**

**USE: GitLab Maintainer Profile** (your original training account)

1.  **Create a New Project and Subgroup**
    
    First, you will set up the project and organizational structure.
    
    *   **[WHERE: GitLab Maintainer Profile Browser Window]**
        *   Navigate to the `gabors-gitlab-training-20251013` group dashboard.
        *   Click **"New project"** and create a new blank project:
            *   **Project name:** `student-<firstname>-<lastname>-lab-permissions`
            *   **Visibility:** Private
            *    **Check** "Initialize repository with a README"
            *   Click **"Create project"**
        *   After the project is created, navigate back to the `gabors-gitlab-training-20251013` group dashboard.
        *   Click **"New subgroup"**.
        *   **Subgroup name:** `My Team's Services`
        *   **Subgroup URL:** Will auto-populate (this is fine)
        *   **Visibility:** `Private`
        *   Click **"Create subgroup"**.

2.  **Invite Your Developer Account as a Developer**
    
    Now, you will onboard your developer account to your new project with limited permissions.
    
    *   **[WHERE: GitLab Maintainer Profile Browser Window]**
        *   Navigate to your `student-<firstname>-<lastname>-lab-permissions` project.
        *   In the left-hand navigation, go to **Manage → Members**.
        *   Click **"Invite members"**.
        *   **Enter your Developer account's GitLab username** (e.g., `yourname-dev` or whatever username you created)
            *    **TIP:** If you don't remember the exact username, switch to your Developer profile window and check the username in the top-right corner
        *   Under "Select a role," choose **`Developer`**.
        *   **Expiration date:** Leave blank
        *   Click **"Invite"**.
        *   **Verification:** You should see your developer account appear in the members list with the "Developer" role.

#### **Part 2: Acting as the New Developer**

 **SWITCH TO: GitLab Developer Profile** (your newly created account)

Your developer account has now been invited to the project. You will attempt an action that this role should prevent.

3.  **Accept the Invitation and Access the Project**
    
    *   **[WHERE: GitLab Developer Profile Browser Window]**
        *   You may see a notification in GitLab, or you may need to check your email (the one you used for the developer account).
        *   **Navigate to:** `https://gitlab.com/dashboard/projects`
        *   You should see the `student-<firstname>-<lastname>-lab-permissions` project listed (it may take a moment to appear)
        *   **If you don't see it immediately:** Check your email for an invitation link and click it
        *   Click on the project name to open it.

4.  **Attempt to Commit Directly to the `main` Branch**
    
    *   **[WHERE: GitLab Developer Profile Browser Window]**
        *   On the project's main page, click on the `README.md` file to open it.
        *   Click the **"Edit"** dropdown button (on the right side) and select **"Edit single file"**.
        *   Make a small change to the text. For example, add a new line:
            ```
            ## Developer Change
            This line was added by the developer account.
            ```
        *   Scroll down to the "Commit message" section.
        *   **Observe:** The target branch is set to `main` (this is the default).
        *   **Do not change the target branch.** Leave it as `main`.
        *   Click **"Commit changes"**.

5.  **Observe the Permission Error**
    
    *   **[WHERE: GitLab Developer Profile Browser Window]**
        *   **Expected Result:** GitLab will block the action and display an error message: 
            ```
            "You are not allowed to push code to protected branches on this project."
            ```
            or similar wording indicating you cannot push to the `main` branch.
        *   **This demonstrates that the `Developer` role is restricted from pushing directly to protected branches.**
        *    **Your commit was NOT saved.** This is the correct behavior.

#### **Part 3: Promoting Roles and Verifying Permissions**

**SWITCH BACK TO: GitLab Maintainer Profile** (your original training account)

Now, you will grant your developer account elevated permissions to demonstrate the difference in roles.

6.  **Promote Your Developer Account to Maintainer**
    
    *   **[WHERE: GitLab Maintainer Profile Browser Window]**
        *   Navigate back to your project's **Manage → Members** page.
        *   Find your developer account in the member list.
        *   In the **"Max role"** column, click the dropdown (which currently shows `Developer`).
        *   Select **`Maintainer`**.
        *   The change is saved automatically. You should see the role update to "Maintainer" immediately.

 **SWITCH BACK TO: GitLab Developer Profile** (but now with Maintainer permissions!)

7.  **Re-attempt to Commit Directly to the `main` Branch**
    
    *   **[WHERE: GitLab Developer Profile Browser Window]**
        *   You may need to **refresh the page** for the new permissions to take effect.
        *   Navigate back to the `README.md` file.
        *   Click **"Edit" → "Edit single file"** again.
        *   Make another change to the text. For example, add another new line:
            ```
            ## Maintainer Change
            This line was added after promotion to maintainer role.
            ```
        *   Scroll down to the "Commit message" section.
        *   **Verify:** The target branch is still set to `main`.
        *   Click **"Commit changes"**.

---

### **5. Verification**

1.  **Verification for Step 5 (Developer role restriction):**
    *    Your developer account **must** have received the "You are not allowed to push code to protected branches" error (or similar).
    *    The commit was NOT saved to the `main` branch.
    *   This confirms the `Developer` role is correctly restricted from pushing to protected branches.

2.  **Verification for Step 7 (Maintainer role permissions):**
    *    Your developer account (now promoted to `Maintainer`) **must** be able to successfully commit the change directly to the `main` branch.
    *    You should see a success message after clicking "Commit changes."
    *   This confirms the `Maintainer` role has elevated permissions.

3.  **Final Verification - Check the Commit History:**
    
    *   **[WHERE: Either Profile - Both will see the same history]**
        *   Navigate to **Code → Commits** in your project.
        *   You should see the commit(s) on the `main` branch.
        *   **Verify:** The most recent commit should be authored by your **developer account's username** (e.g., `yourname-dev`).
        *   This proves that the developer account, after being promoted to Maintainer, successfully pushed directly to the protected `main` branch.

---

### **6. Discussion**

This lab provided a practical, hands-on demonstration of GitLab's **Role-Based Access Control (RBAC)** system. By acting as both a maintainer and a developer yourself, you experienced firsthand the different permission levels associated with these two common roles.

**Key Takeaways:**

*   **Protected Branches Enforce Workflow:** The `main` branch is protected by default in GitLab. This protection is not just a suggestion—it is actively enforced by the platform. When you attempted to push directly to `main` as a Developer, GitLab immediately blocked the action. This forces teams to follow a controlled workflow using Merge Requests, which include automated CI checks and peer code review.

*   **The Developer Role:** This role is designed for day-to-day development work. Developers can:
    *   Clone the repository
    *   Create new branches
    *   Push commits to non-protected branches
    *   Create and comment on Merge Requests
    *   However, they **cannot** push directly to protected branches or modify sensitive project settings.

*   **The Maintainer Role:** This role is designed for team leads, senior developers, or those responsible for the repository's overall health. In addition to all Developer permissions, Maintainers can:
    *   Push directly to protected branches (like `main`)
    *   Manage project members and their roles
    *   Edit project settings and repository rules
    *   Approve and merge Merge Requests
    *   Configure protected branches and CI/CD settings

*   **Principle of Least Privilege:** GitLab's permission model follows this security best practice. Most team members should have the `Developer` role by default, receiving only the permissions they need for their daily work. The `Maintainer` role should be granted sparingly to those who genuinely need elevated access. This reduces the risk of accidental or malicious changes to critical parts of the codebase.

*   **Permissions Are Hierarchical:** When a user has different roles at different levels (group vs. project), GitLab always applies the **highest** permission level for that specific resource. For example, if you're a Developer at the group level but invited as a Maintainer to a specific project, you will have Maintainer permissions within that project.

---

### **7. Questions**

1.  What is the primary reason for protecting the `main` branch in a project?
2.  Besides being able to push to a protected branch, name two other key permissions that a `Maintainer` has but a `Developer` does not.
3.  In a typical team workflow, why would a `Developer` create a Merge Request instead of pushing directly to `main` (even if they had permission)?
4.  If a user is a `Developer` at the group level but is invited to a specific project as a `Maintainer`, what is their effective role within that project?
5.  Why is it a good security practice to give most team members the `Developer` role by default, rather than making everyone a `Maintainer`?
6.  **Bonus:** In this lab, we manually promoted the developer account to Maintainer. In what real-world scenarios might you need to quickly promote or demote a team member's permissions?

---

### **8. Solution**

This lab is process-oriented. The key artifact is the final state of the project and its members.

#### **8.1. Final State**
*   **Subgroup:** A new subgroup named `My Team's Services` exists within the `gabors-gitlab-training-20251013` group.
*   **Project:** A project named `student-<firstname>-<lastname>-lab-permissions` exists with:
    *   Your maintainer account (original training account) as the project Owner
    *   Your developer account listed as a Maintainer (promoted from Developer)
*   **Commit History:** The `main` branch has at least one commit authored by your developer account (created after promotion to Maintainer).
*   **Browser Profiles:** Two functional Edge browser profiles with desktop shortcuts that can be used for future multi-account testing.

---

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**

1.  **Why protect `main`?**
    
    The `main` branch is protected to ensure it remains stable, high-quality, and always deployable. Protection enforces a controlled integration process where all changes must go through Merge Requests. This allows for:
    *   Automated CI/CD pipeline checks (tests, security scans, code quality analysis)
    *   Peer code review and approval
    *   Discussion and documentation of changes
    *   Ability to revert problematic changes
    
    Without protection, developers could accidentally push breaking changes directly to production code.

2.  **Two other `Maintainer` permissions:**
    
    a) **Manage Project Settings:** Maintainers can access and modify project settings (`Settings → General`), including repository settings, CI/CD configuration, merge request settings, and protected branch rules.
    
    b) **Manage Project Members:** Maintainers can invite new members to the project, change existing members' roles, and remove members from the project (`Manage → Members`).
    
    Additional examples: Configure webhooks, manage deploy keys, edit project visibility, delete the project, manage project integrations.

3.  **Why use Merge Requests even with permission?**
    
    Even if a developer had permission to push directly to `main`, using Merge Requests is a best practice because it:
    *   Enables peer code review, catching bugs and improving code quality
    *   Creates a documented history of why changes were made
    *   Allows for discussion and collaboration on the proposed changes
    *   Triggers automated testing and validation through CI/CD pipelines
    *   Provides a clear audit trail for compliance and troubleshooting
    *   Makes it easy to revert changes if issues are discovered
    
    Direct pushes to `main` should be reserved for emergency hotfixes only.

4.  **What is the effective role?**
    
    The user's effective role in that specific project would be **`Maintainer`**. GitLab always applies the *highest* permission level a user has been granted for a specific resource. Project-level permissions override group-level permissions when project-level permissions are higher.

5.  **Why default to the `Developer` role?**
    
    This follows the **Principle of Least Privilege**, a fundamental security principle. By granting users only the minimum permissions they need to perform their job, you:
    *   Reduce the risk of accidental damage (e.g., accidentally deleting important branches or misconfiguring project settings)
    *   Minimize the impact of compromised accounts (if an account is hacked, the attacker has limited permissions)
    *   Enforce better workflows (developers must use Merge Requests, ensuring code review and CI checks)
    *   Create clear accountability (only a small number of Maintainers can make critical changes)
    *   Simplify compliance and auditing
    
    A developer doesn't need to change project settings, force-push to `main`, or delete branches to do their daily work, so those permissions should not be granted by default.

6.  **Bonus Answer - Real-world permission change scenarios:**
    
    *   **Promotion:** A developer is promoted to senior developer or team lead and needs Maintainer permissions
    *   **Demotion/Offboarding:** An employee leaves the company or moves to a different team and should have permissions reduced or removed
    *   **Temporary Access:** A contractor or external consultant needs elevated permissions for a specific task, then should be downgraded afterward
    *   **Incident Response:** During a security incident, you may need to quickly restrict permissions to prevent further damage
    *   **Project Handover:** When ownership of a project transfers between teams, roles need to be adjusted
    *   **Compliance Requirements:** Audit findings may require adjusting permissions to meet security policies

---

### **10. Cleanup (Optional)**

If you want to keep your browser profiles for future labs, you can keep both desktop shortcuts. However, if you prefer to clean up:

*   You can delete the desktop shortcuts (this won't delete your GitLab accounts)
*   The Edge profile data will remain in `%LOCALAPPDATA%\EdgeProfile_Maintainer` and `%LOCALAPPDATA%\EdgeProfile_Developer` unless you manually delete these folders
*   Your developer GitLab account will remain active - you may want to keep it for future testing scenarios, or you can delete it from GitLab account settings

---

**End of Lab**