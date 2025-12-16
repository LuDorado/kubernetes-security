# Kubernetes Security Project

Secure containerized application deployment with hardened security configurations.

## Prerequisites

- Kubernetes cluster (minikube, kind, or cloud provider)
- kubectl configured
- Ingress controller (nginx-ingress recommended)

## Architecture

- **Frontend**: Nginx serving static content
- **API**: FastAPI backend with JWT authentication
- **Database**: PostgreSQL with secure configuration

## Security Features

- **Container Security**: Non-root containers (runAsUser: 1000)
- **Filesystem Security**: Read-only root filesystems with writable volumes
- **Capabilities**: Dropped all capabilities (ALL)
- **Privilege Escalation**: Disabled (allowPrivilegeEscalation: false)
- **Seccomp**: Runtime default security profiles
- **RBAC**: Service accounts with minimal permissions
- **Network Policies**: Pod-to-pod communication restrictions
- **Pod Security**: Standards enforcement
- **Resource Limits**: CPU and memory constraints
- **Persistent Storage**: Secure data persistence
- **Secrets Management**: Base64 encoded secrets (no plaintext comments)

## Deployment

### 1. Create namespace and RBAC
```bash
kubectl apply -f 01-namespace.yaml
kubectl apply -f 02-rbac.yaml
```

### 2. Create secrets and configmaps
```bash
kubectl apply -f 03-secrets.yaml
kubectl apply -f 04-configmaps.yaml
```

### 3. Create persistent volume claims
```bash
kubectl apply -f 05-pvc.yaml
```

### 4. Deploy database
```bash
kubectl apply -f 06-deployments/db-deployment.yaml
kubectl apply -f 07-services/db-service.yaml
```

### 5. Deploy API
```bash
kubectl apply -f 06-deployments/api-deployment.yaml
kubectl apply -f 07-services/api-service.yaml
```

### 6. Deploy frontend
```bash
kubectl apply -f 06-deployments/frontend-deployment.yaml
kubectl apply -f 07-services/frontend-service.yaml
```

### 7. Create ingress and security policies
```bash
kubectl apply -f 08-ingress.yaml
kubectl apply -f 09-network-policies.yaml
kubectl apply -f 10-pod-security.yaml
```

## Verification

```bash
# Check all pods are running
kubectl get pods -n docker-security

# Check services
kubectl get services -n docker-security

# Check ingress
kubectl get ingress -n docker-security

# View logs
kubectl logs -n docker-security deployment/api-deployment
kubectl logs -n docker-security deployment/frontend-deployment
kubectl logs -n docker-security deployment/db-deployment
```

## Access the Application

### Web Access
```bash
# Option 1: Using NodePort (check the port from services)
kubectl get services -n docker-security
# Access: http://<NODE_IP>:<NODEPORT>

# Option 2: Using minikube
minikube service frontend-service -n docker-security --url

# Option 3: Port forwarding
kubectl port-forward -n docker-security service/frontend-service 8080:80
# Access: http://localhost:8080

# Option 4: Using Ingress (add to /etc/hosts)
echo "$(minikube ip) docker-security.local" | sudo tee -a /etc/hosts
# Access: http://docker-security.local (HTTP only - TLS not configured)
```

### Health Checks
```bash
# Check API health
kubectl port-forward -n docker-security service/api-service 8000:8000
curl http://localhost:8000/health

# Check database connectivity
kubectl exec -n docker-security deployment/db-deployment -- pg_isready -U app -d appdb

# Check pod health status
kubectl get pods -n docker-security -o wide
kubectl describe pods -n docker-security
```

## Troubleshooting

- **Ingress**: Ensure controller is installed: `minikube addons enable ingress`
- **Pod Security**: Check security contexts are compatible with images
- **Service Discovery**: Verify service names match application configurations
- **Database Storage**: Requires persistent volumes and writable directories
- **LoadBalancer**: If shows `<pending>`, use NodePort or port-forward instead
- **Image Pull**: Verify images exist in registry (some use custom images)
- **RBAC**: Ensure service accounts have proper permissions
- **Network Policies**: May block traffic if not configured correctly
- **Resource Limits**: Pods may fail if insufficient resources available