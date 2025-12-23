---
title: "Section 4: DevOps & Infrastructure"
parent: "Part 5: Specialized Workflows & Domains"
nav_order: 4
---

# Section 4: DevOps & Infrastructure

**Part of:** [Part 5: Specialized Workflows & Domains](README.md)  
**Related:** [Core Workflows](../01-fundamentals-core-concepts/03-core-workflows.md) | [Backend Development](02-backend-development.md)

---

## 📋 Overview

This section demonstrates how GitHub Copilot accelerates DevOps and infrastructure workflows, from Docker container creation to Kubernetes deployments, CI/CD pipelines, and Infrastructure as Code. Copilot understands YAML configurations, Terraform syntax, and deployment best practices across cloud providers.

**What you'll learn:**
- Dockerfile generation and multi-stage builds
- Docker Compose for local development
- Kubernetes manifests and Helm charts
- CI/CD pipelines (GitHub Actions, GitLab CI)
- Infrastructure as Code (Terraform, CDK, Pulumi)
- Shell scripting for automation
- Configuration file generation
- Deployment strategies

**Time to master:** 50-60 minutes

---

## 🎯 Key Takeaways

1. **Container expertise:** Copilot generates optimized Dockerfiles with multi-stage builds
2. **K8s fluency:** Auto-generate Kubernetes manifests, deployments, services, ingress
3. **CI/CD automation:** Build complete GitHub Actions workflows with testing and deployment
4. **IaC mastery:** Generate Terraform modules for AWS, Azure, GCP
5. **Config generation:** Create nginx, Apache, cloud-init configurations
6. **Security built-in:** Include best practices for secrets, scanning, RBAC

---

## 🚀 Quick Wins: DevOps & Infrastructure

### Pattern 1: Dockerfile Generation

**Generate optimized multi-stage Dockerfile:**

```
Chat:

"Create production-ready Dockerfile for Node.js Express app:
- Multi-stage build for small image size
- Non-root user for security
- Health check
- Build cache optimization
- Alpine Linux base"
```

**Result:**

```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies (including dev dependencies for build)
RUN npm ci

# Copy source code
COPY . .

# Build TypeScript (if applicable)
RUN npm run build

# Remove dev dependencies
RUN npm prune --production

# Stage 2: Production
FROM node:20-alpine AS production

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

# Create app directory
WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy built application from builder
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package*.json ./

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Use dumb-init to handle signals properly
ENTRYPOINT ["dumb-init", "--"]

# Start application
CMD ["node", "dist/index.js"]
```

**Python FastAPI Dockerfile:**

```
Chat:

"Generate Dockerfile for Python FastAPI app with:
- Poetry for dependency management
- Multi-stage build
- Security scanning
- Minimal final image"
```

**Result:**

```dockerfile
# Stage 1: Build
FROM python:3.11-slim AS builder

# Install Poetry
RUN pip install --no-cache-dir poetry==1.7.1

# Set working directory
WORKDIR /app

# Copy dependency files
COPY pyproject.toml poetry.lock ./

# Install dependencies
RUN poetry config virtualenvs.in-project true && \
    poetry install --only main --no-interaction --no-ansi

# Copy application code
COPY . .

# Stage 2: Production
FROM python:3.11-slim AS production

# Install security updates
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN useradd -m -u 1000 appuser

WORKDIR /app

# Copy virtual environment from builder
COPY --from=builder --chown=appuser:appuser /app/.venv ./.venv
COPY --from=builder --chown=appuser:appuser /app .

# Switch to non-root user
USER appuser

# Add venv to PATH
ENV PATH="/app/.venv/bin:$PATH"

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health').read()"

# Start application with Uvicorn
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

### Pattern 2: Docker Compose for Local Development

**Generate Docker Compose setup:**

```
Chat:

"Create docker-compose.yml for full-stack development:
- React frontend (hot reload)
- Node.js backend
- PostgreSQL database
- Redis cache
- Nginx reverse proxy
Include volume mounts and networking"
```

**Result:**

```yaml
version: '3.9'

