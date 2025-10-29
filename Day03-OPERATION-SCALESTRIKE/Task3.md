# Task 3: Scale the Deployment to Increase Pod Count

**Duration:** 45 minutes  
**Environment:** WSL with Docker, kubectl, and Minikube  
**Target Audience:** .NET Core Developers

---

## Prerequisites
- Docker installed and running in WSL
- kubectl installed in WSL
- Minikube installed and cluster running in WSL
- Completed Task 2 (Deployment `web-deploy` should exist)
- Basic understanding of Kubernetes Deployments and ReplicaSets

---

## Objectives
- Learn how to scale Kubernetes Deployments
- Understand horizontal scaling in Kubernetes
- Observe how ReplicaSets manage pod replicas
- Document changes in ReplicaSet behavior during scaling
- Apply scaling concepts to .NET Core applications

---

## Task Steps

### 1. Verify Current Deployment State

Before scaling, check the current state of your deployment:
```bash
kubectl get deployments
```

**Expected Output:**
```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
web-deploy   1/1     1            1           10m
```

Check current ReplicaSet:
```bash
kubectl get rs
```

**Expected Output:**
```
NAME                    DESIRED   CURRENT   READY   AGE
web-deploy-xxxxxxxxxx   1         1         1       10m
```

Check current Pods:
```bash
kubectl get pods
```

**Expected Output:**
```
NAME                          READY   STATUS    RESTARTS   AGE
web-deploy-xxxxxxxxxx-xxxxx   1/1     Running   0          10m
```

**Note:** Currently, you have 1 replica running.

---

### 2. Scale the Deployment to 3 Replicas

Execute the scaling command:
```bash
kubectl scale deployment web-deploy --replicas=3
```

**Expected Output:**
```
deployment.apps/web-deploy scaled
```

**What's Happening:**
- The Deployment updates the desired replica count to 3
- The ReplicaSet detects the change
- The ReplicaSet creates 2 additional Pods to meet the desired state
- Kubernetes scheduler assigns Pods to available nodes

---

### 3. Verify the Scaled Deployment

#### 3.1 Check Deployment Status
```bash
kubectl get deployments
```

**Expected Output:**
```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
web-deploy   3/3     3            3           15m
```

**Observation Notes:**
- `READY` changed from `1/1` to `3/3`
- All 3 replicas are up-to-date and available
- The deployment name remains the same

---

#### 3.2 Check ReplicaSet Status
```bash
kubectl get rs
```

**Expected Output:**
```
NAME                    DESIRED   CURRENT   READY   AGE
web-deploy-xxxxxxxxxx   3         3         3       15m
```

**Critical Observation - ReplicaSet Changes:**
- **Same ReplicaSet**: The existing ReplicaSet is updated (notice the same name and hash)
- **DESIRED** changed from `1` to `3`
- **CURRENT** changed from `1` to `3`
- **READY** changed from `1` to `3`
- **AGE** remains the same (no new ReplicaSet was created)
- **NO new ReplicaSet was created** - the existing one manages all replicas

**Key Insight:** Scaling doesn't create a new ReplicaSet; it updates the existing one!

---

#### 3.3 Check Pods Status
```bash
kubectl get pods
```

**Expected Output:**
```
NAME                          READY   STATUS    RESTARTS   AGE
web-deploy-xxxxxxxxxx-xxxxx   1/1     Running   0          15m
web-deploy-xxxxxxxxxx-yyyyy   1/1     Running   0          30s
web-deploy-xxxxxxxxxx-zzzzz   1/1     Running   0          30s
```

**Observation Notes:**
- **3 Pods total** (increased from 1)
- All share the same ReplicaSet prefix
- Original Pod remains running
- 2 new Pods were created
- New Pods have different unique suffixes
- AGE shows original Pod is older, new Pods are recent

---

### 4. Watch Scaling in Real-Time (Optional)

To observe scaling as it happens, open a new terminal and run:
```bash
kubectl get pods -w
```

