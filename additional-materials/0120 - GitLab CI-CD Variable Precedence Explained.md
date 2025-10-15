# GitLab CI/CD Variable Precedence Explained

version: 1.1 revised by Gábor at 2025-10-15

## Introduction

In GitLab CI/CD, variables can be defined at multiple levels. When the same variable name exists in different locations, GitLab follows a specific precedence order to determine which value is used. Understanding this hierarchy is crucial for effective pipeline management and troubleshooting.

** Variables defined at higher precedence levels override those at lower levels.**

---

## Complete Variable Precedence Hierarchy

Here's the complete list from **HIGHEST to LOWEST** priority:

### 1. **Trigger Variables** (Highest Priority)
- **Where**: Passed via API when triggering a pipeline
- **Use case**: External systems triggering pipelines with custom parameters
- **Example**: CI/CD triggers from other projects or external automation
- **Access level**: Anyone with permission to trigger pipelines via API

### 2. **Scheduled Pipeline Variables**
- **Where**: Defined when creating a pipeline schedule
- **Use case**: Different variable values for nightly builds vs. weekly releases
- **Example**: `DEPLOY_ENV=staging` for nightly schedule, `DEPLOY_ENV=production` for weekly
- **Access level**: Maintainers and above

### 3. **Manual Pipeline Run Variables**
- **Where**: Entered in the UI when clicking "Run Pipeline"
- **Use case**: Quick overrides for testing or one-off deployments
- **Example**: Setting `DEBUG=true` for a single test run
- **Access level**: Developers and above (if they can run pipelines)

### 4. **Project Variables**
- **Where**: Project Settings → CI/CD → Variables
- **Use case**: Project-specific credentials, URLs, or configuration
- **Example**: `AWS_ACCESS_KEY`, `DATABASE_URL` for this specific project
- **Access level**: Maintainers and above

### 5. **Group Variables**
- **Where**: Group Settings → CI/CD → Variables
- **Use case**: Shared credentials or settings across multiple projects
- **Example**: Shared Docker registry credentials for all team projects
- **Access level**: Group owners and maintainers

### 6. **Instance Variables** (Self-managed GitLab only)
- **Where**: Admin Area → Settings → CI/CD → Variables
- **Use case**: Organization-wide defaults or credentials
- **Example**: Corporate proxy settings, company-wide license keys
- **Access level**: GitLab administrators only

### 7. **Job-level Variables**
- **Where**: In `.gitlab-ci.yml` under a specific job
- **Use case**: Variables that only apply to one specific job
- **Example**: Different build flags for different build jobs
- **Access level**: Anyone who can modify `.gitlab-ci.yml`

### 8. **Global/Pipeline-level Variables**
- **Where**: Top of `.gitlab-ci.yml` under `variables:` keyword
- **Use case**: Default values for the entire pipeline
- **Example**: Default Node.js version, common build flags
- **Access level**: Anyone who can modify `.gitlab-ci.yml`

### 9. **Deployment Variables**
- **Where**: Associated with deployment environments
- **Use case**: Environment-specific variables (production vs. staging)
- **Example**: Different API endpoints per environment
- **Access level**: Maintainers and above

### 10. **Predefined Variables** (Lowest Priority)
- **Where**: Built into GitLab
- **Use case**: Information about the pipeline, commit, branch, etc.
- **Example**: `CI_COMMIT_SHA`, `CI_PROJECT_NAME`, `CI_PIPELINE_ID`
- **Access level**: Read-only, cannot be modified

---

## Visual Precedence Diagram

```
┌─────────────────────────────────────────┐
│  TRIGGER VARIABLES                      │ ← Wins over everything
├─────────────────────────────────────────┤
│  SCHEDULED PIPELINE VARIABLES           │
├─────────────────────────────────────────┤
│  MANUAL PIPELINE RUN VARIABLES          │
├─────────────────────────────────────────┤
│  PROJECT VARIABLES (UI)                 │
├─────────────────────────────────────────┤
│  GROUP VARIABLES (UI)                   │
├─────────────────────────────────────────┤
│  INSTANCE VARIABLES (UI)                │
├─────────────────────────────────────────┤
│  JOB-LEVEL VARIABLES (.gitlab-ci.yml)   │
├─────────────────────────────────────────┤
│  GLOBAL/PIPELINE VARIABLES (.gitlab-ci) │
├─────────────────────────────────────────┤
│  DEPLOYMENT VARIABLES                   │
├─────────────────────────────────────────┤
│  PREDEFINED VARIABLES                   │ ← Lowest priority
└─────────────────────────────────────────┘
```

