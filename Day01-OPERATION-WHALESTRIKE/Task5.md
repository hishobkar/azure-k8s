# .NET Core Task: Deploy a Sample App to Understand Cluster Workflow

## Task 5: Deploy a Sample App to Understand Cluster Workflow (1 hour)

### Objective
Deploy a sample application to your local Minikube Kubernetes cluster to understand the workflow of deployments, services, and accessing apps.

### Steps

1. **Create a Simple Deployment**
   - In your WSL terminal, run:
     ```bash
     kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
     ```
   - This creates a deployment named `hello-minikube` using a simple echo server image.

2. **Expose the Deployment**
   - Expose the deployment as a service with a NodePort:
     ```bash
     kubectl expose deployment hello-minikube --type=NodePort --port=8080
     ```

3. **Get Service URL**
   - Retrieve the URL to access the service:
     ```bash
     minikube service hello-minikube --url
     ```
   - Open the URL in your browser to verify the application is running.

4. **Clean Up**
   - Delete the service and deployment to free resources:
     ```bash
     kubectl delete service hello-minikube
     kubectl delete deployment hello-minikube
     ```

### Estimated Time
1 hour

### Notes
- Ensure Minikube cluster is running (`minikube start`) before deploying the app.
- The `NodePort` exposes your app on a port that can be accessed from your host machine.
- This task helps understand Kubernetes workflow: **Deployment → Service → Access → Cleanup**.
