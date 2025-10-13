# Docker Core Concepts for DevOps

**Essential Docker Knowledge for CI/CD Pipelines**

---

## Part 1: Core Fundamentals

### The Three Building Blocks

Understanding Docker requires mastering three fundamental concepts: **Dockerfile**, **Image**, and **Container**. Each serves a distinct purpose in the Docker ecosystem.

---

## 1. Dockerfile: The Blueprint

**What it is:** A text file containing instructions to build a Docker image.

**Purpose:** Define your application environment as code.

**Analogy:** Like a recipe or architectural blueprint.

```dockerfile
# Example Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

**Key Characteristics:**
- Plain text file named `Dockerfile`
- Contains step-by-step instructions
- Defines the environment, dependencies, and commands
- Version-controlled alongside your code
- Repeatable and shareable

**Common Instructions:**
- `FROM` - Base image to start from
- `WORKDIR` - Set working directory
- `COPY` / `ADD` - Copy files into image
- `RUN` - Execute commands during build
- `EXPOSE` - Document which ports the app uses
- `CMD` / `ENTRYPOINT` - Command to run when container starts

---

## 2. Image: The Template

**What it is:** A read-only template with everything needed to run an application.

**Purpose:** Package your application and all its dependencies into a distributable unit.

**Analogy:** Like a class in OOP, or a stamped CD/DVD.

**Visual Representation:**
```
┌─────────────────────────────────────┐
│         Docker Image                │
│  (node-app:1.0)                     │
├─────────────────────────────────────┤
│ Layer 5: CMD ["npm", "start"]      │  ← Your start command
├─────────────────────────────────────┤
│ Layer 4: COPY . . (app code)       │  ← Your application files
├─────────────────────────────────────┤
│ Layer 3: RUN npm install            │  ← Dependencies installed
├─────────────────────────────────────┤
│ Layer 2: COPY package.json          │  ← Package manifest
├─────────────────────────────────────┤
│ Layer 1: FROM node:18-alpine        │  ← Base OS + Node.js
└─────────────────────────────────────┘
```

**Key Characteristics:**
- Immutable (read-only)
- Composed of multiple layers
- Each Dockerfile instruction creates a new layer
- Layers are cached for efficiency
- Stored in registries (Docker Hub, GitLab Container Registry, etc.)
- Tagged for versioning (e.g., `node:18-alpine`, `myapp:v1.2.3`)

**Image Layer Benefits:**
1. **Caching** - Unchanged layers are reused, speeding up builds
2. **Efficiency** - Multiple images can share base layers
3. **Fast distribution** - Only changed layers need to be transferred

**Example Layer Sharing:**
```
Image A: nginx:alpine + custom config = 50MB
Image B: nginx:alpine + different config = 50MB
Total storage: ~50MB (base layer shared)
```

---

## 3. Container: The Running Instance

**What it is:** A runnable instance of an image with its own writable layer.

**Purpose:** Execute your application in an isolated environment.

**Analogy:** Like an object instantiated from a class, or a running program from an executable.

**Visual Representation:**
```
┌─────────────────────────────────────┐
│      Running Container              │
│  (container-id: a3f9b8c2)           │
├─────────────────────────────────────┤
│ Writable Layer (Container Layer)    │  ← Changes made at runtime
│ - Log files                         │  ← (deleted when container stops)
│ - Temporary data                    │
│ - Modified files                    │
├═════════════════════════════════════┤
│      Docker Image (Read-Only)       │  ← Underlying image
│  All image layers...                │  ← (unchanged, reusable)
└─────────────────────────────────────┘
```

**Key Characteristics:**
- Ephemeral by default (data lost when container stops)
- Isolated from other containers and the host
- Has its own filesystem, networking, and process space
- Multiple containers can run from the same image
- Lightweight (shares host OS kernel)
- Started, stopped, deleted independently

**Container States:**
- **Created** - Container exists but not running
- **Running** - Container is executing
- **Paused** - Container execution is paused
- **Stopped** - Container has exited
- **Deleted** - Container is removed

---

## The Relationship: Dockerfile → Image → Container

```
┌──────────────┐    docker build    ┌──────────────┐    docker run    ┌──────────────┐
│  Dockerfile  │ ──────────────────> │    Image     │ ───────────────> │  Container   │
│              │                     │              │                  │              │
│ (Blueprint)  │                     │  (Template)  │                  │  (Instance)  │
│              │                     │              │    (multiple)    │              │
│  One file    │                     │  One image   │ ─────────────┬──> │ Container 1  │
│              │                     │              │              │   │              │
└──────────────┘                     └──────────────┘              ├──> │ Container 2  │
                                                                   │   │              │
                                                                   └──> │ Container 3  │
                                                                       └──────────────┘
