# Raveendra Coffee Shop – End-to-End AWS EKS Kubernetes Project

## 1. Project Overview

This project demonstrates a complete production-style deployment of a Coffee Shop application on AWS EKS.

Application components:

- Frontend: HTML/CSS/JavaScript served through NGINX
- Backend: Java Spring Boot application
- Database: MySQL
- Containerization: Docker
- Container Registry: Amazon ECR
- Kubernetes Platform: Amazon EKS
- Infrastructure: AWS VPC, EC2, IAM, EKS
- Kubernetes resources: Namespace, ConfigMap, Secret, Deployment, Service, StorageClass, PV/PVC
- Storage: Amazon EBS through EBS CSI Driver
- Configuration: Kubernetes ConfigMap and Secret
- Networking: Kubernetes Service and NGINX
- CI/CD: Git, Jenkins/GitHub Actions, Docker, ECR, Kubernetes/GitOps
- Infrastructure as Code: Terraform
- Monitoring/troubleshooting: kubectl, logs, describe, events

## 2. End-to-End Architecture

Developer
   |
   v
GitHub Repository
   |
   v
Source Code
   |
   +-------------------+
   |                   |
   v                   v
Frontend            Backend
NGINX               Spring Boot
   |                   |
   |                   v
   |                MySQL
   |                   |
   |                   v
   |              EBS Volume
   |                   |
   +-------- Kubernetes/EKS --------+
                    |
                 AWS EKS
                    |
              Worker Nodes
                    |
                 AWS VPC
                    |
             Internet / Users

Complete deployment flow:

Code
 -> GitHub
 -> Build
 -> Test
 -> Docker Image
 -> Amazon ECR
 -> Kubernetes Deployment
 -> EKS Pods
 -> Services
 -> Frontend
 -> Backend
 -> MySQL
 -> Persistent EBS Storage

## 3. Recommended Repository Structure

```text
AWS-EKS/
|
+-- coffee-shop-frontend/
|   +-- index.html
|   +-- css/
|   +-- images/
|   +-- Dockerfile
|   +-- nginx.conf
|
+-- coffee-shop-backend/
|   +-- src/
|   +-- pom.xml
|   +-- Dockerfile
|
+-- k8s/
|   +-- 00-namespace.yaml
|   +-- 01-secret.yaml
|   +-- 02-configmap.yaml
|   +-- 03-storageclass.yaml
|   +-- 04-mysql-pvc.yaml
|   +-- 05-mysql-deployment.yaml
|   +-- 06-mysql-service.yaml
|   +-- 07-backend-configmap.yaml
|   +-- 08-backend-deployment.yaml
|   +-- 09-backend-service.yaml
|   +-- 10-nginx-configmap.yaml
|   +-- 11-frontend-deployment.yaml
|   +-- 12-frontend-service.yaml
|   +-- 13-ingress.yaml
|
+-- terraform/
|   +-- providers.tf
|   +-- variables.tf
|   +-- vpc.tf
|   +-- eks.tf
|   +-- outputs.tf
|
+-- Jenkinsfile
+-- README.md
```

## 4. AWS Infrastructure

Required AWS components:

1. AWS Account
2. IAM
3. VPC
4. Public subnets
5. Private subnets
6. Internet Gateway
7. NAT Gateway
8. Route Tables
9. Security Groups
10. EKS Cluster
11. Managed Node Group or EC2 worker nodes
12. ECR repositories
13. EBS volumes
14. IAM OIDC provider
15. EBS CSI Driver

Recommended EKS design:

```text
AWS Region
 |
 +-- Availability Zone A
 |    +-- Public Subnet
 |    +-- Private Subnet
 |
 +-- Availability Zone B
      +-- Public Subnet
      +-- Private Subnet

EKS Control Plane
       |
       +-- Worker Node A
       |
       +-- Worker Node B
```

## 5. GitHub

Create the repository and push:

- Frontend source code
- Backend source code
- Dockerfiles
- Kubernetes YAML files
- Terraform files
- Jenkinsfile
- Documentation

Typical commands:

