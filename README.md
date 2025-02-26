Kubernetes CI/CD Pipeline with Monitoring
=========================================

Project Overview
----------------

This project sets up a complete CI/CD pipeline for deploying applications on Kubernetes. It integrates monitoring using Prometheus and Grafana to track application performance and health. The pipeline ensures automatic deployment of new versions based on Git commits and includes observability for troubleshooting.

Features
--------

-   **Automated Deployment**: Uses Git commit tags to dynamically generate Kubernetes deployment files.

-   **Containerized Application**: The application is built and deployed in Docker containers.

-   **Kubernetes Orchestration**: Manages application lifecycle using Kubernetes.

-   **Monitoring & Observability**: Prometheus and Grafana are used for system monitoring and visualization.

-   **CI/CD Pipeline**: Automates testing and deployment using Jenkins.

Technologies Used
-----------------

-   **Kubernetes** (Minikube for local testing)

-   **Docker** (for containerization)

-   **Jenkins** (for CI/CD automation)

-   **Helm** (for Kubernetes package management)

-   **Prometheus** (for metrics collection)

-   **Grafana** (for visualization and dashboards)

-   **Flask** (sample web application)

Setup Instructions
------------------

### Prerequisites

-   Install **Docker**, **Kubernetes (Minikube/K3s)**, **Kubectl**, **Helm**, and **Jenkins** on your local machine.

-   Ensure you have Prometheus and Grafana installed using Helm.

### 1\. Clone the Repository

```
git clone <repo-url>
cd <project-directory>
```

### 2\. Build and Push the Docker Image

```
docker build -t my-app:latest .
docker tag my-app:latest my-dockerhub-username/my-app:<git-tag>
docker push my-dockerhub-username/my-app:<git-tag>
```

### 3\. Deploy to Kubernetes

```
helm upgrade --install my-app ./helm-chart --set image.tag=<git-tag>
```

### 4\. Verify Deployment

```
kubectl get pods
kubectl get svc
```

### 5\. Access the Application

Forward the service port to access the app locally:

```
kubectl port-forward svc/flask-service 8080:80
```

Then visit: <http://localhost:8080>

### 6\. Monitor Using Prometheus & Grafana

#### Access Prometheus:

```
kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090
```

Visit: <http://localhost:9090>

#### Access Grafana:

```
kubectl port-forward svc/prometheus-grafana 3000:80
```

Visit: <http://localhost:3000> Use the default credentials:

-   **Username**: admin

-   **Password**: (check Kubernetes secret)

```
kubectl get secret prometheus-grafana -o yaml
```

CI/CD Pipeline
--------------

The Jenkins pipeline automates:

1.  Pulling the latest code from Git.

2.  Building and pushing the Docker image.

3.  Deploying the latest image to Kubernetes.

4.  Running tests to ensure stability.

5.  Sending notifications on deployment status.

### Triggering a New Deployment

Push a new tag to trigger the pipeline:

```
git tag -a v1.0.1 -m "New release"
git push origin v1.0.1
```

Future Enhancements
-------------------

-   Implement **GitOps** with ArgoCD for automated deployments.

-   Improve **alerting** with Prometheus Alertmanager.

-   Add **logging aggregation** using ELK stack.