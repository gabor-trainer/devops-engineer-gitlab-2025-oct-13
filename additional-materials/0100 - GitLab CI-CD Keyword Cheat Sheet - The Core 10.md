# GitLab CI/CD Keyword Cheat Sheet: The Core 10

**Quick Reference for `.gitlab-ci.yml` Configuration**

This cheat sheet covers the 10 most essential keywords you'll use when building GitLab CI/CD pipelines. Master these, and you'll be able to create powerful, efficient automation workflows.

---

## 1. `stages`

**Purpose:** Defines the execution order of jobs in your pipeline. Jobs in the same stage run in parallel, while stages execute sequentially.

**Scope:** Global (top-level of `.gitlab-ci.yml`)

**Syntax:**
```yaml
stages:
  - build
  - test
  - deploy
```

**Key Points:**
- Default stages (if not specified): `.pre`, `build`, `test`, `deploy`, `.post`
- Jobs are assigned to stages using the `stage` keyword
- If a job in a stage fails, subsequent stages won't execute (unless configured otherwise)
- Stages execute in the order defined

**Example:**
```yaml
stages:
  - prepare
  - build
  - unit-test
  - integration-test
  - deploy-staging
  - deploy-production

build-app:
  stage: build
  script:
    - npm run build
```

**Pro Tips:**
- Keep stage names descriptive and aligned with your workflow
- Use 3-5 stages for most projects; too many can complicate visualization
- Consider using `.pre` and `.post` for setup/cleanup jobs that always run

---

## 2. `image`

**Purpose:** Specifies the Docker image to use for job execution. All commands run inside this container.

**Scope:** Global (applies to all jobs) or per-job

**Syntax:**
```yaml
# Global
image: node:18-alpine

# Per-job
job-name:
  image: python:3.11-slim
  script:
    - python script.py
```

**Key Points:**
- Can use images from Docker Hub, GitLab Container Registry, or private registries
- Job-level `image` overrides global `image`
- Include tags for reproducibility (avoid `latest`)
- Use Alpine variants for faster pulls and smaller footprint

**Example:**
```yaml
image: node:18-alpine  # Default for all jobs

build:
  image: node:18        # Override with full Debian-based image
  script:
    - npm install
    - npm run build

deploy:
  image: alpine:3.18    # Different image for deployment
  script:
    - apk add --no-cache aws-cli
    - aws s3 sync ./dist s3://my-bucket
```

**Pro Tips:**
- Use specific version tags (e.g., `node:18.17.0`) for production pipelines
- Create custom images with pre-installed dependencies to speed up jobs
- Consider multi-stage Docker builds for complex scenarios

---

## 3. `before_script`

**Purpose:** Defines commands that run before the main `script` in every job (or specific jobs).

**Scope:** Global (runs before all jobs) or per-job

**Syntax:**
```yaml
# Global
before_script:
  - echo "Starting job"
  - date

# Per-job
job-name:
  before_script:
    - export PATH=$PATH:/custom/bin
  script:
    - run-command
```

**Key Points:**
- Executes after any `cache` restoration
- Job-level `before_script` replaces (not appends to) global `before_script`
- Useful for setup tasks: installing dependencies, setting environment
- If `before_script` fails, the job fails and `script` won't run

**Example:**
```yaml
before_script:
  - echo "Pipeline started at $(date)"

variables:
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"

test:
  image: python:3.11
  before_script:
    - pip install --upgrade pip
    - pip install -r requirements.txt
  script:
    - pytest tests/
```

**Pro Tips:**
- Use for dependency installation that's common across jobs
- Keep it fast; move lengthy operations to separate jobs if possible
- Use anchors/extends to reuse `before_script` blocks across jobs

---

## 4. `script`

**Purpose:** The main commands that execute in a job. This is the only required keyword for a job.

**Scope:** Per-job (required)

**Syntax:**
```yaml
job-name:
  script:
    - command1
    - command2
    - command3
```

**Key Points:**
- Each line is a separate shell command
- Commands execute sequentially; if one fails, the job fails
- Can be a single command or a list
- Runs in the shell of the Docker image (`sh` for Alpine, `bash` for Debian)

**Example:**
```yaml
build:
  script:
    - npm install
    - npm run lint
    - npm run build
    - ls -la dist/

test:
  script:
    - |
      echo "Running comprehensive tests"
      npm run test:unit
      npm run test:integration
      npm run test:coverage
```

