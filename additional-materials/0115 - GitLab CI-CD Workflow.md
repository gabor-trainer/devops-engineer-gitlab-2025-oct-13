# GitLab CI/CD Workflow


## Table of Contents

1. [Introduction to Workflow](#introduction-to-workflow)
2. [Workflow Rules](#workflow-rules)
3. [Workflow Name](#workflow-name)
4. [Workflow Auto-Cancel](#workflow-auto-cancel)
5. [Real-World Workflow Examples](#real-world-workflow-examples)


## Introduction to Workflow

### What is `workflow`?

The `workflow` keyword is a **top-level keyword** that controls whether an **entire pipeline** is created or not. Think of it as the **master switch** for your pipeline.

```yaml
workflow:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "main"'

# Rest of your pipeline configuration
stages:
  - build
  - test
```

### Workflow vs Job Rules: What's the Difference?

| Aspect          | `workflow:rules`                   | Job `rules`                       |
| --------------- | ---------------------------------- | --------------------------------- |
| **Scope**       | Controls entire pipeline creation  | Controls individual job execution |
| **Location**    | Top-level, before jobs             | Inside specific jobs              |
| **Purpose**     | "Should this pipeline run at all?" | "Should this job run?"            |
| **Evaluation**  | First thing evaluated              | After pipeline is created         |
| **When to use** | Prevent unnecessary pipelines      | Control specific job behavior     |

**Mental Model:**
```
┌─────────────────────────────────────┐
│  1. workflow:rules evaluated        │
│     ↓                                │
│  Decision: Create pipeline? YES/NO  │
└─────────────────────────────────────┘
              ↓ YES
┌─────────────────────────────────────┐
│  2. Pipeline created                │
│     ↓                                │
│  3. Each job's rules evaluated      │
│     ↓                                │
│  Decision: Run this job? YES/NO     │
└─────────────────────────────────────┘
```

### Why Use Workflow?

**Problem Without Workflow:**

Imagine you push to a branch. GitLab might create:
- A branch pipeline
- A merge request pipeline (if there's an open MR)
- Multiple pipelines for the same commit

This wastes CI/CD minutes and creates confusion.

**Solution With Workflow:**

Control exactly when pipelines are created, avoiding duplicates and unnecessary runs.

---

## Workflow Rules

### Basic Syntax

```yaml
workflow:
  rules:
    - if: CONDITION
      when: always    # or never
    - when: never     # Default fallback
```

### Key Characteristics

1. **Only two `when` values allowed:**
   - `always` - Create the pipeline
   - `never` - Don't create the pipeline
   - (No `on_success`, `on_failure`, `manual`, or `delayed`)

2. **First matching rule wins** (just like job rules)

3. **Default behavior if no rules match:**
   - Pipeline is created (unless you add `- when: never` as a fallback)

### Common Pattern: Explicit Control

```yaml
workflow:
  rules:
    # Define when to create pipelines
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "main"'
    - if: '$CI_COMMIT_TAG'
    
    # Explicitly block everything else
    - when: never
```

This pattern says: "Only create pipelines for MRs, main branch, or tags. Block everything else."

---

## Real-World Workflow Examples

### Example 1: Prevent Duplicate Pipelines

**Problem:** You have an open MR. When you push, GitLab creates both:
- A branch pipeline
- A merge request pipeline

**Solution:**

```yaml
workflow:
  rules:
    # Only create MR pipelines when MR exists
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    
    # Create branch pipelines only for main/develop when NO MR exists
    - if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "push"'
    - if: '$CI_COMMIT_BRANCH == "develop" && $CI_PIPELINE_SOURCE == "push"'
    
    # Create tag pipelines
    - if: '$CI_COMMIT_TAG'
    
    # Create scheduled pipelines
    - if: '$CI_PIPELINE_SOURCE == "schedule"'
    
    # Block everything else (feature branches without MRs)
    - when: never

stages:
  - test
  - deploy

test:
  script: echo "Testing"

deploy:
  script: echo "Deploying"
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```

**Result:**
- Feature branch with MR → MR pipeline only
- Feature branch without MR → No pipeline
- Main branch → Branch pipeline (no MR)
- Tags → Tag pipeline
- Scheduled → Scheduled pipeline

---

### Example 2: Skip CI for Certain Commits

**Scenario:** Developers want to skip CI for documentation-only changes or when commit contains `[skip ci]`.

```yaml
workflow:
  rules:
    # Never run if commit message has [skip ci]
    - if: '$CI_COMMIT_MESSAGE =~ /\[skip ci\]/i'
      when: never
    
    # Never run if commit message has [ci skip]
    - if: '$CI_COMMIT_MESSAGE =~ /\[ci skip\]/i'
      when: never
    
    # Never run if ONLY documentation files changed
    # (You'd need to use changes: in jobs for this)
    
    # Run for merge requests
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    
    # Run for main branch
    - if: '$CI_COMMIT_BRANCH == "main"'
    
    # Block everything else
    - when: never

stages:
  - test

test:
  script: npm test
```

**Usage:**
```bash
git commit -m "Update README [skip ci]"
git push
# No pipeline created!
```

---

### Example 3: Different Pipelines for Different Branches

**Scenario:** You want different pipeline behavior based on branch type.

```yaml
workflow:
  rules:
    # Production pipeline (main branch)
    - if: '$CI_COMMIT_BRANCH == "main"'
      variables:
        PIPELINE_TYPE: "production"
    
    # Staging pipeline (develop branch)
    - if: '$CI_COMMIT_BRANCH == "develop"'
      variables:
        PIPELINE_TYPE: "staging"
    
    # Feature pipeline (merge requests)
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      variables:
        PIPELINE_TYPE: "feature"
    
    # Hotfix pipeline (hotfix/* branches)
    - if: '$CI_COMMIT_BRANCH =~ /^hotfix\/.*/'
      variables:
        PIPELINE_TYPE: "hotfix"
    
    - when: never

stages:
  - test
  - deploy

unit_tests:
  stage: test
  script:
    - echo "Running tests for $PIPELINE_TYPE pipeline"
    - npm test

integration_tests:
  stage: test
  script: npm run test:integration
  rules:
    # Only run integration tests for staging and production
    - if: '$PIPELINE_TYPE == "staging" || $PIPELINE_TYPE == "production"'

deploy:
  stage: deploy
  script:
    - echo "Deploying to $PIPELINE_TYPE environment"
    - ./deploy.sh $PIPELINE_TYPE
  rules:
    # Deploy only for non-feature pipelines
    - if: '$PIPELINE_TYPE != "feature"'
```

**Result:**
- Main branch → Full pipeline with deployment
- Develop branch → Full pipeline with staging deployment
- MR → Tests only, no deployment
- Hotfix branch → Full pipeline with hotfix deployment

---

### Example 4: Scheduled vs Manual Pipelines

**Scenario:** Different behavior for nightly builds vs on-demand runs.

```yaml
workflow:
  rules:
    # Scheduled pipelines (nightly builds)
    - if: '$CI_PIPELINE_SOURCE == "schedule"'
      variables:
        RUN_FULL_SUITE: "true"
        DEPLOY_ENABLED: "false"
    
    # Manual pipelines (web UI trigger)
    - if: '$CI_PIPELINE_SOURCE == "web"'
      variables:
        RUN_FULL_SUITE: "false"
        DEPLOY_ENABLED: "true"
    
    # Merge request pipelines
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      variables:
        RUN_FULL_SUITE: "false"
        DEPLOY_ENABLED: "false"
    
    # Main branch
    - if: '$CI_COMMIT_BRANCH == "main"'
      variables:
        RUN_FULL_SUITE: "false"
        DEPLOY_ENABLED: "true"
    
    - when: never

stages:
  - test
  - deploy

quick_tests:
  stage: test
  script: npm test
  rules:
    - if: '$RUN_FULL_SUITE == "false"'

comprehensive_tests:
  stage: test
  script: npm run test:comprehensive
  rules:
    - if: '$RUN_FULL_SUITE == "true"'

performance_tests:
  stage: test
  script: npm run test:performance
  rules:
    - if: '$RUN_FULL_SUITE == "true"'

security_scan:
  stage: test
  script: npm audit
  rules:
    - if: '$RUN_FULL_SUITE == "true"'

deploy:
  stage: deploy
  script: ./deploy.sh
  rules:
    - if: '$DEPLOY_ENABLED == "true"'
      when: manual
```

**Result:**
- Scheduled (nightly) → Comprehensive tests, no deploy
- Manual trigger → Quick tests, manual deploy
- MR → Quick tests only
- Main branch → Quick tests, manual deploy

---

### Example 5: Multi-Project Coordination

**Scenario:** Only run downstream project pipelines for certain conditions.

```yaml
# In upstream project
workflow:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      variables:
        TRIGGER_DOWNSTREAM: "false"
    
    - if: '$CI_COMMIT_BRANCH == "main"'
      variables:
        TRIGGER_DOWNSTREAM: "true"
    
    - if: '$CI_COMMIT_TAG'
      variables:
        TRIGGER_DOWNSTREAM: "true"
    
    - when: never

stages:
  - test
  - trigger

test:
  stage: test
  script: npm test

trigger_downstream:
  stage: trigger
  trigger:
    project: my-group/downstream-project
    branch: main
  rules:
    - if: '$TRIGGER_DOWNSTREAM == "true"'
```

---

## Workflow Name

### Dynamic Pipeline Names

You can customize the pipeline name that appears in the GitLab UI.

```yaml
workflow:
  name: 'Pipeline for $CI_COMMIT_BRANCH'
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'

stages:
  - test
```

**Advanced Example:**

```yaml
workflow:
  name: |
    $CI_PIPELINE_SOURCE: $CI_COMMIT_REF_NAME
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      variables:
        WORKFLOW_NAME: "MR !$CI_MERGE_REQUEST_IID: $CI_COMMIT_TITLE"
    - if: '$CI_COMMIT_TAG'
      variables:
        WORKFLOW_NAME: "Release $CI_COMMIT_TAG"
    - if: '$CI_COMMIT_BRANCH == "main"'
      variables:
        WORKFLOW_NAME: "Production Pipeline"
    - if: '$CI_PIPELINE_SOURCE == "schedule"'
      variables:
        WORKFLOW_NAME: "Nightly Build - $CI_COMMIT_SHORT_SHA"
    - when: always
      variables:
        WORKFLOW_NAME: "Branch: $CI_COMMIT_BRANCH"

# Note: You can use $WORKFLOW_NAME in jobs if needed
```

**Result in UI:**
- MR pipeline → "MR !123: Add new feature"
- Tag pipeline → "Release v1.2.3"
- Main branch → "Production Pipeline"
- Scheduled → "Nightly Build - a1b2c3d"
- Other branches → "Branch: feature/new-thing"

---

## Workflow Auto-Cancel

### Automatic Pipeline Cancellation

Control what happens when new commits are pushed while a pipeline is running.

```yaml
workflow:
  auto_cancel:
    on_new_commit: interruptible  # or none, conservative
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "main"'
```

### Auto-Cancel Options

| Value           | Behavior                                                       |
| --------------- | -------------------------------------------------------------- |
| `interruptible` | Cancel running pipeline if new commit pushed (default for MRs) |
| `none`          | Never cancel running pipelines                                 |
| `conservative`  | Cancel only if the pipeline hasn't started critical jobs       |

**Real-World Example:**

```yaml
workflow:
  auto_cancel:
    on_new_commit: interruptible
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "main"'
      variables:
        AUTO_CANCEL_MODE: "none"  # Don't cancel production pipelines
    - when: never

stages:
  - test
  - deploy

test:
  stage: test
  interruptible: true  # Can be interrupted
  script: npm test

deploy_staging:
  stage: deploy
  interruptible: true
  script: ./deploy.sh staging
  rules:
    - if: '$CI_COMMIT_BRANCH == "develop"'

deploy_production:
  stage: deploy
  interruptible: false  # NEVER interrupt production deploys
  script: ./deploy.sh production
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual
```

**Behavior:**
- In MR: Push new commit → Old pipeline cancelled immediately
- On main: Deploy job running → New commit doesn't cancel it
- Staging: Can be interrupted
- Production: Never interrupted (even if auto_cancel is set)

---

## Common Workflow Patterns

### Pattern 1: The "Smart Default"

Works for most projects with standard branching.

```yaml
workflow:
  rules:
    # MR pipelines
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    
    # Main/default branch
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
    
    # Tags
    - if: '$CI_COMMIT_TAG'
    
    # Scheduled
    - if: '$CI_PIPELINE_SOURCE == "schedule"'
    
    # Explicit web triggers
    - if: '$CI_PIPELINE_SOURCE == "web"'
    
    # Block all other scenarios
    - when: never
```

### Pattern 2: The "Environment Specific"

Different pipelines for different environments.

```yaml
workflow:
  rules:
    # Production
    - if: '$CI_COMMIT_BRANCH == "main"'
      variables:
        ENVIRONMENT: "production"
        DEPLOY_MANUAL: "true"
        FULL_TESTS: "true"
    
    # Staging
    - if: '$CI_COMMIT_BRANCH == "staging"'
      variables:
        ENVIRONMENT: "staging"
        DEPLOY_MANUAL: "false"
        FULL_TESTS: "true"
    
    # Development
    - if: '$CI_COMMIT_BRANCH == "develop"'
      variables:
        ENVIRONMENT: "development"
        DEPLOY_MANUAL: "false"
        FULL_TESTS: "false"
    
    # MRs (no deployment)
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      variables:
        ENVIRONMENT: "test"
        DEPLOY_MANUAL: "false"
        FULL_TESTS: "false"
    
    - when: never
```

### Pattern 3: The "Feature Flag Controller"

Control entire pipeline with feature flags.

```yaml
workflow:
  rules:
    # Pipeline disabled globally
    - if: '$PIPELINE_DISABLED == "true"'
      when: never
    
    # Maintenance mode
    - if: '$MAINTENANCE_MODE == "true"'
      when: never
    
    # Emergency only mode
    - if: '$EMERGENCY_ONLY == "true"'
      variables:
        SKIP_TESTS: "true"
        FAST_DEPLOY: "true"
    
    # Normal operation
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "main"'
    - when: never
```