```

**Real-World Analogy:**
- **Dockerfile** = House blueprints
- **Image** = Prefabricated house template
- **Container** = Actual house where someone lives

---

## The Docker Lifecycle: Build → Ship → Run

### Phase 1: BUILD

**Create the image from your Dockerfile**

```bash
# Build an image from Dockerfile
docker build -t myapp:1.0 .

# Build with specific Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Build with build arguments
docker build --build-arg NODE_ENV=production -t myapp:1.0 .
```

**What happens:**
1. Docker reads the Dockerfile
2. Executes each instruction sequentially
3. Each instruction creates a new layer
4. Layers are cached for future builds
5. Final image is tagged and stored locally

**Best Practices:**
- Order instructions from least to most frequently changed
- Use `.dockerignore` to exclude unnecessary files
- Minimize number of layers (combine RUN commands)
- Use specific base image tags (not `latest`)

---

### Phase 2: SHIP

**Distribute the image to a registry**

```bash
# Tag image for registry
docker tag myapp:1.0 registry.example.com/myapp:1.0

# Login to registry
docker login registry.example.com

# Push image to registry
docker push registry.example.com/myapp:1.0

# Pull image from registry
docker pull registry.example.com/myapp:1.0
```

**Common Registries:**
- **Docker Hub** - Public registry (hub.docker.com)
- **GitLab Container Registry** - Integrated with GitLab repos
- **Amazon ECR** - AWS Elastic Container Registry
- **Google GCR** - Google Container Registry
- **Azure ACR** - Azure Container Registry
- **Private registries** - Self-hosted solutions

**Why Ship Images?**
- Share images across teams
- Deploy to different environments
- CI/CD pipelines pull images for testing/deployment
- Version control for application artifacts

---

### Phase 3: RUN

**Create and start containers from the image**

```bash
# Run a container
docker run -d -p 3000:3000 --name myapp-container myapp:1.0

# Run with environment variables
docker run -d -e NODE_ENV=production myapp:1.0

# Run with volume mount
docker run -d -v /host/data:/app/data myapp:1.0

# Run interactively
docker run -it myapp:1.0 /bin/sh
```

**Container Management:**
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop a container
docker stop myapp-container

# Start a stopped container
docker start myapp-container

# View logs
docker logs myapp-container

# Execute command in running container
docker exec -it myapp-container /bin/sh

# Remove container
docker rm myapp-container

# Remove image
docker rmi myapp:1.0
```

---

## Understanding Image Layers in Detail

### How Layers Work

Each instruction in a Dockerfile creates a new layer. Layers are stacked on top of each other to form the final image.

```dockerfile
FROM ubuntu:20.04          # Layer 1: Base OS (80MB)
RUN apt-get update         # Layer 2: Package index (30MB)
RUN apt-get install -y     # Layer 3: Installed packages (150MB)
    nginx                  
COPY nginx.conf /etc/      # Layer 4: Config file (4KB)
COPY website /var/www      # Layer 5: Website files (2MB)
```

**Layer Caching Example:**

**First Build:**
```
Step 1: FROM ubuntu:20.04          [Downloaded]
Step 2: RUN apt-get update         [Executed - 45s]
Step 3: RUN apt-get install nginx  [Executed - 120s]
Step 4: COPY nginx.conf            [Executed - 1s]
Step 5: COPY website               [Executed - 2s]
Total: ~3 minutes
```

**Second Build (only website changed):**
```
Step 1: FROM ubuntu:20.04          [Using cache]
Step 2: RUN apt-get update         [Using cache]
Step 3: RUN apt-get install nginx  [Using cache]
Step 4: COPY nginx.conf            [Using cache]
Step 5: COPY website               [Executed - 2s]
Total: ~2 seconds!
```

### Optimizing Layers

**❌ Inefficient (creates many layers):**
```dockerfile
FROM node:18
RUN npm install express
RUN npm install mongoose
RUN npm install dotenv
RUN npm install cors
COPY . .
```

**✅ Efficient (fewer, optimized layers):**
```dockerfile
FROM node:18
COPY package*.json ./
RUN npm ci --only=production
COPY . .
```

