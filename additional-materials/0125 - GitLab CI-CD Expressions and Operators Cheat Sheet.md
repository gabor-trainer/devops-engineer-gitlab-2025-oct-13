# GitLab CI/CD Expressions & Operators Cheat Sheet

## Table of Contents
1. [Basic Syntax](#basic-syntax)
2. [Comparison Operators](#comparison-operators)
3. [Logical Operators](#logical-operators)
4. [Pattern Matching Operators](#pattern-matching-operators)
5. [Variable Existence & Null Checks](#variable-existence--null-checks)
6. [Common Predefined Variables](#common-predefined-variables)
7. [Real-World Examples](#real-world-examples)
8. [Advanced Patterns](#advanced-patterns)
9. [Common Pitfalls](#common-pitfalls)

---

## Basic Syntax

### Where Expressions Are Used

```yaml
# 1. Job rules (most common)
job_name:
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'

# 2. Workflow rules (pipeline-level)
workflow:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'

# 3. Include rules (conditional file inclusion)
include:
  - local: 'path/to/file.yml'
    rules:
      - if: '$CI_COMMIT_TAG'
```

### Expression Structure

```yaml
rules:
  - if: 'EXPRESSION'           # The condition
    when: on_success           # What to do if true (default)
    allow_failure: false       # Optional configuration
```

---

## Comparison Operators

### Equality Operators

| Operator | Description  | Example                          |
| -------- | ------------ | -------------------------------- |
| `==`     | Equal to     | `$CI_COMMIT_BRANCH == "main"`    |
| `!=`     | Not equal to | `$CI_COMMIT_BRANCH != "develop"` |

**Examples:**

```yaml
# Check if on main branch
rules:
  - if: '$CI_COMMIT_BRANCH == "main"'

# Check if NOT a merge request
rules:
  - if: '$CI_PIPELINE_SOURCE != "merge_request_event"'

# Check specific variable value
rules:
  - if: '$DEPLOY_ENVIRONMENT == "production"'

# Check multiple values
rules:
  - if: '$BUILD_TYPE == "release" || $BUILD_TYPE == "hotfix"'
```

### Comparison Operators (Numbers & Strings)

| Operator | Description           | Example                 |
| -------- | --------------------- | ----------------------- |
| `<`      | Less than             | `$RUNNER_COUNT < "5"`   |
| `>`      | Greater than          | `$VERSION_NUMBER > "2"` |
| `<=`     | Less than or equal    | `$MAX_RETRIES <= "3"`   |
| `>=`     | Greater than or equal | `$MIN_VERSION >= "1.5"` |

**Important**: All comparisons are **string-based** in GitLab CI/CD.

**Examples:**

```yaml
# Numeric comparison (as strings)
rules:
  - if: '$PARALLEL_COUNT > "1"'

# Version comparison (lexicographic)
rules:
  - if: '$NODE_VERSION >= "18"'

# Combined with equality
rules:
  - if: '$RETRY_COUNT >= "3" && $RETRY_COUNT <= "10"'
```

⚠️ **Warning**: String comparison can be tricky:
```yaml
# This might not work as expected!
- if: '$VERSION > "9"'  # "10" < "9" in string comparison!

# Better approach - pad numbers or use regex
- if: '$VERSION =~ /^1[0-9]$/ || $VERSION =~ /^[2-9][0-9]$/'
```

---

## Logical Operators

### AND, OR, NOT

| Operator | Description | Syntax                       |
| -------- | ----------- | ---------------------------- |
| `&&`     | Logical AND | `condition1 && condition2`   |
| `\|\|`   | Logical OR  | `condition1 \|\| condition2` |
| `!`      | Logical NOT | `!condition`                 |

### Operator Precedence (Highest to Lowest)

1. `!` (NOT)
2. `&&` (AND)
3. `||` (OR)

**Examples:**

```yaml
# AND: Both conditions must be true
rules:
  - if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "push"'

# OR: Either condition can be true
rules:
  - if: '$CI_COMMIT_BRANCH == "main" || $CI_COMMIT_BRANCH == "develop"'

# NOT: Invert condition
rules:
  - if: '!($CI_COMMIT_BRANCH == "main")'
    # Runs when NOT on main

# Complex combination
rules:
  - if: '$CI_COMMIT_BRANCH == "main" && ($DEPLOY_ENV == "prod" || $DEPLOY_ENV == "staging")'

# NOT with variable existence
rules:
  - if: '!($CI_MERGE_REQUEST_ID)'
    # Runs when NOT a merge request

# Multiple ANDs
rules:
  - if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "push" && $DEPLOY_ENABLED == "true"'
```

### Parentheses for Grouping

```yaml
# Without parentheses (may not work as intended)
rules:
  - if: '$A == "x" || $B == "y" && $C == "z"'
    # Evaluated as: $A == "x" || ($B == "y" && $C == "z")

# With parentheses (explicit)
rules:
  - if: '($A == "x" || $B == "y") && $C == "z"'
    # Now it's clear what we want
```

---

## Pattern Matching Operators

### Regex Match

| Operator | Description                  | Example                                         |
| -------- | ---------------------------- | ----------------------------------------------- |
| `=~`     | Matches regex pattern        | `$CI_COMMIT_BRANCH =~ /^feature\/.*/`           |
| `!~`     | Does NOT match regex pattern | `$CI_COMMIT_TAG !~ /^v[0-9]+\.[0-9]+\.[0-9]+$/` |

**Important**: Regex patterns must be enclosed in `/pattern/`.

### Common Regex Patterns

```yaml
# Branch name patterns
rules:
  # Feature branches (feature/*, feature-*)
  - if: '$CI_COMMIT_BRANCH =~ /^feature\/.*/'
  
  # Release branches (release/1.0, release/2.5)
  - if: '$CI_COMMIT_BRANCH =~ /^release\/[0-9]+\.[0-9]+$/'
  
  # Hotfix branches
  - if: '$CI_COMMIT_BRANCH =~ /^hotfix\/.*/'
  
  # Main or master branch
  - if: '$CI_COMMIT_BRANCH =~ /^(main|master)$/'

# Tag patterns
rules:
  # Semantic versioning tags (v1.0.0, v2.3.1)
  - if: '$CI_COMMIT_TAG =~ /^v[0-9]+\.[0-9]+\.[0-9]+$/'
  
  # Pre-release tags (v1.0.0-beta.1)
  - if: '$CI_COMMIT_TAG =~ /^v[0-9]+\.[0-9]+\.[0-9]+-.*$/'
  
  # Any tag
  - if: '$CI_COMMIT_TAG =~ /.*/'

# Commit message patterns
rules:
  # Contains [skip ci] or [ci skip]
  - if: '$CI_COMMIT_MESSAGE =~ /\[(skip ci|ci skip)\]/'
    when: never
  
  # Starts with "feat:" or "fix:"
  - if: '$CI_COMMIT_MESSAGE =~ /^(feat|fix):/'
  
  # Contains JIRA ticket
  - if: '$CI_COMMIT_MESSAGE =~ /[A-Z]+-[0-9]+/'

# File path patterns (for changes)
rules:
  # Only run if backend files changed
  - if: '$CI_COMMIT_MESSAGE =~ /backend/'
  
  # Check for documentation changes
  - if: '$CI_COMMIT_BRANCH =~ /^docs\/.*/'
```

### Case-Insensitive Regex

```yaml
# Case-insensitive flag: /pattern/i
rules:
  - if: '$CI_COMMIT_MESSAGE =~ /\[skip ci\]/i'
    # Matches: [SKIP CI], [Skip CI], [skip ci], etc.
```

### Negative Regex Match

```yaml
# Does NOT match pattern
rules:
  # Run only if NOT a documentation branch
  - if: '$CI_COMMIT_BRANCH !~ /^docs\/.*/'
  
  # Run only if NOT a pre-release tag
  - if: '$CI_COMMIT_TAG !~ /-(alpha|beta|rc)/'
```

---

## Variable Existence & Null Checks

### Check if Variable Exists

| Pattern             | Description                         | Use Case           |
| ------------------- | ----------------------------------- | ------------------ |
| `$VARIABLE`         | Variable exists and is not empty    | Check if defined   |
| `$VARIABLE == null` | Variable does not exist             | Check if undefined |
| `$VARIABLE != null` | Variable exists (may be empty)      | Check if defined   |
| `$VARIABLE == ""`   | Variable exists but is empty string | Check if empty     |
| `$VARIABLE != ""`   | Variable exists and is not empty    | Check if has value |

**Examples:**

```yaml
# Run only if variable is defined and not empty
rules:
  - if: '$DEPLOY_KEY'
    # True if DEPLOY_KEY has any value

# Run only if variable is NOT defined
rules:
  - if: '$CI_COMMIT_TAG == null'
    # True if no tag

# Run if variable is defined (even if empty)
rules:
  - if: '$MY_VAR != null'

# Run if variable has a value
rules:
  - if: '$MY_VAR != null && $MY_VAR != ""'

# Run if variable is empty or not defined
rules:
  - if: '$MY_VAR == null || $MY_VAR == ""'
```

### Common Existence Checks

```yaml
# Check if this is a tag
rules:
  - if: '$CI_COMMIT_TAG'
    # Short for: $CI_COMMIT_TAG != null && $CI_COMMIT_TAG != ""

# Check if this is NOT a tag
rules:
  - if: '$CI_COMMIT_TAG == null'

# Check if merge request
rules:
  - if: '$CI_MERGE_REQUEST_IID'

# Check if NOT a merge request
rules:
  - if: '$CI_MERGE_REQUEST_IID == null'

# Check if scheduled pipeline
rules:
  - if: '$CI_PIPELINE_SOURCE == "schedule"'

# Check if manual trigger
rules:
  - if: '$CI_PIPELINE_SOURCE == "web"'
```

---

## Common Predefined Variables

### Pipeline Source Variables

```yaml
# $CI_PIPELINE_SOURCE possible values:
- push                    # Git push
- web                     # Manual trigger from UI
- schedule                # Scheduled pipeline
- api                     # API trigger
- external                # External CI trigger
- pipeline                # Multi-project pipeline
- chat                    # ChatOps trigger
- merge_request_event     # Merge request
- parent_pipeline         # Parent-child pipeline
- trigger                 # Legacy trigger token

# Examples:
rules:
  - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  - if: '$CI_PIPELINE_SOURCE == "schedule"'
  - if: '$CI_PIPELINE_SOURCE == "web"'
```

### Branch & Tag Variables

```yaml
$CI_COMMIT_BRANCH         # Branch name (null if tag)
$CI_COMMIT_TAG           # Tag name (null if branch)
$CI_COMMIT_REF_NAME      # Branch or tag name
$CI_DEFAULT_BRANCH       # Default branch (usually "main")
$CI_MERGE_REQUEST_SOURCE_BRANCH_NAME  # MR source branch
$CI_MERGE_REQUEST_TARGET_BRANCH_NAME  # MR target branch

# Examples:
rules:
  - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
  - if: '$CI_COMMIT_TAG =~ /^v.*/'
  - if: '$CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "main"'
```

### Merge Request Variables

```yaml
$CI_MERGE_REQUEST_IID              # MR internal ID
$CI_MERGE_REQUEST_ID               # MR global ID
$CI_MERGE_REQUEST_TITLE            # MR title
$CI_MERGE_REQUEST_LABELS           # MR labels (comma-separated)
$CI_MERGE_REQUEST_SOURCE_BRANCH_NAME
$CI_MERGE_REQUEST_TARGET_BRANCH_NAME

# Examples:
rules:
  - if: '$CI_MERGE_REQUEST_LABELS =~ /\bready\b/'
  - if: '$CI_MERGE_REQUEST_TITLE =~ /^WIP:/'
    when: never
```

### Commit Variables

```yaml
$CI_COMMIT_MESSAGE       # Full commit message
$CI_COMMIT_TITLE         # First line of commit message
$CI_COMMIT_SHA           # Full commit SHA
$CI_COMMIT_SHORT_SHA     # Short commit SHA
$CI_COMMIT_AUTHOR        # Commit author

# Examples:
rules:
  - if: '$CI_COMMIT_MESSAGE =~ /\[skip ci\]/'
    when: never
  - if: '$CI_COMMIT_TITLE =~ /^feat:/'
```

### Project & User Variables

```yaml
$CI_PROJECT_NAME         # Project name
$CI_PROJECT_PATH         # Project path with namespace
$CI_PROJECT_NAMESPACE    # Project namespace
$GITLAB_USER_LOGIN       # User who triggered pipeline
$GITLAB_USER_EMAIL       # User email

# Examples:
rules:
  - if: '$GITLAB_USER_LOGIN == "deploy-bot"'
  - if: '$CI_PROJECT_NAMESPACE == "production"'
```

---

## Real-World Examples

### Example 1: Branch-Based Deployment

```yaml
deploy_production:
  script:
    - echo "Deploying to production"
  rules:
    # Deploy to prod only on main branch, pushed (not MR), not scheduled
    - if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "push"'

deploy_staging:
  script:
    - echo "Deploying to staging"
  rules:
    # Deploy to staging on develop or release branches
    - if: '$CI_COMMIT_BRANCH == "develop"'
    - if: '$CI_COMMIT_BRANCH =~ /^release\/.*/'
```

### Example 2: Skip Pipeline for Certain Commits

```yaml
workflow:
  rules:
    # Don't run pipeline if commit message contains [skip ci]
    - if: '$CI_COMMIT_MESSAGE =~ /\[skip ci\]/i'
      when: never
    
    # Don't run pipeline for draft merge requests
    - if: '$CI_MERGE_REQUEST_TITLE =~ /^(Draft|WIP):/i'
      when: never
    
    # Run pipeline for everything else
    - when: always
```

### Example 3: Tag-Based Release

```yaml
release_production:
  script:
    - echo "Creating production release"
  rules:
    # Only on semantic version tags (v1.0.0, v2.3.1)
    - if: '$CI_COMMIT_TAG =~ /^v[0-9]+\.[0-9]+\.[0-9]+$/'

release_prerelease:
  script:
    - echo "Creating pre-release"
  rules:
    # Only on pre-release tags (v1.0.0-beta.1, v2.0.0-rc.2)
    - if: '$CI_COMMIT_TAG =~ /^v[0-9]+\.[0-9]+\.[0-9]+-(alpha|beta|rc)/'
```

### Example 4: Merge Request Pipelines

```yaml
test_mr:
  script:
    - npm test
  rules:
    # Run on merge requests targeting main or develop
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event" && ($CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "main" || $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "develop")'

review_app:
  script:
    - deploy_review_app.sh
  rules:
    # Deploy review app for MRs, but not drafts
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event" && $CI_MERGE_REQUEST_TITLE !~ /^(Draft|WIP):/'
```

### Example 5: Scheduled vs Manual Triggers

```yaml
nightly_tests:
  script:
    - run_comprehensive_tests.sh
  rules:
    # Only on scheduled pipelines
    - if: '$CI_PIPELINE_SOURCE == "schedule"'

manual_deploy:
  script:
    - deploy.sh
  rules:
    # Only when manually triggered from UI
    - if: '$CI_PIPELINE_SOURCE == "web"'
      when: manual
```

### Example 6: Feature Flag Control

```yaml
experimental_feature:
  script:
    - run_experimental.sh
  rules:
    # Run only if feature flag is enabled AND on non-production branch
    - if: '$ENABLE_EXPERIMENTAL == "true" && $CI_COMMIT_BRANCH != "main"'

performance_tests:
  script:
    - run_perf_tests.sh
  rules:
    # Run perf tests if explicitly enabled OR on release branches
    - if: '$RUN_PERF_TESTS == "true"'
    - if: '$CI_COMMIT_BRANCH =~ /^release\/.*/'
```

### Example 7: Environment-Specific Jobs

```yaml
database_migration:
  script:
    - run_migrations.sh
  rules:
    # Run migrations only when DEPLOY_ENV is set and on main
    - if: '$DEPLOY_ENV && $CI_COMMIT_BRANCH == "main"'

security_scan:
  script:
    - security_scan.sh
  rules:
    # Run security scans on MRs to main, tags, and scheduled pipelines
    - if: '$CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "main"'
    - if: '$CI_COMMIT_TAG'
    - if: '$CI_PIPELINE_SOURCE == "schedule"'
```

### Example 8: User-Based Restrictions

```yaml
production_deploy:
  script:
    - deploy_to_prod.sh
  rules:
    # Only specific users can deploy to production
    - if: '$CI_COMMIT_BRANCH == "main" && ($GITLAB_USER_LOGIN == "admin" || $GITLAB_USER_LOGIN == "deploy-bot")'
      when: manual

auto_deploy_staging:
  script:
    - deploy_to_staging.sh
  rules:
    # Anyone can deploy to staging automatically
    - if: '$CI_COMMIT_BRANCH == "develop"'
```

### Example 9: Label-Based CI/CD

```yaml
e2e_tests:
  script:
    - run_e2e_tests.sh
  rules:
    # Run E2E tests if MR has 'test:e2e' label
    - if: '$CI_MERGE_REQUEST_LABELS =~ /\btest:e2e\b/'

skip_tests:
  script:
    - echo "Skipping tests"
  rules:
    # Skip if MR has 'skip-ci' label
    - if: '$CI_MERGE_REQUEST_LABELS =~ /\bskip-ci\b/'
      when: never
```

### Example 10: Complex Multi-Condition Logic

```yaml
conditional_deploy:
  script:
    - deploy.sh
  rules:
    # Deploy if:
    # - On main branch AND manually triggered
    # - OR on a release tag
    # - OR on develop branch with FORCE_DEPLOY enabled
    # - BUT NEVER if commit message has [no-deploy]
    
    - if: '$CI_COMMIT_MESSAGE =~ /\[no-deploy\]/'
      when: never
    
    - if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "web"'
    
    - if: '$CI_COMMIT_TAG =~ /^v[0-9]+\.[0-9]+\.[0-9]+$/'
    
    - if: '$CI_COMMIT_BRANCH == "develop" && $FORCE_DEPLOY == "true"'
```

---

## Advanced Patterns

### Pattern 1: Multiple Rules with Different Actions

```yaml
deploy:
  script:
    - deploy.sh
  rules:
    # Automatic on main
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: on_success
    
    # Manual on develop
    - if: '$CI_COMMIT_BRANCH == "develop"'
      when: manual
    
    # Delayed on staging
    - if: '$CI_COMMIT_BRANCH == "staging"'
      when: delayed
      start_in: 30 minutes
    
    # Never run otherwise
    - when: never
```

### Pattern 2: Matrix of Conditions

```yaml
test:
  script:
    - run_tests.sh
  rules:
    # Run on: [main, develop] × [push, merge_request]
    - if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "push"'
    - if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "develop" && $CI_PIPELINE_SOURCE == "push"'
    - if: '$CI_COMMIT_BRANCH == "develop" && $CI_PIPELINE_SOURCE == "merge_request_event"'
```

### Pattern 3: Cascading Defaults

```yaml
.base_rules:
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'

test:
  extends: .base_rules
  script:
    - test.sh
  rules:
    # Override base rules
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - !reference [.base_rules, rules]
```

### Pattern 4: File-Based Triggers with Changes

```yaml
backend_tests:
  script:
    - test_backend.sh
  rules:
    # Run only if backend files changed
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      changes:
        - backend/**/*
        - package.json

frontend_tests:
  script:
    - test_frontend.sh
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      changes:
        - frontend/**/*
        - package.json
```

### Pattern 5: Environment Variable Expansion

```yaml
dynamic_deploy:
  script:
    - deploy.sh $TARGET_ENV
  rules:
    # Use variable to determine environment
    - if: '$TARGET_ENV == "production" && $CI_COMMIT_TAG'
    - if: '$TARGET_ENV == "staging" && $CI_COMMIT_BRANCH == "develop"'
  environment:
    name: $TARGET_ENV
```

---

## Common Pitfalls

### ❌ Pitfall 1: String vs Number Comparison

```yaml
# WRONG - String comparison gives unexpected results
rules:
  - if: '$VERSION > "9"'
    # "10" is LESS than "9" in string comparison!

# CORRECT - Use regex for numeric ranges
rules:
  - if: '$VERSION =~ /^[1-9][0-9]+$/ || $VERSION == "9"'
    # Matches 9, 10, 11, 12, etc.
```

### ❌ Pitfall 2: Forgetting Quotes

```yaml
# WRONG
rules:
  - if: $CI_COMMIT_BRANCH == "main"
    # Missing quotes around variable!

# CORRECT
rules:
  - if: '$CI_COMMIT_BRANCH == "main"'
```

### ❌ Pitfall 3: Empty String vs Null

```yaml
# These are DIFFERENT:
rules:
  - if: '$MY_VAR'              # True if exists AND not empty
  - if: '$MY_VAR != null'      # True if exists (even if empty)
  - if: '$MY_VAR == ""'        # True if exists AND is empty

# Example:
# MY_VAR="" (exists but empty)
- if: '$MY_VAR'           # FALSE
- if: '$MY_VAR != null'   # TRUE
- if: '$MY_VAR == ""'     # TRUE
```

### ❌ Pitfall 4: Regex Delimiter

```yaml
# WRONG - Missing regex delimiters
rules:
  - if: '$CI_COMMIT_BRANCH =~ ^feature/.*'
    # This will fail!

# CORRECT
rules:
  - if: '$CI_COMMIT_BRANCH =~ /^feature\/.*/'
    # Regex must be in /pattern/
```

### ❌ Pitfall 5: Operator Precedence

```yaml
# WRONG - Unexpected precedence
rules:
  - if: '$A == "x" || $B == "y" && $C == "z"'
    # Evaluated as: $A == "x" || ($B == "y" && $C == "z")

# CORRECT - Use parentheses
rules:
  - if: '($A == "x" || $B == "y") && $C == "z"'
```

### ❌ Pitfall 6: Multiple Conditions Without Proper Logic

```yaml
# WRONG - This creates multiple separate rules
rules:
  - if: '$CI_COMMIT_BRANCH == "main"'
  - if: '$CI_PIPELINE_SOURCE == "push"'
    # Both conditions checked separately (OR logic)!

# CORRECT - Use && for AND logic
rules:
  - if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "push"'
```

### ❌ Pitfall 7: Case Sensitivity

```yaml
# Branch names are case-sensitive!
rules:
  - if: '$CI_COMMIT_BRANCH == "Main"'
    # Won't match "main"!

# Use regex with case-insensitive flag if needed
rules:
  - if: '$CI_COMMIT_BRANCH =~ /^main$/i'
```

---

## Quick Reference Table

### Operators Summary

| Category       | Operator  | Example                    |
| -------------- | --------- | -------------------------- |
| **Equality**   | `==`      | `$VAR == "value"`          |
|                | `!=`      | `$VAR != "value"`          |
| **Comparison** | `<`       | `$VAR < "5"`               |
|                | `>`       | `$VAR > "5"`               |
|                | `<=`      | `$VAR <= "5"`              |
|                | `>=`      | `$VAR >= "5"`              |
| **Logical**    | `&&`      | `$A == "x" && $B == "y"`   |
|                | `\|\|`    | `$A == "x" \|\| $B == "y"` |
|                | `!`       | `!($VAR == "x")`           |
| **Pattern**    | `=~`      | `$VAR =~ /pattern/`        |
|                | `!~`      | `$VAR !~ /pattern/`        |
| **Null**       | `== null` | `$VAR == null`             |
|                | `!= null` | `$VAR != null`             |

### Common Patterns Quick Copy

```yaml
# Is this the main branch?
- if: '$CI_COMMIT_BRANCH == "main"'

# Is this a tag?
- if: '$CI_COMMIT_TAG'

# Is this a merge request?
- if: '$CI_PIPELINE_SOURCE == "merge_request_event"'

# Is this a manual trigger?
- if: '$CI_PIPELINE_SOURCE == "web"'

# Is this a scheduled pipeline?
- if: '$CI_PIPELINE_SOURCE == "schedule"'

# Feature branch?
- if: '$CI_COMMIT_BRANCH =~ /^feature\/.*/'

# Release tag?
- if: '$CI_COMMIT_TAG =~ /^v[0-9]+\.[0-9]+\.[0-9]+$/'

# Skip if commit says so?
- if: '$CI_COMMIT_MESSAGE =~ /\[skip ci\]/i'
  when: never

# Main branch on push (not MR)?
- if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "push"'

# MR targeting main?
- if: '$CI_PIPELINE_SOURCE == "merge_request_event" && $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "main"'
```

---

## Best Practices

### ✅ DO:
1. **Use quotes** around variables: `'$VAR == "value"'`
2. **Use parentheses** for complex logic: `'($A || $B) && $C'`
3. **Use regex delimiters**: `=~ /pattern/`
4. **Check for null** explicitly when needed: `$VAR != null`
5. **Use case-insensitive** regex when appropriate: `/pattern/i`
6. **Test your expressions** in a test pipeline first
7. **Comment complex rules** to explain intent

### ❌ DON'T:
1. Don't forget quotes around variables
2. Don't assume numeric comparison works
3. Don't mix string and null checks carelessly
4. Don't create overly complex expressions (split into multiple rules)
5. Don't forget that all comparisons are string-based
6. Don't ignore operator precedence
7. Don't use reserved variable names

---

## Testing Your Expressions

### Debug Rule

Add this job to test what variables are available:

```yaml
debug_rules:
  stage: .pre
  script:
    - echo "Branch=$CI_COMMIT_BRANCH"
    - echo "Tag=$CI_COMMIT_TAG"
    - echo "Source=$CI_PIPELINE_SOURCE"
    - echo "MR_IID=$CI_MERGE_REQUEST_IID"
    - env | grep CI_ | sort
  rules:
    - if: '$DEBUG_PIPELINE == "true"'
      when: always
    - when: manual
```

Run with: `DEBUG_PIPELINE=true` to see all variables.

---

## Summary

**Key Takeaways:**
- ✅ All comparisons are **string-based**
- ✅ Always use **quotes** around variables
- ✅ Use **regex** for pattern matching with `/pattern/`
- ✅ Combine conditions with **&&** (AND), **||** (OR)
- ✅ Check existence with **null** checks
- ✅ Test complex expressions in isolation first
- ✅ Use **parentheses** to clarify precedence

**Most Common Expression:**
```yaml
rules:
  - if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE == "push"'
```
