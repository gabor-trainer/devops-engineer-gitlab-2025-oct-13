# GitLab Permissions Matrix - Developer vs. Maintainer


## GitLab Role Hierarchy

```
Guest < Reporter < Developer < Maintainer < Owner
```

| Role           | Typical Use Case                                             |
| -------------- | ------------------------------------------------------------ |
| **Guest**      | External stakeholders, view-only access to issues            |
| **Reporter**   | QA team, product managers (can create issues, view code)     |
| **Developer**  | Software engineers (can push code, run pipelines)            |
| **Maintainer** | Tech leads, senior engineers (can manage branches, settings) |
| **Owner**      | Project owners, admins (full control, can delete project)    |

**This guide focuses on Developer vs. Maintainer** - the roles most developers encounter daily.

---

## Quick Decision Guide

### Assign **Developer** role to:
- Regular team members who write code
- Contributors who should not modify protected branches
- Team members who don't need to manage project settings
- External contractors with limited access needs

### Assign **Maintainer** role to:
- Tech leads and senior engineers
- Release managers who merge to protected branches
- Team members who manage CI/CD configuration
- Anyone who needs to configure project settings

---

## Comprehensive Permissions Matrix

### 🔹 Repository & Code Access

| Permission                           | Developer            | Maintainer           |
| ------------------------------------ | -------------------- | -------------------- |
| **Clone repository**                 | ✅ Yes                | ✅ Yes                |
| **Create branches**                  | ✅ Yes                | ✅ Yes                |
| **Push to regular branches**         | ✅ Yes                | ✅ Yes                |
| **Push to protected branches**       | ❌ No*                | ✅ Yes                |
| **Force push to protected branches** | ❌ No                 | ❌ No (configurable)  |
| **Delete protected branches**        | ❌ No                 | ✅ Yes                |
| **Delete regular branches**          | ✅ Yes (own branches) | ✅ Yes (any branch)   |
| **Create tags**                      | ✅ Yes                | ✅ Yes                |
| **Delete tags**                      | ❌ No                 | ✅ Yes (configurable) |
| **Push to protected tags**           | ❌ No                 | ✅ Yes (configurable) |

*_Can be configured to allow Developers if needed_

---

### 🔹 Merge Requests

| Permission                      | Developer       | Maintainer           |
| ------------------------------- | --------------- | -------------------- |
| **Create merge requests**       | ✅ Yes           | ✅ Yes                |
| **Approve merge requests**      | ✅ Yes*          | ✅ Yes                |
| **Merge to regular branches**   | ✅ Yes           | ✅ Yes                |
| **Merge to protected branches** | ❌ No**          | ✅ Yes                |
| **Override merge checks**       | ❌ No            | ✅ Yes (configurable) |
| **Edit merge request**          | ✅ Yes (own MRs) | ✅ Yes (any MR)       |
| **Delete merge request**        | ❌ No            | ✅ Yes                |
| **Rebase merge requests**       | ✅ Yes (own MRs) | ✅ Yes (any MR)       |

*_Depends on approval rules configuration_  
**_Unless explicitly granted in protected branch settings_

---

### 🔹 CI/CD Pipelines

| Permission                           | Developer   | Maintainer                             |
| ------------------------------------ | ----------- | -------------------------------------- |
| **View pipelines**                   | ✅ Yes       | ✅ Yes                                  |
| **Run pipelines**                    | ✅ Yes       | ✅ Yes                                  |
| **Cancel pipelines**                 | ✅ Yes (own) | ✅ Yes (any)                            |
| **Retry pipelines**                  | ✅ Yes       | ✅ Yes                                  |
| **Edit `.gitlab-ci.yml`**            | ✅ Yes       | ✅ Yes                                  |
| **Manage CI/CD variables (project)** | ❌ No        | ✅ Yes                                  |
| **Manage CI/CD variables (group)**   | ❌ No        | ❌ No (needs Maintainer at group level) |
| **View protected variables**         | ❌ No        | ✅ Yes                                  |
| **Create/edit pipeline schedules**   | ✅ Yes (own) | ✅ Yes (any)                            |
| **Run manual jobs**                  | ✅ Yes       | ✅ Yes                                  |
| **Manage runners**                   | ❌ No        | ✅ Yes                                  |
| **Clear runner caches**              | ❌ No        | ✅ Yes                                  |

---

### 🔹 Environments & Deployments

