# GitLab CI/CD Reusability Patterns


## The Three Reusability Patterns

| Pattern          | **Scope**               | **Best For**                           | **Complexity**  |
| ---------------- | ----------------------- | -------------------------------------- | --------------- |
| **`extends`**    | Single file             | Reusing complete job configurations    | ⭐ Beginner      |
| **`!reference`** | Single file             | Reusing specific script sections       | ⭐⭐ Intermediate |
| **`include`**    | Multiple files/projects | Sharing configurations across projects | ⭐⭐⭐ Advanced    |

---

## Pattern 1: `extends` - Job Template Inheritance

### What It Does

The `extends` keyword allows a job to inherit configuration from a hidden template job (prefixed with `.`). It's like object-oriented inheritance for CI/CD jobs.

### When to Use

✅ **Use `extends` when:**
- You have multiple jobs with similar configurations
- You want to define base configurations (image, tags, rules)
- You're working within a single `.gitlab-ci.yml` file
- You need simple, readable inheritance

### Basic Example

```yaml
# Define a hidden template (jobs starting with . are not executed)
.deploy-template:
  stage: deploy
  image: alpine:latest
  tags:
    - docker
  before_script:
    - echo "Preparing deployment..."
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

# Jobs that extend the template
deploy-staging:
  extends: .deploy-template
  script:
    - echo "Deploying to staging..."
  environment:
    name: staging

deploy-production:
  extends: .deploy-template
  script:
    - echo "Deploying to production..."
  environment:
    name: production
  when: manual  # Override: add manual trigger for production
```

**Result:** Both jobs inherit `stage`, `image`, `tags`, `before_script`, and `rules` from the template, but have their own `script` and `environment`.

### Advanced Example: Multiple Inheritance

```yaml
# Base template
.base-job:
  image: node:18
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/

# Test template (extends base)
.test-template:
  extends: .base-job
  stage: test
  before_script:
    - npm install

# Specific test jobs
unit-tests:
  extends: .test-template
  script:
    - npm run test:unit
  coverage: '/Statements\s+:\s+(\d+\.\d+)%/'

integration-tests:
  extends: .test-template
  script:
    - npm run test:integration
  services:
    - postgres:14  # Add database service
```

### Overriding Behavior

```yaml
.base-template:
  image: node:18
  tags:
    - docker
  variables:
    NODE_ENV: "production"

custom-job:
  extends: .base-template
  image: node:20  # Override: use different Node version
  variables:
    NODE_ENV: "development"  # Override: change environment
    DEBUG: "true"  # Add: new variable
```

**Merge behavior:**
- **Scalars** (strings, numbers): Replaced completely
- **Arrays** (tags, services): Replaced completely (not merged!)
- **Hashes** (variables, cache): Merged (new keys added, existing keys overridden)

---

## Pattern 2: `!reference` - YAML Anchors on Steroids

### What It Does

The `!reference` tag allows you to reference specific sections of configuration from other jobs. Unlike `extends`, which inherits entire job configurations, `!reference` lets you cherry-pick individual arrays or scripts.

### When to Use

✅ **Use `!reference` when:**
- You need to reuse specific script sections (not entire jobs)
- You want to compose scripts from multiple sources
- You need more granular control than `extends` provides
- You're building scripts from reusable building blocks

### Basic Example

```yaml
.setup-scripts:
  script:
    - echo "Installing dependencies..."
    - npm install
    - echo "Dependencies installed"

.test-scripts:
  script:
    - echo "Running tests..."
    - npm test

# Use !reference to combine scripts
build-and-test:
  stage: test
  script:
    - !reference [.setup-scripts, script]  # Insert setup scripts
    - echo "Building application..."
    - npm run build
    - !reference [.test-scripts, script]  # Insert test scripts
```

**Result:** The `build-and-test` job script becomes:
```bash
echo "Installing dependencies..."
npm install
echo "Dependencies installed"
echo "Building application..."
npm run build
echo "Running tests..."
npm test
```

