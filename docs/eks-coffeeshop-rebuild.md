# Phase 0 — EKS Rebuild & Coffee Shop Deployment

A step-by-step runbook: EKS cluster → Pod Identity → EBS CSI Driver → Coffee Shop app (MySQL + Backend + Frontend).

## Table of Contents

1. [Cluster Setup](#1-cluster-setup)
2. [EKS Pod Identity](#2-eks-pod-identity)
3. [EBS CSI Driver](#3-ebs-csi-driver)
4. [Manifest Directory](#4-manifest-directory)
5. [Deploy Coffee Shop App](#5-deploy-coffee-shop-app)
6. [Apply Order Cheat Sheet](#6-apply-order-cheat-sheet)

---

## 1. Cluster Setup

Create the cluster script:

```bash
vi create-eks-cluster.sh
```

Verify the cluster is ready:

```bash
kubectl get nodes
kubectl get nodes -o wide
```

---

## 2. EKS Pod Identity

### Step 1 — Install the EKS Pod Identity Agent

1. Open **EKS Console → Clusters →** select your cluster (`eksdemo1`)
2. Go to **Add-ons → Get more add-ons**
3. Search for **EKS Pod Identity Agent**
4. Click **Next → Create**

This installs a DaemonSet (`eks-pod-identity-agent`) that enables Pod Identity associations.

```bash
# List k8s PIA resources
kubectl get daemonset -n kube-system

# List pods
kubectl get pods -n kube-system
```

### Step 2 — Deploy AWS CLI pod (test *without* Pod Identity association)

**`01_k8s_service_account.yaml`**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: aws-cli-sa
  namespace: default
```

**`02_k8s_aws_cli_pod.yaml`**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: aws-cli
  namespace: default
spec:
  serviceAccountName: aws-cli-sa
  containers:
  - name: aws-cli
    image: amazon/aws-cli
    command: ["sleep", "infinity"]
```

Deploy and test:

```bash
kubectl apply -f kube-manifests/
kubectl get pods

# Expected: FAILS (no permissions yet)
kubectl exec -it aws-cli -- aws s3 ls
```

### Step 3 — Create IAM role for Pod Identity

1. **IAM Console → Roles → Create Role**
2. Trusted entity type → **Custom trust policy**
3. Use this trust policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "pods.eks.amazonaws.com"
            },
            "Action": [
                "sts:AssumeRole",
                "sts:TagSession"
            ]
        }
    ]
}
```

4. Attach policy: **AmazonS3ReadOnlyAccess**
5. Role name (example): `EKS-PodIdentity-S3-ReadOnly-Role-101`

### Step 4 — Create Pod Identity association

**EKS Console → Cluster → Access → Pod Identity Associations → Create**

| Field           | Value                                  |
|-----------------|----------------------------------------|
| Namespace       | `default`                              |
| Service Account | `aws-cli-sa`                           |
| IAM Role        | `EKS-PodIdentity-S3-ReadOnly-Role-101` |

### Step 5 — Re-test

> **Tip:** Pods pick up credentials at start time. Restart the pod after creating the association.

```bash
kubectl delete pod aws-cli
kubectl apply -f kube-manifests/
kubectl exec -it aws-cli -- aws s3 ls     # should now list buckets
```

---

## 3. EBS CSI Driver

### Step 1 — Install

1. **Add-ons → Get more add-ons**
2. Search **Amazon EBS CSI driver** (`aws-ebs-csi-driver`)
3. On the **Permissions** step → choose **Create recommended role**

The console creates an IAM role with the managed policies `AmazonEBSCSIDriverPolicy` and `AmazonEKSClusterPolicy`. Trust is automatically set to `pods.eks.amazonaws.com`.

### Step 2 — Verify

```bash
kubectl get pods -n kube-system
kubectl get ds -n kube-system
kubectl get deploy -n kube-system
```

---

## 4. Manifest Directory

Existing YAML files are kept separate from the Dockerfiles.

```bash
cd /eks/coffee-shop-frontend
ls
```

Expected:

```
coffee-shop-backend
coffee-shop-frontend
manifest
```

```bash
cd /eks/coffee-shop-frontend/manifest
ls -1
```

Files:

```
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
```

---

## 5. Deploy Coffee Shop App

All commands run from:

```bash
cd /eks/coffee-shop-frontend/manifest
```

### Step 1 — StorageClass

**`00-ebs-storageclass.yaml`**

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

```bash
kubectl apply -f 00-ebs-storageclass.yaml
kubectl get storageclass
```

### Step 2 — Namespace

**`00-namespace.yaml`**

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: coffeeshop
```

```bash
kubectl apply -f 00-namespace.yaml
```

### Step 3 — MySQL Secret

**`05-mysql-secret.yaml`**

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: mysql-secret
  namespace: coffeeshop

type: Opaque

stringData:
  MYSQL_ROOT_PASSWORD: dbpassword11
```

```bash
kubectl apply -f 05-mysql-secret.yaml
kubectl get secret -n coffeeshop
kubectl describe secret mysql-secret -n coffeeshop
```

> ⚠️ **Note:** The password is plain text in the YAML. Fine for a demo, but don't commit this file to Git for real projects.

### Step 4 — MySQL PVC (5Gi)

**`06-mysql-pvc.yaml`**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: mysql-pvc
  namespace: coffeeshop

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 5Gi

  storageClassName: ebs-sc
```

```bash
kubectl apply -f 06-mysql-pvc.yaml
kubectl get storageclass
kubectl get pvc -n coffeeshop
```

> The PVC stays `Pending` until a pod uses it — this is expected because of `WaitForFirstConsumer`.

### Step 5 — MySQL Deployment

**`07-mysql-deployment.yaml`**

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: mysql
  namespace: coffeeshop

spec:
  replicas: 1

  strategy:
    type: Recreate

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

          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql

      volumes:
        - name: mysql-storage
          persistentVolumeClaim:
            claimName: mysql-pvc
```

```bash
kubectl apply -f 07-mysql-deployment.yaml
kubectl get pvc -n coffeeshop
kubectl get pv

# Check MySQL logs
kubectl logs deployment/mysql -n coffeeshop
```

> **Remember:** PVC is namespace-scoped; PV is cluster-scoped.

### Step 6 — MySQL Service

**`08-mysql-service.yaml`**

```yaml
apiVersion: v1
kind: Service

metadata:
  name: mysql
  namespace: coffeeshop

spec:
  type: ClusterIP

  selector:
    app: mysql

  ports:
    - port: 3306
      targetPort: 3306
```

```bash
kubectl apply -f 08-mysql-service.yaml
kubectl get svc -n coffeeshop
kubectl get endpoints mysql -n coffeeshop
kubectl get pods -n coffeeshop --show-labels
```

### Step 7 — Backend Deployment

**`03-backend-deployment.yaml`**

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: coffee-backend
  namespace: coffeeshop

spec:
  replicas: 1

  selector:
    matchLabels:
      app: coffee-backend

  template:
    metadata:
      labels:
        app: coffee-backend

    spec:
      containers:
        - name: coffee-backend

          image: raveendradudyala/coffee-shop-backend:5.0.0

          ports:
            - containerPort: 8080

          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
```

```bash
kubectl apply -f 03-backend-deployment.yaml
kubectl get pods -n coffeeshop

# Backend logs
kubectl logs deployment/coffee-backend -n coffeeshop
```

### Step 8 — Backend Service

**`04-backend-service.yaml`**

```yaml
apiVersion: v1
kind: Service

metadata:
  name: coffee-backend
  namespace: coffeeshop

spec:
  type: ClusterIP

  selector:
    app: coffee-backend

  ports:
    - port: 8080
      targetPort: 8080
```

```bash
kubectl apply -f 04-backend-service.yaml
kubectl get svc -n coffeeshop
kubectl get endpoints coffee-backend -n coffeeshop
```

### Step 9 — Frontend Deployment

**`01-frontend-deployment.yaml`**

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
        - name: coffee-frontend

          image: raveendradudyala/coffee-shop-frontend:3.0.1

          ports:
            - containerPort: 80
```

```bash
kubectl apply -f 01-frontend-deployment.yaml
kubectl get pods -n coffeeshop
kubectl get deployment -n coffeeshop
```

### Step 10 — Frontend Service (NodePort)

**`02-frontend-service.yaml`**

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

```bash
kubectl apply -f 02-frontend-service.yaml
kubectl get svc -n coffeeshop
kubectl get endpoints coffee-frontend -n coffeeshop
```

> To access the app, open `http://<node-public-ip>:30080` and make sure the node security group allows inbound TCP **30080**.

---

## 6. Apply Order Cheat Sheet

| Order | File                         | Purpose                     |
|-------|------------------------------|-----------------------------|
| 1     | `00-ebs-storageclass.yaml`   | gp3 StorageClass (`ebs-sc`) |
| 2     | `00-namespace.yaml`          | `coffeeshop` namespace      |
| 3     | `05-mysql-secret.yaml`       | MySQL root password         |
| 4     | `06-mysql-pvc.yaml`          | 5Gi persistent volume claim |
| 5     | `07-mysql-deployment.yaml`   | MySQL pod                   |
| 6     | `08-mysql-service.yaml`      | MySQL ClusterIP service     |
| 7     | `03-backend-deployment.yaml` | Backend pod                 |
| 8     | `04-backend-service.yaml`    | Backend ClusterIP service   |
| 9     | `01-frontend-deployment.yaml`| Frontend pods (x2)          |
| 10    | `02-frontend-service.yaml`   | Frontend NodePort (30080)   |

Final check:

```bash
kubectl get all -n coffeeshop
kubectl get pvc -n coffeeshop
```