```bash
git init
git add .
git commit -m "Initial Coffee Shop project"
git branch -M main
git remote add origin <GITHUB_REPOSITORY_URL>
git push -u origin main
```

## 6. Frontend Application

Frontend is served by NGINX.

Responsibilities:

- Display Coffee Shop UI
- Home page
- Menu
- Orders
- Coffee products
- Images
- Backend API calls

NGINX serves files from:

```text
/usr/share/nginx/html
```

NGINX listens on:

```text
Port 80
```

## 7. Backend Application

Backend is a Java Spring Boot application.

Example technology stack:

```text
Java 17
Spring Boot
Maven
REST API
MySQL
```

Build:

```bash
mvn clean package
```

Expected artifact:

```text
coffee-shop-backend-1.0.0.jar
```

Run locally:

```bash
java -jar coffee-shop-backend-1.0.0.jar
```

Default application port:

```text
8080
```

Important troubleshooting:

If:

```text
Port 8080 was already in use
```

check:

```bash
ss -lntp | grep 8080
```

or:

```bash
lsof -i :8080
```

Then stop the conflicting process or configure another port.

## 8. Docker – Frontend

Example Dockerfile:

```dockerfile
FROM nginx:alpine

COPY . /usr/share/nginx/html/

EXPOSE 80
```

Build:

```bash
docker build -t coffee-shop-frontend:1.0 .
```

Run:

```bash
docker run -d -p 8080:80 coffee-shop-frontend:1.0
```

Test:

```bash
curl http://localhost:8080
```

## 9. Docker – Backend

Example:

```dockerfile
FROM eclipse-temurin:17-jre

WORKDIR /app

COPY target/coffee-shop-backend-1.0.0.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java","-jar","app.jar"]
```

Build:

```bash
mvn clean package
docker build -t coffee-shop-backend:1.0 .
```

Run:

```bash
docker run -d -p 8080:8080 coffee-shop-backend:1.0
```

## 10. Amazon ECR

Create repositories:

```text
coffee-shop-frontend
coffee-shop-backend
```

Authenticate Docker:

```bash
aws ecr get-login-password --region us-east-1 \
| docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
```

Tag:

```bash
docker tag coffee-shop-frontend:1.0 \
<AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/coffee-shop-frontend:1.0

docker tag coffee-shop-backend:1.0 \
<AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/coffee-shop-backend:1.0
```

Push:

```bash
docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/coffee-shop-frontend:1.0

docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/coffee-shop-backend:1.0
```

## 11. EKS Cluster

Example:

```bash
eksctl create cluster \
  --name eksdemo1 \
  --region us-east-1 \
  --zones us-east-1a,us-east-1b
```

Verify:

```bash
eksctl get cluster
kubectl get nodes
kubectl get pods -A
```

Expected:

```text
kubectl get nodes
```

should show worker nodes in Ready state.

## 12. IAM OIDC

Associate OIDC:

```bash
eksctl utils associate-iam-oidc-provider \
  --region us-east-1 \
  --cluster eksdemo1 \
  --approve
```

OIDC is required for IAM roles used by Kubernetes workloads.

Important example:

```text
EKS Pod
 |
 +-- Kubernetes ServiceAccount
       |
       +-- IAM Role
             |
             +-- AWS permissions
```

## 13. Kubernetes Namespace

Create:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: coffeeshop
```

Apply:

```bash
kubectl apply -f 00-namespace.yaml
```

Verify:

```bash
kubectl get namespace
```

## 14. Kubernetes Secret

Store sensitive database information.

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: coffeeshop
type: Opaque
stringData:
  MYSQL_ROOT_PASSWORD: rootpassword
  MYSQL_DATABASE: coffeeshop
  MYSQL_USER: coffeeuser
  MYSQL_PASSWORD: coffeepassword
```

Apply:

```bash
kubectl apply -f 01-secret.yaml
```

Verify:

```bash
kubectl get secret -n coffeeshop
```

Do not commit real production passwords to GitHub.

## 15. ConfigMap

ConfigMap stores non-sensitive configuration.

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: backend-config
  namespace: coffeeshop