**Multi-Stage Builds (advanced optimization):**
```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production
CMD ["node", "dist/index.js"]
```

**Benefits:**
- Final image only contains production dependencies
- Build tools and cache are left behind
- Smaller, more secure images

---

## How This Relates to GitLab CI/CD

When you see `image: node:18` in your `.gitlab-ci.yml`:

```yaml
build:
  image: node:18-alpine
  script:
    - npm install
    - npm run build
```

**What's happening:**

1. **GitLab Runner pulls the image** `node:18-alpine` from Docker Hub
2. **Creates a container** from that image
3. **Runs your script** inside the container
4. **Container is destroyed** after the job completes

**Why this matters:**
- Your job runs in a clean, consistent environment every time
- No "works on my machine" issues
- Easy to test different Node versions (just change the tag)
- All dependencies are defined and reproducible

**Complete CI/CD Example:**
```yaml
stages:
  - build
  - test
  - publish

variables:
  IMAGE_NAME: $CI_REGISTRY_IMAGE/myapp

# Build application
build:
  stage: build
  image: node:18-alpine
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/

# Run tests
test:
  stage: test
  image: node:18-alpine
  script:
    - npm ci
    - npm test

# Build and publish Docker image
publish:
  stage: publish
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $IMAGE_NAME:$CI_COMMIT_SHORT_SHA .
    - docker push $IMAGE_NAME:$CI_COMMIT_SHORT_SHA
```

---

## Part 2: Advanced Concepts

### Docker Networking

**Purpose:** Enable containers to communicate with each other and the outside world.

#### Network Types

**1. Bridge Network (default)**
- Isolated network for containers on same host
- Containers can communicate by name
- Access host via special DNS name

```bash
# Create custom bridge network
docker network create myapp-network

# Run containers on same network
docker run -d --name database --network myapp-network postgres:15
docker run -d --name api --network myapp-network myapi:1.0

# API can connect to database using hostname "database"
```

**2. Host Network**
- Container shares host's network stack
- No isolation, better performance
- Useful for network-intensive applications

```bash
docker run -d --network host nginx
```

**3. None Network**
- No networking
- Complete isolation
- Useful for batch processing

**4. Overlay Network**
- Multi-host networking
- Used in Docker Swarm/Kubernetes
- Containers across different hosts can communicate

#### Network Commands

```bash
# List networks
docker network ls

# Inspect network
docker network inspect myapp-network

# Connect running container to network
docker network connect myapp-network my-container

# Disconnect container from network
docker network disconnect myapp-network my-container

# Remove network
docker network rm myapp-network
```

#### Port Mapping

Expose container ports to host:

```bash
# Map port 8080 on host to port 80 in container
docker run -d -p 8080:80 nginx

# Map to specific host IP
docker run -d -p 127.0.0.1:8080:80 nginx

# Map all exposed ports to random host ports
docker run -d -P nginx
```

---

### Docker Volumes

**Purpose:** Persist data beyond container lifecycle and share data between containers.

**The Problem:** Container writable layer is deleted when container is removed!

**The Solution:** Volumes store data outside the container filesystem.

#### Volume Types

**1. Named Volumes (recommended)**
- Managed by Docker
- Stored in Docker's directory
- Easy to backup and migrate

```bash
# Create volume
docker volume create myapp-data

# Use volume
docker run -d -v myapp-data:/app/data myapp:1.0

# List volumes
docker volume ls

# Inspect volume
docker volume inspect myapp-data

# Remove volume
docker volume rm myapp-data
```

**2. Bind Mounts**
- Mount host directory into container
- Useful for development (hot reload)
- Full host filesystem access

```bash
# Mount current directory
docker run -d -v $(pwd):/app node:18

# Read-only mount
docker run -d -v $(pwd):/app:ro node:18
```

**3. tmpfs Mounts**
- Store in host memory
- Never written to disk
- For sensitive temporary data

```bash
docker run -d --tmpfs /app/temp myapp:1.0
```

#### Volume Best Practices

```bash
# Database with persistent storage
docker run -d \
  --name postgres \
  -v postgres-data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Application with development code mount
docker run -d \
  --name dev-app \
  -v $(pwd)/src:/app/src \
  -v node_modules:/app/node_modules \
  node:18

# Backup volume
docker run --rm \
  -v myapp-data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/backup.tar.gz /data
```

---

### Docker Compose

**Purpose:** Define and run multi-container applications using a YAML file.

