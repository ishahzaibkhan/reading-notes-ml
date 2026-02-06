# Dockerizing FastAPI Applications

## Overview
Complete guide to containerizing a FastAPI ML API, pushing to Docker Hub, and distributing for deployment.

---

## Why Docker for ML APIs?

### Key Benefits
- **Consistency** - Runs identically on any machine
- **Portability** - Easy distribution to testers and deployment platforms
- **Isolation** - Self-contained environment with all dependencies
- **Scalability** - Ready for cloud deployment (AWS, etc.)

---

## Prerequisites

### Required Setup
1. **Docker Desktop** - Install from official Docker website
2. **Docker Hub Account** - Create account at hub.docker.com
3. **Docker Fundamentals** - Understanding of basic Docker concepts

### Core Concepts
- **Image** - Result of Dockerizing an application (blueprint)
- **Container** - Running instance of a Docker image
- **Docker Hub** - Repository for hosting and sharing Docker images

---

## Dockerfile Structure

### Purpose
Instruction manual that guides Docker on building the image step-by-step

### Essential Components

#### 1. Base Image
- Operating system with pre-installed Python version
- Foundation for the application

#### 2. Working Directory
- Set up application directory (e.g., `/app`)
- All subsequent commands execute here

#### 3. Install Dependencies
- Copy `requirements.txt` to working directory
- Install all required Python libraries

#### 4. Copy Application Code
- Transfer all application files (code, models, configs)
- Maintains folder structure

#### 5. Expose Port
- Declare which port the API uses (e.g., 8000 for FastAPI)
- Documents network accessibility

#### 6. Run Command
- Define startup command: `uvicorn app.app --host 0.0.0.0 --port 8000`
- `--host 0.0.0.0` accepts requests from any IP address
- Essential for external accessibility

---

## Building Docker Image

### Build Process

**Command:**
```
docker build -t <username>/<image-name> .
```

**Components:**
- `<username>` - Docker Hub username
- `<image-name>` - Descriptive name for your API
- `.` - Build context (current directory)

**Example:**
```
docker build -t tws24/insurance-premium-api .
```

### Build Behavior
- Builds layer by layer
- Uses caching for faster subsequent builds
- Each Dockerfile instruction creates a new layer

### Verification
- Check Docker Desktop "Images" tab
- Confirm image appears with correct name and size

---

## Pushing to Docker Hub

### Authentication
**Command:**
```
docker login
```
- Prompts for Docker Hub username and password (first time)
- Authenticates your machine with Docker Hub

### Upload Image
**Command:**
```
docker push <username>/<image-name>
```

**Process:**
- Uploads layer by layer
- May take time depending on image size (e.g., 1GB+)
- Default tag is `latest` if not specified

### Verification
- Refresh Docker Hub account page
- Image appears with pull commands and documentation
- Shareable link available for distribution

---

## Pulling and Running (Tester Perspective)

### Download Image
**Command:**
```
docker pull <username>/<image-name>
```
- Downloads image from Docker Hub
- All dependencies included

### Run Container
**Command:**
```
docker run -p 8000:8000 <username>/<image-name>
```
- `-p 8000:8000` maps container port to host port
- Makes API accessible at `localhost:8000`

### Testing the API
Access these endpoints:
- `localhost:8000` - Home endpoint
- `localhost:8000/health` - Health check
- `localhost:8000/docs` - Interactive API documentation
- `localhost:8000/predict` - Prediction endpoint

---

## Complete Workflow

### Developer Side
1. Write Dockerfile
2. Build Docker image locally
3. Test image locally
4. Login to Docker Hub
5. Push image to Docker Hub
6. Share image link

### Tester/User Side
1. Pull image from Docker Hub
2. Run container locally
3. Access API endpoints
4. Test functionality

### Deployment Side
1. Pull image on cloud platform
2. Run container in production
3. API accessible globally

---

## Best Practices

### Dockerfile Optimization
- Use specific base image versions (not `latest`)
- Order instructions by change frequency (dependencies before code)
- Leverage layer caching for faster builds
- Minimize image size by removing unnecessary files

### Image Management
- Use descriptive image names
- Tag versions appropriately (not just `latest`)
- Document image usage in README
- Clean up unused images regularly

### Security Considerations
- Don't include sensitive data in images
- Use `.dockerignore` to exclude unnecessary files
- Keep base images updated
- Scan images for vulnerabilities

---

## Common Commands Reference
```
docker build -t <name> .              # Build image
docker images                         # List images
docker login                          # Authenticate to Docker Hub
docker push <name>                    # Upload image
docker pull <name>                    # Download image
docker run -p <host>:<container> <name>  # Run container
docker ps                             # List running containers
docker stop <container-id>            # Stop container
docker rm <container-id>              # Remove container
docker rmi <image-id>                 # Remove image
```

---

## Key Advantages

### For Development
- Consistent development environment
- Easy onboarding for new team members
- Version control for entire environment

### For Testing
- Test in production-like environment
- No dependency installation needed
- Reproducible testing conditions

### For Deployment
- Cloud-ready containerized application
- Easy scaling and orchestration
- Platform-independent deployment

---

## Next Steps in Production Pipeline

1. **Dockerization** ✓ (Current)
2. **Cloud Deployment** - Deploy to AWS, Azure, or GCP
3. **Orchestration** - Use Kubernetes for scaling
4. **CI/CD Integration** - Automate build and deployment
5. **Monitoring** - Track container health and performance
