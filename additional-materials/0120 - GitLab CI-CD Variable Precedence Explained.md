# GitLab CI/CD Variable Precedence Explained

**Understanding the Override Hierarchy: Which Variable Wins?**

---

## The Variable Precedence Pyramid

When the same variable is defined in multiple locations, GitLab follows a strict precedence order. Variables at the top of the pyramid override those below.

```
                    ┌─────────────────────────────┐
                    │   1. Trigger/API Variables  │  HIGHEST PRIORITY
                    │   (Pipeline API calls)      │  (Always wins)
                    └─────────────────────────────┘
                               ▲
                    ┌─────────────────────────────┐
                    │  2. Scheduled Pipeline Vars │
                    │  (Pipeline schedule settings)│
                    └─────────────────────────────┘
                               ▲
                    ┌─────────────────────────────┐
                    │  3. Manual Pipeline Run     │
                    │  (Run Pipeline form)        │
                    └─────────────────────────────┘
                               ▲
                    ┌─────────────────────────────┐
                    │  4. Job-Level Variables     │
                    │  (Specific job in YAML)     │
                    └─────────────────────────────┘
                               ▲
                    ┌─────────────────────────────┐
                    │  5. Project CI/CD Variables │
                    │  (Settings > CI/CD > Vars)  │
                    └─────────────────────────────┘
                               ▲
                    ┌─────────────────────────────┐
                    │  6. Group-Level Variables   │
                    │  (Group Settings > CI/CD)   │
                    └─────────────────────────────┘
                               ▲
                    ┌─────────────────────────────┐
                    │  7. Instance Variables      │
                    │  (Admin Area - if available)│
                    └─────────────────────────────┘
                               ▲
                    ┌─────────────────────────────┐
                    │  8. Global YAML Variables   │  LOWEST PRIORITY
                    │  (.gitlab-ci.yml variables) │  (Overridden by all)
                    └─────────────────────────────┘
```

**Golden Rule:** Higher levels ALWAYS override lower levels for the same variable name.

---

## Detailed Precedence Levels

### Level 1: Trigger/API Variables (Highest Priority)

**Location:** Passed via GitLab API or trigger token

**Use Case:** External systems triggering pipelines with custom parameters

**Example:**
```bash
# Trigger pipeline via API with variables
curl -X POST \
  -F token=YOUR_TRIGGER_TOKEN \
  -F ref=main \
  -F "variables[DEPLOY_ENV]=production" \
  -F "variables[DEBUG_MODE]=true" \
  https://gitlab.com/api/v4/projects/PROJECT_ID/trigger/pipeline
```

**Characteristics:**
- Overrides ALL other variable sources
- Dynamic per-pipeline execution
- Not visible in repository
- Requires API access or trigger token
- Can be sensitive (use carefully)

**When to use:**
- External CI/CD orchestration
- Dynamic deployment targets
- Integration with other tools (Jenkins, GitHub Actions)
- Override behavior for specific runs

---

### Level 2: Scheduled Pipeline Variables

**Location:** CI/CD > Schedules > Edit schedule > Variables

**Use Case:** Recurring jobs with specific configurations (nightly builds, weekly reports)

**Example:**
```
Schedule: Nightly Production Backup
Cron: 0 2 * * *
Variables:
  BACKUP_TARGET = "production"
  RETENTION_DAYS = "30"
  NOTIFY_SLACK = "true"
```

**Characteristics:**
- Schedule-specific overrides
- Different variables per schedule
- Visible in schedule configuration
- Only applies to scheduled runs

**When to use:**
- Different behavior for scheduled vs manual runs
- Environment-specific schedules
- Automated maintenance windows

---

### Level 3: Manual Pipeline Run Variables

**Location:** CI/CD > Pipelines > Run Pipeline button > Variables section

**Use Case:** On-demand pipeline execution with custom parameters

**Visual:**
```
┌──────────────────────────────────────────┐
│  Run Pipeline for "main" branch          │
├──────────────────────────────────────────┤
│  Variables (optional)                    │
│                                          │
│  Key                 Value               │
│  ┌─────────────┐   ┌─────────────┐     │
│  │ DEPLOY_ENV  │   │ staging     │     │
│  └─────────────┘   └─────────────┘     │
│  ┌─────────────┐   ┌─────────────┐     │
│  │ DRY_RUN     │   │ true        │     │
│  └─────────────┘   └─────────────┘     │
│                                          │
│  [Add variable]       [Run Pipeline]    │
└──────────────────────────────────────────┘
```

