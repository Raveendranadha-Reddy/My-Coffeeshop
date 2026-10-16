# MyCoffeeShop Project — Current Work Summary

## 1. Project Goal

Organize all MyCoffeeShop work completed so far into one project folder. Keep the frontend, backend, database, Docker, Kubernetes/EKS manifests, CI/CD notes, and documentation in clearly separated directories.

Repository previously referenced:
- https://github.com/REDDYRN26/AWS-EKS/tree/main

## 2. Known Project Details

The following details were shared during earlier troubleshooting:

- Frontend: React application served using NGINX.
- Frontend Kubernetes Service: `coffee-frontend`
- Service type: `NodePort`
- NodePort: `30080`
- ClusterIP observed: `10.100.177.42`
- Frontend endpoints observed: `192.168.63.11:80`, `192.168.10.88:80`
- Backend: Java Spring Boot application.
- Java: OpenJDK 17.0.20.1
- Maven: 3.6.3 (Red Hat 3.6.3-23)
- Spring Boot: 3.5.5
- Backend artifact observed: `coffee-shop-backend-1.0.0.jar` (approximately 24 MB)
- A backend startup attempt reported: `Port 8080 already in use`.
- Kubernetes testing used `curlimages/curl`.
- Worker nodes were described as Amazon Linux / RHEL 9.

These are previously observed values, not a guarantee that the current environment still has the same configuration. Confirm them against the actual files and cluster before using them.

## 3. Proposed Folder Structure

```text
MyCoffeeShop/
├── README.md
├── frontend/
│   ├── src/
│   ├── public/
│   ├── package.json
│   ├── Dockerfile
│   └── nginx.conf
├── backend/
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
├── database/
│   ├── schema.sql
│   └── sample-data.sql
├── docker/
│   └── docker-compose.yml
├── kubernetes/
│   ├── namespace.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── database-deployment-or-statefulset.yaml
│   ├── database-service.yaml
│   ├── pvc.yaml
│   └── ingress-or-loadbalancer.yaml
├── terraform/
│   └── README.md
├── ci-cd/
│   └── README.md
└── docs/
    ├── architecture.md
    ├── installation.md
    ├── deployment.md
    └── troubleshooting.md
```

Important:
- This is a proposed structure, not a record that all these files already exist.
- Keep only the database and Kubernetes resources that match the actual application.
- Do not commit passwords, access keys, Kubernetes secrets containing real credentials, `.env` files, `node_modules/`, build output, or large generated JAR files unless there is a specific reason.
- Add a `.gitignore` appropriate for React, Java/Maven, Docker, and Terraform.

## 4. Organizing the Existing Work

1. Locate the current MyCoffeeShop project directory.
2. Identify the existing frontend, backend, Dockerfiles, Kubernetes YAML files, database files, and scripts.
3. Move or copy files into the proposed folders without overwriting working versions.
4. Check frontend API URLs and backend service names so they match the Kubernetes configuration.
5. Check image names and tags in Deployment manifests.
6. Validate YAML files before deploying.
7. Test the frontend, backend, and database connectivity.
8. Document the commands that actually work in `README.md` and `docs/`.

## 5. Useful Validation Commands

Run these from the appropriate project or cluster environment:

```bash
# Inspect project files
find . -maxdepth 3 -type f | sort

# Check Kubernetes resources
kubectl get all -A

# Inspect frontend Service
kubectl get svc coffee-frontend -o wide

# Inspect application pods
kubectl get pods -A -o wide

# Check backend logs (replace namespace and pod name as needed)
kubectl logs -n <namespace> <backend-pod-name>

# Validate a manifest by asking the API server to perform a dry run
kubectl apply --dry-run=server -f kubernetes/
```

The final command assumes the directory contains valid manifests and the current Kubernetes context has access to the cluster. If the manifests are spread across subdirectories, validate each file or directory separately.

## 6. Troubleshooting Notes

### Backend reports port 8080 is already in use

Check whether another process or application instance is already listening on port 8080. In a container, also check whether the container starts more than one application process.

```bash
sudo ss -ltnp | grep ':8080'
```

Do not terminate a process until you confirm what it is and whether it is safe to stop.

### Frontend is accessible internally but not publicly

Check the Service type, endpoints, cloud load balancer or ingress configuration, security-group rules, and application logs. A `NodePort` Service alone does not guarantee public internet access. Confirm the intended exposure method before changing networking.

## 7. Next Step

The exact current project files have not yet been inspected in this document. To create an accurate inventory, open the directory that contains the existing MyCoffeeShop files and run:

```bash
find . -maxdepth 4 -type f | sort
```

Save the output and compare it with the proposed structure. Then update this document with the real file paths, image names, namespaces, Services, and deployment commands.
