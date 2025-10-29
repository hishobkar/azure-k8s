# Task 5: Update Deployment & Observe Rolling Update

**Duration:** 30 minutes  
**Environment:** WSL with Docker, kubectl, and Minikube  
**Target Audience:** .NET Core Developers

---

## Prerequisites
- Docker installed and running in WSL
- kubectl installed in WSL
- Minikube installed and cluster running in WSL
- Completed Task 4 (Deployment `app-deploy` should exist from deployment-demo.yaml)
- Basic understanding of container images and versioning

---

## Objectives
- Understand Kubernetes rolling update strategy
- Update container images in a running Deployment
- Monitor deployment rollout progress in real-time
- View deployment revision history
- Perform rollback to previous versions
- Apply zero-downtime deployment concepts to .NET Core applications

---

## Introduction: Rolling Updates

### What is a Rolling Update?

A rolling update gradually replaces old Pods with new ones, ensuring:
- Zero downtime during updates
- Gradual rollout of changes
- Automatic rollback on failure
- Controlled update process

### Rolling Update Process

1. New ReplicaSet is created with updated Pod template
2. New Pods are created gradually
3. Old Pods are terminated gradually
4. Process continues until all Pods are updated
5. Old ReplicaSet is retained for rollback purposes

---

## Task Steps

### 1. Verify Current Deployment State

Ensure the deployment from Task 4 exists:
```bash
kubectl get deployments
```

**Expected Output:**
```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
app-deploy   2/2     2            2           15m
```

Check current image:
```bash
kubectl describe deployment app-deploy | grep Image
```

**Expected Output:**
```
    Image:        nginx
```

Check current Pods:
```bash
kubectl get pods -l app=myapp
```

**Expected Output:**
```
NAME                          READY   STATUS    RESTARTS   AGE
app-deploy-xxxxxxxxxx-aaaaa   1/1     Running   0          15m
app-deploy-xxxxxxxxxx-bbbbb   1/1     Running   0          15m
```

Note the ReplicaSet hash (xxxxxxxxxx) - this will change after the update.

---

### 2. Check Current ReplicaSets
```bash
kubectl get rs
```

**Expected Output:**
```
NAME                    DESIRED   CURRENT   READY   AGE
app-deploy-xxxxxxxxxx   2         2         2       15m
```

Note: Currently only ONE ReplicaSet exists.

---

### 3. Update the Container Image

Update the nginx image from `nginx` (latest) to `nginx:alpine`:
```bash
kubectl set image deployment/app-deploy myapp-container=nginx:alpine
```

**Command Breakdown:**
- `kubectl set image` - Command to update container image
- `deployment/app-deploy` - Target deployment
- `myapp-container` - Container name (from YAML spec.containers.name)
- `nginx:alpine` - New image with tag

**Expected Output:**
```
deployment.apps/app-deploy image updated
```

**Alternative Method (Declarative):**
Edit deployment-demo.yaml and change:
```yaml
containers:
  - name: myapp-container
    image: nginx:alpine  # Changed from nginx
```

Then apply:
```bash
kubectl apply -f deployment-demo.yaml
```

---

### 4. Monitor the Rollout in Real-Time

Watch the rollout status:
```bash
kubectl rollout status deployment/app-deploy
```

**Expected Output (live updates):**
```
Waiting for deployment "app-deploy" rollout to finish: 1 out of 2 new replicas have been updated...
Waiting for deployment "app-deploy" rollout to finish: 1 out of 2 new replicas have been updated...
Waiting for deployment "app-deploy" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "app-deploy" rollout to finish: 1 old replicas are pending termination...
deployment "app-deploy" successfully rolled out
```

**What's Happening:**
1. New ReplicaSet created with nginx:alpine
2. New Pod created (1/2 updated)
3. Old Pod terminated after new Pod is ready
4. Second new Pod created (2/2 updated)
5. Second old Pod terminated
6. Rollout complete

---

### 5. Observe Rolling Update in Another Terminal (Optional)