services:
  # PostgreSQL Database
  postgres:
    image: postgres:16-alpine
    container_name: dev-postgres
    environment:
      POSTGRES_DB: ${DB_NAME:-appdb}
      POSTGRES_USER: ${DB_USER:-appuser}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-secret}
      PGDATA: /var/lib/postgresql/data/pgdata
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER:-appuser}"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: dev-redis
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
    networks:
      - app-network

  # Backend API
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    container_name: dev-backend
    environment:
      NODE_ENV: development
      DATABASE_URL: postgresql://${DB_USER:-appuser}:${DB_PASSWORD:-secret}@postgres:5432/${DB_NAME:-appdb}
      REDIS_URL: redis://redis:6379
      JWT_SECRET: ${JWT_SECRET:-dev-secret}
      PORT: 3000
    volumes:
      - ./backend:/app
      - /app/node_modules
      - ./backend/uploads:/app/uploads
    ports:
      - "3000:3000"
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    command: npm run dev
    networks:
      - app-network

  # Frontend React App
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    container_name: dev-frontend
    environment:
      REACT_APP_API_URL: http://localhost/api
      CHOKIDAR_USEPOLLING: "true"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    ports:
      - "5173:5173"
    depends_on:
      - backend
    command: npm run dev -- --host
    networks:
      - app-network

  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    container_name: dev-nginx
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - frontend
      - backend
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  postgres-data:
  redis-data:
```

**Nginx configuration:**

```nginx
# nginx/conf.d/default.conf
upstream backend {
    server backend:3000;
}

upstream frontend {
    server frontend:5173;
}

