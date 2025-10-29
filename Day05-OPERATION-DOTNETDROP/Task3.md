# Task 3: Create Deployment and Service YAML Files (45 minutes)

## Prerequisites
- Completed Task 2 (Docker image `minikube-demo-api:latest` built)
- Minikube running (`minikube status`)
- Currently in the `MinikubeDemoAPI` project root directory
- kubectl configured to use Minikube context

## Step 1: Verify Minikube and kubectl Setup

Check Minikube status:
```bash
minikube status
```

Verify kubectl is connected to Minikube:
```bash
kubectl cluster-info
kubectl config current-context
```

**Expected Output:** Should show `minikube` as current context

## Step 2: Create Kubernetes Manifests Directory (Optional but Recommended)
```bash
mkdir k8s
cd k8s
```

**Note:** You can also create YAML files in the project root directory

## Step 3: Create deployment.yaml

Create the deployment manifest file:
```bash
touch deployment.yaml
nano deployment.yaml
# OR
code deployment.yaml
```

Add the following content:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dotnet-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dotnetapi
  template:
    metadata:
      labels:
        app: dotnetapi
    spec:
      containers:
        - name: dotnet-container
          image: minikube-demo-api:latest
          imagePullPolicy: Never
          ports:
            - containerPort: 80
```

**Important Addition:** Note the `imagePullPolicy: Never` - this tells Kubernetes to use the local image from Minikube's Docker daemon instead of trying to pull from a registry.

Save and close the file.

## Step 4: Understanding deployment.yaml

**Key Components:**

- **apiVersion:** Specifies Kubernetes API version for Deployments
- **kind:** Defines resource type (Deployment)
- **metadata.name:** Unique name for this deployment (`dotnet-deploy`)
- **spec.replicas:** Number of pod instances to run (1 for demo)
- **selector.matchLabels:** How deployment finds pods to manage
- **template.metadata.labels:** Labels applied to pods (must match selector)
- **spec.containers:** Container specifications
  - **name:** Container name within the pod
  - **image:** Docker image to use (our built image)
  - **imagePullPolicy:** Never = use local image, don't pull from registry
  - **containerPort:** Port the container exposes (80 for our API)

## Step 5: Create service.yaml

Create the service manifest file:
```bash
touch service.yaml
nano service.yaml
# OR
code service.yaml
```

Add the following content:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: dotnet-service
spec:
  type: NodePort
  selector:
    app: dotnetapi
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30081
```

Save and close the file.

## Step 6: Understanding service.yaml

**Key Components:**

- **apiVersion:** Kubernetes API version for Services
- **kind:** Defines resource type (Service)
- **metadata.name:** Unique name for this service (`dotnet-service`)
- **spec.type:** NodePort = accessible from outside cluster via node IP
- **selector:** Matches pods with label `app: dotnetapi` (from deployment)
- **ports:**
  - **port:** Service's internal port (80)
  - **targetPort:** Container port to forward to (80)
  - **nodePort:** External port on node (30081, range: 30000-32767)

**Service Types:**
- **ClusterIP** (default): Internal cluster access only
- **NodePort:** External access via node IP + nodePort
- **LoadBalancer:** Cloud provider load balancer (not available in Minikube)

## Step 7: Verify YAML Files

Check files were created correctly:
```bash
ls -la
cat deployment.yaml
cat service.yaml
```

Validate YAML syntax (optional):
```bash
kubectl apply -f deployment.yaml --dry-run=client
kubectl apply -f service.yaml --dry-run=client
```

**Expected Output:** Should show no errors, only what would be created

## Step 8: Apply Deployment

Deploy the application to Kubernetes:
```bash
kubectl apply -f deployment.yaml
```

**Expected Output:**
```
deployment.apps/dotnet-deploy created
```

## Step 9: Apply Service

Create the service:
```bash
kubectl apply -f service.yaml
```

**Expected Output:**
```
service/dotnet-service created
```

## Step 10: Verify Deployment

Check deployment status:
```bash
kubectl get deployments
```

**Expected Output:**
```
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
dotnet-deploy   1/1     1            1           30s
```

Check pods are running:
```bash
kubectl get pods
```

**Expected Output:**
```
NAME                             READY   STATUS    RESTARTS   AGE
dotnet-deploy-xxxxxxxxxx-xxxxx   1/1     Running   0          45s
```

View detailed pod information:
```bash
kubectl describe pod <pod-name>
```

## Step 11: Verify Service

Check service status:
```bash
kubectl get services
# OR
kubectl get svc
```

**Expected Output:**
```
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
dotnet-service   NodePort    10.96.xxx.xxx   <none>        80:30081/TCP   1m
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP        5d
```

Get detailed service information:
```bash
kubectl describe service dotnet-service
```

## Step 12: Access the Application

Get Minikube IP address:
```bash
minikube ip
```

**Example Output:** `192.168.49.2`

Access the API using the NodePort:
```bash
# Replace <minikube-ip> with actual IP from previous command
curl http://<minikube-ip>:30081/WeatherForecast

# Example:
curl http://192.168.49.2:30081/WeatherForecast
```

**Expected Output:** JSON response with weather forecast data

Test custom endpoint (if you created one in Task 1):
```bash
curl http://192.168.49.2:30081/hello
```

## Step 13: Alternative Access Method - Minikube Service

Minikube provides a convenient command to access NodePort services:
```bash
minikube service dotnet-service
```

This will automatically open your default browser with the service URL.