data:
  DB_HOST: mysql
  DB_PORT: "3306"
  DB_NAME: coffeeshop
```

Apply:

```bash
kubectl apply -f 02-configmap.yaml
```

## 16. StorageClass

For AWS EBS:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
parameters:
  type: gp3
```

Apply:

```bash
kubectl apply -f 03-storageclass.yaml
```

Important fields:

- apiVersion: Kubernetes API version
- kind: resource type
- metadata.name: resource name
- provisioner: EBS CSI driver
- volumeBindingMode: delays volume creation until pod scheduling
- reclaimPolicy: controls what happens when PVC is deleted
- type: gp3 EBS volume

## 17. MySQL PVC

Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  namespace: coffeeshop
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ebs-sc
  resources:
    requests:
      storage: 10Gi
```

Apply:

```bash
kubectl apply -f 04-mysql-pvc.yaml
```

Verify:

```bash
kubectl get pvc -n coffeeshop
kubectl get pv
```

Flow:

```text
PVC
 |
 v
StorageClass
 |
 v
EBS CSI Driver
 |
 v
AWS EBS Volume
 |
 v
PersistentVolume
 |
 v
Pod
```

## 18. MySQL Deployment

MySQL requires persistent storage.

Example structure:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: coffeeshop
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_DATABASE
          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-storage
          persistentVolumeClaim:
            claimName: mysql-pvc
```

Apply:

```bash
kubectl apply -f 05-mysql-deployment.yaml
```

Verify:

```bash
kubectl get pods -n coffeeshop
kubectl logs -n coffeeshop deployment/mysql
```

## 19. MySQL Service

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: coffeeshop
spec:
  selector:
    app: mysql
  ports:
    - port: 3306
      targetPort: 3306
```

Apply:

```bash
kubectl apply -f 06-mysql-service.yaml
```

The backend can connect using:

```text
mysql:3306
```

Kubernetes DNS resolves the service name.

## 20. Backend Deployment

Backend receives database configuration from ConfigMap and Secret.

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coffee-backend
  namespace: coffeeshop
spec:
  replicas: 2
  selector:
    matchLabels:
      app: coffee-backend
  template:
    metadata:
      labels:
        app: coffee-backend
    spec:
      containers:
        - name: backend
          image: <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/coffee-shop-backend:1.0
          ports:
            - containerPort: 8080
          env:
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  name: backend-config
                  key: DB_HOST
            - name: DB_PORT
              valueFrom:
                configMapKeyRef:
                  name: backend-config
                  key: DB_PORT
            - name: DB_NAME
              valueFrom:
                configMapKeyRef:
                  name: backend-config
                  key: DB_NAME
            - name: DB_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_USER
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_PASSWORD
```

Apply:

```bash
kubectl apply -f 08-backend-deployment.yaml
```

Verify:

```bash
kubectl get pods -n coffeeshop
kubectl logs -n coffeeshop deployment/coffee-backend
```

## 21. Backend Service

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: coffee-backend
  namespace: coffeeshop
spec:
  selector:
    app: coffee-backend
  ports:
    - port: 8080
      targetPort: 8080
```

Apply:

```bash
kubectl apply -f 09-backend-service.yaml
```

Check:

```bash
kubectl get svc -n coffeeshop
```

## 22. NGINX ConfigMap

NGINX can proxy frontend API requests to backend.

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-nginx-config
  namespace: coffeeshop
data:
  default.conf: |
    server {
        listen 80;
        server_name _;

        root /usr/share/nginx/html;
        index index.html;

        location / {
            try_files $uri $uri/ /index.html;
        }

        location /api/ {
            proxy_pass http://coffee-backend:8080/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
```

Apply:

```bash
kubectl apply -f 10-nginx-configmap.yaml
```

## 23. Frontend Deployment

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coffee-frontend
  namespace: coffeeshop
