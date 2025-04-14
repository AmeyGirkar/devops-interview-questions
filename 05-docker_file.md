# Docker Commands Reference Guide

## FROM
* Specifies the base image for your Docker container
* Must be the first instruction in a Dockerfile
* Defines the starting point of your container's filesystem
* Example: `FROM ubuntu:20.04` or `FROM python:3.9-slim`

**Additional Examples:**
```dockerfile
# Using the latest Ubuntu image
FROM ubuntu:latest

# Using a specific version of Node.js
FROM node:14.17.0

# Using Alpine Linux for a minimal image
FROM alpine:3.14

# Multi-stage build example
FROM golang:1.16 AS builder
```

## LABEL
* Adds metadata to an image
* Used for documentation, organization, and description

**Examples:**
```dockerfile
LABEL version="1.0"
LABEL maintainer="your.email@example.com"
LABEL description="My application container"

# Multiple labels in a single instruction (preferred approach)
LABEL version="2.1" \
      maintainer="dev.team@example.com" \
      description="Web server for our application" \
      vendor="Company Name"

# Labels can be used for filtering and organization
LABEL environment="production" \
      team="backend"
```

## RUN
* Executes commands during the build process
* Creates a new layer in the image
* Used for installing dependencies, setting up the environment

**Examples:**
```dockerfile
# Basic usage
RUN apt-get update

# Combining commands to reduce layers
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    python3 \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

# Running shell scripts
RUN chmod +x /app/setup.sh && /app/setup.sh

# Using different shell
RUN ["/bin/bash", "-c", "echo $HOME"]
```

## ADD vs COPY

Both commands copy files and directories to the container, but have important differences:

| Feature | COPY | ADD |
|---------|------|-----|
| Copies files/directories | ✅ | ✅ |
| Fetches files from URLs | ❌ | ✅ |
| Automatically extracts archives | ❌ | ✅ |
| Explicit and predictable | ✅ (preferred) | ❌ (less explicit) |
| Regular expression support | ❌ | ✅ |

**COPY Examples:**
```dockerfile
# Copy a file to a destination
COPY app.py /app/

# Copy multiple files
COPY file1.txt file2.txt /destination/

# Copy all files in a directory
COPY ./src/ /app/

# Copy with specific permissions (available in newer Docker versions)
COPY --chmod=755 script.sh /app/
```

**ADD Examples:**
```dockerfile
# Copy local files (similar to COPY)
ADD source.txt /app/

# Add a compressed file that gets automatically extracted
ADD project.tar.gz /app/

# Add a file from URL
ADD https://example.com/file.txt /app/

# Add with permissions (available in newer Docker versions)
ADD --chmod=644 config.json /app/
```

## WORKDIR
* Sets the working directory for subsequent instructions (RUN, CMD, ENTRYPOINT, COPY, ADD)
* Creates the directory if it doesn't exist
* Can be used multiple times in a Dockerfile

**Examples:**
```dockerfile
# Set working directory
WORKDIR /app

# Can use relative paths after setting WORKDIR
WORKDIR /usr/src
WORKDIR app  # Resulting path is /usr/src/app

# Use with variables
ENV APP_HOME=/application
WORKDIR $APP_HOME
```

## CMD and ENTRYPOINT
Both define what command runs when the container starts, but with different purposes:

### CMD
* Provides default command and/or parameters
* Can be overridden from command line when docker container runs
* Only the last CMD instruction is effective if multiple are specified

**Examples:**
```dockerfile
# Shell form
CMD echo "Hello World"

# Exec form (preferred)
CMD ["python", "app.py"]

# Default parameters for ENTRYPOINT
CMD ["--help"]
```

### ENTRYPOINT
* Configures the container to run as an executable
* Command line arguments to docker run are appended after all ENTRYPOINT arguments
* Not easily overridden