**Advanced Techniques:**
```yaml
# Multi-line script with pipes
complex-job:
  script:
    - |
      for file in src/*.js; do
        echo "Processing $file"
        node $file
      done
    - echo "All files processed"

# Conditional execution
conditional-job:
  script:
    - if [ "$CI_COMMIT_BRANCH" == "main" ]; then make deploy-prod; else make deploy-dev; fi
```

**Pro Tips:**
- Use `|` (pipe) for multi-line scripts while preserving formatting
- Add `set -e` at the start to ensure script fails on first error
- Use `&&` to chain commands that must all succeed
- Consider external script files for complex logic

---

## 5. `after_script`

**Purpose:** Commands that run after `script`, regardless of job success or failure. Ideal for cleanup operations.

**Scope:** Global or per-job

**Syntax:**
```yaml
after_script:
  - echo "Cleaning up..."
  - rm -rf temp/

job-name:
  script:
    - build-app
  after_script:
    - upload-logs
    - notify-slack
```

**Key Points:**
- Always executes, even if `script` fails
- Runs in a separate shell context (variables from `script` aren't available)
- Job-level `after_script` replaces global `after_script`
- Failure in `after_script` doesn't affect job status

**Example:**
```yaml
test:
  script:
    - npm test
  after_script:
    - cat test-results.log
    - curl -X POST $WEBHOOK_URL -d "Tests completed"

deploy:
  script:
    - deploy-to-staging
  after_script:
    - |
      if [ -f deployment.log ]; then
        echo "Uploading deployment logs..."
        aws s3 cp deployment.log s3://logs-bucket/
      fi
```

**Pro Tips:**
- Use for uploading logs, sending notifications, or cleanup
- Don't rely on `after_script` for critical operations (use separate jobs)
- Remember: it runs in a separate shell, so source any needed environment
- Great for debugging failed jobs by inspecting state

---

## 6. `variables`

**Purpose:** Define custom environment variables for jobs. Essential for configuration management and avoiding hardcoded values.

**Scope:** Global, per-job, or in GitLab UI (Settings > CI/CD > Variables)

**Syntax:**
```yaml
# Global
variables:
  DATABASE_URL: "postgres://localhost/mydb"
  NODE_ENV: "production"

# Per-job
job-name:
  variables:
    CUSTOM_VAR: "value"
  script:
    - echo $CUSTOM_VAR
```

**Key Points:**
- Job variables override global variables
- Can reference other variables: `NEW_VAR: "$EXISTING_VAR/suffix"`
- GitLab provides many predefined variables (e.g., `$CI_COMMIT_SHA`, `$CI_PROJECT_DIR`)
- Variables set in GitLab UI are available in all jobs

**Example:**
```yaml
variables:
  DOCKER_DRIVER: overlay2
  CACHE_KEY: "$CI_COMMIT_REF_SLUG"
  API_ENDPOINT: "https://api.example.com"

build:
  variables:
    BUILD_ENV: "staging"
  script:
    - echo "Building for $BUILD_ENV"
    - echo "Commit SHA: $CI_COMMIT_SHORT_SHA"
    - npm run build --env=$BUILD_ENV

deploy:
  variables:
    DEPLOY_TARGET: "production"
  script:
    - echo "Deploying to $DEPLOY_TARGET"
    - curl -X POST $API_ENDPOINT/deploy -d "target=$DEPLOY_TARGET"
```

**Important Predefined Variables:**
- `$CI_COMMIT_SHA` - Full commit hash
- `$CI_COMMIT_SHORT_SHA` - Short commit hash (8 chars)
- `$CI_COMMIT_BRANCH` - Branch name
- `$CI_COMMIT_TAG` - Tag name (if triggered by tag)
- `$CI_PROJECT_DIR` - Full path where repo is cloned
- `$CI_PIPELINE_ID` - Unique pipeline ID
- `$CI_JOB_ID` - Unique job ID

**Pro Tips:**
- Store secrets in GitLab UI variables (marked as "protected" and "masked")
- Use descriptive variable names in UPPER_SNAKE_CASE
- Document variables in your README
- Use `value` and `description` for better documentation in YAML

---

## 7. `rules`

**Purpose:** Advanced conditional logic to control when jobs run. Replaces the older `only`/`except` keywords.

**Scope:** Per-job

**Syntax:**
```yaml
job-name:
  script:
    - echo "Running"
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
    - when: manual
```

**Key Points:**
- Evaluates top-to-bottom; first match wins
- Can use `if`, `changes`, `exists`, `when`, `allow_failure`
- More powerful and flexible than `only`/`except`
- Default behavior if no rules match: job doesn't run

**Common Operators:**
- `==` equals
- `!=` not equals
- `=~` matches regex
- `!~` doesn't match regex

**Example:**
```yaml
# Run only on main branch
deploy-production:
  script:
    - deploy-to-prod
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'

# Run on merge requests only
test-mr:
  script:
    - npm test
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'

# Run on branches matching pattern
deploy-feature:
  script:
    - deploy-to-dev
  rules:
    - if: '$CI_COMMIT_BRANCH =~ /^feature\/.*/'

# Run when specific files change
build-frontend:
  script:
    - npm run build
  rules:
    - changes:
        - src/frontend/**/*
        - package.json

# Complex rules with multiple conditions
deploy-staging:
  script:
    - deploy-to-staging
  rules:
    - if: '$CI_COMMIT_BRANCH == "develop" && $CI_PIPELINE_SOURCE == "push"'
      when: on_success
    - if: '$CI_COMMIT_TAG =~ /^v.*/'
      when: manual

# Always run, but manual on main
integration-test:
  script:
    - run-integration-tests
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual
      allow_failure: true
    - when: always
```

**Rule Clauses:**
- `when: on_success` - Run if previous stages succeeded (default)
- `when: on_failure` - Run if previous stage failed
- `when: always` - Always run, regardless of status
- `when: manual` - Require manual trigger
- `when: never` - Never run (useful for exclusions)
- `allow_failure: true` - Job can fail without blocking pipeline

**Pro Tips:**
- Use `when: never` as first rule to exclude specific cases
- Combine multiple conditions with `&&` (AND) or `||` (OR)
- Test regex patterns at regex101.com before using
- Use workflow rules at global level to control entire pipeline

---

## 8. `cache`

**Purpose:** Store and reuse files between pipeline runs to speed up jobs. Ideal for dependencies that don't change often.

**Scope:** Global or per-job

**Syntax:**
```yaml
cache:
  key: "$CI_COMMIT_REF_SLUG"
  paths:
    - node_modules/
    - .npm/
```

**Key Points:**
- Caches are best-effort; not guaranteed to be available
- Use for build dependencies, not for passing data between jobs (use `artifacts` instead)
- `key` determines cache uniqueness; same key = same cache
- Can define `policy: pull` or `policy: push` to control behavior

**Cache Policies:**
- `pull-push` (default) - Download before job, upload after
- `pull` - Only download, don't update
- `push` - Only upload, don't download

**Example:**
```yaml
# Global cache for all jobs
cache:
  key: "$CI_COMMIT_REF_SLUG"
  paths:
    - node_modules/
    - .npm/

build:
  script:
    - npm ci
    - npm run build

test:
  script:
    - npm test
  cache:
    key: "$CI_COMMIT_REF_SLUG-test"
    paths:
      - node_modules/
      - coverage/

# Cache per-branch with fallback
cache-example:
  cache:
    - key: 
        files:
          - package-lock.json
      paths:
        - node_modules/
    - key: "$CI_COMMIT_REF_SLUG"
      paths:
        - .cache/
  script:
    - npm install
```

**Advanced Patterns:**
```yaml
# Pull-only cache for test jobs (don't update)
test-fast:
  cache:
    key: "$CI_COMMIT_REF_SLUG"
    paths:
      - node_modules/
    policy: pull
  script:
    - npm test

# Cache based on file changes
python-job:
  cache:
    key:
      files:
        - requirements.txt
    paths:
      - .pip-cache/
  script:
    - pip install -r requirements.txt --cache-dir .pip-cache
```

**Cache Key Strategies:**
```yaml
# Branch-based
key: "$CI_COMMIT_REF_SLUG"

# File-based (updates when dependencies change)
key:
  files:
    - Gemfile.lock
    - package-lock.json

# Combination
key: "$CI_COMMIT_REF_SLUG-$CI_JOB_NAME"

# Static (shared across all pipelines)
key: "global-cache"
```

**Pro Tips:**
- Clear cache if builds behave unexpectedly: CI/CD > Pipelines > Clear Cache
- Don't cache git repository (it's cloned fresh each time)
- Use file-based keys for language package managers
- Consider separate caches for different job types
- Cache size limit is typically 4GB per cache