spec:
  replicas: 2
  selector:
    matchLabels:
      app: coffee-frontend
  template:
    metadata:
      labels:
        app: coffee-frontend
    spec:
      containers:
        - name: frontend
          image: <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/coffee-shop-frontend:1.0
          ports:
            - containerPort: 80
          volumeMounts:
            - name: nginx-config
              mountPath: /etc/nginx/conf.d/default.conf
              subPath: default.conf
      volumes:
        - name: nginx-config
          configMap:
            name: frontend-nginx-config
```

Apply:

```bash
kubectl apply -f 11-frontend-deployment.yaml
```

## 24. Frontend Service

For lab access, NodePort can be used.

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: coffee-frontend
  namespace: coffeeshop
spec:
  type: NodePort
  selector:
    app: coffee-frontend
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Apply:

```bash
kubectl apply -f 12-frontend-service.yaml
```

Verify:

```bash
kubectl get svc -n coffeeshop
```

Expected:

```text
coffee-frontend   NodePort   ...   80:30080/TCP
```

## 25. Application Connectivity

Request flow:

```text
Browser
  |
  v
Frontend Service
  |
  v
Frontend Pod
  |
  v
NGINX
  |
  | /api/
  v
Backend Service
  |
  v
Backend Pod
  |
  v
MySQL Service
  |
  v
MySQL Pod
  |
  v
PVC
  |
  v
EBS Volume
```

## 26. Deployment Order

Recommended order:

```text
1. AWS Infrastructure
2. EKS Cluster
3. OIDC
4. EBS CSI Driver
5. Namespace
6. Secret
7. ConfigMap
8. StorageClass
9. PVC
10. MySQL
11. MySQL Service
12. Backend
13. Backend Service
14. NGINX ConfigMap
15. Frontend
16. Frontend Service
17. Ingress / Load Balancer
18. Testing
19. Monitoring
```

## 27. Basic Verification

Check nodes:

```bash
kubectl get nodes
```

Check all application resources:

```bash
kubectl get all -n coffeeshop
```

Check PVC:

```bash
kubectl get pvc -n coffeeshop
```

Check PV:

```bash
kubectl get pv
```

Check services:

```bash
kubectl get svc -n coffeeshop
```

Check endpoints:

```bash
kubectl get endpoints -n coffeeshop
```

Check pods:

```bash
kubectl get pods -n coffeeshop -o wide
```

Check events:

```bash
kubectl get events -n coffeeshop --sort-by=.lastTimestamp
```

## 28. Application Testing

Frontend:

```bash
curl http://<NODE-IP>:30080
```

From inside a test pod:

```bash
kubectl run curl-test \
  --image=curlimages/curl \
  -it --rm \
  -- sh
```

Test backend:

```bash
curl http://coffee-backend:8080
```

Test MySQL DNS:

```bash
nslookup mysql
```

Test MySQL port:

```bash
nc -zv mysql 3306
```

## 29. Important Kubernetes Troubleshooting

### Pod Pending

Commands:

```bash
kubectl get pod -n coffeeshop
kubectl describe pod <pod-name> -n coffeeshop
kubectl get events -n coffeeshop
```

Common causes:

- Insufficient CPU/memory
- PVC pending
- Node selector mismatch
- Taints/tolerations
- Image pull issue
- Volume scheduling issue

### CrashLoopBackOff

Commands:

```bash
kubectl get pod -n coffeeshop
kubectl logs <pod-name> -n coffeeshop
kubectl logs <pod-name> -n coffeeshop --previous
kubectl describe pod <pod-name> -n coffeeshop
```

Common causes:

- Application startup failure
- Wrong environment variables
- Database unavailable
- Wrong credentials
- Port conflict
- Missing configuration
- Failed health probe

### ImagePullBackOff

Check:

```bash
kubectl describe pod <pod-name> -n coffeeshop
```

Common causes:

- Wrong ECR image
- Wrong image tag
- ECR authentication
- IAM permissions
- Image does not exist

### Service Not Working

Check:

```bash
kubectl get svc -n coffeeshop
kubectl get endpoints -n coffeeshop
kubectl get pods -n coffeeshop --show-labels
```

Most common issue:

Service selector does not match pod labels.

### Backend Cannot Connect to MySQL

Check:

```bash
kubectl get svc mysql -n coffeeshop
kubectl get endpoints mysql -n coffeeshop
kubectl get pods -n coffeeshop
```

Test DNS:

```bash
nslookup mysql
```

Test port:

```bash
nc -zv mysql 3306
```

Check backend logs:

```bash
kubectl logs deployment/coffee-backend -n coffeeshop
```

### PVC Pending

Check:

```bash
kubectl get pvc -n coffeeshop
kubectl describe pvc mysql-pvc -n coffeeshop
kubectl get storageclass
```

Check EBS CSI:

```bash
kubectl get pods -n kube-system | grep ebs
```

## 30. Health Probes

Production deployment should use:

- startupProbe
- readinessProbe
- livenessProbe

Example:

```yaml
startupProbe:
  httpGet:
    path: /actuator/health
    port: 8080
  failureThreshold: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /actuator/health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10