Then in your original terminal, scale to a different number:
```bash
kubectl scale deployment web-deploy --replicas=5
```

**You'll see:**
- New Pods being created in real-time
- Status transitions: `Pending` → `ContainerCreating` → `Running`

Scale back down:
```bash
kubectl scale deployment web-deploy --replicas=2
```

**You'll see:**
- Pods being terminated
- Status transitions: `Running` → `Terminating`

*(Press `Ctrl+C` to stop watching)*

Return to 3 replicas for consistency:
```bash
kubectl scale deployment web-deploy --replicas=3
```

---

### 5. Detailed Verification Commands

#### 5.1 View Detailed Deployment Information
```bash
kubectl describe deployment web-deploy
```

**Key Information to Note:**
```
Name:                   web-deploy
Namespace:              default
Replicas:               3 desired | 3 updated | 3 total | 3 available
StrategyType:           RollingUpdate
Pod Template:
  Containers:
   nginx:
    Image:        nginx
Events:
  Type    Reason             Message
  ----    ------             -------
  Normal  ScalingReplicaSet  Scaled up replica set web-deploy-xxxxxxxxxx to 3
```

**Note:** The Events section shows the scaling action.

---

#### 5.2 View Detailed ReplicaSet Information
```bash
kubectl describe rs <replicaset-name>
```

**Key Information to Note:**
```
Name:           web-deploy-xxxxxxxxxx
Namespace:      default
Selector:       app=web-deploy,pod-template-hash=xxxxxxxxxx
Replicas:       3 current / 3 desired
Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Events:
  Type    Reason            Message
  ----    ------            -------
  Normal  SuccessfulCreate  Created pod: web-deploy-xxxxxxxxxx-yyyyy
  Normal  SuccessfulCreate  Created pod: web-deploy-xxxxxxxxxx-zzzzz
```

**Note:** Events show when new Pods were created during scaling.

---

#### 5.3 View Pod Distribution Across Nodes
```bash
kubectl get pods -o wide
```

**Expected Output:**
```
NAME                          READY   STATUS    RESTARTS   AGE   IP           NODE
web-deploy-xxxxxxxxxx-xxxxx   1/1     Running   0          20m   172.17.0.3   minikube
web-deploy-xxxxxxxxxx-yyyyy   1/1     Running   0          5m    172.17.0.4   minikube
web-deploy-xxxxxxxxxx-zzzzz   1/1     Running   0          5m    172.17.0.5   minikube
```

**Observation Notes:**
- Each Pod gets a unique IP address
- In Minikube, all Pods typically run on the same node
- In multi-node clusters, Pods would be distributed across nodes

---

## How ReplicaSet Changes During Scaling

### Summary of ReplicaSet Behavior

| Aspect | Before Scaling | After Scaling | Changed? |
|--------|----------------|---------------|----------|
| ReplicaSet Name | `web-deploy-xxxxxxxxxx` | `web-deploy-xxxxxxxxxx` | No |
| ReplicaSet Count | 1 ReplicaSet | 1 ReplicaSet | No |
| DESIRED Replicas | 1 | 3 | Yes |
| CURRENT Replicas | 1 | 3 | Yes |
| READY Replicas | 1 | 3 | Yes |
| Pod Count | 1 Pod | 3 Pods | Yes |
| AGE | Original | Same | No |

### Key Observations

- **The same ReplicaSet is updated** - No new ReplicaSet is created during scaling  
- **Only the replica count changes** - The ReplicaSet maintains the same template  
- **New Pods inherit the same spec** - All Pods are identical copies  
- **Original Pods remain running** - Scaling up doesn't replace existing Pods  
- **Scaling is declarative** - You specify desired state, Kubernetes handles the rest  

### When Does a New ReplicaSet Get Created?

**New ReplicaSet is created ONLY when:**
- You update the Pod template (e.g., change image version)
- You modify container specifications
- You change environment variables, volumes, or labels in the template

