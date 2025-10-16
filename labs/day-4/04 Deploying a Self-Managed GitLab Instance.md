# **Lab: Deploying a Self-Managed GitLab Instance**

**Module:** Deploying a Self-Managed GitLab Instance
**Time:** Approx. 60 minutes (includes time for GitLab to initialize)

---

### **1. Objective(s)**

By the end of this lab, you will be able to:
*   Prepare a local Windows environment for hosting a persistent GitLab instance.
*   Launch a full GitLab Enterprise Edition (EE) instance as a single Docker container.
*   Retrieve the initial administrator password and perform the first login.
*   Obtain a 30-day Ultimate trial license from the GitLab customer portal.
*   Install the trial license in your self-managed instance to unlock all enterprise features.
*   Perform a basic smoke test to verify the instance is fully functional.

### **2. Scenario**

Your development team needs a private, isolated GitLab instance for internal projects and to evaluate enterprise features without using the public cloud. The fastest and most efficient way to set up such an environment on a local machine is by using Docker.

Your task is to deploy a complete, all-in-one GitLab EE instance on your Windows machine using Docker Desktop. You will configure it to use persistent storage, ensuring that your data (repositories, users, etc.) survives container restarts. After the initial deployment, you will secure the instance, activate an Ultimate trial license, and perform a smoke test to confirm it is ready for use.

### **3. Prerequisites**

*   You are on a Windows machine.
*   Docker Desktop is installed and running.
*   You have an active GitLab.com account (from Day 1) to use for obtaining the trial license.

### **4. Steps**

_**Note:** All commands in this lab should be executed in a standard Windows **PowerShell** terminal. GitLab is a large application and may take 5-10 minutes for the initial startup._

1.  **Prepare the Host Environment for Persistent Storage**
    To ensure your GitLab data is not lost when the container is removed, you must map directories from your host machine into the container.
    *   **[WHERE: Windows PowerShell]**
        *   Open a new PowerShell terminal.
        *   Create a root directory for your GitLab data and set an environment variable to point to it. This makes the `docker run` command cleaner and more portable.
    ```powershell
    # Create the main directory for all GitLab data
    mkdir C:\gitlab

    # Set a session-specific environment variable for the GitLab home directory
    $env:GITLAB_HOME="C:\gitlab"

    # Create the subdirectories that will be used for volume mounts
    mkdir "$env:GITLAB_HOME\config", "$env:GITLAB_HOME\logs", "$env:GITLAB_HOME\data"
    ```

2.  **Launch the GitLab EE Container**
    Now, you will execute the `docker run` command to download and start the GitLab instance.
    *   **[WHERE: Windows PowerShell]**
        *   Copy and paste the entire multi-line command block below into your PowerShell terminal and press Enter.
    ```powershell
    docker run --detach `
      --hostname gitlab.local `
      --publish 443:443 --publish 80:80 --publish 8822:22 `
      --name gitlab-instance `
      --restart always `
      --volume "$env:GITLAB_HOME\config:/etc/gitlab" `
      --volume "$env:GITLAB_HOME\logs:/var/log/gitlab" `
      --volume "$env:GITLAB_HOME\data:/var/opt/gitlab" `
      --shm-size 256m `
      gitlab/gitlab-ee:latest
    ```
    *   **Note on Port `22`:** We are mapping host port `8822` to the container's SSH port `22` to avoid conflicts with any SSH server running on your Windows host.

3.  **Monitor the Startup and Retrieve the Root Password**
    GitLab will now start up in the background. This can take several minutes.
    *   **[WHERE: Windows PowerShell]**
        *   You can monitor the progress by tailing the logs:
        ```powershell
        docker logs -f gitlab-instance
        ```
        *   Wait until you see a "GitLab Reconfigured!" message. Press `Ctrl+C` to stop tailing the logs.
        *   Once the instance is ready, retrieve the initial, temporary password for the `root` user with this command:
        ```powershell
        docker exec -it gitlab-instance grep 'Password:' /etc/gitlab/initial_root_password
        ```
        *   Copy the password (it will be a long, random string).

4.  **Perform the First Login**
    *   **[WHERE: Web Browser]**
        *   Navigate to `http://localhost`.
        *   Log in with the username **`root`** and the password you just copied.
        *   You will be immediately forced to change your password. Set a new, memorable password for the `root` user.

5.  **Obtain the Ultimate Trial License File**
    Now, you will get a trial license for your self-managed instance.
    *   **[WHERE: Web Browser]**
        *   In a new tab, navigate to the GitLab Customer Portal: `customers.gitlab.com`.
        *   Sign in using your **GitLab.com account** credentials (from Day 1).
        *   Navigate to the **"Free Trials"** section of the portal.
        *   Click the button to **"Start an Ultimate free trial"**.
        *   You will be asked to fill out a form with your information.
        *   Once submitted, you will be taken to a page with a **"Download license"** button. Click it to download the `.gitlab-license` file.

