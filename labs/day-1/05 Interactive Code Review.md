# **Lab 1.5: Interactive Code Review**

**Module:** 1.4: Branching Strategies & Collaboration
**Time:** Approx. 40 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Navigate to and review a peer's open Merge Request (MR).
*   Leave line-specific comments to ask questions and provide feedback as part of a formal review.
*   Use the "Suggestion" feature to propose specific, actionable code changes.
*   Apply a received suggestion as the author of an MR with a single click.
*   Formally approve an MR to signify that it meets quality standards.

### **2. Scenario**

In a professional DevOps team, no code is merged into the `main` branch without a peer review. This process is the primary gate for ensuring code quality, sharing knowledge, and mentoring teammates. Your team has just adopted GitLab, and your lead has mandated that all Merge Requests must be reviewed and approved by at least one other developer before being merged.

In this lab, you will act as both the **author** of an MR and the **reviewer** of a teammate's MR. You will practice giving and receiving constructive feedback directly within the GitLab UI, using its powerful, integrated code review tools.

### **3. Prerequisites**

*   You have successfully completed the previous labs.
*   You have a project created within our `gabors-gitlab-training-20251013` group.
*   You have an open Merge Request ready for review. If you have already merged all your previous MRs, you must create a new one for this exercise:
    1.  Create a new branch locally (e.g., `feature/review-content`).
    2.  Edit the `README.md` file to add one or two new lines of text.
    3.  Commit and push this branch.
    4.  Create a new Merge Request in the GitLab UI.
*   You will be paired with another student in the class.

### **4. Steps**

_**Note:** This lab is interactive. You will switch between acting as a **Reviewer** on your partner's MR and an **Author** on your own MR._

1.  **Pair Up and Assign Reviewers**
    *   **[WHERE: GitLab UI - Browser]**
        *   Pair up with a teammate and exchange project names.
        *   Navigate to your own open Merge Request.
        *   In the right-hand sidebar, find the **Reviewers** section and click **"Edit"**.
        *   Select your partner from the list to formally assign them as the reviewer. They will receive a notification.