**Examples:**
```dockerfile
# Shell form
ENTRYPOINT echo "Hello, $NAME"

# Exec form (preferred)
ENTRYPOINT ["nginx", "-g", "daemon off;"]

# Common pattern: ENTRYPOINT with CMD as default parameters
ENTRYPOINT ["python", "app.py"]
CMD ["--config=default.conf"]
```

## USER
* Sets the user name or UID to use when running the container
* Affects RUN, CMD and ENTRYPOINT instructions that follow it
* Good security practice to avoid running processes as root

**Examples:**
```dockerfile
# Using username
USER appuser

# Using numeric UID
USER 1001

# Using both UID and GID
USER 1001:1001

# Create user then switch to it
RUN useradd -r -u 1001 -g appgroup appuser
USER appuser
```

## ENV
* Sets environment variables in the container
* Available during build and when container runs
* Can be overridden at runtime

**Examples:**
```dockerfile
# Basic usage
ENV APP_VERSION=1.0.0

# Multiple variables at once
ENV APP_HOME=/app \
    NODE_ENV=production \
    DEBUG=false

# Using variables in other instructions
ENV PATH="/opt/app/bin:${PATH}"
```

## VOLUME
* Creates a mount point for external volumes
* Used to persist data or share data between containers
* Cannot map to host directories in Dockerfile (done at runtime)

**Examples:**
```dockerfile
# Single volume
VOLUME /data

# Multiple volumes
VOLUME ["/data", "/logs", "/config"]

# Often used for database storage or configuration
VOLUME /var/lib/postgresql/data
```

## EXPOSE
* Documents which ports the container listens on at runtime
* Serves as documentation between container builder and container runner
* Does not actually publish the port (use -p or -P at runtime)

**Examples:**
```dockerfile
# Expose a single port
EXPOSE 8080

# Expose multiple ports
EXPOSE 80 443

# Specify protocol (defaults to TCP)
EXPOSE 53/udp

# Common ports by application type
EXPOSE 3306  # MySQL
EXPOSE 27017  # MongoDB
EXPOSE 80 443  # Web server
```

## Additional Useful Instructions

### ARG
* Defines variables that users can pass at build-time
* Only available during the build of the image

**Examples:**
```dockerfile
# Define build argument with default value
ARG VERSION=latest

# Use in other instructions
FROM ubuntu:${VERSION}

# Multiple ARGs
ARG USER=appuser
ARG BUILD_DATE

# Scope is after the FROM
FROM alpine
ARG SETTINGS  # Need to repeat ARG after FROM
```

### HEALTHCHECK
* Instructs Docker how to check if container is still working
* Helps Docker know the state of the container

**Examples:**
```dockerfile
# Basic check with curl
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1

# Check with custom script
HEALTHCHECK --interval=5m --timeout=3s \
  CMD ["/healthcheck.sh"]

# Disable inherited healthcheck
HEALTHCHECK NONE
```

### SHELL
* Changes the default shell used for shell form instructions

**Examples:**
```dockerfile
# Switch to PowerShell on Windows
SHELL ["powershell", "-command"]

# Use bash with specific options
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
```

### ONBUILD
* Adds a trigger instruction to be executed when the image is used as a base

**Examples:**
```dockerfile
# Run npm install when this image is used as a base
ONBUILD RUN npm install

# Copy application code when image is used as base
ONBUILD COPY . /app
```

## Best Practices

1. **Use multi-stage builds** to create smaller images
   ```dockerfile
   FROM node:14 AS builder
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   RUN npm run build
   
   FROM nginx:alpine
   COPY --from=builder /app/dist /usr/share/nginx/html
   ```

2. **Minimize layers** by combining related commands
   ```dockerfile
   # Bad: Creates 3 layers
   RUN apt-get update
   RUN apt-get install -y python
   RUN rm -rf /var/lib/apt/lists/*
   
   # Good: Creates 1 layer
   RUN apt-get update && \
       apt-get install -y python && \
       rm -rf /var/lib/apt/lists/*
   ```

3. **Use .dockerignore** to exclude unnecessary files
   ```
   .git
   node_modules
   logs
   *.md
   ```