server {
    listen 80;
    server_name localhost;

    # Frontend
    location / {
        proxy_pass http://frontend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        
        # WebSocket support for HMR
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Backend API
    location /api {
        rewrite ^/api/(.*) /$1 break;
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Health check
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
```

---

### Pattern 3: Kubernetes Manifests

**Generate K8s deployment:**

```
Chat:

"Create Kubernetes manifests for production deployment:
- Deployment with 3 replicas
- Horizontal Pod Autoscaler
- Service (ClusterIP)
- Ingress with TLS
- ConfigMap and Secret
- Resource limits
- Liveness and readiness probes"
```

**Result:**

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    name: production
    environment: production

---
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: production
data:
  NODE_ENV: "production"
  LOG_LEVEL: "info"
  PORT: "3000"

---
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: production
type: Opaque
stringData:
  DATABASE_URL: "postgresql://user:password@postgres:5432/appdb"
  JWT_SECRET: "your-jwt-secret-here"
  REDIS_URL: "redis://redis:6379"

---
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
  namespace: production
  labels:
    app: myapp
    version: v1
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        version: v1
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "3000"
        prometheus.io/path: "/metrics"
    spec:
      # Use node affinity for production nodes
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - myapp
                topologyKey: kubernetes.io/hostname
      
      # Security context
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
      
      containers:
      - name: app
        image: myregistry.azurecr.io/myapp:latest
        imagePullPolicy: Always
        
        ports:
        - name: http
          containerPort: 3000
          protocol: TCP
        
        # Environment variables
        envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: app-secrets
        
        # Resource limits
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        
        # Liveness probe
        livenessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        # Readiness probe
        readinessProbe:
          httpGet:
            path: /ready
            port: http
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        
        # Security context
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
          capabilities:
            drop:
              - ALL
        
        # Volume mounts
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: cache
          mountPath: /app/.cache
      
      volumes:
      - name: tmp
        emptyDir: {}
      - name: cache
        emptyDir: {}

---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service
  namespace: production
  labels:
    app: myapp
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: http
    protocol: TCP
    name: http
  selector:
    app: myapp

---
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 2
        periodSeconds: 15
      selectPolicy: Max

---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: production
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: app-tls-secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```

---

### Pattern 4: GitHub Actions CI/CD Pipeline

**Generate complete CI/CD workflow:**

```
Chat → Agent mode

"Create GitHub Actions workflow for Node.js app:
- Trigger on push to main and PRs
- Run tests and linting
- Build Docker image
- Push to Azure Container Registry
- Deploy to Azure Kubernetes Service
- Run smoke tests
- Notify Slack on success/failure
Include caching and security scanning"
```

**Result:**

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  workflow_dispatch:

env:
  NODE_VERSION: '20.x'
  REGISTRY: myregistry.azurecr.io
  IMAGE_NAME: myapp
  AZURE_CLUSTER_NAME: prod-aks
  AZURE_RESOURCE_GROUP: prod-resources
  NAMESPACE: production

jobs:
  # Job 1: Code Quality & Testing
  test:
    name: Test & Lint
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linter
      run: npm run lint
    
    - name: Run type check
      run: npm run type-check
    
    - name: Run unit tests
      run: npm run test:unit -- --coverage
    
    - name: Run integration tests
      run: npm run test:integration
    
    - name: Upload coverage reports
      uses: codecov/codecov-action@v4
      with:
        file: ./coverage/coverage-final.json
        flags: unittests
        name: codecov-umbrella
    
    - name: SonarCloud Scan
      uses: SonarSource/sonarcloud-github-action@master
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  # Job 2: Security Scanning
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Run Snyk security scan
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      with:
        args: --severity-threshold=high
    
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        scan-ref: '.'
        format: 'sarif'
        output: 'trivy-results.sarif'
    
    - name: Upload Trivy results to GitHub Security
      uses: github/codeql-action/upload-sarif@v3
      with:
        sarif_file: 'trivy-results.sarif'

  # Job 3: Build & Push Docker Image
  build:
    name: Build & Push Image
    runs-on: ubuntu-latest
    needs: [test, security]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
    
    - name: Log in to Azure Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ secrets.ACR_USERNAME }}
        password: ${{ secrets.ACR_PASSWORD }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=sha,prefix={{branch}}-
          type=semver,pattern={{version}}
          type=semver,pattern={{major}}.{{minor}}
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=registry,ref=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:buildcache
        cache-to: type=registry,ref=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:buildcache,mode=max
        build-args: |
          NODE_ENV=production
    
    - name: Scan Docker image with Trivy
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: ${{ steps.meta.outputs.tags }}
        format: 'sarif'
        output: 'trivy-image-results.sarif'
    
    - name: Upload image scan results
      uses: github/codeql-action/upload-sarif@v3
      with:
        sarif_file: 'trivy-image-results.sarif'

  # Job 4: Deploy to AKS
  deploy:
    name: Deploy to AKS
    runs-on: ubuntu-latest
    needs: build
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://myapp.example.com
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    
    - name: Set AKS context
      uses: azure/aks-set-context@v3
      with:
        resource-group: ${{ env.AZURE_RESOURCE_GROUP }}
        cluster-name: ${{ env.AZURE_CLUSTER_NAME }}
    
    - name: Create namespace if not exists
      run: |
        kubectl create namespace ${{ env.NAMESPACE }} --dry-run=client -o yaml | kubectl apply -f -
    
    - name: Create image pull secret
      run: |
        kubectl create secret docker-registry acr-secret \
          --docker-server=${{ env.REGISTRY }} \
          --docker-username=${{ secrets.ACR_USERNAME }} \
          --docker-password=${{ secrets.ACR_PASSWORD }} \
          --namespace=${{ env.NAMESPACE }} \
          --dry-run=client -o yaml | kubectl apply -f -
    
    - name: Deploy to Kubernetes
      uses: azure/k8s-deploy@v4
      with:
        namespace: ${{ env.NAMESPACE }}
        manifests: |
          k8s/namespace.yaml
          k8s/configmap.yaml
          k8s/secret.yaml
          k8s/deployment.yaml
          k8s/service.yaml
          k8s/hpa.yaml
          k8s/ingress.yaml
        images: |
          ${{ needs.build.outputs.image-tag }}
        imagepullsecrets: |
          acr-secret
    
    - name: Wait for rollout
      run: |
        kubectl rollout status deployment/app-deployment -n ${{ env.NAMESPACE }} --timeout=5m
    
    - name: Get deployment info
      run: |
        kubectl get pods -n ${{ env.NAMESPACE }}
        kubectl get svc -n ${{ env.NAMESPACE }}
        kubectl get ingress -n ${{ env.NAMESPACE }}

  # Job 5: Smoke Tests
  smoke-test:
    name: Smoke Tests
    runs-on: ubuntu-latest
    needs: deploy
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Run smoke tests
      run: |
        # Wait for application to be ready
        sleep 30
        
        # Health check
        curl -f https://myapp.example.com/health || exit 1
        
        # API smoke test
        curl -f https://myapp.example.com/api/status || exit 1
        
        echo "✅ Smoke tests passed"

  # Job 6: Notifications
  notify:
    name: Send Notifications
    runs-on: ubuntu-latest
    needs: [test, build, deploy, smoke-test]
    if: always()
    
    steps:
    - name: Send Slack notification on success
      if: needs.deploy.result == 'success' && needs.smoke-test.result == 'success'
      uses: slackapi/slack-github-action@v1.24.0
      with:
        payload: |
          {
            "text": "✅ Deployment Successful",
            "blocks": [
              {
                "type": "section",
                "text": {
                  "type": "mrkdwn",
                  "text": "*Deployment to Production*\n✅ Status: Success\n🚀 Version: `${{ github.sha }}`\n👤 Author: ${{ github.actor }}\n🔗 <https://myapp.example.com|View Application>"
                }
              }
            ]
          }
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
    
    - name: Send Slack notification on failure
      if: needs.deploy.result == 'failure' || needs.smoke-test.result == 'failure'
      uses: slackapi/slack-github-action@v1.24.0
      with:
        payload: |
          {
            "text": "❌ Deployment Failed",
            "blocks": [
              {
                "type": "section",
                "text": {
                  "type": "mrkdwn",
                  "text": "*Deployment to Production*\n❌ Status: Failed\n🚀 Version: `${{ github.sha }}`\n👤 Author: ${{ github.actor }}\n🔗 <${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}|View Logs>"
                }
              }
            ]
          }
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

### Pattern 5: Terraform Infrastructure

**Generate Terraform for AWS infrastructure:**

```
Chat:

"Create Terraform configuration for:
- VPC with public/private subnets
- ECS Fargate cluster
- Application Load Balancer
- RDS PostgreSQL
- ElastiCache Redis
- S3 bucket for static assets
- CloudWatch logging
Include variables and outputs"
```

**Result:**

```hcl
# variables.tf
variable "project_name" {
  description = "Project name"
  type        = string
  default     = "myapp"
}

variable "environment" {
  description = "Environment (dev, staging, prod)"
  type        = string
  default     = "production"
}

variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

variable "db_instance_class" {
  description = "RDS instance class"
  type        = string
  default     = "db.t3.micro"
}

variable "db_username" {
  description = "Database master username"
  type        = string
  sensitive   = true
}

variable "db_password" {
  description = "Database master password"
  type        = string
  sensitive   = true
}

# main.tf
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  backend "s3" {
    bucket         = "myapp-terraform-state"
    key            = "production/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Project     = var.project_name
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}

# vpc.tf
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
  
  name = "${var.project_name}-${var.environment}-vpc"
  cidr = var.vpc_cidr
  
  azs             = var.availability_zones
  private_subnets = [for k, v in var.availability_zones : cidrsubnet(var.vpc_cidr, 8, k)]
  public_subnets  = [for k, v in var.availability_zones : cidrsubnet(var.vpc_cidr, 8, k + 10)]
  
  enable_nat_gateway   = true
  single_nat_gateway   = false
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  public_subnet_tags = {
    Type = "Public"
  }
  
  private_subnet_tags = {
    Type = "Private"
  }
}

# security-groups.tf
resource "aws_security_group" "alb" {
  name_prefix = "${var.project_name}-${var.environment}-alb-"
  description = "Security group for Application Load Balancer"
  vpc_id      = module.vpc.vpc_id
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "HTTP from anywhere"
  }
  
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "HTTPS from anywhere"
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow all outbound"
  }
  
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_security_group" "ecs_tasks" {
  name_prefix = "${var.project_name}-${var.environment}-ecs-tasks-"
  description = "Security group for ECS tasks"
  vpc_id      = module.vpc.vpc_id
  
  ingress {
    from_port       = 3000
    to_port         = 3000
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
    description     = "Allow from ALB"
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow all outbound"
  }
  
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_security_group" "rds" {
  name_prefix = "${var.project_name}-${var.environment}-rds-"
  description = "Security group for RDS"
  vpc_id      = module.vpc.vpc_id
  
  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.ecs_tasks.id]
    description     = "PostgreSQL from ECS tasks"
  }
  
  lifecycle {
    create_before_destroy = true
  }
}

# alb.tf
resource "aws_lb" "main" {
  name               = "${var.project_name}-${var.environment}-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = module.vpc.public_subnets
  
  enable_deletion_protection = true
  enable_http2              = true
  enable_cross_zone_load_balancing = true
  
  access_logs {
    bucket  = aws_s3_bucket.logs.bucket
    prefix  = "alb"
    enabled = true
  }
}

resource "aws_lb_target_group" "app" {
  name        = "${var.project_name}-${var.environment}-tg"
  port        = 3000
  protocol    = "HTTP"
  vpc_id      = module.vpc.vpc_id
  target_type = "ip"
  
  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200"
    path                = "/health"
    port                = "traffic-port"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 2
  }
  
  deregistration_delay = 30
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.main.arn
  port              = "80"
  protocol          = "HTTP"
  
  default_action {
    type = "redirect"
    
    redirect {
      port        = "443"
      protocol    = "HTTPS"
      status_code = "HTTP_301"
    }
  }
}

resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.main.arn
  port              = "443"
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS13-1-2-2021-06"
  certificate_arn   = aws_acm_certificate.main.arn
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}

# rds.tf
resource "aws_db_subnet_group" "main" {
  name       = "${var.project_name}-${var.environment}-db-subnet"
  subnet_ids = module.vpc.private_subnets
}

resource "aws_db_instance" "postgresql" {
  identifier = "${var.project_name}-${var.environment}-db"
  
  engine         = "postgres"
  engine_version = "16.1"
  instance_class = var.db_instance_class
  
  allocated_storage     = 20
  max_allocated_storage = 100
  storage_type          = "gp3"
  storage_encrypted     = true
  
  db_name  = replace(var.project_name, "-", "_")
  username = var.db_username
  password = var.db_password
  
  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.rds.id]
  
  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "mon:04:00-mon:05:00"
  
  enabled_cloudwatch_logs_exports = ["postgresql", "upgrade"]
  
  skip_final_snapshot       = false
  final_snapshot_identifier = "${var.project_name}-${var.environment}-final-${formatdate("YYYYMMDDhhmmss", timestamp())}"
  
  deletion_protection = true
  
  tags = {
    Name = "${var.project_name}-${var.environment}-postgresql"
  }
}

# outputs.tf
output "vpc_id" {
  description = "VPC ID"
  value       = module.vpc.vpc_id
}

output "alb_dns_name" {
  description = "ALB DNS name"
  value       = aws_lb.main.dns_name
}

output "db_endpoint" {
  description = "RDS endpoint"
  value       = aws_db_instance.postgresql.endpoint
  sensitive   = true
}

output "ecs_cluster_name" {
  description = "ECS cluster name"
  value       = aws_ecs_cluster.main.name
}
```

---

## 💡 Pro Tips

### 1. **Helm Chart Generation**

```
Chat:

"Create Helm chart for Node.js application with:
- Configurable replicas
- Secret management
- Ingress configuration
- ServiceMonitor for Prometheus
- Values for dev/staging/prod"
```

### 2. **Shell Script Automation**

```
Chat:

"Create bash script for zero-downtime deployment:
1. Build Docker image
2. Push to registry
3. Update Kubernetes deployment
4. Wait for rollout
5. Run health checks
6. Rollback if checks fail
Include logging and error handling"
```

### 3. **CloudFormation Template**

```
Chat:

"Generate CloudFormation template for serverless API:
- API Gateway
- Lambda functions (Node.js)
- DynamoDB table
- CloudWatch alarms
- IAM roles with least privilege"
```

---

## 🚨 Common Pitfalls

### ❌ Don't:
- ❌ Run containers as root user
- ❌ Store secrets in code or Docker images
- ❌ Skip health checks and readiness probes
- ❌ Use `latest` tag in production
- ❌ Forget resource limits on containers
- ❌ Deploy without rollback strategy

### ✅ Do:
- ✅ Use multi-stage Docker builds
- ✅ Implement proper health checks
- ✅ Use specific image tags or digests
- ✅ Set resource requests and limits
- ✅ Scan images for vulnerabilities
- ✅ Test deployments in staging first

---

## 📚 Related Resources

**Official Documentation:**
- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Docs](https://kubernetes.io/docs/)
- [GitHub Actions](https://docs.github.com/actions)
- [Terraform Documentation](https://www.terraform.io/docs)
- [AWS CDK Guide](https://docs.aws.amazon.com/cdk/)
- [Helm Documentation](https://helm.sh/docs/)

**Copilot Features Used:**
- Inline completions for YAML/HCL
- Chat Agent mode for multi-file infrastructure
- Custom instructions for deployment standards
- Slash commands for Dockerfile generation

---

## 🎯 Next Steps

- **Return to:** [Part 5 Overview](README.md)
- **Related:** [Backend Development](02-backend-development.md)
- **Practice:** Deploy full-stack app to Kubernetes
- **Experiment:** Create custom DevOps agent for deployments

---

**Updated:** December 2025  
**Part 5, Section 4 of 4**

