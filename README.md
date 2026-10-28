# GitOps Implementation Using Argo CD and Kubernetes

## Project Overview

This project demonstrates a **GitOps-based Continuous Delivery workflow** using Argo CD, Kubernetes, Docker, Minikube, and GitHub.

An Nginx application is deployed to a Kubernetes cluster using YAML manifests stored in GitHub. Argo CD monitors the repository and automatically synchronizes the desired configuration with the cluster.

## Technologies Used

| Technology | Purpose |
|---|---|
| Docker | Container runtime |
| Minikube | Local Kubernetes cluster |
| Kubernetes | Container orchestration |
| Argo CD | GitOps continuous delivery |
| GitHub | Source code and configuration management |
| Nginx | Sample application |

## GitOps Architecture

```text
          GitHub Repository
             Gitops-Demo
                  |
                  | Kubernetes Manifests
                  v
                Argo CD
                  |
                  | Automatic Sync
                  v
          Kubernetes (Minikube)
                  |
                  v
           Nginx Deployment
                  |
             2 Nginx Pods
                  |
                  v
            Nginx Service
                  |
                  v
          Application Access
```

## 1. Docker and Minikube Setup

Docker provides the container runtime, while Minikube creates a local Kubernetes cluster using the Docker driver.

Start Minikube:

```bash
minikube start --driver=docker
```

Verify Kubernetes:

```bash
minikube status
kubectl get nodes
```

**Result:** Kubernetes v1.37.0 was successfully configured, and the Minikube node reached the `Ready` state.

## 2. Argo CD Installation

Create the namespace:

```bash
kubectl create namespace argocd
```

Install Argo CD:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Verify Argo CD components:

```bash
kubectl get pods -n argocd
```

**Result:** All seven Argo CD pods were successfully running.

Access the Argo CD dashboard:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Dashboard: `https://localhost:8080`

## 3. GitHub Repository Configuration

Created a GitHub repository named **Gitops-Demo** to store Kubernetes manifests.

### Repository Structure

```text
Gitops-Demo/
│
├── README.md
├── deployment.yaml
└── service.yaml
```

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
```

This manifest creates an Nginx Deployment with two replicas.

### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

This Service provides internal access to the Nginx pods.

## 4. Argo CD Application Configuration

Created an Argo CD application with the following configuration:

| Parameter | Value |
|---|---|
| Application Name | nginx-gitops |
| Project | default |
| Repository | Gitops-Demo |
| Target Revision | HEAD |
| Source Path | . |
| Destination Cluster | in-cluster |
| Namespace | default |
| Sync Policy | Automatic |

Argo CD reads the Kubernetes manifests from GitHub and deploys the resources to Minikube.

### Synchronization

**Automatic Sync:** Applies detected Git changes to Kubernetes.

**Prune:** Removes managed resources deleted from Git when enabled.

**Self-Heal:** Restores resources to the desired Git configuration when enabled.

## 5. Kubernetes Deployment Verification

Verify the Nginx Deployment:

```bash
kubectl get deployments
```

Output:

```text
NAME               READY   UP-TO-DATE   AVAILABLE
nginx-deployment   2/2     2            2
```

Verify the pods:

```bash
kubectl get pods
```

Output:

```text
NAME                                READY   STATUS
nginx-deployment-75cc477c85-crkfx   1/1     Running
nginx-deployment-75cc477c85-sdpbg   1/1     Running
```

**Result:** Both Nginx pods were successfully deployed through Argo CD.

## 6. Nginx Application Access

Expose the Nginx Service locally:

```bash
kubectl port-forward svc/nginx-service 8081:80
```

Access the application:

`http://localhost:8081`

**Result:** The Nginx welcome page was displayed successfully.

## 7. How GitOps Works

1. Kubernetes YAML files are maintained in GitHub.
2. Developers commit configuration changes.
3. Argo CD monitors the Git repository.
4. Argo CD compares the desired state in Git with the actual Kubernetes state.
5. Argo CD automatically synchronizes changes when automated sync is enabled.
6. Kubernetes creates or updates the application resources.

### Example: Automatic Scaling

Change the replica count in `deployment.yaml`:

```yaml
replicas: 2
```

To:

```yaml
replicas: 3
```

Commit the change to GitHub.

Argo CD detects the updated configuration and automatically scales the Deployment to three replicas.

This eliminates the need to manually apply Kubernetes manifests for each change.

**Note:** Automatic scaling is the next planned experiment.

## 8. Troubleshooting

During the setup, the following issues were resolved:

| Issue | Resolution |
|---|---|
| Kubernetes context misconfigured | `minikube update-context` |
| Argo CD destination cluster not found | Selected `in-cluster` |
| Argo CD application path not found | Updated repository path to `.` |
| Argo CD health showing Progressing | Verified Nginx Deployment and pods were ready |

## Project Status

| Task | Status |
|---|---|
| Docker and Minikube Setup | Completed |
| Kubernetes Cluster Setup | Completed |
| Argo CD Installation | Completed |
| GitHub Repository Integration | Completed |
| Nginx Deployment | Completed |
| Kubernetes Service Configuration | Completed |
| Nginx Application Verification | Completed |
| Automatic Scaling Test | Pending |
| Self-Healing Test | Pending |

## Conclusion

Successfully implemented a **GitOps-based Kubernetes deployment using Argo CD**.

The Nginx application was deployed using Kubernetes manifests stored in GitHub, with Argo CD managing synchronization between Git and the Kubernetes cluster.

This project demonstrates the fundamentals of **GitOps, Kubernetes orchestration, declarative configuration, and automated continuous delivery**.

---

**Author:** Darshil Gohel