---

## 9. `artifacts`

**Purpose:** Save files created by jobs to pass to subsequent jobs or download after pipeline completion.

**Scope:** Per-job

**Syntax:**
```yaml
job-name:
  script:
    - build-app
  artifacts:
    paths:
      - dist/
      - build/logs/*.log
    expire_in: 1 week
```

**Key Points:**
- Artifacts persist between jobs in the same pipeline
- Stored on GitLab and downloadable from pipeline view
- Use `expire_in` to control storage (default: 30 days)
- Can be downloaded from GitLab UI or via API
- Impacts storage quota; manage retention carefully

**Artifact Properties:**
- `paths` - Files/directories to save
- `exclude` - Patterns to exclude
- `expire_in` - How long to keep (e.g., `1 day`, `2 weeks`, `3 months`, `never`)
- `when` - When to upload (`on_success` (default), `on_failure`, `always`)
- `name` - Custom archive name
- `reports` - Special artifact types (test reports, coverage, etc.)

**Example:**
```yaml
build:
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
      - build-info.json
    expire_in: 1 day
    name: "$CI_JOB_NAME-$CI_COMMIT_SHORT_SHA"

test:
  script:
    - npm test
  artifacts:
    when: always
    paths:
      - coverage/
      - test-results.xml
    reports:
      junit: test-results.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    expire_in: 30 days

# Save logs only on failure
deploy:
  script:
    - deploy-app.sh
  artifacts:
    when: on_failure
    paths:
      - logs/
    expire_in: 1 week

# Exclude sensitive files
publish:
  script:
    - build-package
  artifacts:
    paths:
      - release/
    exclude:
      - release/**/*.map
      - release/**/*.log
```

