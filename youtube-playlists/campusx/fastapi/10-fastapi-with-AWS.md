# Deploying FastAPI to AWS - Production Deployment

## Overview
Final step in ML API deployment pipeline - hosting Dockerized FastAPI application on AWS for global accessibility.

---

## Deployment Architecture

### Complete Pipeline Recap
1. **Model Building** - Train ML model for insurance premium prediction
2. **API Development** - Create FastAPI endpoint with best practices
3. **Dockerization** - Containerize application and push to Docker Hub
4. **Cloud Deployment** - Deploy to AWS for public access ✓ (Current)

### End-to-End Flow
User → Frontend (Streamlit) → AWS Deployed API → ML Model → Prediction → Response

---

## Prerequisites

### Required Knowledge
- Basic cloud computing fundamentals
- AWS terminology understanding
- Docker basics

### Required Resources
- AWS account (free tier eligible)
- Debit/credit card for verification (no charges within free tier)
- Docker image on Docker Hub

### Deployment Method
- **Service:** AWS EC2 (Elastic Compute Cloud)
- **Level:** Basic setup for beginners
- **Note:** For production-scale applications, use advanced deployment methods (ECS, EKS, Lambda)

---

## Step-by-Step Deployment Process

### 1. Create AWS Account
- Visit aws.amazon.com
- Register with card verification (free tier available)
- No charges if staying within free tier limits
- Sign in to AWS Console

---

### 2. Launch EC2 Instance

#### Navigate to EC2
- Search "EC2" in AWS dashboard
- Click "Launch Instances"

#### Configure Instance Settings

**Name:** Choose descriptive name (e.g., "my-fastapi-server")

**Operating System:** Ubuntu

**Instance Type:** T2 Micro
- 1GB memory
- **Free tier eligible** - Critical to avoid charges
- Sufficient for basic API deployment

**Key Pair:**
- Select existing or create new
- Used for SSH remote access
- AWS requires this (even if not used in this flow)

**Network Settings:**
- Keep defaults
- **Enable:** "Allow SSH traffic from Anywhere"
- Required for connection

**Storage:** 8GB (sufficient for this application)

#### Launch
- Click "Launch Instance"
- Wait for instance state: "Running"

---

### 3. Connect to EC2 and Install Docker

#### Connection Method
Two options:
1. SSH client (e.g., PuTTY)
2. **AWS Console "Connect" button** (easier - opens browser terminal)

#### Execute Commands Sequentially
```bash
# Update packages
sudo apt-get update

# Install Docker
sudo apt-get install -y docker.io

# Start Docker service
sudo systemctl start docker

# Enable Docker auto-start on reboot
sudo systemctl enable docker

# Grant permissions to pull Docker images
sudo usermod -aG docker $USER

# Exit and reconnect for permissions to take effect
exit
```

#### Why Each Command Matters
- **update** - Ensures latest package versions
- **install docker.io** - Installs Docker engine
- **start docker** - Activates Docker service
- **enable docker** - Auto-starts Docker if server reboots (prevents downtime)
- **usermod** - **Critical:** Allows pulling images from Docker Hub
- **exit** - Required to apply permission changes

---

### 4. Reconnect and Deploy Application

#### Reconnect to Instance
- Click "Connect" button again
- Verify Docker is running

#### Pull Docker Image
```bash
docker pull <username>/<image-name>:latest
```
Example: `docker pull tweakster24/insurance-premium-api:latest`

**Purpose:** Downloads application from Docker Hub to EC2

#### Run Docker Container
```bash
docker run -p 8000:8000 <username>/<image-name>
```

**Port Mapping:**
- `-p 8000:8000` - Maps EC2 port 8000 to container port 8000
- Starts Uvicorn server

---

### 5. Configure Security Groups (Critical Step)

#### Problem
- API running but not externally accessible
- Port 8000 blocked to external IPs by default

#### Solution: Open Port 8000

**Navigate:**
- EC2 Instance Details → Security → Security Groups
- Click "Edit inbound rules"

**Add New Rule:**
- **Type:** Custom TCP
- **Port Range:** 8000
- **Source:** Anywhere (0.0.0.0/0)
- Save rules