**Why use it?**
- Manage multiple containers as a single application
- Simple configuration in `docker-compose.yml`
- Easy to start, stop, and rebuild entire stack
- Perfect for development and testing environments

#### Basic Structure

```yaml
version: '3.8'

services:
  # Service definitions
  
networks:
  # Network definitions
  
volumes:
  # Volume definitions
```

#### Common Commands

```bash
# Start all services
docker-compose up -d

# Stop all services
docker-compose down

# View logs
docker-compose logs -f

# Rebuild images
docker-compose build

# List running services
docker-compose ps

# Execute command in service
docker-compose exec api /bin/sh

# Scale service
docker-compose up -d --scale api=3
```

---

## Complete Example: GitLab CE Self-Hosted Stack

This example demonstrates a production-ready GitLab setup with separate containers for each component.

### Architecture Overview

```
┌─────────────────────────────────────────────────┐
│              GitLab Application                 │
│                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐ │
│  │  GitLab  │───>│PostgreSQL│    │  Redis   │ │
│  │   Web    │    │ Database │<───│  Cache   │ │
│  └──────────┘    └──────────┘    └──────────┘ │
│       │                                         │
│       v                                         │
│  ┌──────────┐                                  │
│  │  Shared  │                                  │
│  │ Volumes  │                                  │
│  └──────────┘                                  │
└─────────────────────────────────────────────────┘
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  # PostgreSQL Database
  postgresql:
    image: postgres:15-alpine
    container_name: gitlab-postgresql
    restart: always
    environment:
      POSTGRES_USER: gitlab
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: gitlabhq_production
    volumes:
      - postgresql-data:/var/lib/postgresql/data
    networks:
      - gitlab-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U gitlab"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: gitlab-redis
    restart: always
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - gitlab-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # GitLab Application
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    restart: always
    hostname: gitlab.example.com
    ports:
      - "80:80"        # HTTP
      - "443:443"      # HTTPS
      - "22:22"        # SSH
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://gitlab.example.com'
        
        # PostgreSQL connection
        postgresql['enable'] = false
        gitlab_rails['db_adapter'] = 'postgresql'
        gitlab_rails['db_encoding'] = 'utf8'
        gitlab_rails['db_host'] = 'postgresql'
        gitlab_rails['db_port'] = 5432
        gitlab_rails['db_database'] = 'gitlabhq_production'
        gitlab_rails['db_username'] = 'gitlab'
        gitlab_rails['db_password'] = '${POSTGRES_PASSWORD}'
        
        # Redis connection
        redis['enable'] = false
        gitlab_rails['redis_host'] = 'redis'
        gitlab_rails['redis_port'] = 6379
        
        # Email settings
        gitlab_rails['gitlab_email_enabled'] = true
        gitlab_rails['gitlab_email_from'] = 'gitlab@example.com'
        gitlab_rails['gitlab_email_display_name'] = 'GitLab'
        
        # Backup settings
        gitlab_rails['backup_keep_time'] = 604800  # 7 days
        
        # Performance tuning
        puma['worker_processes'] = 2
        sidekiq['max_concurrency'] = 10
    volumes:
      - gitlab-config:/etc/gitlab
      - gitlab-logs:/var/log/gitlab
      - gitlab-data:/var/opt/gitlab
    networks:
      - gitlab-network
    depends_on:
      postgresql:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/-/health"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 120s

# Network definition
networks:
  gitlab-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16

# Volume definitions
volumes:
  postgresql-data:
    driver: local
  redis-data:
    driver: local
  gitlab-config:
    driver: local
  gitlab-logs:
    driver: local
  gitlab-data:
    driver: local
```

### Environment Variables (.env file)

Create a `.env` file in the same directory:

```bash
# PostgreSQL
POSTGRES_PASSWORD=your-secure-password-here

# GitLab
GITLAB_ROOT_PASSWORD=your-root-password-here
```

### Deployment Steps

```bash
# 1. Create .env file with secrets
cat > .env << EOF
POSTGRES_PASSWORD=$(openssl rand -base64 32)
GITLAB_ROOT_PASSWORD=$(openssl rand -base64 32)
EOF

# 2. Start the stack
docker-compose up -d

# 3. Check status
docker-compose ps

# 4. View logs
docker-compose logs -f gitlab

# 5. Wait for GitLab to start (can take 2-5 minutes)
docker-compose logs -f gitlab | grep "gitlab Reconfigured"

# 6. Access GitLab
# Open browser: http://localhost
# Login: root / <password from .env>
```