**Characteristics:**
- Quick temporary overrides
- Testing different configurations
- Visible in pipeline variables list
- Not persisted (must re-enter each time)

**When to use:**
- Testing pipeline changes
- One-time deployments
- Debugging specific scenarios
- Override default environment

---

### Level 4: Job-Level Variables

**Location:** `.gitlab-ci.yml` within specific job definition

**Use Case:** Job-specific configuration that overrides global settings

**Example:**
```yaml
# Global variable
variables:
  DEPLOY_ENV: "staging"
  NODE_ENV: "development"

# This job uses global variables
test:
  script:
    - echo "Testing in $DEPLOY_ENV"  # Output: staging
    - npm test

# This job overrides with job-level variables
deploy-production:
  variables:
    DEPLOY_ENV: "production"      # Overrides global
    NODE_ENV: "production"        # Overrides global
    ENABLE_MONITORING: "true"     # New variable
  script:
    - echo "Deploying to $DEPLOY_ENV"  # Output: production
    - deploy.sh

# This job inherits global + adds new
deploy-staging:
  variables:
    ENABLE_MONITORING: "false"    # New variable only
  script:
    - echo "Deploying to $DEPLOY_ENV"  # Output: staging (global)
    - deploy.sh
```

**Characteristics:**
- Job-specific behavior
- Version controlled in repository
- Clear and explicit in YAML
- Easy to review in merge requests

**When to use:**
- Different behavior per job
- Override defaults for specific tasks
- Provide job-specific configuration

---

### Level 5: Project CI/CD Variables

**Location:** Project Settings > CI/CD > Variables

**Use Case:** Project-specific configuration, secrets, credentials

**Visual:**
```
Settings > CI/CD > Variables

┌──────────────────────────────────────────────────────┐
│  Add Variable                                        │
├──────────────────────────────────────────────────────┤
│  Key:        API_KEY                                 │
│  Value:      sk-1234567890abcdef                     │
│  Type:       [X] Variable  [ ] File                  │
│  Flags:      [X] Protected  [X] Masked  [ ] Expanded │
│  Scope:      All environments (default)              │
│              [ ] Specific environments: _________    │
└──────────────────────────────────────────────────────┘
```

**Variable Types:**

**Variable (default):**
```yaml
# Stored as environment variable
deploy:
  script:
    - echo $DATABASE_URL
    - deploy --db=$DATABASE_URL
```

**File:**
```yaml
# Stored as file, path in environment variable
deploy:
  script:
    - cat $SERVICE_ACCOUNT_JSON      # Variable contains file path
    - gcloud auth activate-service-account --key-file=$SERVICE_ACCOUNT_JSON
```

**Flags:**

**Protected:**
- Only available on protected branches/tags
- Perfect for production secrets
- Won't leak in feature branches

**Masked:**
- Value hidden in job logs
- Prevents accidental exposure
- Must meet requirements (no spaces, min 8 chars, base64)

**Expanded:**
- Allows variable references (`$OTHER_VAR`)
- Disabled by default for security

**Environment Scopes:**
```
Scope: production
  API_KEY = "prod-key-123"
  
Scope: staging
  API_KEY = "stage-key-456"
  
Scope: *
  API_KEY = "dev-key-789"
```

**Characteristics:**
- Secure storage for secrets
- Not in repository (gitignored by nature)
- Can be protected/masked
- Environment-scoped
- Requires project access to view

**When to use:**
- API keys and credentials
- Environment URLs
- Configuration specific to the project
- Any secret that shouldn't be in code

---

### Level 6: Group-Level Variables

**Location:** Group Settings > CI/CD > Variables

**Use Case:** Variables shared across multiple projects in a group

**Example Structure:**
```
Group: MyCompany
├─ Variables:
│  ├─ AWS_REGION = "us-east-1"
│  ├─ DOCKER_REGISTRY = "registry.company.com"
│  └─ SLACK_WEBHOOK = "https://hooks.slack.com/..."
│
├─ Project: Backend API
│  └─ Inherits all group variables
│
├─ Project: Frontend App
│  └─ Inherits all group variables
│
└─ Project: Mobile App
   └─ Inherits all group variables
```

**Characteristics:**
- Share common configuration
- Reduce duplication
- Centralized management
- All group projects inherit
- Can be overridden at project level

**When to use:**
- Organization-wide settings
- Shared infrastructure credentials
- Common registry/repository URLs
- Company standards (coding style, tools)

