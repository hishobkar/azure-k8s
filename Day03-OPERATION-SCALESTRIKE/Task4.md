# Task 4: Create Deployment Using YAML

**Duration:** 45 minutes  
**Environment:** WSL with Docker, kubectl, and Minikube  
**Target Audience:** .NET Core Developers

---

## Prerequisites
- Docker installed and running in WSL
- kubectl installed in WSL
- Minikube installed and cluster running in WSL
- Completed Tasks 1-3 (understanding of Deployments and ReplicaSets)
- Basic text editor (nano, vim, or VS Code)

---

## Objectives
- Understand YAML syntax for Kubernetes resources
- Create Deployments using declarative configuration (YAML)
- Learn the structure of Deployment manifests
- Apply best practices for Infrastructure as Code (IaC)
- Compare imperative vs declarative approaches
- Apply these concepts to .NET Core application deployments

---

## Introduction: Imperative vs Declarative

### Imperative Approach (Previous Tasks)
```bash
kubectl create deployment web-deploy --image=nginx
kubectl scale deployment web-deploy --replicas=3
```
**Characteristics:**
- Command-line based
- Immediate execution
- Not version-controlled
- Hard to reproduce
- Not recommended for production

### Declarative Approach (This Task)
```bash
kubectl apply -f deployment-demo.yaml
```
**Characteristics:**
- YAML configuration files
- Version-controlled (Git)
- Reproducible and auditable
- Industry best practice
- Recommended for production

---

## Task Steps

### 1. Ensure Minikube is Running

Verify your cluster is active:
```bash
minikube status
```

If not running:
```bash
minikube start
```

Check current context:
```bash
kubectl config current-context
```

---

### 2. Create the Deployment YAML File

#### Option A: Using VS Code (Recommended)
```bash
code deployment-demo.yaml
```

#### Option B: Using nano
```bash
nano deployment-demo.yaml
```

#### Option C: Using vim
```bash
vim deployment-demo.yaml
```

---

### 3. Add the Deployment Configuration

Copy and paste the following YAML content into `deployment-demo.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deploy
spec:
  replicas: 2
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
          image: nginx
```

**Save the file:**
- **VS Code**: `Ctrl+S` or `File > Save`
- **nano**: `Ctrl+O`, `Enter`, then `Ctrl+X`
- **vim**: Press `Esc`, type `:wq`, press `Enter`

---

### 4. Understanding the YAML Structure

Let's break down each section of the YAML file:

#### 4.1 API Version and Kind
```yaml
apiVersion: apps/v1
kind: Deployment
```

**Explanation:**
- `apiVersion: apps/v1` - Kubernetes API version for Deployments
- `kind: Deployment` - Type of Kubernetes resource to create

---

#### 4.2 Metadata Section
```yaml
metadata:
  name: app-deploy
```

**Explanation:**
- `metadata` - Information about the Deployment
- `name: app-deploy` - Unique name for this Deployment
- This name is used to reference the Deployment in kubectl commands

**Note:** Names must be unique within a namespace and follow DNS subdomain rules (lowercase alphanumeric, hyphens).

---

#### 4.3 Spec Section - Deployment Configuration
```yaml
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
```

**Explanation:**
- `spec` - Desired state of the Deployment
- `replicas: 2` - Create and maintain 2 Pod replicas
- `selector` - How the Deployment finds Pods to manage
- `matchLabels` - Pods must have these labels to be managed
- `app: myapp` - Label selector matching the Pod template

**Critical:** The `selector.matchLabels` MUST match the `template.metadata.labels` exactly!

---

#### 4.4 Template Section - Pod Specification
```yaml
  template:
    metadata:
      labels:
        app: myapp
```

**Explanation:**
- `template` - Template for creating Pods
- `metadata.labels` - Labels applied to each Pod
- `app: myapp` - Label that matches the selector above

---

#### 4.5 Container Specification
```yaml
    spec:
      containers:
        - name: myapp-container
          image: nginx
```

**Explanation:**
- `spec` - Pod specification (not to be confused with Deployment spec)
- `containers` - List of containers in the Pod
- `name: myapp-container` - Container name (for logging and identification)
- `image: nginx` - Docker image to run (from Docker Hub)

