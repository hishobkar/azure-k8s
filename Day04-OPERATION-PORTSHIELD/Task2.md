# Task 2: Create a Deployment to Expose

**Duration:** 30 minutes  
**Environment:** WSL with Docker, kubectl, and Minikube installed  
**Target Audience:** .NET Core Developer

---

## Objective

Create a Kubernetes Deployment, scale it to multiple replicas, and verify the Pods are running. This deployment will be used in subsequent tasks to demonstrate how Services provide stable networking.

---

## Prerequisites

- Minikube is running in WSL
- kubectl is configured and connected to Minikube cluster
- Basic understanding of Pods and Deployments

**Verify prerequisites:**
```bash
# Check Minikube status
minikube status

# If not running, start it
minikube start

# Verify kubectl connection
kubectl cluster-info
```

---

## Step 1: Create a Deployment

**Command:**
```bash
kubectl create deployment svc-demo --image=nginx
```

**What this does:**
- Creates a Deployment named `svc-demo`
- Uses the `nginx` Docker image (we'll use nginx for simplicity, but imagine this as your ASP.NET Core app)
- Kubernetes automatically creates 1 Pod by default
- The Deployment manages the Pod lifecycle

**Expected output:**
```
deployment.apps/svc-demo created
```

**For .NET Core developers:**
In a real scenario, you would use your own ASP.NET Core image:
```bash
kubectl create deployment myapi --image=myregistry/myapi:latest
```

---

## Step 2: Verify Deployment Created

**Command:**
```bash
kubectl get deployments
```

**Expected output:**
```
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
svc-demo   1/1     1            1           30s
```

**Understanding the columns:**
- **NAME:** Deployment name
- **READY:** Number of ready Pods / Desired Pods
- **UP-TO-DATE:** Pods updated to latest configuration
- **AVAILABLE:** Pods available to serve traffic
- **AGE:** Time since deployment was created

**Get detailed information:**
```bash
kubectl describe deployment svc-demo
```

---

## Step 3: Scale the Deployment to 2 Replicas

**Command:**
```bash
kubectl scale deployment svc-demo --replicas=2
```

**What this does:**
- Scales the deployment from 1 Pod to 2 Pods
- Kubernetes automatically creates an additional Pod
- Both Pods run the same container image
- Simulates a load-balanced application with multiple instances

**Expected output:**
```
deployment.apps/svc-demo scaled
```

**For .NET Core developers:**
This is equivalent to running multiple instances of your ASP.NET Core application behind a load balancer for high availability and better performance.

---

## Step 4: Verify Pods are Running

**Command:**
```bash
kubectl get pods -o wide
```

**Expected output:**
```
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE       
svc-demo-7d4b8c9f5d-abc12   1/1     Running   0          2m    172.17.0.3   minikube   
svc-demo-7d4b8c9f5d-xyz89   1/1     Running   0          30s   172.17.0.4   minikube   
```

**Understanding the output:**
- **NAME:** Unique Pod name (deployment-name + random suffix)
- **READY:** Containers ready / Total containers (1/1 means ready)
- **STATUS:** Current state (Running, Pending, Error, etc.)
- **RESTARTS:** Number of times container has restarted
- **AGE:** Time since Pod was created
- **IP:** Pod's internal IP address (note: these are ephemeral!)
- **NODE:** Which Node the Pod is running on

**Key observation for this task:**
Notice that each Pod has a different IP address (172.17.0.3 and 172.17.0.4). These IPs are dynamic and will change if Pods are recreated. This is the problem Services solve!

---

## Step 5: Additional Verification Commands

### View Pod labels (important for Services)
```bash
kubectl get pods --show-labels
```

**Expected output:**
```
NAME                        READY   STATUS    RESTARTS   AGE   LABELS
svc-demo-7d4b8c9f5d-abc12   1/1     Running   0          3m    app=svc-demo,pod-template-hash=7d4b8c9f5d
svc-demo-7d4b8c9f5d-xyz89   1/1     Running   0          1m    app=svc-demo,pod-template-hash=7d4b8c9f5d
```

**Important:** Both Pods have the label `app=svc-demo`. Services will use this label to identify and route traffic to these Pods.

### Get detailed Pod information
```bash
kubectl describe pod <pod-name>
```

Replace `<pod-name>` with actual Pod name from the previous output.

### Watch Pods in real-time
```bash
kubectl get pods -w
```

Press `Ctrl+C` to stop watching.

---

## Step 6: Test Pod Accessibility (Optional)

### Access a Pod directly (for testing)
```bash
# Get Pod name
POD_NAME=$(kubectl get pods -l app=svc-demo -o jsonpath='{.items[0].metadata.name}')

# Execute command inside the Pod
kubectl exec -it $POD_NAME -- /bin/bash

# Inside the Pod, test nginx
curl localhost

# Exit the Pod
exit
```

**For .NET Core developers:**
If this were an ASP.NET Core app, you could test endpoints:
```bash
kubectl exec -it $POD_NAME -- curl http://localhost:5000/api/health
```

---

## Step 7: Understanding the Problem This Demonstrates

**Current situation:**
- You have 2 Pods running nginx
- Each Pod has a different IP address
- If a Pod crashes and restarts, it gets a new IP address
- How do you reliably connect to these Pods?

**Questions to consider:**
1. How would other applications discover and connect to these Pods?
2. What happens if one Pod is deleted and recreated?
3. How do you load balance traffic across both Pods?

**Answer:** Kubernetes Services! (covered in next tasks)

---

## Troubleshooting

### Pods not starting
```bash
# Check Pod events
kubectl describe pod <pod-name>

# View Pod logs
kubectl logs <pod-name>

# Check if Minikube has enough resources
minikube status
```

### Deployment not scaling
```bash
# Check deployment status
kubectl get deployment svc-demo -o yaml

# Check ReplicaSet
kubectl get replicasets
```

### Cannot pull image
```bash
# For Minikube, ensure it can access Docker images
minikube ssh
docker images
```

---

## Cleanup (After Completing All Tasks)

**Delete the deployment:**
```bash
kubectl delete deployment svc-demo
```

**Verify deletion:**
```bash
kubectl get pods
kubectl get deployments
```

---

## Key Takeaways for .NET Core Developers

1. **Deployments manage Pod lifecycle** - Similar to how IIS Application Pools manage worker processes
2. **Scaling is simple** - One command creates multiple replicas of your app
3. **Pods are ephemeral** - Their IP addresses change, which is why we need Services
4. **Labels are crucial** - They're used by Services to identify Pods (like tags in Azure)
5. **Multiple replicas** - Provide high availability and load distribution

---

## Next Steps

- Create a ClusterIP Service to provide stable internal networking
- Create a NodePort Service to expose the deployment externally
- Test connectivity and load balancing across Pods
- Understand how Services abstract away Pod IP changes

---

## Summary

You have successfully:
- Created a Deployment named `svc-demo` with nginx image
- Scaled it to 2 replicas
- Verified both Pods are running with unique IP addresses
- Observed Pod labels that Services will use for routing
- Understood the networking challenge that Services solve

The deployment is now ready to be exposed via Kubernetes Services!