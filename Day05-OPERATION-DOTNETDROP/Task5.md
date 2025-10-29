# Task 5: Scale, Update & Cleanup (30 minutes)

## Prerequisites
- Completed Tasks 1-4 (Application deployed and accessible)
- kubectl configured and connected to Minikube
- Current deployment running with 1 replica

## Part A: Scaling the Application

### Step 1: Check Current Deployment Status

View current deployment and replicas:
```bash
kubectl get deployments
```

**Expected Output:**
```
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
dotnet-deploy   1/1     1            1           15m
```

View current pods:
```bash
kubectl get pods
```

**Expected Output:**
```
NAME                             READY   STATUS    RESTARTS   AGE
dotnet-deploy-xxxxxxxxxx-xxxxx   1/1     Running   0          15m
```

Check detailed deployment information:
```bash
kubectl describe deployment dotnet-deploy
```

### Step 2: Scale Deployment to 2 Replicas

Scale the deployment using kubectl command:
```bash
kubectl scale deployment dotnet-deploy --replicas=2
```

**Expected Output:**
```
deployment.apps/dotnet-deploy scaled
```

**What This Does:**
- Creates an additional pod running the same container image
- Load balances traffic across both pods
- Provides high availability (if one pod fails, the other continues)
- Increases application capacity

### Step 3: Verify Scaling Operation

Check deployment status immediately after scaling:
```bash
kubectl get deployments
```

**Expected Output:**
```
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
dotnet-deploy   2/2     2            2           16m
```

**Understanding the Output:**
- **READY:** 2/2 means 2 out of 2 desired replicas are ready
- **UP-TO-DATE:** Number of replicas updated to latest configuration
- **AVAILABLE:** Number of replicas available to serve requests

Watch pods being created in real-time:
```bash
kubectl get pods -w
```

**Expected Output:**
```
NAME                             READY   STATUS    RESTARTS   AGE
dotnet-deploy-xxxxxxxxxx-xxxxx   1/1     Running   0          16m
dotnet-deploy-xxxxxxxxxx-yyyyy   0/1     Pending   0          2s
dotnet-deploy-xxxxxxxxxx-yyyyy   0/1     ContainerCreating   0          4s
dotnet-deploy-xxxxxxxxxx-yyyyy   1/1     Running             0          8s
```

Press `Ctrl+C` to stop watching.

### Step 4: Verify Both Pods Are Running

List all pods:
```bash
kubectl get pods
```

**Expected Output:**
```
NAME                             READY   STATUS    RESTARTS   AGE
dotnet-deploy-xxxxxxxxxx-xxxxx   1/1     Running   0          17m
dotnet-deploy-xxxxxxxxxx-yyyyy   1/1     Running   0          45s
```

Get more detailed information:
```bash
kubectl get pods -o wide
```

**Expected Output:**
```
NAME                             READY   STATUS    RESTARTS   AGE   IP           NODE       
dotnet-deploy-xxxxxxxxxx-xxxxx   1/1     Running   0          17m   172.17.0.3   minikube
dotnet-deploy-xxxxxxxxxx-yyyyy   1/1     Running   0          50s   172.17.0.4   minikube
```

**Note:** Each pod has its own IP address.

### Step 5: Verify Load Balancing

Check service endpoints to see both pods registered:
```bash
kubectl get endpoints dotnet-service
```

**Expected Output:**
```
NAME             ENDPOINTS                     AGE
dotnet-service   172.17.0.3:80,172.17.0.4:80   20m
```

Both pod IPs should be listed as endpoints.

Test load balancing by sending multiple requests:
```bash
# In one terminal, watch logs from all pods
kubectl logs -f -l app=dotnetapi --prefix=true

# In another terminal, send requests
for i in {1..10}; do
  curl -s http://$(minikube ip):30081/WeatherForecast > /dev/null
  echo "Request $i completed"
  sleep 1
done
```

You should see logs from both pods showing requests being distributed.

### Step 6: Test High Availability

Simulate pod failure by deleting one pod:
```bash
# Get pod name
kubectl get pods

# Delete one pod
kubectl delete pod <pod-name>
```

Immediately check pod status:
```bash
kubectl get pods -w
```

**Expected Behavior:**
- Deleted pod shows "Terminating" status
- New pod is automatically created to maintain 2 replicas
- Service continues to work without interruption

Test API during pod recreation:
```bash
# While pod is being recreated
curl http://$(minikube ip):30081/WeatherForecast
```

**Expected:** API still responds (served by remaining pod).

### Step 7: Scale to Different Replica Counts

Scale up to 3 replicas:
```bash
kubectl scale deployment dotnet-deploy --replicas=3
kubectl get pods
```