---

### 5. Validate the YAML File

Before applying, validate the YAML syntax:
```bash
kubectl apply -f deployment-demo.yaml --dry-run=client
```

**Expected Output:**
```
deployment.apps/app-deploy created (dry run)
```

**What `--dry-run=client` does:**
- Validates YAML syntax
- Checks API version compatibility
- Does NOT create actual resources
- Safe way to test configurations

---

### 6. Apply the Deployment

Create the Deployment in your cluster:
```bash
kubectl apply -f deployment-demo.yaml
```

**Expected Output:**
```
deployment.apps/app-deploy created
```

**What happens:**
- Kubernetes API server receives the configuration
- Deployment controller creates the Deployment object
- ReplicaSet controller creates a ReplicaSet
- Scheduler assigns Pods to nodes
- Kubelet starts containers on nodes

---

### 7. Verify the Deployment

#### 7.1 Check Deployments
```bash
kubectl get deployments
```

**Expected Output:**
```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
app-deploy   2/2     2            2           30s
```

**Observations:**
- Deployment name: `app-deploy` (from YAML metadata.name)
- 2/2 replicas ready (from YAML spec.replicas)
- All replicas are up-to-date and available

---

#### 7.2 Check ReplicaSets
```bash
kubectl get rs
```

**Alternative:**
```bash
kubectl get replicasets
```

**Expected Output:**
```
NAME                    DESIRED   CURRENT   READY   AGE
app-deploy-xxxxxxxxxx   2         2         2       1m
```

**Observations:**
- ReplicaSet name: `app-deploy-xxxxxxxxxx` (deployment name + hash)
- DESIRED: 2 (matches replicas in YAML)
- CURRENT: 2 (all replicas created)
- READY: 2 (all replicas running)
- One ReplicaSet automatically created by the Deployment

---

#### 7.3 Check Pods
```bash
kubectl get pods
```

**Expected Output:**
```
NAME                          READY   STATUS    RESTARTS   AGE
app-deploy-xxxxxxxxxx-aaaaa   1/1     Running   0          1m
app-deploy-xxxxxxxxxx-bbbbb   1/1     Running   0          1m
```

**Observations:**
- 2 Pods running (matches spec.replicas: 2)
- Pod names: `app-deploy-xxxxxxxxxx-xxxxx` (deployment-replicaset-random)
- All containers ready (1/1)
- Status: Running
- No restarts

---

#### 7.4 Check Pod Labels
```bash
kubectl get pods --show-labels
```

**Expected Output:**
```
NAME                          READY   STATUS    RESTARTS   AGE   LABELS
app-deploy-xxxxxxxxxx-aaaaa   1/1     Running   0          2m    app=myapp,pod-template-hash=xxxxxxxxxx
app-deploy-xxxxxxxxxx-bbbbb   1/1     Running   0          2m    app=myapp,pod-template-hash=xxxxxxxxxx
```

**Observations:**
- `app=myapp` label (from template.metadata.labels)
- `pod-template-hash` label (automatically added by Kubernetes)
- These labels match the selector.matchLabels

---

### 8. Get Detailed Information

#### 8.1 Describe the Deployment
```bash
kubectl describe deployment app-deploy
```

**Key Information:**
```
Name:                   app-deploy
Namespace:              default
CreationTimestamp:      [timestamp]
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=myapp
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=myapp
  Containers:
   myapp-container:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   app-deploy-xxxxxxxxxx (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  3m    deployment-controller  Scaled up replica set app-deploy-xxxxxxxxxx to 2
```

**Key Observations:**
- Selector matches your YAML (app=myapp)
- Pod Template shows container configuration
- Events show the ReplicaSet creation

---

#### 8.2 Describe the ReplicaSet
```bash
kubectl describe rs app-deploy-xxxxxxxxxx
```

*(Replace `xxxxxxxxxx` with actual hash from `kubectl get rs`)*

Or use label selector:
```bash
kubectl describe rs -l app=myapp
```

