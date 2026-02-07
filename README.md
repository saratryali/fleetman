# Fleet Management Kubernetes Application

This repository contains Kubernetes manifests for deploying a microservices-based Fleet Management application. The application consists of multiple services that work together to track and manage fleet operations.

## Architecture Overview

The application follows a microservices architecture with the following components:

- **Web Application**: Angular-based frontend for user interface
- **API Gateway**: Central gateway for routing API requests
- **Position Tracker**: Service for tracking vehicle positions
- **Position Simulator**: Generates simulated position data
- **Message Queue**: Handles asynchronous messaging between services
- **MongoDB Database**: Persistent storage for application data

## Files Description

### 📄 `workload.yaml`
Contains Kubernetes Deployments for all application microservices:

- **webapp**: Angular frontend application (port 80)
- **api-gateway**: Main API gateway service (port 8080)
- **position-tracker**: Position tracking service (port 8080)
- **position-simulator**: Generates simulated position data
- **queue**: Message queue service (ports 8161, 61616)

All deployments are configured with Spring profiles set to `production-microservice`.

### 📄 `services.yaml`
Defines Kubernetes Services that expose the deployments:

| Service | Type | Ports | External Access |
|---------|------|-------|----------------|
| fleetman-webapp | NodePort | 80:30080 | ✅ Browser access |
| fleetman-api-gateway | NodePort | 8080:30020 | ✅ API access |
| fleetman-queue | NodePort | 8161:30010, 61616 | ✅ Queue management |
| fleetman-position-tracker | ClusterIP | 8080 | ❌ Internal only |

### 📄 `mango-stack.yaml`
MongoDB database configuration:

- **mongodb**: MongoDB 3.6.5 deployment with ClusterIP service
- **Port**: 27017 (internal cluster access only)

## Quick Start

### Prerequisites
- Kubernetes cluster (minikube, Docker Desktop, or cloud cluster)
- `kubectl` configured to access your cluster

### Deploy the Application

1. **Deploy MongoDB:**
   ```bash
   kubectl apply -f mango-stack.yaml
   ```

2. **Deploy Application Services:**
   ```bash
   kubectl apply -f workload.yaml
   ```

3. **Expose Services:**
   ```bash
   kubectl apply -f services.yaml
   ```

### Access the Application

- **Web App**: http://localhost:30080
- **API Gateway**: http://localhost:30020
- **Queue Management**: http://localhost:30010

### Verify Deployment

```bash
# Check all deployments
kubectl get deployments

# Check all services
kubectl get services

# Check pod status
kubectl get pods

# View application logs
kubectl logs -l app=webapp
```

## Service Dependencies

```
┌─────────────┐    ┌──────────────┐    ┌─────────────────┐
│   Browser   │───▶│   Web App    │───▶│  API Gateway    │
└─────────────┘    └──────────────┘    └─────────────────┘
                                               │
                   ┌─────────────────────────────┼──────────────┐
                   │                             │              │
                   ▼                             ▼              ▼
            ┌──────────────┐              ┌─────────────┐  ┌─────────┐
            │Position Track│              │    Queue    │  │ MongoDB │
            └──────────────┘              └─────────────┘  └─────────┘
                   ▲                             ▲
                   │                             │
            ┌──────────────┐                     │
            │Position Sim  │─────────────────────┘
            └──────────────┘
```

## Scaling the Application

```bash
# Scale individual services
kubectl scale deployment webapp --replicas=3
kubectl scale deployment api-gateway --replicas=2

# Auto-scale based on CPU (requires metrics server)
kubectl autoscale deployment webapp --cpu-percent=50 --min=1 --max=10
```

## Troubleshooting

### Common Issues

1. **Pods in Pending state:**
   ```bash
   kubectl describe pod <pod-name>
   ```

2. **Service not accessible:**
   ```bash
   kubectl get endpoints
   kubectl describe service <service-name>
   ```

3. **View application logs:**
   ```bash
   kubectl logs -f deployment/webapp
   kubectl logs -f deployment/api-gateway
   ```

### Health Checks

```bash
# Check if MongoDB is running
kubectl exec -it deployment/mongodb -- mongo --eval "db.stats()"

# Test API Gateway
curl http://localhost:30020/health

# Test Queue connection
curl http://localhost:30010
```

## Cleanup

To remove all resources:

```bash
kubectl delete -f services.yaml
kubectl delete -f workload.yaml
kubectl delete -f mango-stack.yaml
```

## Development

This configuration uses production Docker images from `richardchesterwood/k8s-fleetman-*` repositories. For development:

1. Update image tags in `workload.yaml` to point to your development images
2. Consider using `imagePullPolicy: Always` for latest development builds
3. Adjust resource limits and requests based on your requirements

## Notes

- All services are configured with Spring Boot production profiles
- MongoDB data is not persisted (no persistent volumes configured)
- External access is provided via NodePort services suitable for development/testing
- For production, consider using Ingress controllers and persistent storage