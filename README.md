# MEAN Stack CRUD Application - AWS Deployment

A CRUD application built with MongoDB, Express.js, Angular 15, and Node.js, containerized with Docker and deployed on AWS EC2 with CI/CD pipeline.

## Live Application

- **URL**: http://13.61.149.169
- **GitHub**: https://github.com/vedant015/discover-dollar-assignment
- **Docker Hub**: 
  - Frontend: https://hub.docker.com/r/vedant151007/crud-dd-frontend
  - Backend: https://hub.docker.com/r/vedant151007/crud-dd-backend

## Technologies Used

- **Frontend**: Angular 15, TypeScript, Bootstrap, Nginx
- **Backend**: Node.js 18, Express.js, Mongoose
- **Database**: MongoDB (Docker image)
- **DevOps**: Docker, Docker Compose, GitHub Actions, AWS EC2 (Ubuntu 22.04)

## Containerization

### Backend Dockerfile
```dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 8080
CMD ["npm", "start"]
```

### Frontend Dockerfile
```dockerfile
FROM node:18 as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist/angular-15-crud /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## Database Setup

Using **MongoDB Docker Image** (`mongo:latest`)

**Configuration:**
- Database: `dd_db`
- Port: 27017
- Connection: `mongodb://mongo:27017/dd_db`
- Storage: Docker volume for data persistence

## AWS EC2 Deployment

**Infrastructure:**
- Instance: t2.medium
- OS: Ubuntu 22.04 LTS
- IP: 13.61.149.169

**Security Groups:**

| Port | Protocol | Purpose |
|------|----------|---------|
| 22   | TCP      | SSH     |
| 80   | HTTP     | Web App |
| 8080 | TCP      | Backend API |

**Deployment Steps:**

1. **Connect to EC2:**
```bash
ssh -i your-key.pem ubuntu@13.61.149.169
```

2. **Install Docker:**
```bash
sudo apt-get update
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

3. **Install Docker Compose:**
```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

4. **Deploy Application:**
```bash
mkdir -p ~/discover-dollar-project
cd ~/discover-dollar-project
# Create docker-compose.prod.yml (see below)
sudo docker-compose -f docker-compose.prod.yml up -d
```

**Production Docker Compose:**
```yaml
version: "3.8"
services:
  mongo:
    image: mongo:latest
    container_name: mongodb
    restart: unless-stopped
    volumes:
      - mongo_data:/data/db
    networks:
      - app-network

  backend:
    image: vedant151007/crud-dd-backend:latest
    container_name: backend
    restart: unless-stopped
    ports:
      - "8080:8080"
    depends_on:
      - mongo
    networks:
      - app-network

  frontend:
    image: vedant151007/crud-dd-frontend:latest
    container_name: frontend
    restart: unless-stopped
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - app-network

networks:
  app-network:
volumes:
  mongo_data:
```

## CI/CD Pipeline

**GitHub Actions Workflow** (`.github/workflows/ci-cd.yml`):

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    - name: Build and push images
      run: |
        docker build -t ${{ secrets.DOCKER_USERNAME }}/crud-dd-backend:latest ./backend
        docker push ${{ secrets.DOCKER_USERNAME }}/crud-dd-backend:latest
        docker build -t ${{ secrets.DOCKER_USERNAME }}/crud-dd-frontend:latest ./frontend
        docker push ${{ secrets.DOCKER_USERNAME }}/crud-dd-frontend:latest

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
    - name: Deploy to EC2
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ${{ secrets.EC2_USERNAME }}
        key: ${{ secrets.EC2_SSH_KEY }}
        script: |
          cd /home/ubuntu/discover-dollar-project
          sudo docker-compose -f docker-compose.prod.yml pull
          sudo docker-compose -f docker-compose.prod.yml up -d
          sudo docker image prune -af
```

**Required GitHub Secrets:**
- `DOCKER_USERNAME` - Docker Hub username
- `DOCKER_PASSWORD` - Docker Hub token
- `EC2_HOST` - EC2 public IP (13.61.149.169)
- `EC2_USERNAME` - ubuntu
- `EC2_SSH_KEY` - Private SSH key content

## Nginx Configuration

**Reverse Proxy Setup** (`frontend/nginx.conf`):

```nginx
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://backend:8080/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## Screenshots

### CI/CD Pipeline
- GitHub Actions workflow execution
- Build and push logs
- Successful deployment

### Docker Hub
- Frontend image repository
- Backend image repository

### AWS Deployment
- EC2 instance details
- Security group configuration
- Running containers (`docker ps`)

### Application UI
- Homepage
- Tutorial list
- Add/Edit tutorial forms
- Search functionality

## Local Development

```bash
# Backend
cd backend
npm install
node server.js

# Frontend
cd frontend
npm install
ng serve --port 8081
```

## Troubleshooting

**Application not accessible:**
- Check security groups (port 80 open)
- Verify containers: `sudo docker ps`
- Use HTTP (not HTTPS)

**CI/CD fails:**
- Verify GitHub secrets
- Check EC2 SSH key is complete
- Review GitHub Actions logs

**Containers not starting:**
```bash
sudo docker-compose -f docker-compose.prod.yml logs
sudo docker-compose -f docker-compose.prod.yml restart
```

---

**Author**: Vedant  
**Date**: February 2026  
**Status**: ✅ Deployed and Running