6.  **Install the Trial License**
    *   **[WHERE: Web Browser - Your Local GitLab Instance]**
        *   Navigate back to your local GitLab instance at `http://localhost`.
        *   In the top navigation bar, click the **"Admin Area"** icon (the wrench).
        *   In the left-hand navigation, click on **"Subscription"**.
        *   Click **"Upload license"**.
        *   Choose the `.gitlab-license` file you just downloaded and upload it.

7.  **Smoke Test the Instance**
    The final step is to verify that the core functionality is working.
    *   **[WHERE: Web Browser - Your Local GitLab Instance]**
        *   From the Admin Area, create a new standard user for yourself.
        *   Log out as `root` and log back in as your new standard user.
        *   Create a new group and then create a new project within that group.

### **5. Verification**

1.  **Verify Container is Running:** In PowerShell, run `docker ps`. You should see the `gitlab-instance` container with a status of "Up".
2.  **Verify Data Persistence:** In Windows File Explorer, navigate to `C:\gitlab\data`. You should see that the directory is now populated with many files and folders created by GitLab.
3.  **Verify License Installation:** In the Admin Area under "Subscription," you should see the details of your Ultimate trial, including the number of seats and the expiration date.
4.  **Verify Smoke Test:** You must be able to successfully create a new user, a group, and a project without any errors.

### **6. Discussion**

In this lab, you deployed a complete, self-managed GitLab instance, a task that once required a dedicated physical server, now accomplished with a single Docker command. This demonstrates the power of containerization for complex application delivery.

The most critical concept was the use of **Docker volumes** (`--volume`). By mapping host directories (`C:\gitlab\config`) to directories inside the container (`/etc/gitlab`), you ensured that all of GitLab's critical, stateful data is stored persistently on your local machine. This means you can stop, remove, and even upgrade the GitLab container, and as long as you re-attach these volumes, your users, repositories, and configurations will be preserved.

You also performed two key administrative tasks: the initial secure login process and the installation of an **Ultimate trial license**. This is the standard procedure for any team evaluating GitLab's enterprise features in a private, self-managed environment.

### **7. Questions**

1.  In the `docker run` command, what is the purpose of the `--restart always` flag?
2.  We mapped three volumes: `config`, `logs`, and `data`. Which of these is the most critical to back up, and why?
3.  We used the `gitlab/gitlab-ee:latest` image. What is the difference between this and the `gitlab/gitlab-ce:latest` image, and why was `ee` necessary for this lab?
4.  If you wanted to access your local GitLab instance's repositories via SSH from your host machine, what `git clone` command would you use? (Hint: Consider the port mapping).
5.  How does this all-in-one Docker setup differ from a true enterprise production architecture? What is the biggest risk or limitation of this setup?

---

### **8. Solution**

This section contains the completed commands for this lab.

#### **8.1. Command Summary**
```powershell
# Phase 1: Prepare the host environment
mkdir C:\gitlab
$env:GITLAB_HOME="C:\gitlab"
mkdir "$env:GITLAB_HOME\config", "$env:GITLAB_HOME\logs", "$env:GITLAB_HOME\data"

# Phase 2: Launch the GitLab container
docker run --detach `
  --hostname gitlab.local `
  --publish 443:443 --publish 80:80 --publish 8822:22 `
  --name gitlab-instance `
  --restart always `
  --volume "$env:GITLAB_HOME\config:/etc/gitlab" `
  --volume "$env:GITLAB_HOME\logs:/var/log/gitlab" `
  --volume "$env:GITLAB_HOME\data:/var/opt/gitlab" `
  --shm-size 256m `
  gitlab/gitlab-ee:latest

# Phase 3: Retrieve the root password
docker exec -it gitlab-instance grep 'Password:' /etc/gitlab/initial_root_password```

### **9. Answers**

This section contains the detailed answers to the questions.

#### **9.1. Answers to Questions**
1.  **Purpose of `--restart always`?**
    This is a policy that tells the Docker daemon to automatically restart the container if it ever stops, whether due to an error, a system reboot, or being manually stopped. It turns the container into a persistent service on your host machine.

2.  **Which volume is most critical to back up?**
    The **`data`** volume (`/var/opt/gitlab`) is the most critical. It contains all the user-generated data, including the Git repositories themselves, uploaded attachments, and other essential assets. The `config` volume is also critical, but the `data` volume is irreplaceable.

3.  **`ee` vs. `ce`?**
    `ce` is the Community Edition (Free tier only). `ee` is the Enterprise Edition, which is a single package that can run in Free, Premium, or Ultimate mode depending on the license you apply. We needed the `ee` image because it is the only one that can accept the Ultimate trial license.

4.  **SSH `git clone` command?**
    Because we mapped the container's port `22` to our host's port `8822`, you would need to specify this custom port in the URL. The command would be: `git clone ssh://git@localhost:8822/group-name/project-name.git`.

5.  **Difference from a production architecture?**
    This all-in-one setup runs every GitLab component (the web server, database, Redis, Gitaly) inside a single container. A true production architecture is **distributed**, with each of these components running as a separate, highly-available service for scalability and resilience. The biggest risk of this setup is that it is a **single point of failure**; if the container or the host machine goes down, the entire GitLab service is offline.