4. **Set specific versions** for predictable builds
   ```dockerfile
   FROM python:3.9.7-slim  # Good
   FROM python:latest      # Less predictable
   ```








# Dockerfile Interview Questions and Answers

## Basic Dockerfile Questions

### 1. What is a Dockerfile and why is it important?
**Answer:** A Dockerfile is a text document containing a series of instructions and commands used to automatically build a Docker image. It's important because it enables infrastructure as code, provides version control for your environment, ensures consistency across deployments, and makes the build process repeatable and automated.

### 2. Explain the purpose of the FROM instruction in a Dockerfile.
**Answer:** The FROM instruction initializes a new build stage and sets the base image for subsequent instructions. Every valid Dockerfile must start with a FROM instruction. It specifies the starting point for your container image, which can be a minimal OS like Alpine Linux or an image that already includes tools and frameworks like Node.js or Python. Examples: `FROM ubuntu:20.04` or `FROM python:3.9-slim`.

### 3. What's the difference between RUN, CMD, and ENTRYPOINT instructions?
**Answer:** 
- **RUN**: Executes commands during image build time and creates new layers in the image. Used for installing packages, setting up the environment, etc.
- **CMD**: Provides default commands and/or parameters for the container when it starts. Can be overridden by command-line arguments.
- **ENTRYPOINT**: Configures the container to run as an executable. Command-line arguments to `docker run` are appended after all ENTRYPOINT arguments.

### 4. How would you optimize a Dockerfile to create smaller images?
**Answer:**
- Use a smaller base image (like Alpine)
- Combine multiple RUN commands with && to reduce layers
- Use multi-stage builds to include only necessary artifacts
- Remove unnecessary files in the same layer they're created
- Use .dockerignore to exclude unnecessary files
- Clean package manager caches (e.g., `apt-get clean`, `rm -rf /var/lib/apt/lists/*`)
- Install only required packages with --no-install-recommends

### 5. Explain the difference between COPY and ADD instructions.
**Answer:** Both COPY and ADD copy files from source to destination in the image. However, ADD has additional features:
- ADD can extract compressed files automatically
- ADD can download files from URLs
- COPY is more transparent and preferred for simple copying
- ADD should be used only when its additional functionality is required

## Intermediate Dockerfile Questions

### 6. What is a multi-stage build in Docker and what problem does it solve?
**Answer:** Multi-stage builds allow you to use multiple FROM statements in your Dockerfile. Each FROM instruction can use a different base image, and begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don't need in the final image. This helps create much smaller production images by:
- Separating build-time dependencies from runtime dependencies
- Including only the necessary artifacts in the final image
- Reducing the security attack surface by removing build tools

### 7. How would you use ARG and ENV instructions and what's the difference?
**Answer:**
- **ARG** defines variables that users can pass at build-time using `--build-arg` flag. ARGs are not available after the image is built.
- **ENV** sets environment variables that persist when a container is running. They're available both during build and runtime.

Example:
```dockerfile
# Build-time argument
ARG VERSION=latest

# Environment variable
ENV APP_HOME=/app
ENV DEBUG=false

# Using ARG with FROM
FROM node:${VERSION}
```

### 8. What is the purpose of the WORKDIR instruction?
**Answer:** WORKDIR sets the working directory for any subsequent RUN, CMD, ENTRYPOINT, COPY, and ADD instructions. If the directory doesn't exist, it will be created. Using WORKDIR is preferred over a series of RUN cd /path commands, as it makes the Dockerfile more readable and avoids path-related issues. It's a best practice to set a specific working directory rather than operating in the root directory.

### 9. How would you handle sensitive data in a Dockerfile?
**Answer:** Sensitive data should not be included directly in a Dockerfile since it becomes part of the image history. Better approaches include:
- Using build arguments (ARG) for build-time secrets, but remember they're still visible in the image history
- Using Docker secrets for runtime secrets
- Using environment variables passed at runtime
- Using volumes to mount sensitive files at runtime
- Using multi-stage builds to avoid leaking build-time secrets
- For newer Docker versions, using the experimental `--secret` flag with BuildKit