**Key Information:**
```
Name:           app-deploy-xxxxxxxxxx
Namespace:      default
Selector:       app=myapp,pod-template-hash=xxxxxxxxxx
Labels:         app=myapp
                pod-template-hash=xxxxxxxxxx
Annotations:    deployment.kubernetes.io/desired-replicas: 2
                deployment.kubernetes.io/max-replicas: 3
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/app-deploy
Replicas:       2 current / 2 desired
Pods Status:    2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=myapp
           pod-template-hash=xxxxxxxxxx
  Containers:
   myapp-container:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  5m    replicaset-controller  Created pod: app-deploy-xxxxxxxxxx-aaaaa
  Normal  SuccessfulCreate  5m    replicaset-controller  Created pod: app-deploy-xxxxxxxxxx-bbbbb
```

**Key Observations:**
- "Controlled By: Deployment/app-deploy" confirms hierarchy
- Events show both Pods being created

---

#### 8.3 Describe a Pod
```bash
kubectl describe pod app-deploy-xxxxxxxxxx-aaaaa
```

Or describe all pods with the label:
```bash
kubectl describe pod -l app=myapp
```

**Key Information to Note:**
- Controlled By: ReplicaSet/app-deploy-xxxxxxxxxx
- Container Image: nginx
- Container State: Running
- Events showing image pull and container start

---

### 9. Verify Pod Output

#### 9.1 View Pod Logs
```bash
kubectl logs -l app=myapp
```

This shows logs from all Pods with the `app=myapp` label.

For a specific Pod:
```bash
kubectl logs app-deploy-xxxxxxxxxx-aaaaa
```

---

#### 9.2 Access Nginx (Optional)

Create a service to access the Pods:
```bash
kubectl expose deployment app-deploy --type=NodePort --port=80
```

Get the URL:
```bash
minikube service app-deploy --url
```

Test with curl:
```bash
curl $(minikube service app-deploy --url)
```

**Expected:** You'll see the default Nginx welcome page HTML.

Or open in browser:
```bash
minikube service app-deploy
```

Clean up the service:
```bash
kubectl delete service app-deploy
```

---

### 10. Modify and Reapply the Deployment

Let's change the number of replicas to demonstrate declarative updates.

Edit `deployment-demo.yaml` and change:
```yaml
spec:
  replicas: 4  # Changed from 2 to 4
```

Apply the changes:
```bash
kubectl apply -f deployment-demo.yaml
```

**Expected Output:**
```
deployment.apps/app-deploy configured
```

**Note:** Output says "configured" (not "created") because the resource already exists.

Verify the change:
```bash
kubectl get deployments
kubectl get pods
```

**Expected:**
- Deployment shows 4/4 replicas
- 4 Pods are running (2 original + 2 new)

---

### 11. View Applied Configuration

See the current applied configuration:
```bash
kubectl get deployment app-deploy -o yaml
```

This shows the complete configuration including system-added fields.

Compare with your source file:
```bash
kubectl diff -f deployment-demo.yaml
```

*(Shows differences between local file and cluster state)*

---

## For .NET Core Developers

### Create a .NET Core Deployment YAML

Create a file named `dotnet-api-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dotnet-api
  labels:
    app: dotnet-api
    environment: development
spec:
  replicas: 3
  selector:
    matchLabels:
      app: dotnet-api
  template:
    metadata:
      labels:
        app: dotnet-api
        version: v1
    spec:
      containers:
        - name: dotnet-api-container
          image: mcr.microsoft.com/dotnet/samples:aspnetapp
          ports:
            - containerPort: 8080
              protocol: TCP
          env:
            - name: ASPNETCORE_ENVIRONMENT
              value: "Development"
            - name: ASPNETCORE_URLS
              value: "http://+:8080"
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "500m"
```

Apply it:
```bash
kubectl apply -f dotnet-api-deployment.yaml
```

Verify:
```bash
kubectl get deployments dotnet-api
kubectl get pods -l app=dotnet-api
```

**Enhancements in this example:**
- Multiple labels for better organization
- Port specification (containerPort: 8080)
- Environment variables for ASP.NET Core configuration
- Resource requests and limits (production best practice)

---

