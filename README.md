# Vercel Clone (Full Stack Deployment Platform)

A simplified, high-performance deployment platform inspired by Vercel. This project automates the process of cloning a Git repository, building the project inside an isolated container, storing the static assets, and serving them via a custom reverse proxy.

## 🏗️ Architecture Overview

The platform operates on a distributed microservices architecture to ensure isolation between the build process and the serving layer.

### Workflow:
- Client → API Server → AWS ECS (Fargate) → S3 → Reverse Proxy

### Components:
- **Client:** Submits a GitHub repository URL and a unique slug.
- **API Server:** Validates requests and triggers an AWS ECS Fargate task.
- **Build Server (ECS):** Clones the repo, installs dependencies, builds the project, and pushes the output to Amazon S3.
- **Redis & Socket.io:** The Build Server pushes real-time logs to Redis; the API Server subscribes to these logs and streams them to the client via WebSockets.
- **S3 Reverse Proxy:** Intercepts incoming requests, extracts the subdomain (slug), and fetches the corresponding content from S3.

## 🛠️ Tech Stack
- **Runtime:** Node.js, Express.js
- **Compute:** AWS ECS (Fargate) & Docker
- **Storage:** AWS S3
- **Messaging/Real-time:** Redis (Pub/Sub) & Socket.io
- **Routing:** Node.js HTTP Proxy

## 📁 Project Structure
```
VERCEL/
├── api-server/           # Orchestrator: Handles project creation & ECS tasks
├── build-server/         # Worker: Builds project & uploads to S3 (Dockerized)
├── s3-reverse-proxy/     # Gateway: Serves deployed sites via subdomains
└── README.md
```
---


## 🚀 Setup Instructions
```bash
git clone https://github.com/your-username/vercel-clone.git
cd vercel-clone
```
1. Create `.env` files in both `api-server` and `build-server` directories with necessary environment variables.
```bash
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=
CLUSTER_ARN=
TASK_ARN=
SUBNET_1=
SUBNET_2=
SUBNET_3=
SECURITY_GROUP=
REDIS_URL=
```
2. Install dependencies:
```bash
cd api-server && npm install
cd ../build-server && npm install
cd ../s3-reverse-proxy && npm install
```
3. Start services:
- Build Server
```bash
docker build -t ${IMAGE_NAME}:${TAG} .
docker tag ${IMAGE_NAME}:${TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}:${TAG}
docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}:${TAG}

```
- API & Socket Server:
```bash
cd api-server
node index.js
defaults for port 9000 for API and 9002 for Socket.
```
- Reverse Proxy:
```bash
cd s3-reverse-proxy
node index.js
defaults for port 8000.
```
Optional Docker command for local Redis:
```bash
docker run -d --name redis-server -p 6379:6379 redis
defaults for Redis server.
```
---
## 🔌 API Reference
### Deploy a Project
POST `http://localhost:9000/project`
Request Body:
ejson {"gitURL": "https://github.com/your-repo.git", "slug": "my-app"}
ejson Response:
ejson {"status": "queued", "data": {"projectSlug": "my-app", "url": "http://my-app.localhost:8000"}}
tests are streamed via Redis logs published on channel `logs:<projectSlug>`.f