Open a second terminal and run:
```bash
kubectl get pods -l app=myapp -w
```

The `-w` flag watches for changes in real-time.

**You'll see:**
```
NAME                          READY   STATUS              RESTARTS   AGE
app-deploy-xxxxxxxxxx-aaaaa   1/1     Running             0          20m
app-deploy-xxxxxxxxxx-bbbbb   1/1     Running             0          20m
app-deploy-yyyyyyyyyy-ccccc   0/1     ContainerCreating   0          1s
app-deploy-yyyyyyyyyy-ccccc   1/1     Running             0          5s
app-deploy-xxxxxxxxxx-aaaaa   1/1     Terminating         0          20m
app-deploy-yyyyyyyyyy-ddddd   0/1     Pending             0          0s
app-deploy-yyyyyyyyyy-ddddd   0/1     ContainerCreating   0          0s
app-deploy-yyyyyyyyyy-ddddd   1/1     Running             0          4s
app-deploy-xxxxxxxxxx-bbbbb   1/1     Terminating         0          20m
```

**Observations:**
- New Pods with different hash (yyyyyyyyyy) are created
- Old Pods (xxxxxxxxxx) are terminated after new ones are ready
- Zero downtime: always at least 2 Pods running

Press `Ctrl+C` to stop watching.

---

### 6. Verify the Update

#### 6.1 Check Deployment Status
```bash
kubectl get deployments
```

**Expected Output:**
```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
app-deploy   2/2     2            2           25m
```

All replicas are up-to-date with the new image.

---

#### 6.2 Check ReplicaSets
```bash
kubectl get rs
```

**Expected Output:**
```
NAME                    DESIRED   CURRENT   READY   AGE
app-deploy-xxxxxxxxxx   0         0         0       25m
app-deploy-yyyyyyyyyy   2         2         2       2m
```

**Critical Observations:**
- TWO ReplicaSets now exist
- OLD ReplicaSet (xxxxxxxxxx): DESIRED=0, scaled down but retained
- NEW ReplicaSet (yyyyyyyyyy): DESIRED=2, managing current Pods
- Old ReplicaSet is kept for rollback purposes

---

#### 6.3 Check Pods
```bash
kubectl get pods -l app=myapp
```

**Expected Output:**
```
NAME                          READY   STATUS    RESTARTS   AGE
app-deploy-yyyyyyyyyy-ccccc   1/1     Running   0          3m
app-deploy-yyyyyyyyyy-ddddd   1/1     Running   0          3m
```

**Observations:**
- New Pods with different ReplicaSet hash
- Both Pods running the updated image
- Old Pods are completely terminated

---

#### 6.4 Verify New Image
```bash
kubectl describe deployment app-deploy | grep Image
```

**Expected Output:**
```
    Image:        nginx:alpine
```

Or check a specific Pod:
```bash
kubectl describe pod <pod-name> | grep Image
```

**Expected Output:**
```
    Image:          nginx:alpine
    Image ID:       docker.io/library/nginx@sha256:...
```

---

### 7. View Rollout History

Check the deployment revision history:
```bash
kubectl rollout history deployment/app-deploy
```

**Expected Output:**
```
deployment.apps/app-deploy
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

**Explanation:**
- REVISION 1: Original deployment (nginx image)
- REVISION 2: Updated deployment (nginx:alpine image)
- CHANGE-CAUSE is empty because we didn't annotate the changes

---

### 8. View Specific Revision Details

Check details of revision 1:
```bash
kubectl rollout history deployment/app-deploy --revision=1
```

**Expected Output:**
```
deployment.apps/app-deploy with revision #1
Pod Template:
  Labels:	app=myapp
	pod-template-hash=xxxxxxxxxx
  Containers:
   myapp-container:
    Image:	nginx
    Port:	<none>
    Host Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

Check details of revision 2:
```bash
kubectl rollout history deployment/app-deploy --revision=2
```