Scale down to 1 replica:
```bash
kubectl scale deployment dotnet-deploy --replicas=1
kubectl get pods
```

**Note:** When scaling down, Kubernetes terminates excess pods gracefully.

Scale back to 2 replicas for remaining exercises:
```bash
kubectl scale deployment dotnet-deploy --replicas=2
kubectl get pods
```

### Step 8: Alternative - Update Deployment YAML

Instead of using kubectl scale, you can edit the YAML file:

Edit `deployment.yaml`:
```bash
nano deployment.yaml
# OR
code deployment.yaml
```

Change the replicas field:
```yaml
spec:
  replicas: 2  # Change this value
```

Apply the changes:
```bash
kubectl apply -f deployment.yaml
```

**Expected Output:**
```
deployment.apps/dotnet-deploy configured
```

This method is preferred for production as changes are version controlled.

## Part B: Updating the Application

### Step 9: Make Changes to Application Code

Navigate to your project directory:
```bash
cd ~/MinikubeDemoAPI
# OR wherever your project is located
```

Edit the WeatherForecastController (example change):
```bash
nano Controllers/WeatherForecastController.cs
```

Add or modify an endpoint:
```csharp
[HttpGet("version")]
public IActionResult GetVersion()
{
    return Ok(new { 
        version = "2.0", 
        message = "Updated version deployed!", 
        timestamp = DateTime.UtcNow 
    });
}
```

Save the file.

### Step 10: Rebuild Docker Image

Ensure you're using Minikube's Docker daemon:
```bash
eval $(minikube docker-env)
```

Rebuild the image with a new tag:
```bash
docker build -t minikube-demo-api:v2 .
```

**Best Practice:** Use version tags instead of overwriting `latest`.

Alternatively, rebuild with `latest` tag:
```bash
docker build -t minikube-demo-api:latest .
```

Verify the new image:
```bash
docker images | grep minikube-demo-api
```

### Step 11: Update Deployment with New Image

**Option 1: Update using kubectl set image**
```bash
kubectl set image deployment/dotnet-deploy dotnet-container=minikube-demo-api:v2
```

**Option 2: Edit deployment directly**
```bash
kubectl edit deployment dotnet-deploy
```

Find the `image:` field and update it to `minikube-demo-api:v2`.

**Option 3: Update YAML and reapply**

Edit `deployment.yaml`:
```yaml
spec:
  containers:
    - name: dotnet-container
      image: minikube-demo-api:v2
```

Apply changes:
```bash
kubectl apply -f deployment.yaml
```

### Step 12: Monitor Rolling Update

Watch the rollout process:
```bash
kubectl rollout status deployment/dotnet-deploy
```

**Expected Output:**
```
Waiting for deployment "dotnet-deploy" rollout to finish: 1 out of 2 new replicas have been updated...
Waiting for deployment "dotnet-deploy" rollout to finish: 1 old replicas are pending termination...
deployment "dotnet-deploy" successfully rolled out
```

Watch pods during update:
```bash
kubectl get pods -w
```

You'll see:
- New pods with new image being created
- Old pods being terminated
- Zero downtime rolling update

### Step 13: Verify Update

Check if new version is deployed:
```bash
curl http://$(minikube ip):30081/version
```

**Expected Output:**
```json
{
  "version": "2.0",
  "message": "Updated version deployed!",
  "timestamp": "2025-10-30T11:15:30.123Z"
}
```

Check deployment history:
```bash
kubectl rollout history deployment/dotnet-deploy
```

**Expected Output:**
```
deployment.apps/dotnet-deploy 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

### Step 14: Rollback Update (If Needed)

If the update has issues, rollback to previous version:
```bash
kubectl rollout undo deployment/dotnet-deploy
```

Rollback to specific revision:
```bash
kubectl rollout undo deployment/dotnet-deploy --to-revision=1
```

Check rollback status:
```bash
kubectl rollout status deployment/dotnet-deploy
```

### Step 15: View Deployment Details

Get detailed information about the deployment:
```bash
kubectl describe deployment dotnet-deploy
```

Check replica set information:
```bash
kubectl get replicasets
```

**Expected Output:**
```
NAME                       DESIRED   CURRENT   READY   AGE
dotnet-deploy-xxxxxxxxxx   2         2         2       25m
dotnet-deploy-yyyyyyyyyy   0         0         0       5m
```

Old replica sets are kept for rollback purposes.

## Part C: Cleanup Resources

### Step 16: View All Resources Before Cleanup

Check everything that will be deleted:
```bash
kubectl get all
```

**Expected Output:**
```
NAME                                 READY   STATUS    RESTARTS   AGE
pod/dotnet-deploy-xxxxxxxxxx-xxxxx   1/1     Running   0          5m
pod/dotnet-deploy-xxxxxxxxxx-yyyyy   1/1     Running   0          5m

NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/dotnet-service   NodePort    10.96.123.45    <none>        80:30081/TCP   30m
service/kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP        5d

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dotnet-deploy   2/2     2            2           30m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/dotnet-deploy-xxxxxxxxxx   2         2         2       30m
```

Save current state for reference (optional):
```bash
kubectl get all -o yaml > backup-before-cleanup.yaml
```

### Step 17: Delete Service

Delete the service first:
```bash
kubectl delete svc dotnet-service
```

**Expected Output:**
```
service "dotnet-service" deleted
```

Verify service is deleted:
```bash
kubectl get svc
```

**Expected Output:**
```
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5d
```

Only the default `kubernetes` service should remain.

**What This Does:**
- Removes the service endpoint
- API is no longer accessible via NodePort
- Pods continue running but are not exposed externally
- No data is lost

### Step 18: Delete Deployment

Delete the deployment:
```bash
kubectl delete deployment dotnet-deploy
```

**Expected Output:**
```
deployment.apps "dotnet-deploy" deleted
```

Verify deployment is deleted:
```bash
kubectl get deployments
```

**Expected Output:**
```
No resources found in default namespace.
```

**What This Does:**
- Terminates all pods managed by the deployment
- Deletes the replica set
- Removes deployment configuration
- Cannot be rolled back after deletion

### Step 19: Verify Complete Cleanup

Check that all related resources are gone:
```bash
kubectl get all
```

**Expected Output:**
```
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5d
```

Check pods are terminating/terminated:
```bash
kubectl get pods
```

**Expected Output:**
```
No resources found in default namespace.
```

Check replica sets:
```bash
kubectl get replicasets
```

**Expected Output:**
```
No resources found in default namespace.
```

### Step 20: Alternative - Delete Using YAML Files

Instead of individual delete commands, delete using the original YAML files:
```bash
# Navigate to directory with YAML files
cd k8s/  # or wherever you saved them

# Delete both resources at once
kubectl delete -f deployment.yaml -f service.yaml
```

**Expected Output:**
```
deployment.apps "dotnet-deploy" deleted
service "dotnet-service" deleted
```

This ensures all resources defined in the files are deleted.

### Step 21: Delete All Resources by Label (Alternative)

If you used labels, delete all resources with that label:
```bash
kubectl delete all -l app=dotnetapi
```

This deletes all resources (pods, services, deployments) with the label `app=dotnetapi`.

### Step 22: Clean Up Docker Images (Optional)

Remove Docker images from Minikube:
```bash
# Ensure you're using Minikube's Docker
eval $(minikube docker-env)

# List images
docker images | grep minikube-demo-api

# Delete specific version
docker rmi minikube-demo-api:v2

# Delete all versions
docker rmi minikube-demo-api:latest minikube-demo-api:v2

# Remove dangling images
docker image prune -f
```

### Step 23: Clean Up Minikube (Complete Reset - Optional)

If you want to completely reset Minikube:
```bash
# Stop Minikube
minikube stop

# Delete Minikube cluster (WARNING: Deletes everything)
minikube delete

# Start fresh Minikube cluster
minikube start
```

**Warning:** This deletes all deployments, services, and data in Minikube.

### Step 24: Clean Up Project Files (Optional)

Remove project files if no longer needed:
```bash
# Navigate to parent directory
cd ~

# Remove project directory
rm -rf MinikubeDemoAPI

# Remove YAML backup
rm backup-before-cleanup.yaml
```

**Warning:** Only do this if you're completely done with the project.

### Step 25: Reset Docker Environment

If you configured Docker to use Minikube's daemon, reset it:
```bash
eval $(minikube docker-env -u)
```

This points Docker back to your local Docker daemon.

Verify:
```bash
docker context ls
docker info | grep -i "Name:"
```

## Troubleshooting

### Issue: Pods stuck in "Terminating" state

**Solution:**
```bash
# Force delete stuck pods
kubectl delete pod <pod-name> --grace-period=0 --force

# If still stuck, check for finalizers
kubectl get pod <pod-name> -o yaml | grep finalizers
```

### Issue: Cannot delete service

**Solution:**
```bash
# Check if service has finalizers
kubectl get svc dotnet-service -o yaml | grep finalizers

# Force delete if necessary
kubectl delete svc dotnet-service --grace-period=0 --force
```

### Issue: Deployment won't delete

**Solution:**
```bash
# Check deployment status
kubectl describe deployment dotnet-deploy

# Force delete deployment
kubectl delete deployment dotnet-deploy --grace-period=0 --force

