# Task 2: Create a Deployment and Observe the ReplicaSet

**Duration:** 30 minutes  
**Environment:** WSL with Docker, kubectl, and Minikube  
**Target Audience:** .NET Core Developers

---

## Prerequisites
- Docker installed and running in WSL
- kubectl installed in WSL
- Minikube installed and cluster running in WSL
- Basic understanding of Kubernetes concepts

---

## Objectives
- Create a Kubernetes Deployment
- Understand the relationship between Deployments, ReplicaSets, and Pods
- Observe automatic ReplicaSet creation
- Learn basic kubectl commands for monitoring resources

---

## Task Steps

### 1. Ensure Minikube is Running
Before starting, verify your Minikube cluster is active:
```bash
minikube status
```

If not running, start it:
```bash
minikube start
```

---

### 2. Create a Deployment

Create a deployment named `web-deploy` using the nginx image:
```bash
kubectl create deployment web-deploy --image=nginx
```

**Expected Output:**
```
deployment.apps/web-deploy created
```

**What's Happening:**
- A Deployment object is created
- The Deployment automatically creates a ReplicaSet
- The ReplicaSet creates the specified number of Pod replicas (default: 1)

---

### 3. Verify the Deployment

Check that the deployment was created successfully:
```bash
kubectl get deployments
```

**Expected Output:**
```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
web-deploy   1/1     1            1           30s
```

**Understanding the Output:**
- `NAME`: Deployment name
- `READY`: Number of ready pods / desired pods
- `UP-TO-DATE`: Number of replicas updated to desired state
- `AVAILABLE`: Number of available replicas
- `AGE`: Time since creation

---

### 4. Verify the ReplicaSet

Check the automatically created ReplicaSet:
```bash
kubectl get rs
```

**Alternative command:**
```bash
kubectl get replicasets
```

**Expected Output:**
```
NAME                    DESIRED   CURRENT   READY   AGE
web-deploy-xxxxxxxxxx   1         1         1       1m
```

**Understanding the Output:**
- `NAME`: ReplicaSet name (deployment name + random hash)
- `DESIRED`: Number of desired pod replicas
- `CURRENT`: Number of current replicas
- `READY`: Number of ready replicas
- `AGE`: Time since creation

**Key Observation:** The ReplicaSet was automatically created by the Deployment!

---

### 5. Verify the Pods

Check the pods created by the ReplicaSet:
```bash
kubectl get pods
```

**Expected Output:**
```
NAME                          READY   STATUS    RESTARTS   AGE
web-deploy-xxxxxxxxxx-xxxxx   1/1     Running   0          2m
```

**Understanding the Output:**
- `NAME`: Pod name (replicaset name + random hash)
- `READY`: Number of ready containers / total containers
- `STATUS`: Current pod status (Running, Pending, etc.)
- `RESTARTS`: Number of container restarts
- `AGE`: Time since creation

---

## Understanding the Hierarchy
```
Deployment (web-deploy)
    └── ReplicaSet (web-deploy-xxxxxxxxxx)
        └── Pod (web-deploy-xxxxxxxxxx-xxxxx)
```

**Key Concepts:**
- **Deployment**: Manages ReplicaSets and provides declarative updates
- **ReplicaSet**: Ensures specified number of pod replicas are running
- **Pod**: The smallest deployable unit containing one or more containers

---

## Additional Verification Commands

### Get Detailed Information

View detailed deployment information:
```bash
kubectl describe deployment web-deploy
```

View detailed ReplicaSet information:
```bash
kubectl describe rs <replicaset-name>
```

View detailed Pod information:
```bash
kubectl describe pod <pod-name>
```

### Watch Resources in Real-Time

Monitor deployments:
```bash
kubectl get deployments -w
```

Monitor pods:
```bash
kubectl get pods -w
```

*(Press `Ctrl+C` to stop watching)*

---

## For .NET Core Developers

### Understanding the Parallel

This nginx deployment works similarly to how you would deploy a .NET Core application:

**Equivalent .NET Core Deployment:**
```bash
kubectl create deployment dotnet-api --image=mcr.microsoft.com/dotnet/samples:aspnetapp
```

**Docker Hub .NET Images:**
- `mcr.microsoft.com/dotnet/aspnet:8.0` - ASP.NET Core runtime
- `mcr.microsoft.com/dotnet/sdk:8.0` - .NET SDK (for building)
- `mcr.microsoft.com/dotnet/samples:aspnetapp` - Sample ASP.NET app

---

## Troubleshooting

### Pod Not Running?

Check pod status:
```bash
kubectl get pods
```

View pod logs:
```bash
kubectl logs <pod-name>
```

View pod events:
```bash
kubectl describe pod <pod-name>
```

### Common Issues

1. **Image Pull Errors**: Check internet connectivity and image name
2. **Insufficient Resources**: Ensure Docker Desktop/Minikube has enough resources
3. **Pending Status**: Check for resource constraints or scheduling issues

---

## Cleanup (Optional)

To delete the deployment (this will also delete the ReplicaSet and Pods):
```bash
kubectl delete deployment web-deploy
```

Verify deletion:
```bash
kubectl get deployments
kubectl get rs
kubectl get pods
```

---

## Key Takeaways

- Deployments automatically create and manage ReplicaSets  
- ReplicaSets manage the desired number of Pod replicas  
- The naming hierarchy shows the relationship between resources  
- kubectl provides various commands to inspect and monitor resources  
- The same concepts apply when deploying .NET Core applications to Kubernetes

---

## Next Steps

- Experiment with scaling the deployment
- Learn about updating deployments (rolling updates)
- Create a deployment from a YAML file
- Deploy a custom .NET Core application to Minikube

---

**Task Completed!** You have successfully created a Deployment and observed how Kubernetes automatically manages ReplicaSets and Pods.