### Real-World .NET Core Deployment Template
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-webapi
  namespace: production
  labels:
    app: my-webapi
    tier: backend
    environment: production
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: my-webapi
  template:
    metadata:
      labels:
        app: my-webapi
        version: v2.1.0
    spec:
      containers:
        - name: webapi
          image: myregistry.azurecr.io/my-webapi:2.1.0
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
          env:
            - name: ASPNETCORE_ENVIRONMENT
              value: "Production"
            - name: ConnectionStrings__DefaultConnection
              valueFrom:
                secretKeyRef:
                  name: db-connection
                  key: connection-string
          resources:
            requests:
              memory: "256Mi"
              cpu: "200m"
            limits:
              memory: "512Mi"
              cpu: "1000m"
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

**Production Features:**
- Namespace for environment isolation
- Multiple descriptive labels
- Rolling update strategy
- Resource requests and limits
- Health checks (liveness and readiness probes)
- Secrets for sensitive data (connection strings)
- Version labeling for tracking

---

## YAML Best Practices

### 1. Indentation
- **Use 2 spaces** (not tabs)
- Be consistent throughout the file
- YAML is whitespace-sensitive

### 2. Label Naming Conventions
```yaml
labels:
  app: myapp                    # Application name
  tier: frontend                # Application tier
  environment: production       # Environment
  version: v1.0.0              # Version
  managed-by: kubectl          # Management tool
```

### 3. Resource Naming
- Use lowercase letters, numbers, and hyphens
- Start and end with alphanumeric characters
- Maximum 253 characters
- Be descriptive: `user-service-api` not `api1`

### 4. Comments
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deploy
  # This deployment manages the frontend application
spec:
  replicas: 2  # Increased from 1 for high availability
```

---

## Common YAML Mistakes and Fixes

### Mistake 1: Selector Mismatch

**Wrong:**
```yaml
spec:
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: different-app  # MISMATCH!
```

**Correct:**
```yaml
spec:
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp  # MATCHES!
```

---

### Mistake 2: Incorrect Indentation

**Wrong:**
```yaml
spec:
  containers:
  - name: myapp-container
  image: nginx  # Wrong indentation
```

**Correct:**
```yaml
spec:
  containers:
    - name: myapp-container
      image: nginx  # Correct indentation
```

---

### Mistake 3: Missing Required Fields

**Wrong:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deploy
spec:
  replicas: 2
  # Missing selector!
  template:
    metadata:
      labels:
        app: myapp
```

**Correct:**
```yaml
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp  # Required field!
```

---

## Troubleshooting

### YAML Validation Errors

**Error: "error validating data"**
```bash
kubectl apply -f deployment-demo.yaml --validate=true
```

Check for:
- Incorrect indentation
- Missing required fields
- Typos in field names

---

### Selector Mismatch Error

**Error: "The Deployment spec.selector does not match"**

Verify:
```bash
kubectl describe deployment app-deploy
```

Ensure `selector.matchLabels` matches `template.metadata.labels`.

---

### Image Pull Errors

**Error: "ErrImagePull" or "ImagePullBackOff"**

Check:
```bash
kubectl describe pod <pod-name>
```

Common causes:
- Image name typo
- Image doesn't exist
- Private registry without credentials
- No internet connection

---

### Pods Not Starting
```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

Look for:
- Container crashes
- Configuration errors
- Resource constraints

---

## Version Control with Git

### Initialize Git Repository
```bash
# Create a directory for your manifests
mkdir k8s-manifests
cd k8s-manifests

# Move your YAML file
mv ~/deployment-demo.yaml .

# Initialize Git
git init
git add deployment-demo.yaml
git commit -m "Initial deployment configuration"
```

---

### .gitignore for Kubernetes

Create `.gitignore`:
```
# Sensitive files
*-secret.yaml
*.key
*.pem

# Temporary files
*.tmp
*.swp
*~

# Environment-specific
*.local.yaml
```

---

### Branching Strategy
```bash
# Development changes
git checkout -b dev
# Edit deployment-demo.yaml (set replicas: 1)
git add deployment-demo.yaml
git commit -m "Reduce replicas for dev environment"