**Expected Output:**
```
deployment.apps/app-deploy with revision #2
Pod Template:
  Labels:	app=myapp
	pod-template-hash=yyyyyyyyyy
  Containers:
   myapp-container:
    Image:	nginx:alpine
    Port:	<none>
    Host Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

**Comparison:**
- Revision 1: Image = nginx
- Revision 2: Image = nginx:alpine

---

### 9. Add Change Annotations (Best Practice)

Update the image again with a change annotation:
```bash
kubectl set image deployment/app-deploy myapp-container=nginx:1.21 \
  --record
```

**Note:** The `--record` flag is deprecated but still functional. It records the command in the revision history.

**Better approach - using kubectl annotate:**
```bash
kubectl set image deployment/app-deploy myapp-container=nginx:1.21
kubectl annotate deployment/app-deploy kubernetes.io/change-cause="Updated to nginx:1.21 for security patch"
```

Monitor the rollout:
```bash
kubectl rollout status deployment/app-deploy
```

View updated history:
```bash
kubectl rollout history deployment/app-deploy
```

**Expected Output:**
```
deployment.apps/app-deploy
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         Updated to nginx:1.21 for security patch
```

Now you can see what changed in revision 3.

---

### 10. Perform a Rollback

Rollback to the previous version (nginx:alpine):
```bash
kubectl rollout undo deployment/app-deploy
```

**Expected Output:**
```
deployment.apps/app-deploy rolled back
```

**What Happens:**
- Kubernetes reverts to the previous revision (revision 2)
- The ReplicaSet for revision 2 is scaled up
- Current ReplicaSet (revision 3) is scaled down
- Same rolling update process, but in reverse

Monitor the rollback:
```bash
kubectl rollout status deployment/app-deploy
```

**Expected Output:**
```
Waiting for deployment "app-deploy" rollout to finish: 1 out of 2 new replicas have been updated...
deployment "app-deploy" successfully rolled out
```

---

### 11. Verify the Rollback

Check current image:
```bash
kubectl describe deployment app-deploy | grep Image
```

**Expected Output:**
```
    Image:        nginx:alpine
```

Image is back to nginx:alpine (revision 2).

Check ReplicaSets:
```bash
kubectl get rs
```

**Expected Output:**
```
NAME                    DESIRED   CURRENT   READY   AGE
app-deploy-xxxxxxxxxx   0         0         0       35m
app-deploy-yyyyyyyyyy   2         2         2       12m
app-deploy-zzzzzzzzzz   0         0         0       2m
```

**Observations:**
- THREE ReplicaSets now exist
- ReplicaSet yyyyyyyyyy (nginx:alpine) is active again with DESIRED=2
- ReplicaSet zzzzzzzzzz (nginx:1.21) is scaled down to DESIRED=0
- All historical ReplicaSets are retained

---

### 12. Rollback to Specific Revision

Rollback to a specific revision (revision 1 - original nginx):
```bash
kubectl rollout undo deployment/app-deploy --to-revision=1
```

**Expected Output:**
```
deployment.apps/app-deploy rolled back
```

Monitor:
```bash
kubectl rollout status deployment/app-deploy
```

Verify:
```bash
kubectl describe deployment app-deploy | grep Image
```

**Expected Output:**
```
    Image:        nginx
```

Now back to the original nginx image (no tag = latest).

---

### 13. View Updated History
```bash
kubectl rollout history deployment/app-deploy
```

**Expected Output:**
```
deployment.apps/app-deploy
REVISION  CHANGE-CAUSE
2         <none>
3         Updated to nginx:1.21 for security patch
4         <none>
5         <none>
```

**Observations:**
- Revision 1 is gone (becomes revision 5 after rollback)
- Revisions are reused when rolling back
- Change cause annotations are preserved

---

## Understanding Rolling Update Strategy

### Default Strategy Parameters

View the deployment's update strategy:
```bash
kubectl describe deployment app-deploy | grep -A 5 "StrategyType"
```

**Expected Output:**
```
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
```

**Parameters Explained:**
- **StrategyType: RollingUpdate** - Gradual Pod replacement
- **maxUnavailable: 25%** - Maximum number of Pods that can be unavailable during update
- **maxSurge: 25%** - Maximum number of Pods that can be created above desired count

**For 2 replicas:**
- maxUnavailable = 25% of 2 = 0.5 (rounded up to 1)
- maxSurge = 25% of 2 = 0.5 (rounded up to 1)

**Update Process:**
1. One new Pod is created (surge)
2. One old Pod is terminated after new Pod is ready
3. Second new Pod is created
4. Second old Pod is terminated

---

### Customize Rolling Update Strategy

Edit deployment-demo.yaml to customize the strategy:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deploy
spec:
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0  # Ensures zero downtime
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp-container
          image: nginx:alpine
```