To just get the URL without opening browser:
```bash
minikube service dotnet-service --url
```

## Step 14: View Application Logs

Check application logs from the pod:
```bash
# Get pod name first
kubectl get pods

# View logs
kubectl logs <pod-name>

# Follow logs in real-time
kubectl logs -f <pod-name>
```

## Troubleshooting

### Issue: Pod status shows "ImagePullBackOff" or "ErrImagePull"

**Solution:** 
```bash
# Check if imagePullPolicy is set to Never
kubectl describe pod <pod-name>

# Delete and recreate deployment with imagePullPolicy: Never
kubectl delete -f deployment.yaml
# Edit deployment.yaml to add imagePullPolicy: Never
kubectl apply -f deployment.yaml
```

### Issue: Pod status shows "CrashLoopBackOff"

**Solution:** 
```bash
# Check pod logs for errors
kubectl logs <pod-name>

# Describe pod for events
kubectl describe pod <pod-name>

# Common causes:
# - Wrong ENTRYPOINT in Dockerfile
# - Application crashes on startup
# - Port mismatch
```

### Issue: Service is running but can't access via curl

**Solution:** 
```bash
# Verify Minikube IP
minikube ip

# Check if port is accessible
minikube service list

# Try accessing from within the cluster
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://dotnet-service
```

### Issue: "connection refused" when accessing the service

**Solution:** 
```bash
# Check if pod is actually running
kubectl get pods

# Verify service endpoints
kubectl get endpoints dotnet-service

# Check pod logs
kubectl logs <pod-name>

# Verify container port matches Dockerfile EXPOSE
kubectl describe pod <pod-name> | grep Port
```

### Issue: kubectl commands hang or timeout

**Solution:** 
```bash
# Restart Minikube
minikube stop
minikube start

# Reapply manifests
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### Issue: YAML validation errors

**Solution:** 
- Check indentation (use spaces, not tabs)
- Verify field names are spelled correctly
- Use online YAML validators
- Run: `kubectl apply -f deployment.yaml --validate=true`

## Useful kubectl Commands

**View all resources:**
```bash
kubectl get all
```

**Watch resources in real-time:**
```bash
kubectl get pods -w
```

**Get resource details in YAML format:**
```bash
kubectl get deployment dotnet-deploy -o yaml
```

**Get resource details in JSON format:**
```bash
kubectl get service dotnet-service -o json
```

**Execute command in pod:**
```bash
kubectl exec -it <pod-name> -- /bin/bash
```

**Port forward (alternative to NodePort):**
```bash
kubectl port-forward service/dotnet-service 8080:80
# Then access at http://localhost:8080
```

## Updating the Deployment

If you make changes to your code and rebuild the image:
```bash
# 1. Rebuild Docker image (with Minikube Docker env)
eval $(minikube docker-env)
docker build -t minikube-demo-api:latest .

# 2. Restart deployment to use new image
kubectl rollout restart deployment dotnet-deploy

# 3. Check rollout status
kubectl rollout status deployment dotnet-deploy
```

## Scaling the Deployment

Scale to multiple replicas:
```bash
# Scale to 3 replicas
kubectl scale deployment dotnet-deploy --replicas=3

# Verify scaling
kubectl get pods
```

To make scaling permanent, edit `deployment.yaml` and change `replicas: 3`, then:
```bash
kubectl apply -f deployment.yaml
```

## Cleanup Resources

When you're done testing:
```bash
# Delete service
kubectl delete -f service.yaml

# Delete deployment
kubectl delete -f deployment.yaml

# Or delete both at once
kubectl delete -f deployment.yaml -f service.yaml

# Verify deletion
kubectl get all
```

## Best Practices for YAML Files

1. **Use imagePullPolicy: Never** for local Minikube images
2. **Add resource limits** (CPU/memory) in production
3. **Use health checks** (liveness/readiness probes)
4. **Label resources** consistently for better organization
5. **Version your manifests** in source control

## Enhanced deployment.yaml (Production-Ready)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dotnet-deploy
  labels:
    app: dotnetapi
spec:
  replicas: 2
  selector:
    matchLabels:
      app: dotnetapi
  template:
    metadata:
      labels:
        app: dotnetapi
    spec:
      containers:
        - name: dotnet-container
          image: minikube-demo-api:latest
          imagePullPolicy: Never
          ports:
            - containerPort: 80
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /WeatherForecast
              port: 80
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /WeatherForecast
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
```

## Verification Checklist

- [ ] deployment.yaml created with correct syntax
- [ ] service.yaml created with correct syntax
- [ ] `imagePullPolicy: Never` added to deployment
- [ ] Deployment applied successfully
- [ ] Service applied successfully
- [ ] Pods are in "Running" status
- [ ] Service shows correct NodePort (30081)
- [ ] Can access API via `curl http://<minikube-ip>:30081/WeatherForecast`
- [ ] Logs show application running correctly
- [ ] `minikube service dotnet-service` works

## Key Takeaways

1. **Deployments** manage pod replicas and updates
2. **Services** provide stable networking and load balancing
3. **NodePort** enables external access on Minikube
4. **Labels and selectors** connect services to pods
5. **imagePullPolicy: Never** is crucial for local images
6. **kubectl** is your primary tool for managing resources

## Next Steps

- Explore ConfigMaps and Secrets for configuration
- Add health checks and resource limits
- Implement horizontal pod autoscaling
- Set up ingress for better routing
- Deploy to cloud Kubernetes clusters (AKS, EKS, GKE)