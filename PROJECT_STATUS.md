# MyCoffeeShop Project Status and Inventory

## Confirmed in saved project records

| Component | Recorded value |
|---|---|
| Frontend | Static HTML/CSS/JavaScript + NGINX |
| Frontend container port | 80 |
| Frontend Kubernetes Service | `coffee-frontend` |
| Frontend Service type | NodePort |
| NodePort | 30080 |
| Frontend image | `raveendradudyala/coffee-shop-frontend:3.0.1` |
| Backend | Java 17 + Spring Boot 3.5.5 + Maven |
| Backend artifact | `coffee-shop-backend-1.0.0.jar` |
| Backend port | 8080 |
| Backend image | `raveendradudyala/coffee-shop-backend:5.0.0` |
| Backend Kubernetes Service | `coffee-backend` (ClusterIP :8080) |
| Database | MySQL 8.0 |
| Namespace in documented manifests | `coffeeshop` |
| Storage | EBS CSI Driver, `gp3`, MySQL PVC |
| API path | `/api/` proxied by NGINX to `coffee-backend:8080` |
| Cluster example | `eksdemo1`, region `us-east-1` (verify before use) |

## Recorded troubleshooting

1. Backend startup error: port 8080 already in use.
2. Frontend external access: check NodePort 30080, node public IP, security group, Service endpoints.
3. Frontend-to-backend calls: verify NGINX proxy target is `coffee-backend:8080` and API path mapping.
4. Backend-to-MySQL connectivity: check MySQL Service DNS, Secret/ConfigMap values, and backend logs.
5. MySQL pod pending: check PVC status, StorageClass, EBS CSI Driver and IAM permissions.

## Source-code recovery checklist

- [ ] `coffee-shop-frontend/index.html`
- [ ] Frontend CSS / JS / images
- [ ] Frontend `Dockerfile`
- [ ] Frontend `nginx.conf`
- [ ] Backend `pom.xml`
- [ ] Backend Java source (`OrderController.java`, `Order.java`, and other classes)
- [ ] Backend `application.properties` (remove credentials before sharing)
- [ ] Backend `Dockerfile`
- [ ] All Kubernetes YAML manifests
- [ ] Terraform `.tf` files
- [ ] `Jenkinsfile` or CI/CD workflow

The checklist is intentionally separate from the documentation inventory because those original files were not available in the retrieved saved records.
