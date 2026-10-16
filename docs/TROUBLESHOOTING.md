# Coffee Shop Kubernetes Deployment - Troubleshooting Guide

## Common Issues and Solutions

### 1. MySQL Pod Not Starting

**Symptoms:**
- `kubectl get pods` shows MySQL pod in `Pending`, `CrashLoopBackOff`, or `CreateContainerConfigError`

**Diagnosis:**
```bash
kubectl describe pod -l app=mysql
kubectl logs deployment/mysql
```

**Solutions:**

**Issue: PVC not bound**
```bash
# Check PVC status
kubectl get pvc mysql-pvc -o wide
kubectl describe pvc mysql-pvc
```
- **Solution**: Ensure StorageClass exists: `kubectl get storageclass`
- **For EBS**: Verify EBS CSI driver is installed
  ```bash
  helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
  helm install aws-ebs-csi-driver aws-ebs-csi-driver/aws-ebs-csi-driver -n kube-system
  ```

**Issue: Secret not found**
```bash
# Check if secret exists
kubectl get secret mysql-secret
kubectl describe secret mysql-secret
```
- **Solution**: Create the secret
  ```bash
  kubectl apply -f 05-mysql-secret-CORRECTED.yaml
  ```

**Issue: Disk space**
```bash
# Check node disk space
kubectl describe nodes
# Check PVC usage
kubectl exec deployment/mysql -- df -h /var/lib/mysql
```
- **Solution**: Increase PVC size
  ```bash
  kubectl patch pvc mysql-pvc -p '{"spec":{"resources":{"requests":{"storage":"10Gi"}}}}'
  ```

---

### 2. Backend Cannot Connect to MySQL

**Symptoms:**
- Backend pod running but not ready (readiness probe failing)
- Logs show connection errors

**Diagnosis:**
```bash
kubectl logs deployment/coffee-backend
# Output might show: "Can't connect to MySQL server on 'mysql:3306'"
```

**Solutions:**

**Issue: MySQL service DNS not resolving**
```bash
# Test DNS from backend pod
kubectl exec deployment/coffee-backend -- nslookup mysql
kubectl exec deployment/coffee-backend -- cat /etc/resolv.conf
```
- **Solution**: Ensure MySQL service is created
  ```bash
  kubectl get svc mysql
  kubectl apply -f 08-mysql-service.yaml
  ```

**Issue: Wrong connection configuration**
```bash
# Check environment variables
kubectl exec deployment/coffee-backend -- env | grep MYSQL
```
- **Expected output:**
  ```
  MYSQL_HOST=mysql
  MYSQL_PORT=3306
  MYSQL_DATABASE=coffeedb
  MYSQL_USER=coffeeapp
  MYSQL_PASSWORD=coffeeapp123
  ```
- **Solution**: Verify environment variables in deployment
  ```bash
  kubectl describe deployment coffee-backend
  ```

**Issue: Network policy blocking access**
```bash
# Check network policies
kubectl get networkpolicies
```
- **Solution**: Ensure no restrictive network policies exist or create permissive policy
  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: allow-backend-to-mysql
  spec:
    podSelector:
      matchLabels:
        app: coffee-backend
    policyTypes:
    - Egress
    egress:
    - to:
      - podSelector:
          matchLabels:
            app: mysql
      ports:
      - protocol: TCP
        port: 3306
  ```

**Test connection from backend:**
```bash
kubectl exec deployment/coffee-backend -- sh -c \
  "mysql -h mysql -u coffeeapp -pcoffeeapp123 -D coffeedb -e 'SELECT 1;'"
```

---

### 3. Frontend Cannot Connect to Backend

**Symptoms:**
- Frontend loads but API calls fail
- Browser console shows 404 or connection refused errors

**Diagnosis:**
```bash
# Check backend service DNS from frontend
kubectl exec deployment/coffee-frontend -- nslookup coffee-backend

# Check environment variables
kubectl exec deployment/coffee-frontend -- env | grep REACT_APP
```

**Solutions:**

**Issue: Wrong API URL**
```bash
# Check if REACT_APP_API_URL is set correctly
kubectl exec deployment/coffee-frontend -- env | grep REACT_APP_API_URL
```
- **Expected**: `REACT_APP_API_URL=http://coffee-backend:8080`
- **Solution**: Update deployment
  ```bash
  kubectl set env deployment/coffee-frontend \
    REACT_APP_API_URL=http://coffee-backend:8080
  ```

**Issue: Backend service not accessible**
```bash
# Verify backend service
kubectl get svc coffee-backend
kubectl describe svc coffee-backend

# Test from frontend pod
kubectl exec deployment/coffee-frontend -- \
  wget -q -O- http://coffee-backend:8080/health || echo "Connection failed"
```

**Issue: CORS misconfiguration**
- **Error**: Browser shows CORS policy error
- **Solution**: Backend needs to allow frontend origin
  - Update backend configuration or add CORS headers
  - For development: Allow all origins in backend