---

### Level 7: Instance-Level Variables (Admin Only)

**Location:** Admin Area > Settings > CI/CD > Variables

**Use Case:** GitLab instance-wide configuration (self-hosted only)

**Characteristics:**
- Available to ALL projects on instance
- Useful for license keys, global proxies
- Requires administrator access
- Not available on GitLab.com
- Lowest precedence among UI variables

**When to use:**
- Self-hosted GitLab instances
- Company-wide tool licenses
- Proxy/firewall configurations
- Global compliance settings

---

### Level 8: Global YAML Variables (Lowest Priority)

**Location:** `.gitlab-ci.yml` at root level

**Use Case:** Default values, development configuration, non-sensitive settings

**Example:**
```yaml
variables:
  # These are defaults - can be overridden by any higher level
  DEPLOY_ENV: "development"
  DEBUG_MODE: "true"
  LOG_LEVEL: "info"
  NODE_ENV: "development"
  CACHE_ENABLED: "true"

stages:
  - build
  - test
  - deploy

build:
  script:
    - echo "Building for $DEPLOY_ENV"
```

**Characteristics:**
- Version controlled
- Visible to all team members
- Great for documentation
- Safe defaults
- LOWEST priority - overridden by everything
- Should NOT contain secrets

**When to use:**
- Default configurations
- Development/testing values
- Non-sensitive settings
- Documentation of expected variables

---

## Precedence in Action: Real-World Examples

### Example 1: Deployment Environment Override

**Scenario:** You have a default staging environment but need to deploy to production.

**Configuration:**
```yaml
# .gitlab-ci.yml (Level 8 - Lowest)
variables:
  DEPLOY_ENV: "staging"

deploy:
  script:
    - echo "Deploying to $DEPLOY_ENV"
    - ./deploy.sh $DEPLOY_ENV
```

**Settings > CI/CD > Variables (Level 5):**
```
Key: DEPLOY_ENV
Value: production
Scope: production (environment)
Protected: Yes
```

**Result:**
- Feature branch runs: `DEPLOY_ENV = "staging"` (from YAML)
- Main branch run: `DEPLOY_ENV = "production"` (from Project variable with environment scope)
- Manual run with variable: Uses manual override (Level 3)

---

### Example 2: The API Key Mystery

**Setup:**
```yaml
# .gitlab-ci.yml
variables:
  API_KEY: "dev-key-123"  # Level 8

# Group Settings > CI/CD
API_KEY: "group-key-456"  # Level 6

# Project Settings > CI/CD
API_KEY: "project-key-789"  # Level 5

# Job-level
test:
  variables:
    API_KEY: "test-key-000"  # Level 4
  script:
    - echo $API_KEY
```

**Question:** What API_KEY does the test job use?

**Answer:** `test-key-000` (Level 4 - Job-level wins)

**Visual Resolution:**
```
Level 4 (Job-level): "test-key-000"  <-- WINNER
   |
   | (overrides)
   |
Level 5 (Project):   "project-key-789"
   |
   | (overrides)
   |
Level 6 (Group):     "group-key-456"
   |
   | (overrides)
   |
Level 8 (YAML):      "dev-key-123"
```

---

### Example 3: Protected Branch Secrets

**Problem:** How to ensure production secrets only work on main branch?

**Solution:**
```yaml
# .gitlab-ci.yml
variables:
  DATABASE_URL: "postgres://localhost/dev"  # Default for feature branches

deploy-production:
  script:
    - echo "Database: $DATABASE_URL"
    - deploy-to-production
  only:
    - main
```

**Project Settings > CI/CD > Variables:**
```
Key: DATABASE_URL
Value: postgres://prod-db.company.com/prod
Protected: Yes (only available on protected branches)
Masked: Yes
Environment Scope: production
```

**Behavior:**
- Feature branch: Uses YAML default (dev database)
- Main branch: Uses protected variable (prod database)
- Variable value never appears in logs (masked)

---

## Variable Resolution Flowchart