---

## Deep Dive: The Four Most Common Levels

### Level 1: Global/Pipeline-Level Variables

**Definition**: Variables defined at the top of `.gitlab-ci.yml` that apply to all jobs in the pipeline.

**Location in `.gitlab-ci.yml`**:
```yaml
variables:
  NODE_VERSION: "18"
  BUILD_TYPE: "production"
  ENABLE_CACHE: "true"

stages:
  - build
  - test
  - deploy
```

**Real-Life Example 1: Default Versions**
```yaml
variables:
  NODE_VERSION: "18.17.0"
  PYTHON_VERSION: "3.11"
  DOCKER_IMAGE: "alpine:3.18"

build_frontend:
  image: node:${NODE_VERSION}
  script:
    - npm install
    - npm run build

build_backend:
  image: python:${PYTHON_VERSION}
  script:
    - pip install -r requirements.txt
    - python setup.py build
```
**Use case**: Set default versions for all jobs. If you need to upgrade Node.js across all jobs, change it once here.

**Real-Life Example 2: Feature Flags**
```yaml
variables:
  ENABLE_EXPERIMENTAL_FEATURES: "false"
  RUN_PERFORMANCE_TESTS: "true"
  DEPLOY_TO_STAGING: "true"

test_job:
  script:
    - |
      if [ "$ENABLE_EXPERIMENTAL_FEATURES" = "true" ]; then
        npm run test:experimental
      fi
    - npm run test
```
**Use case**: Control pipeline behavior across all jobs. Developers can override these via UI for testing.

---

### Level 2: Job-Level Variables

**Definition**: Variables defined within a specific job that only apply to that job.

**Location in `.gitlab-ci.yml`**:
```yaml
build_production:
  variables:
    BUILD_TYPE: "production"
    OPTIMIZE: "true"
  script:
    - npm run build

build_development:
  variables:
    BUILD_TYPE: "development"
    OPTIMIZE: "false"
  script:
    - npm run build
```

**Real-Life Example 1: Different Build Configurations**
```yaml
variables:
  # Global default
  BUILD_ENV: "staging"

build_staging:
  variables:
    BUILD_ENV: "staging"
    API_URL: "https://api-staging.example.com"
    MINIFY: "false"
  script:
    - echo "Building for $BUILD_ENV"
    - npm run build -- --env=$BUILD_ENV --api-url=$API_URL

build_production:
  variables:
    BUILD_ENV: "production"  # Overrides global
    API_URL: "https://api.example.com"
    MINIFY: "true"
  script:
    - echo "Building for $BUILD_ENV"
    - npm run build -- --env=$BUILD_ENV --api-url=$API_URL
```
**Use case**: Each build job needs different configuration. The job-level variables override the global default.

**Real-Life Example 2: Different Docker Registries**
```yaml
variables:
  DOCKER_REGISTRY: "registry.gitlab.com"  # Default

push_to_gitlab:
  variables:
    DOCKER_REGISTRY: "registry.gitlab.com"
    DOCKER_TAG: "latest"
  script:
    - docker push $DOCKER_REGISTRY/myapp:$DOCKER_TAG

push_to_aws:
  variables:
    DOCKER_REGISTRY: "123456789.dkr.ecr.us-east-1.amazonaws.com"
    DOCKER_TAG: "production"
  script:
    - docker push $DOCKER_REGISTRY/myapp:$DOCKER_TAG
```
**Use case**: Push the same image to different registries using job-specific variables.

---

### Level 3: Group Variables

**Definition**: Variables defined at the group level, shared across all projects in the group.

**Location**: Group Settings → CI/CD → Variables

**Real-Life Example 1: Shared Credentials**

Scenario: Your organization has 20 microservices, all need access to the same Docker registry.

**Without Group Variables** (Bad):
- Define `DOCKER_USERNAME` and `DOCKER_PASSWORD` in 20 project variable settings
- When password rotates, update 20 projects manually
- Risk of inconsistency and forgotten updates

**With Group Variables** (Good):
```
Group: "backend-team"
├── Project: user-service
├── Project: payment-service
├── Project: notification-service
└── ... (17 more projects)

Group Variables:
  DOCKER_USERNAME: "gitlab-ci-bot"
  DOCKER_PASSWORD: "*********" (masked)
  SHARED_NPM_TOKEN: "*********" (masked)
```

All 20 projects automatically get these variables. Rotate the password once at the group level.

**Real-Life Example 2: Environment Endpoints**