Apply:
```bash
kubectl apply -f deployment-demo.yaml
```

**Benefits of maxUnavailable: 0:**
- Guarantees all Pods remain available during update
- Essential for production environments
- Requires sufficient cluster resources for surge Pods

---

## For .NET Core Developers

### Update .NET Core Application

Create a deployment for a .NET Core app:

**dotnet-api-deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dotnet-api
  annotations:
    kubernetes.io/change-cause: "Initial deployment - v1.0.0"
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: dotnet-api
  template:
    metadata:
      labels:
        app: dotnet-api
        version: v1.0.0
    spec:
      containers:
        - name: api
          image: myregistry.azurecr.io/dotnet-api:1.0.0
          ports:
            - containerPort: 8080
          env:
            - name: ASPNETCORE_ENVIRONMENT
              value: "Production"
```

Apply:
```bash
kubectl apply -f dotnet-api-deployment.yaml
```

---

### Update to New Version

Update the image to v2.0.0:
```bash
kubectl set image deployment/dotnet-api api=myregistry.azurecr.io/dotnet-api:2.0.0
kubectl annotate deployment/dotnet-api kubernetes.io/change-cause="Updated to v2.0.0 - Added new features"
```

Monitor the rollout:
```bash
kubectl rollout status deployment/dotnet-api
```

---

### Typical .NET Update Workflow
```bash
# 1. Build new Docker image
docker build -t myregistry.azurecr.io/dotnet-api:2.1.0 .

# 2. Run tests
dotnet test

# 3. Push to registry
docker push myregistry.azurecr.io/dotnet-api:2.1.0

# 4. Update Kubernetes deployment
kubectl set image deployment/dotnet-api api=myregistry.azurecr.io/dotnet-api:2.1.0
kubectl annotate deployment/dotnet-api kubernetes.io/change-cause="Release 2.1.0 - Bug fixes"

# 5. Monitor rollout
kubectl rollout status deployment/dotnet-api

# 6. Verify deployment
kubectl get pods -l app=dotnet-api
kubectl logs -l app=dotnet-api --tail=50

# 7. If issues occur, rollback
kubectl rollout undo deployment/dotnet-api
```

---

### Zero-Downtime Deployment for .NET APIs

**Key Requirements:**

1. **Health Checks** (Liveness and Readiness Probes)
```yaml
spec:
  containers:
    - name: api
      image: myregistry.azurecr.io/dotnet-api:2.0.0
      livenessProbe:
        httpGet:
          path: /health/live
          port: 8080
        initialDelaySeconds: 30
        periodSeconds: 10
      readinessProbe:
        httpGet:
          path: /health/ready
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 5
```

2. **Graceful Shutdown** (in Program.cs)
```csharp
var host = builder.Build();

var lifetime = host.Services.GetRequiredService<IHostApplicationLifetime>();
lifetime.ApplicationStopping.Register(() =>
{
    // Complete in-flight requests
    Thread.Sleep(5000);
});

await host.RunAsync();
```

3. **Rolling Update Strategy**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

---

### .NET Core Health Checks Implementation

**Startup configuration:**
```csharp
// Program.cs or Startup.cs
builder.Services.AddHealthChecks()
    .AddCheck("self", () => HealthCheckResult.Healthy())
    .AddSqlServer(connectionString)
    .AddRedis(redisConnection);