```
                    Start: Job needs variable "DEPLOY_ENV"
                                    |
                                    v
                    ┌───────────────────────────────────┐
                    │ Was variable set via API call?    │
                    └───────────┬───────────────┬───────┘
                              YES              NO
                                |               |
                                v               v
                          Use API value    ┌────────────────────────────────┐
                          [DONE]           │ Was variable set in schedule?  │
                                           └──────────┬──────────────┬──────┘
                                                    YES             NO
                                                      |              |
                                                      v              v
                                               Use schedule    ┌──────────────────────────────────┐
                                               [DONE]          │ Was variable set in manual run?  │
                                                               └──────────┬──────────────┬────────┘
                                                                        YES             NO
                                                                          |              |
                                                                          v              v
                                                                    Use manual    ┌───────────────────────────────────┐
                                                                    [DONE]        │ Is variable in job definition?    │
                                                                                  └──────────┬──────────────┬─────────┘
                                                                                           YES             NO
                                                                                             |              |
                                                                                             v              v
                                                                                        Use job     ┌────────────────────────────────────┐
                                                                                        [DONE]      │ Is variable in project settings?   │
                                                                                                    └──────────┬──────────────┬──────────┘
                                                                                                             YES             NO
                                                                                                               |              |
                                                                                                               v              v
                                                                                                         Use project   ┌─────────────────────────────────┐
                                                                                                         [DONE]        │ Is variable in group settings?  │
                                                                                                                       └──────────┬──────────────┬───────┘
                                                                                                                                YES             NO
                                                                                                                                  |              |
                                                                                                                                  v              v
                                                                                                                            Use group    ┌──────────────────────────────────────┐
                                                                                                                            [DONE]       │ Is variable in instance settings?    │
                                                                                                                                         └──────────┬──────────────┬────────────┘
                                                                                                                                                  YES             NO
                                                                                                                                                    |              |
                                                                                                                                                    v              v
                                                                                                                                              Use instance  ┌──────────────────────────────────┐
                                                                                                                                              [DONE]        │ Is variable in .gitlab-ci.yml?   │
                                                                                                                                                            └──────────┬──────────────┬────────┘
                                                                                                                                                                     YES             NO
                                                                                                                                                                       |              |
                                                                                                                                                                       v              v
                                                                                                                                                                  Use YAML      Variable not set
                                                                                                                                                                  [DONE]        [Empty/undefined]
```

---

## Common Confusion & Troubleshooting

### Problem 1: "My variable isn't working!"

**Symptoms:** Variable shows empty or unexpected value in job

**Debug Steps:**
```yaml
debug-variables:
  script:
    - echo "Checking variable sources..."
    - echo "MY_VAR from environment: $MY_VAR"
    - env | grep MY_VAR
    - echo "All CI variables:"
    - env | grep ^CI_
```

