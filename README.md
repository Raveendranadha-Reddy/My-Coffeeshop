# MyCoffeeShop — Consolidated Project Folder

This folder consolidates the MyCoffeeShop / Coffee Shop EKS project records found in prior ChatGPT work. It preserves the available project documentation and troubleshooting notes, rather than claiming that missing application source files have been recovered.

## Current recorded implementation

- Frontend: static HTML/CSS/JavaScript served by NGINX.
- Backend: Java 17, Spring Boot 3.5.5, Maven, REST API, port 8080.
- Database: MySQL 8.0.
- Platform: AWS EKS, Kubernetes namespace `coffeeshop` in documented manifests.
- Storage: Amazon EBS CSI Driver, `gp3` StorageClass, PVC for MySQL.
- Frontend image recorded: `raveendradudyala/coffee-shop-frontend:3.0.1`.
- Backend image recorded: `raveendradudyala/coffee-shop-backend:5.0.0`.
- Database image recorded: `mysql:8.0`.
- Frontend Service recorded: `coffee-frontend`, NodePort `30080`.
- Backend Service recorded: `coffee-backend`, ClusterIP port `8080`.
- NGINX proxies `/api/` requests to `http://coffee-backend:8080/api/`.
- MySQL persistent storage uses a PVC and AWS EBS volume.
- A previous build/run attempt reported `Port 8080 already in use`.
- A documented EKS context example is `eksdemo1` in `us-east-1`; verify the current cluster before running commands.

## Folder contents

```text
MyCoffeeShop_Project/
├── README.md
├── docs/
│   ├── Raveendra_Coffee_Shop_EKS_End_to_End.md
│   ├── eks-coffeeshop-rebuild.md
│   ├── TROUBLESHOOTING.md
│   └── Coffee_Shop_EKS_End_to_End_Project_Documentation.pdf
└── source-records/
    └── MyCoffeeShop_Project_Summary.md  (if present in the source workspace)
```

## Main features documented

- Coffee Shop menu and product UI
- Cart and checkout flow
- Order creation and order history UI
- Backend REST API, including `GET /api/orders` for order history
- MySQL persistence for orders
- NGINX reverse proxy to the backend Service
- Docker image build and push workflow
- EKS deployment using Kubernetes Deployments and Services
- EBS-backed persistent database storage
- EKS Pod Identity and EBS CSI Driver setup notes
- Troubleshooting for PVC binding, MySQL connectivity, frontend/backend connectivity, and public NodePort access
- AWS infrastructure and Terraform/Jenkins CI/CD plans documented in the end-to-end guide

## Important: what is and is not included

Included: previously saved project guides, rebuild runbook, troubleshooting guide, and PDF documentation.

Not recovered as standalone source files in this consolidation: the full frontend source (`index.html`, CSS, JS), the full Spring Boot source tree (`pom.xml`, Java classes, `application.properties`), Dockerfiles, exact Kubernetes manifest files, Terraform `.tf` files, and Jenkinsfile. Existing documentation describes several of these files and configurations, but the complete original source tree was not present in the retrieved records. Do not treat this documentation bundle as a verified, directly deployable source-code repository.

## Important configuration notes

- Never commit real passwords, cloud access keys, or populated Kubernetes Secrets to GitHub.
- Check image tags and namespaces before applying manifests; the records contain examples from different stages of the project.
- The documented frontend image tag is `3.0.1`; backend image tag is `5.0.0`.
- Validate the actual running state with `kubectl get all -n coffeeshop`, `kubectl get pvc -n coffeeshop`, and `kubectl get endpoints -n coffeeshop`.
- Public NodePort access requires a reachable node IP and an appropriate AWS security-group rule; NodePort alone does not guarantee internet access.

## Original repository reference

https://github.com/REDDYRN26/AWS-EKS/tree/main

The repository root inspected for this consolidation did not show a `MyCoffeeShop` directory at the top level, so the source files were not copied from that repository. The documentation files below are preserved from prior ChatGPT project artifacts.