| Permission                           | Developer   | Maintainer  |
| ------------------------------------ | ----------- | ----------- |
| **View environments**                | ✅ Yes       | ✅ Yes       |
| **Deploy to regular environments**   | ✅ Yes       | ✅ Yes       |
| **Deploy to protected environments** | ❌ No*       | ✅ Yes       |
| **Stop environments**                | ✅ Yes (own) | ✅ Yes (any) |
| **Delete environments**              | ❌ No        | ✅ Yes       |
| **Create deployment tokens**         | ❌ No        | ✅ Yes       |

*_Can be configured in protected environment settings_

---

### 🔹 Project Settings & Configuration

| Permission                           | Developer | Maintainer        |
| ------------------------------------ | --------- | ----------------- |
| **View project settings**            | ✅ Partial | ✅ Full            |
| **Edit project description**         | ❌ No      | ✅ Yes             |
| **Configure protected branches**     | ❌ No      | ✅ Yes             |
| **Configure protected tags**         | ❌ No      | ✅ Yes             |
| **Manage webhooks**                  | ❌ No      | ✅ Yes             |
| **Manage integrations**              | ❌ No      | ✅ Yes             |
| **Configure merge request settings** | ❌ No      | ✅ Yes             |
| **Manage project members**           | ❌ No      | ✅ Yes             |
| **Edit project avatar**              | ❌ No      | ✅ Yes             |
| **Archive project**                  | ❌ No      | ✅ Yes             |
| **Transfer project**                 | ❌ No      | ❌ No (Owner only) |
| **Delete project**                   | ❌ No      | ❌ No (Owner only) |

---

### 🔹 Packages & Registries

| Permission                         | Developer | Maintainer |
| ---------------------------------- | --------- | ---------- |
| **Pull from Container Registry**   | ✅ Yes     | ✅ Yes      |
| **Push to Container Registry**     | ✅ Yes     | ✅ Yes      |
| **Delete from Container Registry** | ❌ No      | ✅ Yes      |
| **Publish to Package Registry**    | ✅ Yes     | ✅ Yes      |
| **Delete from Package Registry**   | ❌ No      | ✅ Yes      |
| **Manage cleanup policies**        | ❌ No      | ✅ Yes      |

---

### 🔹 Issues & Issue Boards

| Permission            | Developer | Maintainer |
| --------------------- | --------- | ---------- |
| **Create issues**     | ✅ Yes     | ✅ Yes      |
| **Close issues**      | ✅ Yes     | ✅ Yes      |
| **Reopen issues**     | ✅ Yes     | ✅ Yes      |
| **Delete issues**     | ❌ No      | ✅ Yes      |
| **Edit issue boards** | ✅ Yes     | ✅ Yes      |
| **Manage labels**     | ✅ Yes     | ✅ Yes      |
| **Manage milestones** | ✅ Yes     | ✅ Yes      |

---

### 🔹 Wiki & Documentation

| Permission               | Developer | Maintainer |
| ------------------------ | --------- | ---------- |
| **View wiki**            | ✅ Yes     | ✅ Yes      |
| **Create wiki pages**    | ✅ Yes     | ✅ Yes      |
| **Edit wiki pages**      | ✅ Yes     | ✅ Yes      |
| **Delete wiki pages**    | ❌ No      | ✅ Yes      |
| **Upload files to wiki** | ✅ Yes     | ✅ Yes      |

---

### 🔹 Security & Compliance

| Permission                   | Developer | Maintainer |
| ---------------------------- | --------- | ---------- |
| **View security dashboard**  | ✅ Yes     | ✅ Yes      |
| **Create vulnerabilities**   | ✅ Yes     | ✅ Yes      |
| **Resolve vulnerabilities**  | ✅ Yes     | ✅ Yes      |
| **Dismiss vulnerabilities**  | ❌ No      | ✅ Yes      |
| **View dependency list**     | ✅ Yes     | ✅ Yes      |
| **Manage security policies** | ❌ No      | ✅ Yes      |

---

## Real-World Scenarios

### Scenario 1: Releasing to Production

**Situation:** Your team follows GitFlow with `main` as a protected branch. Only releases should be merged to `main`.

**Setup:**
- **Protected branch:** `main`
- **Allowed to merge:** Maintainers only
- **Team members:** Developers create feature branches and MRs
- **Release manager:** Maintainer role, merges to `main`

```yaml
# .gitlab-ci.yml
deploy-production:
  stage: deploy
  script:
    - ./deploy.sh production
  only:
    - main  # Only runs on main branch
  when: manual  # Requires manual trigger
```

**Why this works:**
- Developers can work freely on feature branches
- Only Maintainers can merge to `main` (protected)
- Only Maintainers can trigger production deployment

---

### Scenario 2: Managing Secrets

**Situation:** Your pipeline needs database credentials that should not be visible to all team members.