**Security Note:** In production, restrict source IPs for better security

---

### 6. Verify Deployment

#### Get Public URL
- Copy "Public IPv4 address" from EC2 instance details
- Format: `http://<PUBLIC_IP>:8000`
- **Use HTTP, not HTTPS** (no SSL certificate configured)

#### Test Endpoints
- `http://<PUBLIC_IP>:8000/` - Home endpoint
- `http://<PUBLIC_IP>:8000/health` - Health check
- `http://<PUBLIC_IP>:8000/docs` - Interactive API docs
- `http://<PUBLIC_IP>:8000/predict` - Prediction endpoint

#### Test Prediction
- Access `/docs` endpoint
- Input test data
- Verify prediction response

---

### 7. Update Frontend (Optional)

#### Modify Streamlit Code
**Change API URL:**
```python
# Before (local)
API_URL = "http://localhost:8000/predict"

# After (deployed)
API_URL = "http://<PUBLIC_IP>:8000/predict"
```

#### Run Frontend
```bash
streamlit run front.py
```

**Result:** Frontend now communicates with AWS-deployed API

---

## Key Deployment Concepts

### EC2 Instance
- Virtual server in AWS cloud
- Runs Ubuntu operating system
- Hosts Docker container with API

### Security Groups
- Virtual firewall for EC2 instances
- Controls inbound/outbound traffic
- Must explicitly allow port 8000 for API access

### Docker on EC2
- Docker runs inside EC2 instance
- Pulls image from Docker Hub
- Container runs isolated from host system

### Port Mapping
- EC2 port 8000 → Container port 8000
- External requests to EC2:8000 forwarded to container

---

## Common Issues and Solutions

### API Not Accessible
**Check:**
1. Security group allows port 8000
2. Using HTTP (not HTTPS)
3. Container is running
4. Correct Public IP used

### Docker Permission Denied
**Solution:**
- Run `sudo usermod -aG docker $USER`
- Exit and reconnect to EC2

### Container Stops on Reboot
**Solution:**
- Ensure `sudo systemctl enable docker` was run
- Use Docker restart policies

---

## Production Considerations

### Current Setup Limitations
- Single EC2 instance (no redundancy)
- No auto-scaling
- No load balancing
- No SSL/HTTPS
- No custom domain

### Production-Ready Improvements
1. **Load Balancer** - Distribute traffic across multiple instances
2. **Auto Scaling** - Handle traffic spikes
3. **SSL Certificate** - Enable HTTPS (AWS Certificate Manager)
4. **Custom Domain** - Use Route 53 for DNS
5. **Container Orchestration** - ECS or EKS for management
6. **Monitoring** - CloudWatch for logs and metrics
7. **CI/CD Pipeline** - Automated deployment

### Advanced AWS Services
- **ECS (Elastic Container Service)** - Managed container orchestration
- **EKS (Elastic Kubernetes Service)** - Kubernetes on AWS
- **Lambda** - Serverless deployment
- **API Gateway** - Managed API service
- **CloudFront** - CDN for global distribution

---

## Cost Optimization

### Free Tier Benefits
- T2 Micro instance: 750 hours/month free
- 30GB EBS storage
- Inbound data transfer free
- Outbound: 100GB/month free

### Staying Within Free Tier
- Use only T2 Micro instances
- Stop instances when not needed
- Monitor usage in billing dashboard
- Set up billing alerts

---

## Complete Deployment Checklist

✓ AWS account created  
✓ EC2 instance launched (T2 Micro)  
✓ Docker installed and enabled  
✓ Docker image pulled from Hub  
✓ Container running on port 8000  
✓ Security group configured  
✓ API accessible via public IP  
✓ All endpoints tested  
✓ Frontend updated (if applicable)  

---

## Key Takeaways

### What You've Accomplished
1. **Local Development** - Built and improved ML API
2. **Containerization** - Dockerized for consistency
3. **Cloud Deployment** - Globally accessible application

### Skills Developed
- FastAPI development
- Pydantic validation
- Docker containerization
- AWS EC2 deployment
- Cloud security configuration

### Next Level
- Advanced FastAPI features
- Production-grade deployments
- Scalable architectures
- DevOps best practices