**Checklist:**
1. Check variable name spelling (case-sensitive!)
2. Verify variable exists at expected level
3. Check if protected variable on unprotected branch
4. Verify environment scope matches
5. Look for typos in `.gitlab-ci.yml`
6. Check if variable is masked (won't see in logs)

---

### Problem 2: "Protected variable not available"

**Issue:** Variable shows empty on protected branch

**Causes:**
- Branch/tag not marked as "protected" in repository settings
- Variable scope doesn't match environment
- User lacks permission to view variable

**Solution:**
```
1. Go to Settings > Repository > Protected Branches
2. Verify your branch is protected
3. Go to Settings > CI/CD > Variables
4. Check variable has "Protected" flag
5. Verify environment scope matches (or use "*")
```

---

### Problem 3: "Can't override group variable"

**Misconception:** Group variables can't be changed

**Reality:** They CAN be overridden at project or job level!

**Example:**
```yaml
# Group has: AWS_REGION = "us-east-1"

# Project can override in Settings > CI/CD
# Or in .gitlab-ci.yml:

deploy-europe:
  variables:
    AWS_REGION: "eu-west-1"  # Overrides group variable
  script:
    - deploy-to-aws
```

---

### Problem 4: "Which variable is actually being used?"

**Solution:** Use CI/CD debugging output

**Add to your job:**
```yaml
.debug_template:
  before_script:
    - |
      echo "=== Variable Debug Info ==="
      echo "Job: $CI_JOB_NAME"
      echo "Pipeline Source: $CI_PIPELINE_SOURCE"
      echo "Branch: $CI_COMMIT_BRANCH"
      echo "MY_VAR value: $MY_VAR"
      echo "MY_VAR source: Unknown (check precedence)"
      echo "=========================="

my_job:
  extends: .debug_template
  script:
    - actual-work.sh
```

---

## Best Practices

### 1. Use the Right Level for the Right Purpose

| Level       | Best For                                 | Avoid For                       |
| ----------- | ---------------------------------------- | ------------------------------- |
| API/Trigger | External integrations, dynamic overrides | Regular pipeline runs           |
| Schedule    | Schedule-specific config                 | Variables needed in manual runs |
| Manual Run  | Testing, one-off overrides               | Persistent configuration        |
| Job-Level   | Job-specific behavior                    | Secrets (not visible enough)    |
| Project     | Secrets, project config                  | Shared org-wide settings        |
| Group       | Shared team settings                     | Project-specific values         |
| Instance    | Global infrastructure                    | Project-specific anything       |
| YAML        | Defaults, documentation                  | Secrets (visible in repo!)      |

### 2. Security Guidelines

**DO:**
- Store secrets in Project/Group variables (Level 5/6)
- Use "Protected" flag for production secrets
- Use "Masked" flag to hide values in logs
- Use environment scopes to limit variable availability
- Document expected variables in README

**DON'T:**
- Never put secrets in `.gitlab-ci.yml`
- Don't use API variables for permanent config
- Don't share credentials across projects via group vars
- Don't rely on variable precedence for security

### 3. Naming Conventions

```yaml
# Good naming (clear purpose)
variables:
  DEPLOY_TARGET_ENV: "staging"
  AWS_REGION: "us-east-1"
  ENABLE_DEBUG_MODE: "false"
  API_BASE_URL: "https://api.example.com"

# Poor naming (confusing)
variables:
  VAR1: "something"
  TEMP: "value"
  X: "yes"
```

### 4. Documentation Template

Add to your repository README:

```markdown
## Required CI/CD Variables

### Project Variables (Settings > CI/CD > Variables)

| Variable      | Description           | Example        | Protected | Masked |
| ------------- | --------------------- | -------------- | --------- | ------ |
| API_KEY       | Production API key    | sk-...         | Yes       | Yes    |
| DATABASE_URL  | Database connection   | postgres://... | Yes       | Yes    |
| SLACK_WEBHOOK | Notifications webhook | https://...    | No        | Yes    |

### Default Variables (.gitlab-ci.yml)

| Variable   | Default     | Description       |
| ---------- | ----------- | ----------------- |
| DEPLOY_ENV | staging     | Deployment target |
| NODE_ENV   | development | Node environment  |
| LOG_LEVEL  | info        | Logging verbosity |
```

### 5. Testing Variable Precedence

**Create a test job:**
```yaml
test-variables:
  stage: test
  script:
    - |
      echo "=== Variable Precedence Test ==="
      echo "TEST_VAR from YAML: Should show if not overridden"
      echo "TEST_VAR actual value: $TEST_VAR"
      echo ""
      echo "Expected precedence (highest to lowest):"
      echo "1. API/Trigger"
      echo "2. Schedule"
      echo "3. Manual Run"
      echo "4. Job-level"
      echo "5. Project Settings"
      echo "6. Group Settings"
      echo "7. Instance Settings"
      echo "8. YAML Global (this value: $YAML_DEFAULT)"
  variables:
    TEST_VAR: "from-yaml-global"
    YAML_DEFAULT: "yaml-level-8"
  rules:
    - when: manual
```

---

## Quick Reference Card

### Precedence Order (Top Wins)

```
1. API/Trigger          Highest priority
2. Schedule             Schedule-specific
3. Manual Run           One-time override
4. Job-Level (YAML)     Job-specific
5. Project Settings     Project config
6. Group Settings       Shared across projects
7. Instance Settings    Instance-wide (admin)
8. Global YAML          Defaults, lowest priority
```

### Memory Aid: "API Schedules Manual Jobs Privately in Groups Immediately with YAML"

- **A**PI
- **S**chedule
- **M**anual
- **J**ob
- **P**roject
- **G**roup
- **I**nstance
- **Y**AML

---

## Summary

**Key Takeaways:**

1. **Higher levels ALWAYS override lower levels** - No exceptions
2. **YAML variables have the LOWEST priority** - They're defaults only
3. **Project/Group variables are perfect for secrets** - Not visible in code
4. **Job-level variables override global YAML** - Use for job-specific config
5. **Manual/API/Schedule variables override everything below** - Dynamic control
6. **Use protected variables for production** - Security best practice
7. **Document your variables** - Future you will thank you

**When in doubt:**
1. Check the pyramid - higher wins
2. Use debug scripts to see actual values
3. Remember: YAML is just defaults, UI settings override them

Understanding variable precedence prevents confusion, security issues, and debugging headaches. Master this concept, and your CI/CD pipelines will be more secure, maintainable, and predictable.