**Setup:**
1. Create a CI/CD variable `DB_PASSWORD`
2. Mark it as **Protected** ✅
3. Mark it as **Masked** ✅

**Result:**
- **Developers:** Cannot view the variable value
- **Maintainers:** Can view and edit the variable
- **Pipelines on protected branches:** Can use the variable
- **Pipelines on feature branches:** Cannot use the variable (if protected)

```yaml
# This job only runs on protected branches and has access to secrets
deploy-staging:
  script:
    - echo "Deploying with credentials..."
    - ./deploy.sh --password $DB_PASSWORD
  only:
    - main
    - develop
```

---

### Scenario 3: Hotfix Emergency

**Situation:** Production is down, and you need to push a hotfix directly to `main`.

**Without Maintainer role:**
1. Developer creates `hotfix/critical-bug` branch
2. Developer creates MR → `main`
3. **Must wait** for Maintainer to review and merge
4. Maintainer merges
5. Deployment proceeds

**With Maintainer role:**
1. Developer pushes directly to `main` (if protected branch settings allow)
2. Deployment proceeds immediately
3. *(Follow up with MR for review)*

**Best practice:** Even Maintainers should use MRs for most changes, but the permission exists for emergencies.

---

### Scenario 4: Multi-Environment Deployment

**Setup:**
```yaml
deploy-dev:
  stage: deploy
  script:
    - ./deploy.sh dev
  environment:
    name: development
  # Anyone can deploy to dev

deploy-staging:
  stage: deploy
  script:
    - ./deploy.sh staging
  environment:
    name: staging
  when: manual
  # Anyone can deploy to staging

deploy-production:
  stage: deploy
  script:
    - ./deploy.sh production
  environment:
    name: production
  when: manual
  only:
    - main
  # Only runs on protected branch
```

**Protected Environment settings:**
- **Development:** No protection (anyone can deploy)
- **Staging:** No protection (anyone can deploy)
- **Production:** Protected, only Maintainers can deploy

---

## Common Misunderstandings

### ❌ Myth 1: "Developers can't modify CI/CD"

**Reality:** Developers **can** edit `.gitlab-ci.yml` files and push changes. What they **can't** do is:
- Manage CI/CD variables in project settings
- View protected/masked variable values
- Manage runner configuration

**Example:**
```yaml
# Developer CAN add/modify this
new-test-job:
  stage: test
  script:
    - npm run test:new

# But CANNOT access $PROTECTED_VAR if it's marked as protected
# and the branch is not protected
```

---

### ❌ Myth 2: "You must be a Maintainer to run pipelines"

**Reality:** Both Developers and Maintainers can run pipelines. The difference is:
- Developers can run pipelines on any branch they have access to
- Only Maintainers can manage runners, clear caches, or view protected variables

---

### ❌ Myth 3: "Protected branches completely block Developers"

**Reality:** Protected branch settings are **configurable**. You can:
- Allow Developers to push to protected branches
- Allow Developers to merge to protected branches
- Require approvals before merge

**Example configuration:**
```
Protected Branch: main
├─ Allowed to push: Maintainers
├─ Allowed to merge: Developers + Maintainers
├─ Require approval: Yes (2 approvals)
└─ Require passing pipeline: Yes
```

This allows Developers to create MRs to `main`, but they need 2 approvals and a passing pipeline.

---

## Permission Configuration Examples

### Example 1: Standard Workflow (Recommended)

**Goal:** Balance between developer autonomy and protection.

**Protected Branches:**
- `main` - production releases
- `develop` - integration branch

**Settings for `main`:**
```
Allowed to merge: Maintainers only
Allowed to push: No one (merge via MR only)
Allowed to force push: No one
Code owner approval required: Yes (optional)
```

**Settings for `develop`:**
```
Allowed to merge: Developers + Maintainers
Allowed to push: No one (merge via MR only)
Allowed to force push: No one
```

**Roles:**
- **Developers:** Team members - can create MRs to `develop`
- **Maintainers:** Tech leads - can merge to both `main` and `develop`

---

### Example 2: Open Source Project

**Goal:** Allow external contributors while maintaining control.

**Roles:**
- **Guest:** Anonymous users (can view public projects)
- **Developer:** Internal team members
- **Maintainer:** Core maintainers

**Protected Branches:**
```
main:
  Allowed to merge: Maintainers only
  Allowed to push: No one
  
release/*:
  Allowed to merge: Maintainers only
  Allowed to push: Maintainers only
```

**External contributors:**
- Fork the project
- Create MR from fork
- Maintainers review and merge

---

### Example 3: Startup with Fast Iteration

**Goal:** Move quickly with minimal friction.