var app = builder.Build();

app.MapHealthChecks("/health/ready", new HealthCheckOptions
{
    Predicate = _ => true
});

app.MapHealthChecks("/health/live", new HealthCheckOptions
{
    Predicate = check => check.Name == "self"
});
```

**Why separate endpoints?**
- **/health/live**: Basic check - is the app running?
- **/health/ready**: Full check - is the app ready to accept traffic? (includes database, cache, etc.)

---

## Advanced Rollout Commands

### Pause a Rollout

If you need to stop a rollout in progress:
```bash
kubectl rollout pause deployment/app-deploy
```

The rollout pauses at its current state (some Pods old, some new).

Resume the rollout:
```bash
kubectl rollout resume deployment/app-deploy
```

**Use Case:**
- Pause to verify canary deployment behavior
- Investigate issues before completing rollout

---

### Restart Deployment

Restart all Pods without changing the image:
```bash
kubectl rollout restart deployment/app-deploy
```

**Use Cases:**
- Apply ConfigMap or Secret changes
- Recover from application issues
- Force Pod recreation

---

### Watch Rollout Progress

Continuously monitor rollout:
```bash
kubectl rollout status deployment/app-deploy --watch
```

---

## Troubleshooting Rolling Updates

### Update Stuck or Failed

Check deployment status:
```bash
kubectl describe deployment app-deploy
```

Look for:
- Events section for errors
- Conditions section for status

**Common Issues:**
- Image pull errors (wrong tag, registry credentials)
- Resource constraints (insufficient CPU/memory)
- Failed health checks (readiness probe failures)
- Application crashes on startup

---

### View Failed Pod Logs

If new Pods are failing:
```bash
kubectl get pods -l app=myapp
kubectl logs <failing-pod-name>
kubectl describe pod <failing-pod-name>
```

---

### Automatic Rollback

Kubernetes can automatically rollback if:
- New Pods fail readiness probes
- ProgressDeadlineSeconds is exceeded

Set progress deadline:
```yaml
spec:
  progressDeadlineSeconds: 600  # 10 minutes
```

If rollout doesn't complete in 10 minutes, it's marked as failed.

Check with:
```bash
kubectl rollout status deployment/app-deploy
```

---

### Manual Health Check

If you suspect an issue with updated Pods:
```bash
# Get a Pod name
POD_NAME=$(kubectl get pods -l app=myapp -o jsonpath='{.items[0].metadata.name}')

# Port-forward to test
kubectl port-forward $POD_NAME 8080:80

# Test in another terminal
curl http://localhost:8080
```

---

## Rollout History Management

### Limit Revision History

By default, Kubernetes keeps 10 revisions. Customize this:
```yaml
spec:
  revisionHistoryLimit: 5  # Keep only 5 revisions
```

Apply:
```bash
kubectl apply -f deployment-demo.yaml
```

**Benefits:**
- Reduces cluster resource usage
- Cleans up old ReplicaSets
- Limits rollback options to recent versions

---

### Clear Revision History

Old ReplicaSets with 0 replicas can be manually deleted:
```bash
kubectl delete rs <old-replicaset-name>
```

**Warning:** This removes the ability to rollback to that revision.

---

## Best Practices for Production

### 1. Always Use Tagged Images

**Bad:**
```yaml
image: nginx  # Uses 'latest' tag - unpredictable
```

**Good:**
```yaml
image: nginx:1.21.6  # Specific version - reproducible
```

### 2. Annotate Changes
```bash
kubectl annotate deployment/app-deploy \
  kubernetes.io/change-cause="Release v2.1.0 - Security patches"
```

### 3. Test Before Production
```bash
# Deploy to dev/staging first
kubectl apply -f deployment.yaml --namespace=staging

# Monitor for issues
kubectl rollout status deployment/app-deploy -n staging