```
Group Variables:
  KAFKA_BROKER: "kafka.internal.company.com:9092"
  REDIS_HOST: "redis.internal.company.com"
  MONITORING_URL: "https://grafana.company.com"
  SENTRY_DSN: "https://abc123@sentry.io/456789"
```

`.gitlab-ci.yml` in any project:
```yaml
test_integration:
  script:
    - echo "Connecting to Kafka at $KAFKA_BROKER"
    - python run_integration_tests.py
  # KAFKA_BROKER comes from group variables
```

**Use case**: All microservices in the group connect to the same infrastructure. Define once, use everywhere.

---

### Level 4: Project Variables

**Definition**: Variables defined for a specific project, overriding group and lower-level variables.

**Location**: Project Settings → CI/CD → Variables

**Real-Life Example 1: Project-Specific Secrets**

Scenario: You have a group-level `DATABASE_URL` for dev database, but production project needs its own.

```
Group Variables:
  DATABASE_URL: "postgres://dev.db.company.com/shared_dev"

Project Variables (in production-api project):
  DATABASE_URL: "postgres://prod.db.company.com/production"  ← Overrides group
  AWS_S3_BUCKET: "production-media-bucket"
  DEPLOY_KEY: "*********"
```

`.gitlab-ci.yml`:
```yaml
deploy_production:
  script:
    - echo "Deploying to database: $DATABASE_URL"
    - python manage.py migrate
    - python manage.py deploy
  # Uses project-level DATABASE_URL, not group-level
```

**Result**: Development projects use the group's dev database, production uses its own.

**Real-Life Example 2: Feature Flags Per Project**

```
Group Variables:
  ENABLE_NEW_UI: "false"  # Default for all projects

Project Variables (in beta-testing project):
  ENABLE_NEW_UI: "true"  ← Override for this project only
```

`.gitlab-ci.yml`:
```yaml
variables:
  FEATURE_FLAG_SOURCE: "group-default"  # Lowest priority

build_app:
  script:
    - |
      if [ "$ENABLE_NEW_UI" = "true" ]; then
        echo "Building with new UI"
        npm run build:new-ui
      else
        echo "Building with classic UI"
        npm run build:classic
      fi
```

**Use case**: Beta project tests new features, while other projects use stable defaults.

---

## Practical Override Scenarios

### Scenario 1: Emergency Debug Override

**Setup**:
```yaml
# .gitlab-ci.yml
variables:
  LOG_LEVEL: "info"

deploy:
  variables:
    LOG_LEVEL: "info"
  script:
    - deploy_app.sh
```

**Problem**: Production deployment failing, need verbose logs NOW.

**Solution**: Click "Run Pipeline" → Add variable:
```
LOG_LEVEL = debug
```

**Precedence Chain**:
```
Manual Run (debug) ✓ WINS
    ↓ overrides
Job-level (info)
    ↓ overrides
Global (info)
```

The deployment runs with debug logging without touching code or project settings.

---

### Scenario 2: Multi-Environment Strategy

**Setup**:
```
Group Variables:
  API_ENDPOINT: "https://api-dev.company.com"      # Dev default
  CACHE_ENABLED: "true"
  
Project Variables (staging-project):
  API_ENDPOINT: "https://api-staging.company.com"  # Override for staging
  
Project Variables (production-project):
  API_ENDPOINT: "https://api.company.com"          # Override for production
  CACHE_ENABLED: "false"                           # Production needs fresh data
```

`.gitlab-ci.yml` (same in all projects):
```yaml
variables:
  TIMEOUT: "30"  # Global fallback

deploy:
  variables:
    RETRY_COUNT: "3"  # Job-level setting
  script:
    - echo "Deploying to $API_ENDPOINT"
    - echo "Cache: $CACHE_ENABLED, Timeout: $TIMEOUT, Retries: $RETRY_COUNT"
    - ./deploy.sh
```

**Result per project**:
- **Dev projects**: API_ENDPOINT from group, CACHE_ENABLED=true
- **Staging project**: API_ENDPOINT overridden, CACHE_ENABLED=true from group
- **Production project**: Both API_ENDPOINT and CACHE_ENABLED overridden

---

### Scenario 3: The Full Stack Override

Let's trace `DEPLOY_ENV` through all levels:

```yaml
# .gitlab-ci.yml
variables:
  DEPLOY_ENV: "development"  # Level 8: Global

deploy_staging:
  variables:
    DEPLOY_ENV: "staging"    # Level 7: Job
  script:
    - echo $DEPLOY_ENV
```

