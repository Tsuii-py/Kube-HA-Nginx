# Kube-HA-Nginx: A High-Availability Web Architecture on Kubernetes

This repository showcases a practical, high-availability web architecture built with HAProxy, Nginx, and a simple Flask application, all orchestrated by Kubernetes. The entire deployment process is automated from commit to cluster using a CI/CD pipeline powered by GitHub Actions.

It serves as a hands-on blueprint for building resilient, scalable, and automated web services.

## ✨ Core Concepts

This project demonstrates several key principles of modern infrastructure:

-   **Layered Architecture:** Using the right tool for the job: HAProxy as a dedicated edge load balancer and Nginx as a versatile reverse proxy and static content server.
-   **Containerization with Docker:** The Python application is packaged into a lightweight, portable Docker image.
-   **Kubernetes Orchestration:** All components are deployed and managed declaratively using Kubernetes manifests, ensuring a reproducible and scalable environment.
-   **Decoupled Configuration:** Leveraging Kubernetes `ConfigMaps` to manage Nginx and HAProxy configurations without needing to rebuild Docker images.
-   **End-to-End Automation (CI/CD):** Every `push` to the `main` branch automatically builds the application, pushes the container image, and deploys the new version to the cluster.

## 🏗️ Architecture

User traffic flows through the system in the following sequence:

```
          INTERNET
             |
             v
  [Service: LoadBalancer (K8s)]  <-- Exposes HAProxy to the outside world
             |
             v
  [Deployment: HAProxy Pods]     <-- Edge load balancer (Layer 4/7)
             |
             v
  [Service: ClusterIP (Nginx)]   <-- Internal service for Nginx, visible only to HAProxy
             |
             v
  [Deployment: Nginx Pods]       <-- Reverse proxy and static content server
             |
             v
  [Service: ClusterIP (App)]     <-- Internal service for the app, visible only to Nginx
             |
             v
  [Deployment: App Pods]         <-- The actual Python/Flask application
```

## 🚀 Getting Started: Manual Deployment

To get this running on your own, you'll need a running Kubernetes cluster (like `minikube`, `kind`, or a cloud provider's) and `docker` installed.

1.  **Clone the repository:**
    ```bash
    git clone <REPO_URL>
    cd kube-ha-nginx
    ```

2.  **Build the application image:**
    ```bash
    docker build -t kube-ha-nginx-app:v1 ./app
    ```

3.  **Load the image into your cluster** (this example is for `minikube`):
    ```bash
    minikube image load kube-ha-nginx-app:v1
    ```

4.  **Apply the Kubernetes manifests:**
    The `kustomize` flag (`-k`) intelligently applies all resources in the `k8s/` directory.
    ```bash
    kubectl apply -k k8s/
    ```

5.  **Get the access URL** (this example is for `minikube`):
    This command tunnels into the cluster and gives you a direct URL to access the service.
    ```bash
    minikube service haproxy-service --url
    ```

6.  **Test the load balancing!**
    Run the command below a few times. You should see the `Served by pod:` message change, confirming that the load balancing is working!
    ```bash
    curl <URL_FROM_PREVIOUS_STEP>
    ```

## 🤖 Automated CI/CD Pipeline

This project is configured with a GitHub Actions workflow defined in `.github/workflows/ci-cd.yml`.

### How It Works

On every `git push` to the `main` branch, the pipeline automatically performs two jobs:

1.  **Build & Push Docker Image:**
    -   Builds a new Docker image from the `app/` directory.
    -   Tags the image with the unique Git commit SHA for precise versioning.
    -   Logs into the GitHub Container Registry (GHCR) and pushes the new image.

2.  **Deploy to Kubernetes:**
    -   Waits for the `build-and-push` job to succeed.
    -   Connects to a Kubernetes cluster.
    -   Uses `kustomize` to update the `app-deployment` manifest to use the new image tag and applies the changes.

The pipeline relies on GitHub Actions secrets to securely store credentials for the container registry and the Kubernetes cluster. For more details, refer to the workflow file.