### Advanced Example: Composable Scripts

```yaml
.docker-login:
  before_script:
    - echo "$CI_REGISTRY_PASSWORD" | docker login -u "$CI_REGISTRY_USER" --password-stdin "$CI_REGISTRY"

.docker-build:
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .

.docker-push:
  script:
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

.docker-cleanup:
  after_script:
    - docker logout $CI_REGISTRY

# Compose a complete Docker workflow
build-docker-image:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - !reference [.docker-login, before_script]
  script:
    - !reference [.docker-build, script]
    - !reference [.docker-push, script]
  after_script:
    - !reference [.docker-cleanup, after_script]
```

### Referencing Nested Structures

```yaml
.deployment-vars:
  variables:
    DEPLOY_USER: "deployer"
    DEPLOY_PATH: "/var/www/app"

.staging-config:
  variables:
    SERVER: "staging.example.com"

deploy-staging:
  stage: deploy
  variables:
    # Merge multiple variable sets
    !reference [.deployment-vars, variables]
    !reference [.staging-config, variables]
    # Add job-specific variables
    DEPLOY_ENV: "staging"
  script:
    - echo "Deploying to $SERVER as $DEPLOY_USER"
```

**Note:** `!reference` only works with arrays and hashes, not scalar values.

---

## Pattern 3: `include` - Cross-Project and Remote Configuration

### What It Does

The `include` keyword allows you to import CI/CD configuration from:
- Other files in the same project
- Files from other projects
- Remote URLs
- Templates provided by GitLab

### When to Use

✅ **Use `include` when:**
- You want to share configuration across multiple projects
- You're building a CI/CD configuration library
- You need to centralize common patterns
- You want to version control shared CI/CD logic separately

### Include Types

| Type           | Use Case                         | Example                                            |
| -------------- | -------------------------------- | -------------------------------------------------- |
| **`local`**    | File in same repo                | `include: local: '/templates/build.yml'`           |
| **`project`**  | File from another GitLab project | `include: project: 'my-group/templates'`           |
| **`remote`**   | File from external URL           | `include: remote: 'https://example.com/ci.yml'`    |
| **`template`** | GitLab-provided templates        | `include: template: 'Security/SAST.gitlab-ci.yml'` |

### Example 1: Local Include (Organizing Large Pipelines)

**Project structure:**
```
.gitlab-ci.yml
ci/
  ├── build.yml
  ├── test.yml
  └── deploy.yml
```

**File: `.gitlab-ci.yml`**
```yaml
stages:
  - build
  - test
  - deploy

include:
  - local: 'ci/build.yml'
  - local: 'ci/test.yml'
  - local: 'ci/deploy.yml'

variables:
  DOCKER_REGISTRY: "registry.gitlab.com"
```

**File: `ci/build.yml`**
```yaml
build-app:
  stage: build
  image: node:18
  script:
    - npm install
    - npm run build
  artifacts:
    paths:
      - dist/
```

**File: `ci/test.yml`**
```yaml
unit-tests:
  stage: test
  image: node:18
  script:
    - npm test

integration-tests:
  stage: test
  image: node:18
  services:
    - postgres:14
  script:
    - npm run test:integration
```

**File: `ci/deploy.yml`**
```yaml
.deploy-template:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying to $ENVIRONMENT"

deploy-staging:
  extends: .deploy-template
  variables:
    ENVIRONMENT: "staging"

deploy-production:
  extends: .deploy-template
  variables:
    ENVIRONMENT: "production"
  when: manual
```

### Example 2: Project Include (Shared Configuration Library)

**Scenario:** You manage 20 microservices that all need similar CI/CD configurations.

**Project structure:**
```
Organization:
  ├── ci-templates (shared templates repo)
  │     └── .gitlab-ci.yml
  │           ├── node-build.yml
  │           ├── docker-build.yml
  │           └── kubernetes-deploy.yml
  │
  ├── service-a
  │     └── .gitlab-ci.yml
  ├── service-b
  │     └── .gitlab-ci.yml
  └── service-c
        └── .gitlab-ci.yml
```