# If successful, deploy to production
kubectl apply -f deployment.yaml --namespace=production
```

### 4. Set Resource Limits
```yaml
containers:
  - name: api
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

### 5. Implement Health Checks

Always include liveness and readiness probes for zero-downtime deployments.

### 6. Use Blue-Green or Canary Deployments

For critical applications, consider advanced deployment strategies:
- **Blue-Green**: Two identical environments, switch traffic
- **Canary**: Gradual rollout to subset of users

### 7. Monitor and Alert

Integrate with monitoring tools:
- Prometheus + Grafana
- Application Insights (for .NET)
- ELK Stack (Elasticsearch, Logstash, Kibana)

---

## Cleanup

### Option 1: Keep Deployment, Reset to Original State
```bash
kubectl rollout undo deployment/app-deploy --to-revision=1
```

### Option 2: Delete Deployment
```bash
kubectl delete deployment app-deploy
```

Verify:
```bash
kubectl get deployments
kubectl get rs
kubectl get pods
```

---

## Comparison: Update Strategies

### Rolling Update (Default)

**Pros:**
- Zero downtime
- Gradual rollout
- Easy rollback
- Automatic

**Cons:**
- Mixed versions temporarily
- Slower than recreate
- Requires extra resources

**Use Case:** Production applications requiring high availability

---

### Recreate Strategy
```yaml
spec:
  strategy:
    type: Recreate
```

**Pros:**
- Simple and fast
- No mixed versions
- Lower resource usage

**Cons:**
- Downtime during update
- All-or-nothing

**Use Case:** Development environments, batch jobs, applications that can tolerate downtime

---

## Key Takeaways

Rolling updates enable zero-downtime deployments
New ReplicaSet is created, old ReplicaSet is retained
Kubernetes maintains revision history for rollbacks
kubectl rollout commands manage deployment lifecycle
Health checks are critical for safe updates
Annotate changes for better tracking
Always use specific image tags in production
Rolling back is as simple as one command
.NET Core apps benefit from gradual, controlled updates
Monitor rollout status to catch issues early

---

## Next Steps

- Implement health checks in your .NET Core applications
- Practice blue-green deployment strategies
- Explore canary deployments with Istio or Flagger
- Set up CI/CD pipelines with automated rollouts
- Implement monitoring and alerting for deployments
- Learn about Helm for package management
- Study GitOps with ArgoCD or Flux
- Explore progressive delivery patterns

---

## Real-World .NET Deployment Example

**Complete workflow for a .NET Core API update:**
```bash
# Step 1: Code changes and testing
git checkout -b feature/new-api
# ... make changes ...
dotnet test
git commit -am "Add new API endpoints"
git push

# Step 2: Build and tag Docker image
docker build -t myregistry.azurecr.io/my-api:2.1.0 .
docker tag myregistry.azurecr.io/my-api:2.1.0 myregistry.azurecr.io/my-api:latest

# Step 3: Push to registry
docker push myregistry.azurecr.io/my-api:2.1.0
docker push myregistry.azurecr.io/my-api:latest

# Step 4: Update Kubernetes deployment
kubectl set image deployment/my-api api=myregistry.azurecr.io/my-api:2.1.0 -n production
kubectl annotate deployment/my-api kubernetes.io/change-cause="Release 2.1.0 - New API endpoints" -n production

# Step 5: Monitor rollout
kubectl rollout status deployment/my-api -n production --watch

# Step 6: Verify health
kubectl get pods -l app=my-api -n production
kubectl logs -l app=my-api -n production --tail=50 --since=5m

# Step 7: Smoke test
curl https://my-api.production.example.com/health
curl https://my-api.production.example.com/api/v1/users

# Step 8: If issues, immediate rollback
kubectl rollout undo deployment/my-api -n production

# Step 9: Document deployment
kubectl rollout history deployment/my-api -n production
```

**Task Completed!** You have successfully updated a Deployment, monitored rolling updates, viewed revision history, and performed rollbacks. You now understand how to safely deploy updates to production .NET Core applications with zero downtime!