**Special Report Types:**
```yaml
# Test results (displayed in merge request)
artifacts:
  reports:
    junit: test-results.xml

# Code coverage (displayed in merge request)
artifacts:
  reports:
    coverage_report:
      coverage_format: cobertura
      path: coverage/cobertura.xml

# Security scanning
artifacts:
  reports:
    sast: gl-sast-report.json
    dependency_scanning: gl-dependency-scanning.json

# Performance metrics
artifacts:
  reports:
    performance: performance.json
```

**Pro Tips:**
- Keep artifacts small; exclude unnecessary files
- Use short `expire_in` for temporary build artifacts
- Use `never` for release artifacts you want to keep
- Artifacts from all previous stages are available in current job
- Download artifacts: Pipeline view > Job > right sidebar > "Browse"

---

## 10. `needs`

**Purpose:** Create job dependencies and enable parallel execution across stages. Allows jobs to start before entire previous stage completes.

**Scope:** Per-job

**Syntax:**
```yaml
job-name:
  stage: test
  needs: ["build-job"]
  script:
    - run-tests
```

**Key Points:**
- Breaks traditional stage-by-stage execution for faster pipelines
- Job starts as soon as all `needs` jobs complete
- Automatically downloads artifacts from needed jobs
- Can specify jobs from any stage
- Empty `needs: []` means job doesn't wait for any previous jobs

**Example:**
```yaml
stages:
  - build
  - test
  - deploy

build-frontend:
  stage: build
  script:
    - npm run build:frontend
  artifacts:
    paths:
      - dist/frontend/

build-backend:
  stage: build
  script:
    - npm run build:backend
  artifacts:
    paths:
      - dist/backend/

# Runs immediately after build-frontend completes
test-frontend:
  stage: test
  needs: ["build-frontend"]
  script:
    - npm run test:frontend

# Runs immediately after build-backend completes
test-backend:
  stage: test
  needs: ["build-backend"]
  script:
    - npm run test:backend

# Runs immediately after both build jobs complete
integration-test:
  stage: test
  needs: ["build-frontend", "build-backend"]
  script:
    - npm run test:integration

# Deploy only needs integration tests, not all tests
deploy:
  stage: deploy
  needs: ["integration-test"]
  script:
    - deploy-app
```

**Advanced Usage:**
```yaml
# Optional dependencies
flexible-job:
  needs:
    - job: build-app
      optional: true  # Run even if build-app fails
  script:
    - run-regardless

# Control artifact downloads
efficient-job:
  needs:
    - job: build-app
      artifacts: true   # Download artifacts (default)
    - job: compile-assets
      artifacts: false  # Don't download artifacts
  script:
    - use-build-artifacts

# Cross-project dependencies
downstream-job:
  needs:
    - project: group/project-name
      job: build-upstream
      ref: main
  script:
    - use-upstream-artifacts
```

