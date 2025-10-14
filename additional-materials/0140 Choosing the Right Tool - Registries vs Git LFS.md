# Choosing the Right Tool - Registries vs. Git LFS

## The Three Storage Mechanisms

| Tool                   | **Primary Purpose**               | **Best For**                                     | **Access Method**                                      |
| ---------------------- | --------------------------------- | ------------------------------------------------ | ------------------------------------------------------ |
| **Container Registry** | Store Docker/OCI images           | Containerized applications, microservices        | `docker pull`, `docker push`                           |
| **Package Registry**   | Store language-specific packages  | Libraries, dependencies (npm, Maven, PyPI, etc.) | Language-specific tools (`npm install`, `pip install`) |
| **Git LFS**            | Version large binary files in Git | Media files, datasets, design assets             | Git commands (`git lfs pull`)                          |

---

## Decision Tree

```
What type of data are you storing?
│
├─ Container Images (Docker, Helm charts)
│   └─ Use: CONTAINER REGISTRY
│
├─ Language Packages/Libraries
│   ├─ npm packages → Use: PACKAGE REGISTRY (npm)
│   ├─ Maven artifacts → Use: PACKAGE REGISTRY (Maven)
│   ├─ Python packages → Use: PACKAGE REGISTRY (PyPI)
│   ├─ NuGet packages → Use: PACKAGE REGISTRY (NuGet)
│   └─ Generic binaries → Use: PACKAGE REGISTRY (Generic)
│
├─ Large Binary Files
│   ├─ Versioned with code (images, videos, datasets)
│   │   └─ Use: GIT LFS
│   │
│   └─ Build artifacts (compiled binaries, releases)
│       └─ Use: PACKAGE REGISTRY (Generic)
│
└─ Source Code
    └─ Use: Git (regular repository)
```

---

## 1. Container Registry

### When to Use

✅ **Use Container Registry for:**
- Docker images for your applications
- Base images for CI/CD pipelines
- Helm charts (OCI format)
- Container images for microservices architecture

❌ **Don't use for:**
- Language packages (npm, pip, Maven)
- Source code
- Large media files that need Git versioning

### Example Use Case

**Scenario:** You're building a Node.js microservice that needs to be deployed as a container.

```yaml
# .gitlab-ci.yml
build-docker:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

```dockerfile
# Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "server.js"]
```

**Access the image:**
```bash
docker pull registry.gitlab.com/mygroup/myproject:abc123
```

---

## 2. Package Registry

### When to Use

✅ **Use Package Registry for:**
- Publishing internal npm packages
- Storing Maven artifacts (`.jar`, `.war`)
- Python packages for internal use
- NuGet packages for .NET projects
- Generic binary artifacts (compiled executables, releases)

❌ **Don't use for:**
- Container images (use Container Registry)
- Files that need Git versioning (use Git LFS)
- Temporary build artifacts (use CI/CD artifacts)

### Supported Package Types

| Format       | Use Case                        | Install Command Example            |
| ------------ | ------------------------------- | ---------------------------------- |
| **npm**      | JavaScript/Node.js libraries    | `npm install @mygroup/mypackage`   |
| **Maven**    | Java libraries and applications | `mvn install`                      |
| **PyPI**     | Python packages                 | `pip install mypackage`            |
| **NuGet**    | .NET libraries                  | `nuget install MyPackage`          |
| **Composer** | PHP dependencies                | `composer require myorg/mypackage` |
| **Generic**  | Any file type                   | `curl` or `wget`                   |

### Example Use Case: npm Package

**Scenario:** You're creating a shared React component library for multiple projects.

```yaml
# .gitlab-ci.yml
publish-npm:
  stage: deploy
  image: node:18
  script:
    - echo "@mygroup:registry=https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/npm/" > .npmrc
    - echo "//gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/npm/:_authToken=${CI_JOB_TOKEN}" >> .npmrc
    - npm publish
  only:
    - tags
```

**Consuming the package:**
```bash
# In another project
npm install @mygroup/react-components
```

### Example Use Case: Generic Package (Compiled Binary)

**Scenario:** You're releasing compiled binaries for different platforms.

```yaml
# .gitlab-ci.yml
publish-binary:
  stage: deploy
  script:
    - 'curl --header "JOB-TOKEN: $CI_JOB_TOKEN" 
           --upload-file ./bin/myapp-linux 
           "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/generic/myapp/${CI_COMMIT_TAG}/myapp-linux"'
  only:
    - tags
```

**Downloading the binary:**
```bash
curl --header "PRIVATE-TOKEN: <your_token>" \
  "https://gitlab.com/api/v4/projects/123/packages/generic/myapp/v1.0.0/myapp-linux" \
  --output myapp-linux
```

---

## 3. Git LFS (Large File Storage)

### When to Use

✅ **Use Git LFS for:**
- Images, videos, and audio files in your repository
- Design files (PSD, Sketch, Figma exports)
- 3D models and game assets
- Large datasets versioned alongside code
- Documentation with embedded media

❌ **Don't use for:**
- Dependencies (use Package Registry)
- Docker images (use Container Registry)
- Build outputs (use artifacts or Package Registry)
- Files larger than 5GB per file (GitLab limit)

### File Size Guidelines

| File Size     | Recommendation                        |
| ------------- | ------------------------------------- |
| < 10 MB       | Regular Git (no LFS needed)           |
| 10-100 MB     | Git LFS (recommended)                 |
| 100 MB - 5 GB | Git LFS (required)                    |
| > 5 GB        | External storage (S3, object storage) |

### Example Use Case

**Scenario:** Your marketing website repository contains videos and high-resolution images.

**Step 1: Install and configure Git LFS**
```bash
# Install Git LFS
git lfs install