# Production configuration
git checkout main
# deployment-demo.yaml has replicas: 2 (production)
```

---

## Alternative: Using kubectl create with --dry-run

Generate YAML from imperative commands:
```bash
kubectl create deployment app-deploy --image=nginx --replicas=2 \
  --dry-run=client -o yaml > generated-deployment.yaml
```

This creates a YAML file that you can customize and version control.

---

## Comparison: Imperative vs Declarative

| Aspect | Imperative | Declarative |
|--------|-----------|-------------|
| **Method** | CLI commands | YAML files |
| **Example** | `kubectl create deployment` | `kubectl apply -f file.yaml` |
| **Reproducibility** | Hard | Easy |
| **Version Control** | No | Yes |
| **Team Collaboration** | Difficult | Easy |
| **Audit Trail** | Limited | Complete |
| **Production Use** | Not recommended | Best practice |
| **Learning Curve** | Easier | Moderate |
| **Speed** | Fast | Requires file creation |

---

## Managing Multiple Environments

### Development Environment

`deployment-dev.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deploy
  namespace: development
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      environment: dev
  template:
    metadata:
      labels:
        app: myapp
        environment: dev
    spec:
      containers:
        - name: myapp-container
          image: nginx:latest
```

---

### Production Environment

`deployment-prod.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deploy
  namespace: production
spec:
  replicas: 5
  selector:
    matchLabels:
      app: myapp
      environment: prod
  template:
    metadata:
      labels:
        app: myapp
        environment: prod
    spec:
      containers:
        - name: myapp-container
          image: nginx:1.21.6  # Specific version for production
```

---

## Cleanup

### Remove the Deployment

Using the YAML file:
```bash
kubectl delete -f deployment-demo.yaml
```

Or by name:
```bash
kubectl delete deployment app-deploy
```

---

### Verify Deletion
```bash
kubectl get deployments
kubectl get rs
kubectl get pods
```

All resources should be gone.

---

### Clean Up .NET Example (if created)
```bash
kubectl delete -f dotnet-api-deployment.yaml
# Or
kubectl delete deployment dotnet-api
```

---

## Key Takeaways

- **Declarative configuration is the production standard** - YAML files are versioned and reproducible  
- **YAML structure has 4 main sections** - apiVersion, kind, metadata, spec  
- **Selector must match template labels** - Critical for Deployment to manage Pods  
- **kubectl apply is idempotent** - Safe to run multiple times  
- **Version control your manifests** - Use Git for tracking changes  
- **Dry-run validates before applying** - Always validate YAML before deployment  
- **Describe commands show detailed info** - Essential for troubleshooting  
- **Labels organize and select resources** - Use consistent labeling strategy  

---

## Next Steps

- Add resource limits and requests to your Deployments
- Implement health checks (liveness and readiness probes)
- Create Services to expose your Deployments
- Learn about ConfigMaps and Secrets for configuration
- Explore Helm for package management
- Implement CI/CD pipelines with GitOps (ArgoCD, Flux)
- Study Kustomize for environment-specific configurations

---

## Additional Resources for .NET Developers

### Recommended YAML Structure for .NET Projects
```
my-dotnet-app/
├── k8s/
│   ├── base/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── configmap.yaml
│   ├── dev/
│   │   └── kustomization.yaml
│   └── prod/
│       └── kustomization.yaml
├── src/
│   └── (your .NET code)
└── Dockerfile
```

---

### Sample Dockerfile for .NET Core
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 8080

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["MyApp.csproj", "./"]
RUN dotnet restore "MyApp.csproj"
COPY . .
RUN dotnet build "MyApp.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "MyApp.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

---

### Build and Deploy Workflow
```bash
# Build Docker image
docker build -t my-dotnet-app:v1.0.0 .

# Tag for registry
docker tag my-dotnet-app:v1.0.0 myregistry.azurecr.io/my-dotnet-app:v1.0.0

# Push to registry
docker push myregistry.azurecr.io/my-dotnet-app:v1.0.0

# Update deployment YAML with new image
# Then apply
kubectl apply -f deployment.yaml
```

---

**Task Completed!** You have successfully created a Deployment using YAML, applied declarative configuration, and learned industry best practices for managing Kubernetes resources. You're now ready to manage production-grade .NET Core applications in Kubernetes!