**File: `ci-templates/node-build.yml`**
```yaml
.node-build:
  stage: build
  image: node:18
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
  before_script:
    - npm ci
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 day
```

**File: `service-a/.gitlab-ci.yml`**
```yaml
include:
  - project: 'my-org/ci-templates'
    ref: main  # Use specific branch/tag for stability
    file: 'node-build.yml'

stages:
  - build
  - test
  - deploy

build:
  extends: .node-build

test:
  stage: test
  image: node:18
  script:
    - npm test

deploy:
  stage: deploy
  script:
    - echo "Deploying service-a..."
```

**Benefits:**
- Update template once, all 20 services get the improvement
- Version control templates separately
- Enforce organizational standards

### Example 3: Remote Include (Third-Party Integration)

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/awesome-org/ci-templates/main/security-scan.yml'

stages:
  - build
  - security
  - deploy

build-app:
  stage: build
  script:
    - make build

# Job defined in remote file
security-scan:
  extends: .security-template
  stage: security
```

### Example 4: Template Include (GitLab Official Templates)

```yaml
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/Container-Scanning.gitlab-ci.yml

stages:
  - build
  - test
  - security

build:
  stage: build
  script:
    - make build

# SAST, dependency scanning, and container scanning jobs
# are automatically added by the included templates
```

**Available templates:** [Browse GitLab CI/CD templates](https://gitlab.com/gitlab-org/gitlab/-/tree/master/lib/gitlab/ci/templates)

### Advanced: Conditional Includes

```yaml
include:
  - local: 'ci/common.yml'
  - local: 'ci/production.yml'
    rules:
      - if: $CI_COMMIT_BRANCH == "main"
  - local: 'ci/development.yml'
    rules:
      - if: $CI_COMMIT_BRANCH != "main"
```

---

## Comparison Matrix

| Feature                 | `extends`       | `!reference`       | `include`               |
| ----------------------- | --------------- | ------------------ | ----------------------- |
| **Scope**               | Single file     | Single file        | Multiple files/projects |
| **Granularity**         | Entire job      | Specific sections  | Entire files            |
| **Cross-project**       | ❌ No            | ❌ No               | ✅ Yes                   |
| **Dynamic composition** | ❌ Limited       | ✅ Yes              | ❌ No                    |
| **Complexity**          | Low             | Medium             | High                    |
| **Use case**            | Job inheritance | Script composition | Configuration sharing   |
| **Merge behavior**      | Automatic       | Manual (explicit)  | Automatic               |
| **Debugging**           | Easy            | Moderate           | Can be difficult        |

---

## Decision Guide

### Choose `extends` when:
```
✅ You have repetitive job configurations in one file
✅ You want simple, readable inheritance
✅ You need to define base configurations (templates)
✅ Most of the configuration is shared
```

**Example scenario:** You have 5 deployment jobs (dev, staging, prod, etc.) that differ only in environment name and approval requirements.

---

### Choose `!reference` when:
```
✅ You need to reuse specific script snippets
✅ You're building complex scripts from building blocks
✅ extends is too coarse-grained for your needs
✅ You want maximum flexibility in composition
```

**Example scenario:** You have common setup, teardown, and utility scripts that need to be mixed and matched in different combinations across jobs.

---

### Choose `include` when:
```
✅ You want to share configuration across projects
✅ You're managing multiple repos with similar pipelines
✅ You need to centralize organizational standards
✅ You want to version control shared templates separately
```

**Example scenario:** You maintain 50 microservices and want to enforce consistent build, test, and security scanning practices across all of them.

---

## Real-World Combined Example

Here's how all three patterns work together in a real-world scenario:

**File: `ci-templates/base-templates.yml` (shared repository)**
```yaml
.docker-build-template:
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY

.docker-build-scripts:
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
```

**File: `my-app/.gitlab-ci.yml` (application repository)**
```yaml
include:
  - project: 'my-org/ci-templates'
    ref: v1.2.0  # Pin to specific version for stability
    file: 'base-templates.yml'

stages:
  - build
  - test
  - deploy

# Use extends for job inheritance
build-docker:
  extends: .docker-build-template
  stage: build
  script:
    # Use !reference to import specific scripts
    - !reference [.docker-build-scripts, script]
    - echo "Build complete!"

test:
  stage: test
  image: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  script:
    - npm test

deploy-staging:
  extends: .docker-build-template
  stage: deploy
  script:
    - kubectl set image deployment/myapp myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  environment:
    name: staging
```

**What's happening:**
1. **`include`** imports shared templates from a separate project
2. **`extends`** inherits the Docker build configuration
3. **`!reference`** reuses specific build scripts

---

## Common Pitfalls and Best Practices

### ❌ Pitfall 1: Over-nesting with `extends`

```yaml
# DON'T DO THIS - too many layers
.base:
  image: alpine

.with-docker:
  extends: .base
  services:
    - docker:dind

.with-node:
  extends: .with-docker
  image: node:18

.with-cache:
  extends: .with-node
  cache:
    paths:
      - node_modules/

my-job:
  extends: .with-cache
  script:
    - npm test
```

**Problem:** Hard to understand what `my-job` actually inherits. Debugging is difficult.

**✅ Better approach:**
```yaml
.node-docker-job:
  image: node:18
  services:
    - docker:dind
  cache:
    paths:
      - node_modules/

my-job:
  extends: .node-docker-job
  script:
    - npm test
```

---

### ❌ Pitfall 2: Using `!reference` for non-arrays

```yaml
# THIS WILL ERROR
.config:
  image: node:18

my-job:
  image: !reference [.config, image]  # ❌ Can't reference scalars
```

**✅ Correct approach:**
```yaml
my-job:
  extends: .config  # Use extends for entire job config
```

---

### ❌ Pitfall 3: Including without version pinning

```yaml
# RISKY - uses latest version
include:
  - project: 'my-org/templates'
    file: 'build.yml'
```

**Problem:** Template changes can break your pipeline unexpectedly.

**✅ Better approach:**
```yaml
include:
  - project: 'my-org/templates'
    ref: v2.1.0  # Pin to specific tag
    file: 'build.yml'
```

---

### ✅ Best Practice: Documentation

Always document your templates:

```yaml
# Template: .node-build
# Purpose: Standard Node.js build with caching
# Requirements: 
#   - Project must have package.json
#   - Node 18 compatible
# Usage:
#   build:
#     extends: .node-build
.node-build:
  image: node:18
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
  before_script:
    - npm ci
  script:
    - npm run build
```

---

## Quick Reference Cheat Sheet

```yaml
# EXTENDS - Job inheritance
.template:
  stage: test
  image: node:18

my-job:
  extends: .template
  script:
    - npm test

# ═══════════════════════════════════

# !REFERENCE - Script composition
.setup:
  script:
    - npm install

my-job:
  script:
    - !reference [.setup, script]
    - npm test

# ═══════════════════════════════════

# INCLUDE - File importing
include:
  - local: 'ci/templates.yml'
  - project: 'group/repo'
    file: 'templates.yml'
  - remote: 'https://example.com/ci.yml'
  - template: 'Security/SAST.gitlab-ci.yml'
```

---

## Further Reading

- [GitLab Docs: `extends` keyword](https://docs.gitlab.com/ee/ci/yaml/#extends)
- [GitLab Docs: `!reference` tag](https://docs.gitlab.com/ee/ci/yaml/#reference-tags)
- [GitLab Docs: `include` keyword](https://docs.gitlab.com/ee/ci/yaml/#include)
- [GitLab CI/CD Templates Repository](https://gitlab.com/gitlab-org/gitlab/-/tree/master/lib/gitlab/ci/templates)