livenessProbe:
  httpGet:
    path: /actuator/health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 20
```

Purpose:

- startupProbe: application startup
- readinessProbe: whether pod can receive traffic
- livenessProbe: whether container is healthy

## 31. Resource Requests and Limits

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

Requests are used for scheduling.

Limits define maximum resource usage.

## 32. Horizontal Scaling

Frontend:

```bash
kubectl scale deployment coffee-frontend \
  --replicas=3 \
  -n coffeeshop
```

Backend:

```bash
kubectl scale deployment coffee-backend \
  --replicas=3 \
  -n coffeeshop
```

Verify:

```bash
kubectl get pods -n coffeeshop
```

For production, use HPA:

```text
HPA
 |
 +-- CPU/Memory Metrics
 |
 +-- Deployment
 |
 +-- More/Fewer Pods
```

## 33. Ingress / Load Balancer

For production access, use an AWS Load Balancer rather than NodePort.

Recommended:

```text
Internet
   |
   v
AWS ALB
   |
   v
Kubernetes Ingress
   |
   v
Frontend Service
   |
   v
Frontend Pods
```

For EKS, AWS Load Balancer Controller can manage ALB/NLB resources.

## 34. HTTPS

Production application should use HTTPS.

Typical flow:

```text
Route 53
   |
   v
ALB
   |
   v
ACM Certificate
   |
   v
HTTPS :443
   |
   v
Kubernetes Ingress
```

## 35. CI/CD Pipeline

Recommended pipeline:

```text
Developer
   |
   v
Git Push
   |
   v
GitHub
   |
   v
Jenkins / GitHub Actions
   |
   +--> Maven Build
   |
   +--> Unit Test
   |
   +--> SonarQube
   |
   +--> Docker Build
   |
   +--> Security Scan
   |
   +--> Push Image to ECR
   |
   v
Update Kubernetes Manifest
   |
   v
Argo CD
   |
   v
EKS
```

## 36. Jenkins Pipeline Stages

Typical stages:

```text
Checkout
   |
Build
   |
Test
   |
SonarQube
   |
Docker Build
   |
Docker Scan
   |
ECR Login
   |
Docker Push
   |
Deploy
   |
Verify
```

Example commands:

```bash
mvn clean package
```

```bash
docker build -t coffee-shop-backend:${BUILD_NUMBER} .
```

```bash
docker push <ECR_REPOSITORY>:${BUILD_NUMBER}
```

## 37. GitOps with Argo CD

Recommended GitOps flow:

```text
GitHub
   |
   v
Kubernetes Manifests
   |
   v
Argo CD
   |
   v
EKS
```

Argo CD continuously compares:

```text
Desired State
     vs
