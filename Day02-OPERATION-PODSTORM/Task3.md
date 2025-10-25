# .NET Core Task: Create Your First Pod Using YAML

## Task 3: Create Your First Pod Using YAML (45 minutes)

### Objective
Manually define and create a Kubernetes Pod using a YAML configuration file. This helps .NET Core developers understand how Pods are defined declaratively, which is essential when deploying containerized .NET Core applications in Kubernetes.

### Steps

1. **Create a YAML File**
   - In your project or working directory (e.g., within a .NET Core microservices folder), create a file named `pod-demo.yaml`.
   - Add the following content:
     ```yaml
     apiVersion: v1
     kind: Pod
     metadata:
       name: demo-pod
     spec:
       containers:
         - name: demo-container
           image: nginx
     ```

2. **Apply the YAML to Create the Pod**
   - Run the following command in your WSL terminal:
     ```bash
     kubectl apply -f pod-demo.yaml
     ```

3. **Confirm the Pod is Running**
   - Check the Pod status:
     ```bash
     kubectl get pods
     ```
   - You should see `demo-pod` with a status such as `Running` or `ContainerCreating`.

### Optional (Further Inspection)
To view detailed information about the Pod:
```bash
kubectl describe pod demo-pod
```

### Why This Matters for .NET Core Developers

- When deploying a .NET Core API, Worker Service, or microservice, you'll define Pods using YAML files.
- Understanding declarative configuration is essential for working with more complex Kubernetes objects such as Deployments, ReplicaSets, and Services.
- This forms the foundation for CI/CD pipelines that deploy YAML-based manifests.

### Estimated Time : 45 minutes

### Cleanup (Optional)
To delete the Pod:

```bash
kubectl delete pod demo-pod
```

### Summary
In this task, you created your first Kubernetes Pod using a YAML configuration file. This introduced the concept of declarative infrastructure management—an essential skill when deploying .NET Core microservices to Kubernetes.
