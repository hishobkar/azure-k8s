# .NET Core Task: Inspect an Existing Pod Created by a Deployment

## Task 2: Inspect an Existing Pod Created by a Deployment (30 minutes)

### Objective
Create a deployment in Kubernetes (using Minikube or any active cluster), identify the Pod(s) created by that deployment, and inspect the Pod details. This helps .NET Core developers understand how their applications are deployed and managed within Kubernetes Pods.

### Prerequisites
- Minikube or another Kubernetes cluster must be running (e.g., `minikube start`).
- `kubectl` CLI must be installed and configured.
- Docker, Minikube, and kubectl should already be set up from previous tasks.

### Steps

1. **Create a Deployment**
   - Use a sample Nginx deployment:
     ```bash
     kubectl create deployment nginx-deploy --image=nginx
     ```

2. **List Pods to Identify the One Created by the Deployment**
   - Run:
     ```bash
     kubectl get pods
     ```
   - Note the Pod name (e.g., `nginx-deploy-xxxxxx-yyyyy`).

3. **Describe the Pod**
   - Replace `<pod-name>` with your actual Pod name:
     ```bash
     kubectl describe pod <pod-name>
     ```

### What to Observe in the Pod Details

| Detail | Explanation |
|--------|-------------|
| **Status** | Whether the Pod is `Pending`, `Running`, or `Completed`. |
| **Containers** | Name(s) of container(s) inside the Pod. |
| **Image Used** | The Docker image used (here `nginx`). |
| **Pod IP** | The internal IP address for communication inside the cluster. |
| **Node Name** | The node (VM) on which the Pod is running. |
| **Events** | Useful logs on scheduling, pulling images, or container restarts. |

### Why This Matters for .NET Core Developers
- When deploying a .NET Core microservice, it will run inside a Pod similar to this one.
- You will often inspect pods to debug startup failures, image pull issues, or misconfigurations.
- Understanding `kubectl describe pod` helps trace common issues in deployments.

### Estimated Time
🕒 **30 minutes**

### Cleanup (Optional)
To remove the created deployment and its associated Pod:
```bash
kubectl delete deployment nginx-deploy