Live State
```

If they differ, Argo CD can synchronize the cluster.

## 38. Terraform

Terraform can create:

- VPC
- Subnets
- Route tables
- NAT Gateway
- IAM
- EKS
- Node Groups
- Security Groups

Typical commands:

```bash
terraform init
terraform validate
terraform plan
terraform apply
```

Destroy lab infrastructure:

```bash
terraform destroy
```

## 39. Security

Important security practices:

- Do not use AWS root account for daily work.
- Use IAM roles.
- Use least privilege.
- Do not store passwords in Git.
- Use Kubernetes Secrets.
- Use ECR private repositories.
- Enable EKS audit logging where appropriate.
- Use private worker nodes for production.
- Restrict Security Groups.
- Use HTTPS.
- Scan container images.
- Patch base images.
- Use non-root containers where possible.
- Apply Kubernetes RBAC.

## 40. Monitoring

Recommended stack:

```text
Prometheus
Grafana
CloudWatch
Fluent Bit
AWS Container Insights
```

Monitor:

- CPU
- Memory
- Pod restarts
- Node health
- Application latency
- HTTP errors
- Disk usage
- EBS volume status
- Kubernetes events

## 41. Logging

Useful commands:

```bash
kubectl logs <pod> -n coffeeshop
```

Follow logs:

```bash
kubectl logs -f <pod> -n coffeeshop
```

Previous container logs:

```bash
kubectl logs <pod> -n coffeeshop --previous
```

Multiple containers:

```bash
kubectl logs <pod> -c <container> -n coffeeshop
```

## 42. Production Improvements

After the basic lab works, implement:

1. ALB Ingress
2. HTTPS with ACM
3. Route 53 DNS
4. HPA
5. Cluster Autoscaler or Karpenter
6. PodDisruptionBudget
7. NetworkPolicy
8. RBAC
9. Secrets Manager integration
10. External Secrets
11. Prometheus
12. Grafana
13. CloudWatch
14. Fluent Bit
15. Argo CD
16. Image vulnerability scanning
17. Resource requests/limits
18. Pod security controls
19. Multi-AZ deployment
20. Backup and restore strategy

## 43. Final Application Validation

Run:

```bash
kubectl get nodes
kubectl get ns
kubectl get all -n coffeeshop
kubectl get pvc -n coffeeshop
kubectl get pv
kubectl get storageclass
```

Then:

```bash
kubectl get pods -n coffeeshop -o wide
```

All application pods should be:

```text
Running
```

Services should have endpoints:

```bash
kubectl get endpoints -n coffeeshop
```

Frontend should be reachable:

```bash
curl http://<NODE-IP>:30080
```

Backend should communicate with MySQL.

MySQL data should persist after pod restart.

## 44. Complete Troubleshooting Decision Tree

Start:

```text
Application not accessible
        |
        v
Is frontend Pod Running?
        |
    +---+---+
    |       |
   NO      YES
    |       |
logs/      Check Service
describe       |
              v
       Endpoints available?
          |
       +--+--+
       |     |
      NO    YES
       |     |
 selector   Check NodePort/
 labels     LoadBalancer/Ingress
             |
             v
       Frontend working?
             |
          +--+--+
          |     |
         YES    NO
          |     |
          |   Check NGINX
          |   ConfigMap
          |
          v
      Test Backend
          |
          v
   Backend Pod Running?
          |
       +--+--+
       |     |
      NO    YES
       |     |
    logs/   Test API
    describe   |
              v
       MySQL reachable?
          |
       +--+--+
       |     |
      NO    YES
       |     |
    DNS/Service   Check
    /credentials  application
                  logic
```

## 45. Final End-to-End Flow

```text
USER
 |
 | HTTPS
 v
Route 53
 |
 v
AWS ALB
 |
 v
Kubernetes Ingress
 |
 v
Frontend Service
 |
 v
Frontend Pods
 |
 | /api
 v
Backend Service
 |
 v
Backend Pods
 |
 | JDBC
 v
MySQL Service
 |
 v
MySQL Pod
 |
 v
PVC
 |
 v
EBS Volume
```

Infrastructure flow:

```text
Terraform
   |
   v
AWS VPC
   |
   v
EKS
   |
   v
Worker Nodes
   |
   v
Kubernetes
```

CI/CD flow:

```text
Developer
   |
   v
GitHub
   |
   v
Jenkins / GitHub Actions
   |
   +--> Maven
   +--> Tests
   +--> SonarQube
   +--> Docker
   +--> Security Scan
   |
   v
Amazon ECR
   |
   v