**Direct test:**
```bash
# Port-forward and test
kubectl port-forward svc/coffee-backend 8080:8080 &
curl http://localhost:8080/health
kill %1
```

---

### 4. Frontend Cannot Be Accessed from Outside

**Symptoms:**
- Cannot reach frontend from browser
- `http://<NODE_IP>:30080` times out

**Diagnosis:**
```bash
# Get service details
kubectl get svc coffee-frontend -o wide

# Check NodePort
kubectl get svc coffee-frontend -o jsonpath='{.spec.ports[0].nodePort}'

# Get node IP
kubectl get nodes -o wide
```

**Solutions:**

**Issue: Wrong node IP**
- Get the correct node IP (ExternalIP or InternalIP):
  ```bash
  # For external access
  kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="ExternalIP")].address}'
  
  # For internal network
  kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}'
  ```

**Issue: NodePort not in range**
- **Error**: Port outside 30000-32767
- **Solution**: Change nodePort to valid range or use Ingress

**Issue: Security group/firewall blocking**
- **For AWS**: Security group must allow traffic on NodePort
  ```bash
  # Add inbound rule for port 30080
  aws ec2 authorize-security-group-ingress \
    --group-id sg-xxx \
    --protocol tcp \
    --port 30080 \
    --cidr 0.0.0.0/0
  ```

**Alternative - Use port forwarding:**
```bash
kubectl port-forward svc/coffee-frontend 8080:80
# Access at http://localhost:8080
```

**Recommended for production - Use Ingress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: coffee-shop
spec:
  ingressClassName: nginx
  rules:
  - host: coffee.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: coffee-frontend
            port:
              number: 80
```

---

### 5. Pod in CrashLoopBackOff

**Symptoms:**
- Pod keeps restarting
- Status shows `CrashLoopBackOff`

**Diagnosis:**
```bash
# Check logs
kubectl logs deployment/coffee-backend
kubectl logs --previous deployment/coffee-backend  # Previous crash logs

# Check events
kubectl describe pod -l app=coffee-backend
```

**Solutions:**

**Common causes:**
1. **Application error** → Check logs for actual error
2. **Missing dependencies** → Wait for dependencies (MySQL) to be ready
3. **Health check failing** → Verify probe configuration
   ```bash
   kubectl describe deployment coffee-backend | grep -A 20 Liveness
   ```
4. **Out of memory** → Increase resource limits
   ```bash
   kubectl set resources deployment coffee-backend \
     --limits=memory=1Gi,cpu=1000m
   ```

**Reset crash loop:**
```bash
# Temporarily disable health checks
kubectl set probe deployment/coffee-backend --liveness --initial-delay-seconds=300
```

---

### 6. Pods Pending - Not Scheduling

**Symptoms:**
- Pods stay in `Pending` state
- Not moving to `Running`

**Diagnosis:**
```bash
# Check pod events
kubectl describe pod -l app=mysql

# Check node resources
kubectl describe nodes

# Check resource requests
kubectl describe deployment mysql | grep -A 10 Resources
```

**Solutions:**

**Issue: Insufficient resources**
- **Solution**: 
  ```bash
  # Add nodes or reduce resource requests
  kubectl set resources deployment mysql \
    --requests=memory=128Mi,cpu=100m
  ```

**Issue: Node affinity/anti-affinity issues**
```bash
# Check affinity rules
kubectl describe deployment coffee-frontend | grep -A 10 Affinity
```
- **Solution**: Ensure enough nodes for pod distribution

**Issue: Node selector mismatch**
```bash
# Check node labels
kubectl get nodes --show-labels
```

---

### 7. Volume Mount Issues

**Symptoms:**
- Pod fails to mount PVC
- Error: `Unable to attach or mount volumes`

**Diagnosis:**
```bash
# Check PVC status
kubectl get pvc -o wide
kubectl describe pvc mysql-pvc

# Check PV status
kubectl get pv -o wide

# Check EBS volumes (if using AWS)
aws ec2 describe-volumes --filters Name=tag:kubernetes.io/cluster/\*,Values=owned
```

**Solutions:**

**Issue: PVC stuck in Pending**
- **Cause**: StorageClass not available or EBS CSI driver missing
- **Solution**: Install EBS CSI driver or verify StorageClass
  ```bash
  kubectl get storageclass
  kubectl describe storageclass ebs-sc
  ```

**Issue: Pod can't mount volume**
- **Cause**: Pod and PVC on different zones
- **Solution**: Use zone-aware storage or affinity
  ```bash
  kubectl get pvc mysql-pvc -o jsonpath='{.metadata.annotations}'
  ```

---

### 8. High CPU/Memory Usage

**Symptoms:**
- Pods using excessive resources
- Nodes getting overloaded

**Diagnosis:**
```bash
# Check resource usage
kubectl top nodes
kubectl top pods -A