# Track specific file types
git lfs track "*.mp4"
git lfs track "*.psd"
git lfs track "*.mov"
git lfs track "assets/images/*.jpg"

# Commit the .gitattributes file
git add .gitattributes
git commit -m "Configure Git LFS"
```

**Step 2: Add and commit large files normally**
```bash
git add video/demo.mp4
git commit -m "Add product demo video"
git push
```

**Step 3: Clone repository with LFS files**
```bash
# Clones repo and downloads LFS files
git clone https://gitlab.com/mygroup/marketing-site.git

# Or download LFS files after cloning
git lfs pull
```

### Git LFS in CI/CD

```yaml
# .gitlab-ci.yml
build-site:
  stage: build
  variables:
    GIT_LFS_SKIP_SMUDGE: "1"  # Skip LFS download initially
  script:
    - git lfs pull  # Only download when needed
    - npm run build
```

---

## Comparison Matrix

| Criteria              | Container Registry                          | Package Registry                       | Git LFS                       |
| --------------------- | ------------------------------------------- | -------------------------------------- | ----------------------------- |
| **Max file size**     | Unlimited (practical limit ~10GB per layer) | Unlimited                              | 5 GB per file                 |
| **Versioning**        | Tags (semantic versioning)                  | Versions (semantic versioning)         | Git commits                   |
| **Access control**    | Project/group level                         | Project/group level                    | Repository permissions        |
| **Storage cost**      | Counted toward storage quota                | Counted toward storage quota           | Counted toward storage quota  |
| **CLI tool**          | `docker`                                    | Language-specific (`npm`, `pip`, etc.) | `git lfs`                     |
| **Public access**     | Can be public                               | Can be public                          | Follows repo visibility       |
| **CI/CD integration** | Excellent (built-in variables)              | Excellent (token authentication)       | Good (requires configuration) |

---

## Real-World Architecture Examples

### Example 1: Full-Stack Web Application

```
Project Structure:
├─ Frontend (React)
│   └─ Uses: npm packages from Package Registry
│
├─ Backend API (Java)
│   └─ Uses: Maven packages from Package Registry
│
├─ Deployment
│   └─ Uses: Docker images from Container Registry
│
└─ Marketing Assets
    └─ Uses: Git LFS for videos and images
```

### Example 2: Machine Learning Project

```
Project Structure:
├─ Training Code (Python)
│   └─ Uses: pip packages from Package Registry
│
├─ Trained Models (large .h5 files)
│   └─ Uses: Git LFS or Generic Package Registry
│
├─ Inference Service
│   └─ Uses: Docker images from Container Registry
│
└─ Datasets
    └─ Uses: External storage (S3) linked in code
```

---

## Cost Considerations

All three mechanisms count toward your GitLab storage quota:

| GitLab Tier  | Storage Included | Overage Cost                |
| ------------ | ---------------- | --------------------------- |
| **Free**     | 5 GB             | N/A (hard limit)            |
| **Premium**  | 10 GB            | Purchase additional storage |
| **Ultimate** | 10 GB            | Purchase additional storage |

**Optimization tips:**
- Use `.dockerignore` to reduce Docker image sizes
- Set `expire_in` for CI/CD artifacts
- Use package registry retention policies
- Compress LFS files when possible
- Clean up old package versions regularly

---

## Common Anti-Patterns

### ❌ Anti-Pattern 1: Storing Dependencies in Git LFS

```bash
# DON'T DO THIS
git lfs track "node_modules/*"
```

**Why it's wrong:** Dependencies should be installed via package managers, not committed to Git.

**✅ Correct approach:** Use `.gitignore` for `node_modules/` and install via `npm install`.

---

### ❌ Anti-Pattern 2: Using Package Registry for Container Images

```yaml
# DON'T DO THIS
publish-image:
  script:
    - tar -czf image.tar.gz docker-image/
    - 'curl --upload-file image.tar.gz "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/generic/images/v1.0.0/image.tar.gz"'
```

**Why it's wrong:** Container Registry is optimized for Docker images with layer caching and proper metadata.

**✅ Correct approach:** Use `docker push` to Container Registry.

---

### ❌ Anti-Pattern 3: Committing Large Binaries to Regular Git

```bash
# DON'T DO THIS - file is 500MB
git add training-data.zip
git commit -m "Add dataset"
```

**Why it's wrong:** Large files in Git history make cloning extremely slow and bloat the repository permanently.

**✅ Correct approach:** Use Git LFS or store in Package Registry (Generic).

---

## Quick Reference

**"Which tool should I use for..."**

| I need to store...                | Use this                                |
| --------------------------------- | --------------------------------------- |
| Docker image for my app           | **Container Registry**                  |
| Internal npm package              | **Package Registry (npm)**              |
| Video file versioned with code    | **Git LFS**                             |
| Compiled `.jar` file              | **Package Registry (Maven or Generic)** |
| Marketing website images          | **Git LFS**                             |
| Base Docker image                 | **Container Registry**                  |
| Python package for internal use   | **Package Registry (PyPI)**             |
| 3D model for game development     | **Git LFS**                             |
| Release binaries (`.exe`, `.dmg`) | **Package Registry (Generic)**          |

---

## Further Reading

- [GitLab Container Registry Docs](https://docs.gitlab.com/ee/user/packages/container_registry/)
- [GitLab Package Registry Docs](https://docs.gitlab.com/ee/user/packages/package_registry/)
- [GitLab Git LFS Docs](https://docs.gitlab.com/ee/topics/git/lfs/)