**New ReplicaSet is NOT created when:**
- You scale up or down
- You modify replica count only

---

## For .NET Core Developers

### Scaling a .NET Core API

**Scenario:** You have a .NET Core Web API that needs to handle increased traffic.

#### Create a .NET Core Deployment
```bash
kubectl create deployment dotnet-api --image=mcr.microsoft.com/dotnet/samples:aspnetapp
```

#### Scale for High Availability
```bash
kubectl scale deployment dotnet-api --replicas=3
```

**Benefits:**
- Load distribution across 3 instances
- High availability (if one Pod fails, 2 remain)
- Zero-downtime deployments (covered in future tasks)

#### Real-World .NET Scaling Example
```bash
# Development environment - 1 replica
kubectl scale deployment dotnet-api --replicas=1

# Staging environment - 2 replicas
kubectl scale deployment dotnet-api --replicas=2

# Production environment - 5 replicas
kubectl scale deployment dotnet-api --replicas=5

# Black Friday / High traffic - 10 replicas
kubectl scale deployment dotnet-api --replicas=10
```

---

## Experiment: Scale Up and Down

### Scale Up to 5 Replicas
```bash
kubectl scale deployment web-deploy --replicas=5
kubectl get pods
```

**Observe:**
- 2 additional Pods are created
- All Pods share the same ReplicaSet
- Total: 5 running Pods

---

### Scale Down to 2 Replicas
```bash
kubectl scale deployment web-deploy --replicas=2
kubectl get pods
```

**Observe:**
- 3 Pods are terminated
- Kubernetes selects which Pods to remove
- Usually newest Pods are terminated first
- Total: 2 running Pods

---

### Return to 3 Replicas
```bash
kubectl scale deployment web-deploy --replicas=3
kubectl get deployments
kubectl get rs
kubectl get pods
```

---

## Alternative Scaling Methods

### Method 1: Using kubectl scale (Already covered)
```bash
kubectl scale deployment web-deploy --replicas=3
```

---

### Method 2: Using kubectl edit
```bash
kubectl edit deployment web-deploy
```

This opens the deployment YAML in your default editor. Find the `spec.replicas` field and change it:
```yaml
spec:
  replicas: 3  # Change this value
```

Save and close. Kubernetes automatically applies the changes.

---

### Method 3: Using kubectl patch
```bash
kubectl patch deployment web-deploy -p '{"spec":{"replicas":3}}'
```

---

### Method 4: Using YAML file (Best Practice)

Create a file `web-deploy.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-deploy
  template:
    metadata:
      labels:
        app: web-deploy
    spec:
      containers:
      - name: nginx
        image: nginx
```

Apply the configuration:
```bash
kubectl apply -f web-deploy.yaml
```

**Note:** This is the recommended approach for production environments.

---

## Monitoring and Verification

### Check Resource Usage
```bash
kubectl top pods
```

*(Requires metrics-server, may need to enable in Minikube)*

Enable metrics-server in Minikube:
```bash
minikube addons enable metrics-server
```

Wait a minute, then try again:
```bash
kubectl top pods
```

---

