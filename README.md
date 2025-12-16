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

- Non-root containers (runAsUser: 1000)
- Read-only root filesystems
- Dropped capabilities (ALL)
- No privilege escalation
- Network policies ready
- Secrets management

## Deployment

### 1. Create namespace
```bash
kubectl apply -f 01-namespace.yaml
```

### 2. Create secrets
```bash
kubectl apply -f 02-secrets.yaml
```

### 3. Create configmaps
```bash
kubectl apply -f 03-configmaps.yaml
```

### 4. Deploy database
```bash
kubectl apply -f 04-deployments/db-deployment.yaml
kubectl apply -f 05-services/db-service.yaml
```

### 5. Deploy API
```bash
kubectl apply -f 04-deployments/api-deployment.yaml
kubectl apply -f 05-services/api-service.yaml
```

### 6. Deploy frontend
```bash
kubectl apply -f 04-deployments/frontend-deployment.yaml
kubectl apply -f 05-services/frontend-service.yaml
```

### 7. Create ingress
```bash
kubectl apply -f 06-ingress.yaml
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
# Access: http://<NODE_IP>:32382

# Option 2: Using minikube
minikube service frontend-service -n docker-security --url

# Option 3: Port forwarding
kubectl port-forward -n docker-security service/frontend-service 8080:80
# Access: http://localhost:8080

# Option 4: Using Ingress (add to /etc/hosts)
echo "$(minikube ip) docker-security.local" | sudo tee -a /etc/hosts
# Access: http://docker-security.local
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

- Ensure ingress controller is installed: `minikube addons enable ingress`
- Check pod security contexts are compatible with images
- Verify service names match application configurations
- Database needs writable volumes for /var/run/postgresql and /tmp
- If LoadBalancer shows `<pending>`, use NodePort or port-forward instead
- For image pull errors, verify images exist in registry