### Component Explanation

#### PostgreSQL Service
- **Purpose:** Stores all GitLab data (users, projects, issues, etc.)
- **Volume:** `postgresql-data` persists database across restarts
- **Health Check:** Ensures database is ready before GitLab starts
- **Network:** Isolated on `gitlab-network` for security

#### Redis Service
- **Purpose:** Caching layer and job queue for Sidekiq
- **Persistence:** Uses AOF (Append Only File) for data safety
- **Volume:** `redis-data` stores cache and queue data
- **Health Check:** Verifies Redis responds to commands

#### GitLab Service
- **Purpose:** Main application (web UI, API, Git repositories)
- **Depends On:** Waits for PostgreSQL and Redis health checks
- **Volumes:**
  - `gitlab-config`: Configuration files
  - `gitlab-logs`: Application logs
  - `gitlab-data`: Git repositories, uploads, CI artifacts
- **Ports:**
  - 80/443: Web interface
  - 22: Git SSH access

### Network Communication

```
gitlab container
  ├─> connects to "postgresql" hostname on port 5432
  └─> connects to "redis" hostname on port 6379

Docker DNS resolves these to container IPs:
  postgresql = 172.20.0.2
  redis = 172.20.0.3
  gitlab = 172.20.0.4
```

### Volume Structure

```
Host System                    Container
├─ /var/lib/docker/volumes/
   ├─ postgresql-data/         -> /var/lib/postgresql/data
   ├─ redis-data/              -> /data
   ├─ gitlab-config/           -> /etc/gitlab
   ├─ gitlab-logs/             -> /var/log/gitlab
   └─ gitlab-data/             -> /var/opt/gitlab
       ├─ git-data/            (repositories)
       ├─ gitlab-ci/           (CI artifacts)
       └─ uploads/             (attachments)
```

### Management Commands

```bash
# Backup GitLab
docker-compose exec gitlab gitlab-backup create

# Restore from backup
docker-compose exec gitlab gitlab-backup restore BACKUP=<timestamp>

# View PostgreSQL data
docker-compose exec postgresql psql -U gitlab -d gitlabhq_production

# Monitor Redis
docker-compose exec redis redis-cli monitor

# Update GitLab
docker-compose pull gitlab
docker-compose up -d gitlab

# Scale down/up
docker-compose stop
docker-compose start

# Complete teardown (preserves volumes)
docker-compose down

# Complete teardown (removes volumes - DATA LOSS!)
docker-compose down -v
```

### Production Considerations

**1. Resource Allocation**
```yaml
services:
  gitlab:
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 8G
        reservations:
          cpus: '2'
          memory: 4G
```

**2. SSL/TLS Configuration**
```yaml
services:
  gitlab:
    volumes:
      - ./ssl/cert.pem:/etc/gitlab/ssl/gitlab.example.com.crt
      - ./ssl/key.pem:/etc/gitlab/ssl/gitlab.example.com.key
```

**3. Regular Backups**
```bash
# Cron job for daily backups
0 2 * * * docker-compose -f /path/to/docker-compose.yml exec -T gitlab gitlab-backup create
```

**4. Monitoring**
```yaml
services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    networks:
      - gitlab-network

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    networks:
      - gitlab-network
```

---

## Key Takeaways

### Core Concepts
1. **Dockerfile** = Instructions (blueprint)
2. **Image** = Packaged template (stamped copy)
3. **Container** = Running instance (actual execution)

### Build → Ship → Run Workflow
1. **Build**: Create images from Dockerfiles
2. **Ship**: Push images to registries
3. **Run**: Deploy containers from images

### Advanced Features
1. **Networks**: Enable container communication
2. **Volumes**: Persist data and share between containers
3. **Compose**: Orchestrate multi-container applications

### GitLab CI/CD Connection
- `image:` keyword pulls and runs containers
- Each job executes in isolated container
- Reproducible, consistent environments
- Easy to test different versions/configurations

---

## Additional Resources

- **Docker Documentation:** https://docs.docker.com
- **Docker Hub:** https://hub.docker.com
- **Docker Compose Reference:** https://docs.docker.com/compose/compose-file/
- **GitLab Docker Images:** https://docs.gitlab.com/ee/install/docker.html
- **Best Practices:** https://docs.docker.com/develop/dev-best-practices/

---

**Remember:** Docker transforms "works on my machine" into "works everywhere." Understanding these core concepts is essential for effective DevOps and CI/CD pipeline development.