**Roles:**
- All engineers have **Maintainer** role
- Interns/contractors have **Developer** role

**Protected Branches:**
```
main:
  Allowed to merge: Maintainers only
  Allowed to push: Maintainers only
  Require approval: No (trust-based)
```

**Trade-off:** Speed over strict control (appropriate for small teams).

---

## Troubleshooting Common Permission Issues

### Issue 1: "Cannot push to protected branch"

**Error message:**
```
remote: GitLab: You are not allowed to push code to protected branch main on this project.
```

**Solutions:**
1. **Check your role:** You need Maintainer role (or special permission)
2. **Use MR instead:** Create a branch and merge request
3. **Ask for permission:** Request Maintainer to add your user to allowed list
4. **Check branch protection:** Settings → Repository → Protected Branches

---

### Issue 2: "Variable not found in pipeline"

**Error message:**
```
echo: $DB_PASSWORD: parameter not set
```

**Common causes:**
1. **Variable is protected:** Runs only on protected branches
   - **Solution:** Remove "protected" flag or run on protected branch
2. **Variable is masked:** Trying to echo it (masked variables hide output)
   - **Solution:** Don't echo masked variables
3. **Scope mismatch:** Variable defined for different environment
   - **Solution:** Check variable scope in CI/CD settings

---

### Issue 3: "Cannot merge request"

**Symptoms:** "Merge" button is disabled on MR.

**Common causes:**
1. **Failing pipeline:** Pipeline must pass first
2. **Unresolved threads:** All discussions must be resolved
3. **Awaiting approvals:** Required approvals not met
4. **Insufficient permissions:** Target branch is protected
   - **Solution:** Request Maintainer to merge

---

## Best Practices

### ✅ DO: Use Principle of Least Privilege

- Start with **Developer** role for new team members
- Promote to **Maintainer** only when needed
- Use protected branches for critical code paths

### ✅ DO: Document Your Permission Strategy

```markdown
# Team Roles & Responsibilities

## Developers
- All software engineers
- Can create MRs to any branch
- Can deploy to dev/staging

## Maintainers
- Tech leads and senior engineers
- Can merge to main/production
- Manage CI/CD secrets
- Release responsibilities
```

### ✅ DO: Use Protected Environments

```yaml
deploy-production:
  stage: deploy
  environment:
    name: production
  # Configure in Settings → CI/CD → Protected Environments
  # Allow: Maintainers only
```

### ✅ DO: Regular Permission Audits

- Review project members quarterly
- Remove inactive users
- Verify role assignments are still appropriate

### ❌ DON'T: Give Everyone Maintainer Role

**Why:** Reduces security, increases risk of accidental changes.

### ❌ DON'T: Use Owner Role for Daily Work

**Why:** Owner can delete projects. Reserve for actual project owners/admins.

### ❌ DON'T: Store Secrets in Unprotected Variables

**Why:** Developers can see them in pipeline output or variable list.

---

## Quick Reference Card

### When to Escalate to Maintainer

You need **Maintainer** role if you need to:
- ✓ Merge to protected branches regularly
- ✓ Manage CI/CD variables and secrets
- ✓ Configure protected branches/tags
- ✓ Delete tags or branches
- ✓ Manage project integrations
- ✓ Configure runners
- ✓ Manage project members

### When Developer is Sufficient

You have enough access as **Developer** if you:
- ✓ Write code and push to feature branches
- ✓ Create and review merge requests
- ✓ Run and retry pipelines
- ✓ Create issues and manage labels
- ✓ Deploy to development environments
- ✓ View (but not manage) project settings

---

## Summary Table

| Category        | Developer Can            | Maintainer Can                   |
| --------------- | ------------------------ | -------------------------------- |
| **Code**        | Push to regular branches | Push to protected branches       |
| **Merge**       | Create MRs               | Merge to protected branches      |
| **CI/CD**       | Run pipelines            | Manage variables & runners       |
| **Settings**    | View                     | Edit & configure                 |
| **Secrets**     | Use in allowed contexts  | View & manage                    |
| **Branches**    | Create & delete own      | Delete any, configure protection |
| **Deployments** | Regular environments     | Protected environments           |

---

## Further Reading

- [GitLab Docs: Permissions and Roles](https://docs.gitlab.com/ee/user/permissions.html)
- [GitLab Docs: Protected Branches](https://docs.gitlab.com/ee/user/project/protected_branches.html)
- [GitLab Docs: Protected Environments](https://docs.gitlab.com/ee/ci/environments/protected_environments.html)
- [GitLab Docs: CI/CD Variables](https://docs.gitlab.com/ee/ci/variables/)