**Configuration**:
```
Group Variables:
  DEPLOY_ENV: "group-default"

Project Variables:
  DEPLOY_ENV: "production"

Manual Run:
  DEPLOY_ENV: "hotfix"
```

**When you run the pipeline normally**:
```
Precedence resolution:
Project (production) ← WINS
    ↓ would override
Job (staging)
    ↓ would override
Global (development)
    ↓ would override
Group (group-default)

Result: DEPLOY_ENV = "production"
```

**When you manually run with override**:
```
Precedence resolution:
Manual Run (hotfix) ← WINS
    ↓ overrides
Project (production)
    ↓ would override
Job (staging)
    ↓ would override
Global (development)
    ↓ would override
Group (group-default)

Result: DEPLOY_ENV = "hotfix"
```

---

## Best Practices

### 1. **Use the Right Level for the Job**

| Variable Type         | Best Level              | Reason                                |
| --------------------- | ----------------------- | ------------------------------------- |
| Secrets (API keys)    | Project/Group           | Secure, masked, not in code           |
| Default versions      | Global (.gitlab-ci.yml) | Visible, version controlled           |
| Job-specific config   | Job-level               | Clear which job uses what             |
| Shared infrastructure | Group                   | DRY principle, single source of truth |
| Quick testing         | Manual run              | No code changes needed                |

### 2. **Document Your Strategy**

Add comments to your `.gitlab-ci.yml`:
```yaml
variables:
  # Default to Node 18 - can be overridden at project level for legacy apps
  NODE_VERSION: "18"
  
  # API URL is set at group level - production projects override it
  # Group: api-staging.company.com
  # Production projects: api.company.com
  API_ENDPOINT: "fallback-value"
```

### 3. **Avoid Variable Name Collisions**

Use prefixes to avoid conflicts:
```yaml
# Good
CI_IMAGE_TAG: "latest"
CI_DOCKER_REGISTRY: "registry.gitlab.com"
CI_BUILD_TYPE: "production"

# Bad (conflicts with GitLab predefined)
CI_COMMIT_SHA: "custom-value"  # Don't do this!
```

### 4. **Use Masked Variables for Secrets**

In Project/Group Settings → CI/CD → Variables:
- ✅ Check "Mask variable" for passwords, tokens, keys
- ✅ Check "Protect variable" for production credentials
- ✅ Use "File" type for certificates or config files

### 5. **Test Precedence with Debug Job**

Add this to your pipeline:
```yaml
debug_variables:
  stage: .pre
  script:
    - echo "NODE_VERSION=$NODE_VERSION"
    - echo "API_ENDPOINT=$API_ENDPOINT"
    - echo "DEPLOY_ENV=$DEPLOY_ENV"
    - env | grep CI_ | sort
  when: manual
```

Run it to see which values won the precedence battle.

---

## Common Pitfalls

### ❌ Pitfall 1: "Why isn't my variable changing?"

```yaml
variables:
  MY_VAR: "default"

job:
  script:
    - MY_VAR="changed"
    - echo $MY_VAR  # Prints "changed"
    
next_job:
  script:
    - echo $MY_VAR  # Prints "default" - variables don't carry between jobs!
```

**Fix**: Use artifacts or pass via `dotenv` artifacts if needed.

### ❌ Pitfall 2: "My project variable isn't working!"

Check if a job-level variable is overriding it:
```yaml
variables:
  API_KEY: "will-be-overridden"

job:
  variables:
    API_KEY: "job-level-wins"  # This beats project variables!
```

### ❌ Pitfall 3: "Group variable isn't applying"

Ensure the project is actually in that group:
- Check: Project → Settings → General → Project name (should show group/project)
- Variables only inherit from the immediate parent group

---

## Quick Reference Cheat Sheet

**Need to override for all projects?** → Use **Group Variables**

**Need to override for one project?** → Use **Project Variables**

**Need to set defaults in code?** → Use **Global Variables** (`.gitlab-ci.yml`)

**Need job-specific values?** → Use **Job Variables** (`.gitlab-ci.yml`)

**Need to test one run?** → Use **Manual Run Variables** (UI)

**Need to schedule different values?** → Use **Scheduled Pipeline Variables**

---

## Summary

Understanding GitLab's variable precedence is essential for:
- ✅ Writing maintainable CI/CD pipelines
- ✅ Managing secrets securely at the right level
- ✅ Avoiding configuration conflicts
- ✅ Quick debugging and testing
- ✅ Scaling across teams and projects

Remember: **Higher precedence levels always win.** Manual overrides beat UI settings, UI settings beat code, and job-level beats global.

When in doubt, use the debug job to see what's actually running!