### View Pod Events
```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

This shows recent cluster events, including Pod creation during scaling.

---

### Check Pod Logs (All Replicas)
```bash
kubectl logs -l app=web-deploy --all-containers=true
```

This shows logs from all Pods with the `app=web-deploy` label.

---

## Troubleshooting

### Pods Stuck in Pending State

**Check:**
```bash
kubectl describe pod <pod-name>
```

**Common Causes:**
- Insufficient cluster resources (CPU/Memory)
- Image pull issues
- Node selector constraints

**Solution for Minikube:**
```bash
minikube stop
minikube start --cpus=4 --memory=4096
```

---

### Pods Not Scaling

**Verify deployment exists:**
```bash
kubectl get deployment web-deploy
```

**Check for typos in deployment name:**
```bash
kubectl get deployments
```

---

### ReplicaSet Not Creating Pods

**Check ReplicaSet status:**
```bash
kubectl describe rs <replicaset-name>
```

Look for error messages in the Events section.

---

## Load Balancing Verification

### Access Pods Through Service (Preview)

Create a service to access your Pods:
```bash
kubectl expose deployment web-deploy --type=NodePort --port=80
```

Get the service URL:
```bash
minikube service web-deploy --url
```

Test with curl (multiple requests):
```bash
for i in {1..10}; do curl -s $(minikube service web-deploy --url) | grep -i welcome; done
```

**Note:** Requests are load-balanced across all 3 Pod replicas!

Clean up the service:
```bash
kubectl delete service web-deploy
```

---

## Best Practices for Scaling

### For .NET Core Applications

1. **Start Small**: Begin with 2-3 replicas for redundancy
2. **Monitor Performance**: Use Application Insights or Prometheus
3. **Set Resource Limits**: Define CPU and memory limits in Pod specs
4. **Use Horizontal Pod Autoscaler (HPA)**: Automatically scale based on metrics
5. **Consider Stateless Design**: Scaled apps should be stateless for best results
6. **Health Checks**: Implement liveness and readiness probes
7. **Connection Strings**: Use ConfigMaps or Secrets for database connections
8. **Session State**: Use distributed cache (Redis) instead of in-memory sessions

---

## Summary of ReplicaSet Changes

### What Changed:
-Replica count (DESIRED, CURRENT, READY)  
-Number of Pods managed by the ReplicaSet  
-Deployment status showing new replica count  

### What Did NOT Change:
-ReplicaSet name or identifier  
-Number of ReplicaSets (still 1)  
-Pod template or specification  
-ReplicaSet creation timestamp  
-Container image or configuration  

### Important Conclusion:
**Scaling modifies the existing ReplicaSet's replica count without creating a new ReplicaSet. The ReplicaSet controller ensures the desired number of Pod replicas are running at all times.**

---

## Cleanup (Optional)

Scale back to 1 replica:
```bash
kubectl scale deployment web-deploy --replicas=1
kubectl get pods
```

Or delete the deployment entirely:
```bash
kubectl delete deployment web-deploy
```

Verify:
```bash
kubectl get deployments
kubectl get rs
kubectl get pods
```

---

## Key Takeaways

- Scaling is performed at the Deployment level  
- The same ReplicaSet handles the scaled replicas  
- New Pods are created; existing Pods remain running  
- Scaling is declarative - you specify desired state  
- ReplicaSets automatically maintain the desired replica count  
- Multiple scaling methods available (imperative and declarative)  
- Perfect for scaling .NET Core microservices and APIs  
- Horizontal scaling provides high availability and load distribution  

---

## Next Steps

- Learn about Horizontal Pod Autoscaler (HPA) for automatic scaling
- Explore rolling updates and rollback strategies
- Implement health checks (liveness and readiness probes)
- Deploy and scale a custom .NET Core application
- Study resource requests and limits for optimal scaling
- Investigate vertical pod autoscaling

---

## Additional Resources for .NET Developers

**Deploy .NET Core to Kubernetes:**
```bash
# Build and push your .NET app
docker build -t your-registry/dotnet-app:v1 .
docker push your-registry/dotnet-app:v1

# Create deployment
kubectl create deployment dotnet-app --image=your-registry/dotnet-app:v1

# Scale it
kubectl scale deployment dotnet-app --replicas=3
```

**Common .NET Kubernetes Patterns:**
- API Gateway pattern (multiple scaled microservices)
- Background worker services (scaled based on queue depth)
- Stateless web applications (easily scalable)
- gRPC services (high-performance inter-service communication)

---

**Task Completed!** You have successfully scaled a Deployment, observed ReplicaSet behavior, and documented how scaling affects Kubernetes resources. You're now ready to apply these concepts to real-world .NET Core applications!