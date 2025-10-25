# .NET Core Task: Launch Your First Local Kubernetes Cluster

## Task 4: Launch Your First Local Kubernetes Cluster (30 minutes)

### Objective
Start and interact with a local Kubernetes cluster using **Minikube** and **kubectl** in WSL.

### Steps

1. **Start Minikube**
   - Launch your WSL terminal in **VS Code**.
   - Start the Minikube cluster:
     ```bash
     minikube start
     ```
   - Wait for Minikube to provision the cluster. You should see messages indicating that the cluster is running.

2. **Check Cluster Status**
   - Verify the cluster and nodes:
     ```bash
     kubectl get nodes
     ```
   - You should see the Minikube node listed with `STATUS` as `Ready`.

3. **Stop the Cluster**
   - When finished, stop the cluster to free resources:
     ```bash
     minikube stop
     ```

### Estimated Time
30 minutes

### Notes
- Ensure that Docker, kubectl, and Minikube are already installed in WSL.
- Minikube must be started each time you want to work with the local Kubernetes cluster.
- Use `minikube dashboard` to open a web-based Kubernetes dashboard if needed.
