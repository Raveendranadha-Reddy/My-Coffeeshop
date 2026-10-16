MyCoffeeShop Kubernetes YAML bundle

Files:
00-ebs-storageclass.yaml
00-namespace.yaml
01-frontend-deployment.yaml
02-frontend-service.yaml
03-backend-deployment.yaml
04-backend-service.yaml
05-mysql-secret.yaml
06-mysql-pvc.yaml
07-mysql-deployment.yaml
08-mysql-service.yaml

Before applying:
1. Confirm the EBS CSI driver is installed and has required IAM permissions.
2. Edit 05-mysql-secret.yaml and replace CHANGE_ME_STRONG_PASSWORD.
3. Confirm the image tags exist in Docker Hub and the backend image uses these environment variables.
4. The frontend image's NGINX config must proxy /api/ to http://coffee-backend:8080/api/.

Apply from this directory:
kubectl apply -f 00-ebs-storageclass.yaml
kubectl apply -f 00-namespace.yaml
kubectl apply -f 05-mysql-secret.yaml
kubectl apply -f 06-mysql-pvc.yaml
kubectl apply -f 07-mysql-deployment.yaml
kubectl apply -f 08-mysql-service.yaml
kubectl apply -f 03-backend-deployment.yaml
kubectl apply -f 04-backend-service.yaml
kubectl apply -f 01-frontend-deployment.yaml
kubectl apply -f 02-frontend-service.yaml

Verify:
kubectl get all,pvc -n coffeeshop
kubectl get storageclass
kubectl get endpoints -n coffeeshop

External access:
http://<EC2-PUBLIC-IP>:30080
Allow inbound TCP 30080 in the EC2 node security group.

Important:
These are reconstructed manifests based on saved project records, not a byte-for-byte recovery of the original cluster files. Review image names, DB configuration, and API paths against your actual application before deployment. Do not commit real passwords.
