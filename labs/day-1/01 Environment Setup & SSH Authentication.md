# **Lab: Environment Setup & SSH Authentication**

**Module:** Introduction to DevOps & GitLab
**Time:** Approx. 30 minutes

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Create a GitLab.com account and join our central training group.
*   Generate a new, secure ED25519 SSH key pair on your local machine.
*   Add your public SSH key to your GitLab account settings.
*   Verify your SSH connectivity to GitLab.com, enabling secure, password-less Git operations.

### **2. Scenario**

As a professional developer or administrator joining a new team, your first task is to set up your development environment for secure access to the organization's source code repositories. Password-based authentication is inconvenient for daily use and less secure. The industry standard is SSH key authentication, which provides a robust, secure, and password-less method for interacting with Git remotes.

In this lab, you will perform this fundamental setup task, ensuring your local machine is trusted by GitLab and ready for all subsequent development work.

### **3. Prerequisites**

*   A working computer with a command-line terminal (e.g., Git Bash on Windows, Terminal on macOS/Linux).
*   A valid email address to register your GitLab.com account.
*   You have received an email invitation to join the `gabors-gitlab-training-20251013` group.

### **4. Steps**

_**Note:** This section provides the specific commands and UI navigation steps. The "Discussion" section will provide a deeper explanation of the concepts._

1.  **Create Your GitLab Account and Join the Group**
    First, you need to create your personal account and become a member of our shared training environment.
    *   Navigate to `gitlab.com` and sign up for a new, free account.
    *   Activate the 30-day Ultimate trial when prompted. **No credit card is required for this step.**
    *   Check your email for the invitation to the `gabors-gitlab-training-20251013` group and accept it.

2.  **Generate Your SSH Key Pair**
    Next, you will create a cryptographic key pair on your local machine. The private key stays on your computer, and the public key will be uploaded to GitLab.
    *   Open your terminal (or Git Bash on Windows).
    *   Run the following command, replacing `"your.email@example.com"` with the email address you used for your GitLab account. This command creates a secure ED25519 key, which is the current recommended standard.
    ```bash
    ssh-keygen -t ed25519 -C "your.email@example.com"
    ```
    *   When prompted to "Enter a file in which to save the key," press **Enter** to accept the default location (`~/.ssh/id_ed25519`).
    *   When prompted to "Enter passphrase," press **Enter** twice to leave it empty for this training.

3.  **Add Your Public SSH Key to GitLab**
    Now, you will provide GitLab with your public key, which allows it to authenticate you.
    *   First, display your public key in the terminal and copy its entire content to your clipboard.
    ```bash
    cat ~/.ssh/id_ed25519.pub
    ```
    *   In your web browser, navigate to your GitLab.com account.
    *   In the top-right corner, click your avatar and select **Preferences**.
    *   In the left-hand navigation pane, select **SSH Keys**.
    *   In the **Title** field, enter a descriptive name for your key, such as `Work Laptop - Windows`.
    *   Paste the copied public key into the **Key** field.
    *   Click the **"Add key"** button.

4.  **Verify SSH Connectivity to GitLab**
    The final step is to test that GitLab can successfully authenticate your machine using your new key.
    *   In your terminal, run the following command:
    ```bash
    ssh -T git@gitlab.com
    ```
    *   The first time you connect, you will see a host authenticity warning. This is expected. Type **`yes`** and press **Enter**.

### **5. Verification**

You will know you have successfully completed this lab when the `ssh -T git@gitlab.com` command produces the following output:

```
Welcome to GitLab, @your-username!
```

If you receive a "Permission denied (publickey)" error, carefully repeat Step 3 to ensure your public key was copied and pasted correctly. If you see the "Welcome" message, your environment is correctly configured.

### **6. Discussion**

This lab established a secure link between your local machine and GitLab using public-key cryptography. Your machine holds a **private key** (`id_ed25519`), which is a secret that must never be shared. You uploaded the corresponding **public key** (`id_ed25519.pub`) to GitLab, which acts as a "lock."

When you perform a Git operation, your Git client uses your private key to "sign" the request. GitLab then uses your public key to verify that signature. This proves your identity without ever sending a password over the network, which is the foundation of secure, automated interactions in DevOps.

### **7. Questions**

1.  Why is it critical that you only upload the `.pub` file to GitLab and never the file without the `.pub` extension?
2.  What would have happened if you tried to run `git clone git@gitlab.com:gabors-gitlab-training-20251013/group-runner-smoke-test.git` *before* adding your SSH key to GitLab?
3.  The book mentions using HTTPS with Personal Access Tokens. In what scenario might a token be a better choice for authentication than an SSH key?
4.  If you connect to GitLab from a new computer in the future, what steps from this lab will you need to repeat?
5.  Why is it a major security risk to copy your `id_ed25519` private key file to a shared server or commit it to a Git repository?

---

### **8. Solution**

This lab primarily involves configuration steps rather than creating source code. The key artifacts are the commands used.

#### **8.1. Command Summary**
```bash
# 1. Generate the ED25519 SSH key pair
ssh-keygen -t ed25519 -C "your.email@example.com"

# 2. Display the public key to copy it
cat ~/.ssh/id_ed25519.pub

# 3. Test the SSH connection to GitLab
ssh -T git@gitlab.com
```

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **Why is it critical that you only upload the `.pub` file?**
    The file without the `.pub` extension is your **private key**. It is your secret identity. If you upload it, anyone with access to GitLab's server (or if the server is compromised) could impersonate you. The `.pub` file is the **public key**; it is designed to be shared and only allows for verification, not impersonation.

2.  **What would happen if you tried to clone via SSH before adding your key?**
    GitLab would reject the connection with a `Permission denied (publickey)` error. Your machine would present its key, but GitLab's server would have no record of that key in your account's list of authorized keys, so it would deny access.

3.  **When might a token be better than an SSH key?**
    Personal Access Tokens are ideal for **non-interactive, automated scripts or applications**, such as a CI/CD job running on a server that needs to clone another repository. Tokens can be created with specific, limited scopes (e.g., read-only access) and can be easily revoked, which is perfect for service-to-service authentication. SSH keys are better suited for interactive user sessions on a developer's personal workstation.

4.  **If you connect from a new computer, what steps must be repeated?**
    You must repeat **Step 2 (Generate SSH Key Pair)** on the new machine to create a unique key pair for that machine. You would then repeat **Step 3 (Add Public SSH Key to GitLab)** to add the new computer's public key to your GitLab account. You can have multiple SSH keys associated with a single GitLab account.

5.  **Why is sharing your private key a major security risk?**
    Sharing your private key is equivalent to sharing your password. Anyone who possesses your private key can impersonate you on any system that trusts the corresponding public key (in this case, GitLab). They could push malicious code, delete repositories, or access sensitive information under your identity.