**Pipeline Acceleration Pattern:**
```yaml
stages:
  - build
  - test
  - security
  - deploy

# Traditional: All test jobs wait for all build jobs
# With needs: Each test starts when its specific build completes

build-api:
  stage: build
  script: [build api]

build-ui:
  stage: build
  script: [build ui]

build-docs:
  stage: build
  script: [build docs]

test-api:
  stage: test
  needs: ["build-api"]     # Starts immediately when build-api done
  script: [test api]

test-ui:
  stage: test
  needs: ["build-ui"]      # Starts immediately when build-ui done
  script: [test ui]

security-scan:
  stage: security
  needs: ["build-api", "build-ui"]  # Only needs these two
  script: [security scan]

deploy-docs:
  stage: deploy
  needs: ["build-docs"]    # Can deploy docs early!
  script: [deploy docs]

deploy-app:
  stage: deploy
  needs: ["test-api", "test-ui", "security-scan"]
  script: [deploy app]
```

**Pro Tips:**
- Use needs to parallelize independent work and reduce pipeline time
- Visualize dependencies in GitLab: Pipeline view shows the DAG (Directed Acyclic Graph)
- Empty `needs: []` creates isolated jobs that run immediately
- Overusing `needs` can make pipelines hard to understand; balance speed with clarity
- Combine with `rules` for powerful conditional execution

---

## Quick Reference Summary

| Keyword         | Purpose                      | Scope      | Example Use Case                    |
| --------------- | ---------------------------- | ---------- | ----------------------------------- |
| `stages`        | Define execution order       | Global     | Organize build → test → deploy flow |
| `image`         | Specify Docker container     | Global/Job | Use `node:18` for Node.js projects  |
| `before_script` | Pre-job setup commands       | Global/Job | Install dependencies                |
| `script`        | Main job commands (required) | Job        | Run tests, build app, deploy        |
| `after_script`  | Cleanup commands             | Global/Job | Upload logs, send notifications     |
| `variables`     | Environment variables        | Global/Job | Store API URLs, configure tools     |
| `rules`         | Conditional job execution    | Job        | Run only on main branch             |
| `cache`         | Speed up with reusable files | Global/Job | Cache `node_modules/`               |
| `artifacts`     | Pass files between jobs      | Job        | Share build output                  |
| `needs`         | Create job dependencies      | Job        | Start test when build ready         |

---

## Common Patterns & Best Practices

### Pattern 1: Efficient Dependency Management
```yaml
variables:
  CACHE_KEY: "$CI_COMMIT_REF_SLUG"

cache:
  key: "$CACHE_KEY"
  paths:
    - node_modules/

install:
  stage: .pre
  script:
    - npm ci
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 hour

test:
  needs: ["install"]
  script:
    - npm test
```

### Pattern 2: Branch-Specific Workflows
```yaml
deploy-staging:
  script:
    - deploy to staging
  rules:
    - if: '$CI_COMMIT_BRANCH == "develop"'

deploy-production:
  script:
    - deploy to production
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual
```

### Pattern 3: Parallel Testing
```yaml
test-unit:
  needs: ["build"]
  script:
    - npm run test:unit

test-integration:
  needs: ["build"]
  script:
    - npm run test:integration

test-e2e:
  needs: ["build"]
  script:
    - npm run test:e2e
```

---

## Troubleshooting Tips

**Cache not working?**
- Check cache key matches between jobs
- Verify paths exist in job's working directory
- Remember: cache is best-effort, not guaranteed

**Artifacts not available?**
- Ensure previous job succeeded (or use `when: always`)
- Check job has `needs` or is in later stage
- Verify paths are correct (relative to `$CI_PROJECT_DIR`)

**Job not running?**
- Check `rules` - no match means job skipped
- Verify stage name matches stages list
- Look for `when: manual` or `when: never`

**Variables not expanding?**
- Use double quotes: `"$VAR"` not `'$VAR'`
- Check variable is defined at right scope
- Verify predefined variable spelling

---

## Additional Resources

- **GitLab CI/CD Docs:** https://docs.gitlab.com/ee/ci/
- **CI/CD Pipeline Examples:** https://docs.gitlab.com/ee/ci/examples/
- **Predefined Variables:** https://docs.gitlab.com/ee/ci/variables/predefined_variables.html
- **Pipeline Editor:** Use GitLab's built-in pipeline editor with validation and autocomplete

