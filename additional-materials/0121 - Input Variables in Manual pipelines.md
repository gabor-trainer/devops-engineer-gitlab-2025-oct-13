
# Input Variables in Manual Pipelines

**Example:**
```
spec:
  inputs:
    XXX:
      description: "This variable is used to demonstrate variable usage in the build job."
      default: "example_value"
    YYY:
      description: "This variable is not used in any job."
      default: "another_value"

---

stages:
  - build
  - test
  - deploy

build-job:
  stage: build
  image: debian:bookworm
  script:
    # before_script steps
    - echo "Running before_script steps..."
    - apt-get update && apt-get install -y curl
    - echo "Before_script steps completed."

    - echo "This is the build job. Simulating a build process..."
    - echo "Using variable XXX with value - $[[ inputs.XXX ]]"
    - sleep 10
    - echo "Build complete."

test-job:
  stage: test
  script:
    - echo "This is the test job. Running automated tests..."
    - echo "YYY value is $[[ inputs.YYY ]]"
    - sleep 15
    - echo "Tests passed."

deploy-job:
  stage: deploy
  script:
    - echo "This is the deploy job. Deploying application..."
    - sleep 5
    - echo "Deployment complete."
```

## What Can We Write in `spec:`?

Currently, the `spec:` section in GitLab CI/CD is **specifically designed for defining inputs**. As of now, there's only **one main thing** you can define in `spec:`:

### 1. `spec:inputs` (The Primary Feature)

This is what we've been discussing. Here's the complete structure:

```yaml
spec:
  inputs:
    input-name:
      type: string | number | boolean | array
      description: "Human-readable description"
      default: "default-value"
      options: ['option1', 'option2', 'option3']
      regex: ^pattern$
---
# Your CI/CD configuration follows
```

## Complete `spec:inputs` Configuration Options

Here's everything you can configure for each input:

| Property      | Type   | Required | Description                       | Example                                |
| ------------- | ------ | -------- | --------------------------------- | -------------------------------------- |
| `type`        | string | No       | Data type of input                | `string`, `number`, `boolean`, `array` |
| `description` | string | No       | Help text shown in UI             | `"The deployment environment"`         |
| `default`     | any    | No*      | Default value if not provided     | `"production"`, `42`, `true`           |
| `options`     | array  | No       | List of allowed values (dropdown) | `['dev', 'staging', 'prod']`           |
| `regex`       | string | No       | Pattern that value must match     | `^v\d+\.\d+\.\d+$`                     |

*Required if you want automatic pipelines to work (MR pipelines, scheduled, etc.)

## Complete Example with All Features

```yaml
spec:
  inputs:
    # String input with options (dropdown in UI)
    environment:
      type: string
      description: "Target deployment environment"
      options: ['development', 'staging', 'production']
      default: 'development'
    
    # String input with regex validation
    version:
      type: string
      description: "Semantic version tag (e.g., v1.2.3)"
      regex: ^v\d+\.\d+\.\d+$
      default: 'v1.0.0'
    
    # Number input
    parallel_jobs:
      type: number
      description: "Number of parallel test jobs"
      default: 4
    
    # Boolean input (checkbox in UI)
    run_integration_tests:
      type: boolean
      description: "Execute integration tests"
      default: true
    
    # Array input
    test_suites:
      type: array
      description: "List of test suites to run"
      default:
        - unit
        - integration
        - e2e
    
    # Simple string input (mandatory - no default)
    api_key:
      type: string
      description: "API key for deployment"
    
    # String with default and description only
    docker_image:
      description: "Base Docker image to use"
      default: "node:18-alpine"
---
stages:
  - test
  - deploy

test-job:
  stage: test
  image: $[[ inputs.docker_image ]]
  parallel: $[[ inputs.parallel_jobs ]]
  script:
    - echo "Running tests in $[[ inputs.environment ]]"
    - echo "Version: $[[ inputs.version ]]"
    - |
      if $[[ inputs.run_integration_tests ]]; then
        echo "Running integration tests"
      fi

deploy-job:
  stage: deploy
  script:
    - deploy.sh --env=$[[ inputs.environment ]] --key=$[[ inputs.api_key ]]
  rules:
    - if: '$[[ inputs.environment ]] == "production"'
      when: manual
```