# Check resource limits
kubectl describe deployment mysql | grep -A 5 Limits
```

**Solutions:**

**For MySQL:**
```bash
# Check current settings
kubectl exec deployment/mysql -- mysqld --help | grep max_connections

# Increase resource limits gradually
kubectl set resources deployment mysql --limits=memory=1Gi,cpu=1000m
```

**For Backend:**
```bash
# Check for connection leaks
kubectl logs deployment/coffee-backend | grep -i "connection\|error"

# Restart if needed
kubectl rollout restart deployment/coffee-backend
```

---

### 9. Persistent Data Loss After Restart

**Symptoms:**
- Data disappears after pod restarts
- Changes lost when pod is recreated

**Diagnosis:**
```bash
# Check if data is being written to mounted volume
kubectl exec deployment/mysql -- ls -la /var/lib/mysql

# Verify volume mount
kubectl describe pod -l app=mysql | grep -A 5 Mounts
```

**Solutions:**

**Ensure database is writing to mounted volume:**
```bash
# Test database persistence
kubectl exec deployment/mysql -- mysql -uroot -pdbpassword11 -e "CREATE DATABASE test;"
kubectl rollout restart deployment/mysql
kubectl exec deployment/mysql -- mysql -uroot -pdbpassword11 -e "SHOW DATABASES;" | grep test
```

**If data is lost:**
1. Don't delete the PV: `reclaimPolicy: Retain`
2. Recover from backup
3. Implement regular backup schedule

---

### 10. Debugging Commands Reference

```bash
# Pod status
kubectl get pods -A
kubectl get pods -l app=mysql -w  # Watch mode

# Logs
kubectl logs deployment/mysql
kubectl logs deployment/mysql -f  # Follow
kubectl logs deployment/mysql --tail=50
kubectl logs deployment/mysql --timestamps=true

# Execute commands in pod
kubectl exec -it deployment/mysql -- bash
kubectl exec deployment/mysql -- env

# Port forwarding
kubectl port-forward svc/mysql 3306:3306
kubectl port-forward deployment/coffee-backend 8080:8080

# Resource usage
kubectl top nodes
kubectl top pods

# Describe resources
kubectl describe deployment mysql
kubectl describe service coffee-frontend
kubectl describe pvc mysql-pvc

# Edit resources
kubectl edit deployment mysql
kubectl patch deployment mysql -p '{"spec":{"replicas":2}}'

# Delete and recreate
kubectl delete deployment mysql
kubectl apply -f 07-mysql-deployment-CORRECTED.yaml

# Check events
kubectl get events --sort-by='.lastTimestamp'
kubectl describe pod <pod-name>

# Health check status
kubectl get pods -o json | jq '.items[].status.conditions'
```

---

## Performance Tuning

### MySQL Performance
```bash
# Check slow queries
kubectl exec deployment/mysql -- mysql -uroot -pdbpassword11 -e \
  "SET GLOBAL slow_query_log = 'ON'; SET GLOBAL long_query_time = 2;"

# View query cache status
kubectl exec deployment/mysql -- mysqladmin -uroot -pdbpassword11 status
```

### Backend Performance
```bash
# Monitor connections
kubectl exec deployment/coffee-backend -- lsof -i -P | grep ESTABLISHED

# Check request latency
kubectl logs deployment/coffee-backend | grep "response time"
```

### Frontend Performance
```bash
# Check bundle size
kubectl exec deployment/coffee-frontend -- du -sh /usr/share/nginx/html

# Monitor cache hits
kubectl logs deployment/coffee-frontend
```

---

## Maintenance Tasks

### Regular Backups
```bash
# Backup MySQL database
kubectl exec deployment/mysql -- mysqldump \
  -uroot -pdbpassword11 --all-databases > backup.sql

# Backup PV (using snapshots in AWS)
aws ec2 create-snapshot --volume-id vol-xxx
```

### Health Checks
```bash
# Daily health verification
kubectl get all
kubectl get pvc
kubectl top nodes
kubectl top pods
```

### Update Applications
```bash
# Update container image
kubectl set image deployment/coffee-frontend \
  coffee-frontend=raveendradudyala/coffee-shop-frontend:3.0.2

# Monitor rollout
kubectl rollout status deployment/coffee-frontend
kubectl rollout history deployment/coffee-frontend
```

---

## Still Having Issues?

**Collect debug information:**
```bash
# Create debug bundle
kubectl cluster-info dump --output-directory=./cluster-dump

# Export resources
kubectl get all -A -o yaml > resources.yaml
kubectl get events -A > events.log

# Check kubelet logs
journalctl -u kubelet -n 100
```

**Where to get help:**
- Kubernetes docs: https://kubernetes.io/docs
- Coffee app repo: [GitHub Repository]
- Community forums: Kubernetes Slack