2.  **Start a Formal Review**
    As a **Reviewer**, your first task is to examine the changes and ask a clarifying question.
    *   **[WHERE: GitLab UI - Browser - On your partner's MR]**
        *   Navigate to the Merge Request you have been assigned to review.
        *   Click on the **"Changes"** tab to see the code differences (the "diff").
        *   Hover your mouse over the line number of one of the new lines your partner added. A comment bubble icon will appear. Click it.
        *   A comment box will open. Type a question, for example: `Is this the final wording for this section?`
        *   Instead of posting immediately, click the **"Start a review"** button. This batches your feedback into a single, formal review.

3.  **Propose a Change with a Suggestion**
    Next, as a **Reviewer**, you will propose a specific improvement.
    *   **[WHERE: GitLab UI - Browser - On your partner's MR]**
        *   Hover over a different line of code your partner added and click the comment bubble icon again.
        *   In the comment toolbar, click the **"Insert suggestion"** icon (looks like a document with a plus sign).
        *   GitLab will automatically quote your partner's original line inside a `suggestion` block. Modify the text inside this block to your proposed version (e.g., change one word or fix a typo).
        *   Click the **"Add to review"** button to add this to your pending review.

4.  **Submit Your Batched Review**
    *   **[WHERE: GitLab UI - Browser - On your partner's MR]**
        *   At the top of the page, you will see a "Pending comments" bar. Click the **"Finish review"** button.
        *   A pop-up will appear summarizing all your comments and suggestions. Click **"Submit review"**. Your complete feedback is now delivered to the author in a single notification.

5.  **Receive Feedback and Apply the Suggestion**
    Now, switch roles. You are the **Author** of your own MR.
    *   **[WHERE: GitLab UI - Browser - On YOUR MR]**
        *   Navigate to your open Merge Request. You will see the review comments and suggestions left by your partner in the "Overview" tab.
        *   Find the suggestion they made.
        *   Click the **"Apply suggestion"** button. GitLab will automatically create a new commit on your branch with this change and push it for you.

6.  **Verify and Approve the Merge Request**
    Finally, as the **Reviewer**, you will confirm the changes have been applied and give your final approval.
    *   **[WHERE: GitLab UI - Browser - On your partner's MR]**
        *   Refresh the Merge Request page. You will see a new commit in the history indicating your suggestion was applied.
        *   At the top of the "Overview" tab, click the **"Approve"** button. The MR now has your official approval, satisfying the review gate.

### **5. Verification**

1.  **Verify Approval:** On your own Merge Request, you should see a green checkmark and the text "Approved by [Partner's Name]".
2.  **Verify Suggestion Commit:** In your MR's **"Commits"** tab, you will see a new commit with a message like "Apply suggestion to [file]". This commit was created automatically by GitLab when you applied the suggestion.
3.  **Verify Discussion:** In the "Overview" tab, you will see the discussion thread you had with your partner. Reply to their original comment to acknowledge the change and then click **"Resolve thread"** to mark the discussion as complete.

### **6. Discussion**

This lab demonstrated the core of GitLab's collaborative code review process. The **Merge Request** serves as the central forum for all discussions related to a change. By leaving **line-specific comments**, you provided focused, contextual feedback. The key takeaway is the difference between an ad-hoc comment and a **formal review**, which batches comments to reduce notification noise for the author.

The **Suggestion** feature is a powerful tool for accelerating the review cycle. Instead of just describing a change, you provide the exact code, and the author can accept it with a single click. This is far more efficient than the author having to manually copy, paste, and commit the change themselves.

Finally, the **Approve** button provides a formal, auditable quality gate. It serves as a clear signal to the project maintainers that the code has been vetted by a peer and is ready for integration. This combination of automated CI pipelines and manual peer review creates a robust, two-part quality assurance process essential for any high-performing DevOps team.

### **7. Questions**

1.  What is the primary benefit to the author of a reviewer using the "Start a review" feature instead of posting several individual comments?
2.  If a reviewer leaves five suggestions in a single review, what is the most efficient way for the author to apply all of them?
3.  Why is it important to "Resolve thread" after a discussion point has been addressed? What does this signify to the team?
4.  If your project requires two approvals and you are the author, can you approve your own Merge Request to count as one of the two?
5.  In a real project, what are two or three key things a good reviewer should look for beyond just typos or style?

---

### **8. Solution**

This lab is process-oriented. The primary artifact is the final state of the Merge Request.

#### **8.1. Final State of the Merge Request**
*   The "Overview" tab should show at least one approval.
*   The "Commits" tab should include a commit with the message "Apply suggestion...".
*   The "Changes" tab should reflect the final, merged state of the code, including the applied suggestion.
*   All discussion threads on the "Overview" tab should be marked as resolved.

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **Benefit of "Start a review"?**
    It reduces notification spam. The author receives a single, consolidated notification that a full review is complete, rather than receiving a separate email or alert for every single comment the reviewer posts. This allows the author to address all feedback in one focused session.

2.  **How to apply multiple suggestions?**
    If a reviewer has submitted multiple suggestions in a single review, the author will see an **"Apply batch suggestion"** button. This allows them to apply all suggestions with a single click, creating a single, clean commit that incorporates all the proposed changes.

3.  **Why "Resolve thread"?**
    Resolving a thread acts as a visual checklist, signaling that a specific point of feedback or a question has been fully addressed and is considered "done." It provides a clear, at-a-glance status to the author, reviewers, and maintainers, indicating that the MR is closer to being ready to merge.

4.  **Can you approve your own MR?**
    By default, yes, an author can approve their own MR. However, this is considered a bad practice. Professional, enterprise-grade GitLab instances have a setting (`Prevent author approval`) that can be enabled at the project level to forbid this, ensuring that all approvals come from a true peer.

5.  **What should a good reviewer look for?**
    A good reviewer looks for:
    *   **Correctness and Logic:** Does the code correctly solve the problem described in the issue? Are there any logical errors, off-by-one mistakes, or missed edge cases?
    *   **Maintainability and Readability:** Is the code clean, well-structured, and easy for another developer to understand? Does it follow the team's established coding standards and best practices?
    *   **Security and Performance:** Does the change introduce any potential security vulnerabilities (like SQL injection) or performance bottlenecks (like an inefficient database query)?