Argo CD / kubectl
   |
   v
Amazon EKS
```

## 46. Project Skills Demonstrated

This project demonstrates practical knowledge of:

- Linux
- Git
- GitHub
- Maven
- Java
- Spring Boot
- NGINX
- Docker
- Amazon ECR
- AWS VPC
- IAM
- EC2
- EBS
- EKS
- Kubernetes
- Kubernetes Networking
- Kubernetes Storage
- ConfigMap
- Secret
- Deployment
- Service
- PVC
- PV
- StorageClass
- CSI
- Probes
- Scaling
- Ingress
- AWS Load Balancer Controller
- Terraform
- Jenkins
- SonarQube
- Argo CD
- GitOps
- Monitoring
- Logging
- Troubleshooting
- Production deployment practices

## 47. Resume Project Description

Project: Coffee Shop Application – AWS EKS

Designed and deployed a containerized Coffee Shop application on Amazon EKS using a Java Spring Boot backend, NGINX frontend, and MySQL database. Built Docker images and published them to Amazon ECR. Implemented Kubernetes Deployments, Services, ConfigMaps, Secrets, PersistentVolumeClaims, StorageClass, and AWS EBS CSI-based persistent storage. Configured application-to-database communication through Kubernetes Services and DNS. Implemented AWS infrastructure using Terraform and established CI/CD/GitOps deployment patterns using Jenkins, ECR, and Argo CD. Performed Kubernetes troubleshooting for pod failures, CrashLoopBackOff, Pending pods, image pull failures, service connectivity, DNS, storage, and application startup issues.

## 48. Interview Explanation – 1 Minute

"I worked on an end-to-end Coffee Shop application deployed on AWS EKS. The application has an NGINX-based frontend, Java Spring Boot backend, and MySQL database. I containerized the frontend and backend using Docker and pushed the images to Amazon ECR. On EKS, I deployed the application using Kubernetes Deployments and Services, used ConfigMaps for non-sensitive configuration and Secrets for database credentials, and configured persistent MySQL storage using a PVC backed by AWS EBS through the EBS CSI driver. I also worked with AWS VPC, IAM, EKS, Kubernetes networking, storage, troubleshooting, and CI/CD concepts using Git, Jenkins, ECR and Argo CD. I validated the complete flow from the user request through the frontend, backend, MySQL service, PVC, and EBS storage."

## 49. Final Checklist

AWS:

- [ ] IAM configured
- [ ] VPC created
- [ ] Subnets created
- [ ] Route tables configured
- [ ] NAT Gateway configured where required
- [ ] EKS created
- [ ] Worker nodes Ready
- [ ] ECR repositories created
- [ ] OIDC configured
- [ ] EBS CSI Driver installed

Application:

- [ ] Frontend code complete
- [ ] Backend code complete
- [ ] MySQL configuration complete
- [ ] Frontend Docker image built
- [ ] Backend Docker image built
- [ ] Images pushed to ECR

Kubernetes:

- [ ] Namespace
- [ ] Secret
- [ ] ConfigMap
- [ ] StorageClass
- [ ] PVC
- [ ] MySQL Deployment
- [ ] MySQL Service
- [ ] Backend Deployment
- [ ] Backend Service
- [ ] NGINX ConfigMap
- [ ] Frontend Deployment
- [ ] Frontend Service
- [ ] Ingress/ALB for production

CI/CD:

- [ ] GitHub
- [ ] Jenkins/GitHub Actions
- [ ] Maven build
- [ ] Unit tests
- [ ] SonarQube
- [ ] Docker build
- [ ] Image scan
- [ ] ECR push
- [ ] Kubernetes deployment
- [ ] Argo CD/GitOps

Validation:

- [ ] Frontend accessible
- [ ] Backend API working
- [ ] Backend-to-MySQL connectivity working
- [ ] PVC Bound
- [ ] EBS volume attached
- [ ] Data persists after MySQL pod restart
- [ ] Logs checked
- [ ] Events checked
- [ ] No CrashLoopBackOff
- [ ] No ImagePullBackOff
- [ ] No Pending application pods
