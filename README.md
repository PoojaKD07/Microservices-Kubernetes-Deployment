# Microservices Web Application and Kubernetes Resource Management

A containerized Product Management web application built using **Python Flask, Docker, and Kubernetes (Kind)**. This project demonstrates microservices communication, container deployment, service discovery, horizontal scaling, and Kubernetes resource scheduling.

## 1. Project Overview

The application follows a microservices architecture with two independent services:

- **Frontend Service:** Provides the application interface and communicates with the backend.
- **Backend Service:** Provides product data through REST API endpoints.

Docker is used to containerize both services, while Kubernetes manages deployment, networking, scaling, and resource allocation.

## 2. Architecture

```text
                User / Browser
                      |
                      v
              Frontend Service
                      |
                      v
              catalog-service
                      |
                      v
             Backend Deployment
                /     |     \
              Pod    Pod    Pod
```

The frontend communicates with the backend using the Kubernetes service name `catalog-service`.

## 3. Technologies Used

| Technology | Purpose |
|---|---|
| Python | Application development |
| Flask | Frontend and backend services |
| Docker | Containerization |
| Kubernetes | Container orchestration |
| Kind | Local Kubernetes cluster |
| kubectl | Kubernetes management |

## 4. Project Structure

```text
Microservices-Kubernetes-Deployment/
│
├── backend/
│   ├── app.py
│   ├── Dockerfile
│   └── requirements.txt
│
├── frontend/
│   ├── app.py
│   ├── Dockerfile
│   └── requirements.txt
│
├── screenshots/
│   ├── 01-cluster-and-application-verification.png
│   └── 02-resource-scheduling-test.png
│
├── deployment.yaml
└── README.md
```

## 5. Implementation

### A. Microservices Development

Two Flask services were developed independently.

**Backend Service**
- Provides product information.
- Exposes the `/products` API endpoint.
- Provides a `/health` endpoint.
- Returns the hostname of the backend Pod using `socket.gethostname()`.

**Frontend Service**
- Provides the application interface.
- Communicates with the backend through `catalog-service`.
- Displays the product information received from the backend.

### B. Docker Containerization

Separate Docker images were built for the frontend and backend services.

```powershell
docker build -t catalog:v1 .\backend
docker build -t frontend:v1 .\frontend
```

The images were loaded into the Kind cluster:

```powershell
kind load docker-image catalog:v1 --name microservices-lab
kind load docker-image frontend:v1 --name microservices-lab
```

### C. Kubernetes Deployment

The Kubernetes configuration file `deployment.yaml` defines:

- Backend Deployment
- Frontend Deployment
- Backend Service
- Frontend Service
- Container resource requests and limits

Deploy the application:

```powershell
kubectl apply -f deployment.yaml
```

Verify the deployments and services:

```powershell
kubectl get deployments
kubectl get services
kubectl get pods -o wide
```

### D. Service Discovery and Communication

The backend is exposed internally through a Kubernetes ClusterIP service named `catalog-service`.

The frontend accesses the backend using:

```text
http://catalog-service:5000
```

The backend API was tested from the frontend Pod. The response returned HTTP status **200** with the product list.

### E. Horizontal Scaling

The backend Deployment was scaled to three replicas:

```powershell
kubectl scale deployment/catalog-deployment --replicas=3
```

Verify the replicas:

```powershell
kubectl get deployments
kubectl get pods -o wide
kubectl get endpoints catalog-service
```

The three backend Pods provide multiple endpoints behind the same Kubernetes service.

### F. Resource Allocation and Scheduling

Resource requests and limits were configured for the application containers.

| Resource | Request | Limit |
|---|---:|---:|
| CPU | 100m | 500m |
| Memory | 128Mi | 256Mi |

Node resource allocation can be inspected using:

```powershell
kubectl describe nodes
```

### G. Resource Scheduling Experiment

An additional Pod named `resource-test-pod` was created to test Kubernetes scheduling when resource requirements exceed available capacity.

Its resource requests were configured as:

```yaml
resources:
  requests:
    cpu: "100"
    memory: "100Gi"
```

Check its status:

```powershell
kubectl get pods
kubectl describe pod resource-test-pod
```

**Observed result:** The Pod remains in the `Pending` state because Kubernetes cannot find a suitable node to schedule it. The scheduler reports insufficient CPU and memory on nodes, along with an untolerated node taint.

This demonstrates how Kubernetes evaluates resource requests and scheduling constraints before assigning a Pod to a node.

The test Pod is intentionally retained for demonstration.

## 6. Execution Screenshots

### Cluster and Application Verification

The screenshot shows the three Kind nodes in Ready state, successful frontend and backend deployments, Kubernetes services, running Pods, a successful backend API response, and backend service endpoints.

![Cluster and application verification](screenshots/01.png)

### Resource Scheduling Test

The screenshot shows the resource test Pod in Pending state, its CPU and memory requests, and the scheduler's FailedScheduling event.

![Resource scheduling test](screenshots/02.png)

## 7. Final Execution Results

| Component / Test | Observed Result |
|---|---|
| Kind cluster | 3 nodes Ready |
| Backend Deployment | 3/3 replicas Ready |
| Frontend Deployment | 1/1 replica Ready |
| Backend Service | ClusterIP, port 5000 |
| Frontend Service | ClusterIP, port 80 |
| Backend API | HTTP 200 |
| Backend endpoints | 3 Pod endpoints |
| Resource test Pod | Pending |

## 8. Conclusion

This project demonstrates the development and deployment of a Flask-based microservices application using Docker and Kubernetes.

It covers independent service containerization, Kubernetes Deployments and Services, internal service discovery, horizontal scaling, resource requests and limits, and Pod scheduling behavior.

The experiment provides practical understanding of how Kubernetes manages containerized applications and allocates cluster resources.