# Delete replica sets manually if needed
kubectl delete replicaset --all
```

### Issue: Resources still showing after deletion

**Solution:**
```bash
# Wait a few seconds for propagation
sleep 10
kubectl get all

# Check for resources in all namespaces
kubectl get all --all-namespaces

# Delete stuck resources
kubectl delete <resource-type> <resource-name> --force --grace-period=0
```

### Issue: Scaling doesn't work

**Solution:**
```bash
# Check deployment exists
kubectl get deployments

# Check for errors
kubectl describe deployment dotnet-deploy

# Check resource quotas
kubectl describe resourcequota

# Verify image exists in Minikube
eval $(minikube docker-env)
docker images | grep minikube-demo-api
```

## Advanced Scaling Techniques

### Horizontal Pod Autoscaler (HPA)

Create autoscaler based on CPU usage:
```bash
kubectl autoscale deployment dotnet-deploy --cpu-percent=50 --min=1 --max=5
```

View autoscaler status:
```bash
kubectl get hpa
```

Delete autoscaler:
```bash
kubectl delete hpa dotnet-deploy
```

**Note:** Requires metrics-server addon in Minikube:
```bash
minikube addons enable metrics-server
```

### Manual Scaling with Resource Limits

Edit deployment to add resource requests/limits:
```yaml
spec:
  containers:
    - name: dotnet-container
      image: minikube-demo-api:latest
      resources:
        requests:
          memory: "128Mi"
          cpu: "100m"
        limits:
          memory: "256Mi"
          cpu: "500m"
```

This helps Kubernetes make better scaling decisions.

## Best Practices Summary

### Scaling Best Practices
1. Always use even numbers of replicas for better load distribution
2. Monitor resource usage before scaling up
3. Test with 2-3 replicas before scaling to production levels
4. Use HPA for automatic scaling based on metrics
5. Set appropriate resource requests and limits

### Update Best Practices
1. Use version tags (v1, v2) instead of overwriting `latest`
2. Test new versions in a separate namespace first
3. Use rolling update strategy (default) for zero downtime
4. Always monitor rollout status
5. Keep old replica sets for quick rollback

### Cleanup Best Practices
1. Delete service before deployment to prevent orphaned endpoints
2. Use YAML files for consistent deletion
3. Verify deletion with `kubectl get all`
4. Save configurations before deleting for future reference
5. Clean up unused Docker images to save space

## Useful Commands Summary
```bash
# Scaling
kubectl scale deployment dotnet-deploy --replicas=2
kubectl get pods -w
kubectl get endpoints dotnet-service

# Updating
docker build -t minikube-demo-api:v2 .
kubectl set image deployment/dotnet-deploy dotnet-container=minikube-demo-api:v2
kubectl rollout status deployment/dotnet-deploy
kubectl rollout undo deployment/dotnet-deploy

# Cleanup
kubectl delete svc dotnet-service
kubectl delete deployment dotnet-deploy
kubectl delete -f deployment.yaml -f service.yaml
kubectl delete all -l app=dotnetapi

# Verification
kubectl get all
kubectl get pods
kubectl get svc
kubectl get deployments
```

## Verification Checklist

### Scaling
- [ ] Scaled deployment to 2 replicas
- [ ] Both pods are running
- [ ] Service endpoints show both pod IPs
- [ ] Load balancing works across pods
- [ ] High availability tested (pod deletion/recreation)
- [ ] Logs show requests distributed to both pods

### Updating
- [ ] Code changes made to application
- [ ] Docker image rebuilt with new tag
- [ ] Deployment updated with new image
- [ ] Rolling update completed successfully
- [ ] New version verified working
- [ ] Rollback tested (optional)

### Cleanup
- [ ] Service deleted successfully
- [ ] Deployment deleted successfully
- [ ] All pods terminated
- [ ] No replica sets remaining
- [ ] `kubectl get all` shows only kubernetes service
- [ ] Docker images cleaned (optional)
- [ ] Docker environment reset

## Key Takeaways

1. **Scaling** increases application capacity and availability
2. **Load balancing** happens automatically with multiple replicas
3. **Rolling updates** provide zero-downtime deployments
4. **Rollback** capability allows quick recovery from bad updates
5. **Cleanup** should be done in order: service → deployment → images
6. **Labels** make it easy to manage related resources
7. **Resource management** is critical for stable scaling

## Next Steps (Beyond This Tutorial)

- Implement StatefulSets for stateful applications
- Use ConfigMaps and Secrets for configuration management
- Set up persistent volumes for data storage
- Configure ingress controllers for advanced routing
- Implement monitoring with Prometheus and Grafana
- Set up CI/CD pipelines for automated deployments
- Deploy to production Kubernetes clusters (AKS, EKS, GKE)
- Implement service mesh (Istio, Linkerd) for advanced traffic management