### 10. Explain the importance of the .dockerignore file.
**Answer:** The .dockerignore file allows you to specify patterns for files and directories that should be excluded from the build context sent to the Docker daemon. Benefits include:
- Faster builds by reducing the size of the build context
- Preventing sensitive files from being accidentally added to images
- Avoiding unnecessary cache invalidation from files that change frequently but aren't needed in the build
- Reducing image size by excluding unnecessary files

## Advanced Dockerfile Questions

### 11. How can you create a non-root user in a Dockerfile and why is it important?
**Answer:** Creating a non-root user improves security by following the principle of least privilege. Example:
```dockerfile
RUN useradd -r -u 1001 -g appgroup appuser
USER appuser
```
This is important because:
- Prevents container processes from having root access to the host if they escape
- Reduces the attack surface if an application is compromised
- Follows security best practices and may be required by company policies

### 12. What is the HEALTHCHECK instruction and how would you implement it?
**Answer:** HEALTHCHECK tells Docker how to test a container to check if it's still working. If a container fails its health check, it will be marked as unhealthy. Example:
```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```
Parameters include:
- `--interval`: How often to run the check (default: 30s)
- `--timeout`: How long to wait before considering it failed (default: 30s)
- `--start-period`: Initial grace period for startup (default: 0s)
- `--retries`: Number of consecutive failures needed to mark unhealthy (default: 3)

### 13. Explain the ONBUILD instruction and when you might use it.
**Answer:** ONBUILD registers a command to be executed later, when the current image is used as a base for another build. It's triggered when another Dockerfile uses this image in its FROM instruction. It's useful for:
- Creating reusable build templates
- Standardizing build processes across multiple projects
- Automating tasks when teams build child images
- Setting up frameworks or libraries in a consistent way

Example:
```dockerfile
# In a base image Dockerfile
FROM node:14
WORKDIR /app
ONBUILD COPY package*.json ./
ONBUILD RUN npm install
ONBUILD COPY . .

# Then in a child Dockerfile, these commands run automatically
FROM my-node-base
# The ONBUILD commands execute here
CMD ["npm", "start"]
```

### 14. What is the SHELL instruction in a Dockerfile?
**Answer:** The SHELL instruction allows you to change the default shell used for shell-form commands. The default shell on Linux is `/bin/sh -c`, but you can change it using the SHELL instruction. This is useful when:
- Working with Windows containers (using PowerShell)
- Needing specific shell features not available in /bin/sh
- Adding shell options for better error handling

Example:
```dockerfile
# Use bash with pipefail option to catch errors in pipes
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN wget -O - https://example.com | wc -l
```

### 15. How would you implement a proper Docker caching strategy in your Dockerfile?
**Answer:** To optimize Docker's build cache:
- Order instructions from least to most frequently changing
- Place COPY commands for frequently changing files as late as possible
- Separate dependency installation from code copying
- Use specific tags for base images instead of "latest"
- Group related RUN commands into a single instruction
- Use .dockerignore to exclude files that invalidate cache but aren't needed
- Consider using build arguments for cache-busting when needed

Example:
```dockerfile
FROM node:14
WORKDIR /app

# Copy dependency files first
COPY package*.json ./

# Install dependencies in a separate layer
RUN npm install

# Copy source code after dependencies
COPY . .

CMD ["node", "app.js"]
```

### 16. What are some security best practices for Dockerfiles?
**Answer:**
- Use specific image versions instead of "latest" tags
- Create and use non-root users
- Minimize the number of layers and installed packages
- Remove unnecessary tools and development dependencies
- Use multi-stage builds to reduce attack surface
- Scan images for vulnerabilities using tools like Trivy, Clair, or Docker Scout
- Don't store secrets in the Dockerfile or image
- Set filesystem directories and files to be read-only when possible
- Keep base images updated for security patches
- Use COPY instead of ADD when simple file copying is needed
