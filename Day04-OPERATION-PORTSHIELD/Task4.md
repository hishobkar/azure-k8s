# Task 4: Create a NodePort Service via YAML

**Duration:** 45 minutes  
**Environment:** WSL with Docker, kubectl, and Minikube installed  
**Target Audience:** .NET Core Developer

---

## Objective

Create a NodePort Service using a YAML manifest file to expose the deployment externally. Learn the declarative approach to Kubernetes resource management and understand how NodePort differs from ClusterIP.

---

## Prerequisites

- Completed Task 2 (Deployment `svc-demo` with 2 replicas is running)
- Completed Task 3 (Understanding of ClusterIP Services)
- Minikube is running in WSL
- kubectl is configured and connected to cluster

**Verify prerequisites:**
```bash
# Check deployment exists with 2 replicas
kubectl get deployment svc-demo

# Verify Pods are running
kubectl get pods -l app=svc-demo

# Check current services
kubectl get svc
```

---

## Step 1: Understand NodePort vs ClusterIP

### Quick Comparison

| Feature | ClusterIP | NodePort |
|---------|-----------|----------|
| **Access** | Internal only | External + Internal |
| **Port Range** | Any port | 30000-32767 |
| **Use Case** | Service-to-service | Development/testing external access |
| **IP Address** | Cluster virtual IP | Node's IP + NodePort |
| **URL Format** | `http://service-name:port` | `http://<NodeIP>:<NodePort>` |

**For .NET Core developers:**
- **ClusterIP:** Like calling `http://api-service` from within your Kubernetes cluster
- **NodePort:** Like exposing your API at `http://localhost:30080` accessible from your host machine

---

## Step 2: Create the NodePort Service YAML File

### Create the YAML File
```bash
# Create the file using your preferred editor
nano nodeport-svc.yaml
# or
vim nodeport-svc.yaml
# or
code nodeport-svc.yaml  # If using VS Code
```

### YAML Content

Copy and paste the following content:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-node
spec:
  type: NodePort
  selector:
    app: svc-demo
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

**Save the file and exit the editor.**

### Understanding the YAML Structure
```yaml
apiVersion: v1              # API version for Service resource
kind: Service               # Resource type
metadata:
  name: svc-node            # Service name (used for DNS)
spec:
  type: NodePort            # Service type (exposes externally)
  selector:
    app: svc-demo           # Matches Pods with label app=svc-demo
  ports:
    - port: 80              # Service port (internal cluster access)
      targetPort: 80        # Port on the Pod containers
      nodePort: 30080       # External port on each Node (30000-32767)
```

**Port explanation:**
- **nodePort: 30080** - External port accessible on Node IP (from outside cluster)
- **port: 80** - Service port for internal cluster communication
- **targetPort: 80** - Actual port the container listens on (nginx default)

**For .NET Core developers:**
If your ASP.NET Core app runs on port 5000:
```yaml
ports:
  - port: 8080         # Service port
    targetPort: 5000   # Your app's port in the container
    nodePort: 30080    # External access port
```

---

## Step 3: Validate the YAML File (Optional but Recommended)

### Check YAML Syntax
```bash
# Dry run to validate without creating the resource
kubectl apply -f nodeport-svc.yaml --dry-run=client
```

**Expected output:**
```
service/svc-node created (dry run)
```

### View What Will Be Created
```bash
kubectl apply -f nodeport-svc.yaml --dry-run=client -o yaml
```

This shows the full resource definition that Kubernetes will create.

---

## Step 4: Apply the YAML Configuration

### Create the NodePort Service
```bash
kubectl apply -f nodeport-svc.yaml
```

**Expected output:**
```
service/svc-node created
```

**What kubectl apply does:**
- Reads the YAML file
- Creates or updates the resource
- Idempotent (safe to run multiple times)
- Tracks changes for future updates

**For .NET Core developers:**
This is similar to deploying infrastructure as code using ARM templates or Terraform. The YAML file is your infrastructure definition.

---

## Step 5: Verify the Service

### List All Services
```bash
kubectl get svc
```

**Expected output:**
```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        5d
svc-demo     ClusterIP   10.96.123.45    <none>        80/TCP         30m
svc-node     NodePort    10.96.234.56    <none>        80:30080/TCP   10s
```

**Key observations:**
- **TYPE:** NodePort (different from svc-demo's ClusterIP)
- **PORT(S):** `80:30080/TCP` means:
  - Internal port 80
  - External NodePort 30080
- **EXTERNAL-IP:** `<none>` (Minikube doesn't assign external IPs)

### Get Detailed Service Information
```bash
kubectl describe svc svc-node
```

**Expected output:**
```
Name:                     svc-node
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=svc-demo
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.234.56
IPs:                      10.96.234.56
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30080/TCP
Endpoints:                172.17.0.3:80,172.17.0.4:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

**Important fields:**
- **Selector:** `app=svc-demo` (same as ClusterIP service, targets same Pods)
- **NodePort:** 30080 (external access port)
- **Endpoints:** Same 2 Pods as the ClusterIP service
- **External Traffic Policy:** Cluster (load balances across all Pods)

### View Service Endpoints
```bash
kubectl get endpoints svc-node
```

**Expected output:**
```
NAME       ENDPOINTS                         AGE
svc-node   172.17.0.3:80,172.17.0.4:80      2m
```

Both services (ClusterIP and NodePort) target the same Pods!

---

## Step 6: Get the Service URL (Minikube-Specific)

Minikube provides a convenient command to get the service URL.

### Get Service URL
```bash
minikube service svc-node --url
```

**Expected output:**
```
http://192.168.49.2:30080
```

**Note:** The IP address may be different in your environment. This is the Minikube VM's IP address.

### Alternative: Get Node IP Manually
```bash
# Get Minikube IP
minikube ip

# Output example: 192.168.49.2
```

The service is accessible at: `http://<minikube-ip>:30080`

---

## Step 7: Test External Access

### Test from WSL Terminal
```bash
# Using the URL from minikube service command
curl http://192.168.49.2:30080

# Or construct URL using minikube ip
curl http://$(minikube ip):30080
```

**Expected output (if you completed Task 3's load balancing test):**
```
Response from Pod 1: svc-demo-7d4b8c9f5d-abc12
```

Or:
```
Response from Pod 2: svc-demo-7d4b8c9f5d-xyz89
```

### Test Multiple Requests to See Load Balancing
```bash
for i in {1..10}; do
  echo "Request $i:"
  curl http://$(minikube ip):30080
  echo ""
done
```

**Expected output (alternating between Pods):**
```
Request 1:
Response from Pod 1: svc-demo-7d4b8c9f5d-abc12

Request 2:
Response from Pod 2: svc-demo-7d4b8c9f5d-xyz89

Request 3:
Response from Pod 1: svc-demo-7d4b8c9f5d-abc12
...
```

### Test from Windows Browser (Outside WSL)

1. Get the Minikube IP:
```bash
minikube ip
```

2. Open your Windows browser and navigate to:
```
http://192.168.49.2:30080
```

Replace `192.168.49.2` with your actual Minikube IP.

**Expected:** You should see the nginx welcome page or custom response from one of the Pods.

### Test from Windows PowerShell
```powershell
# In Windows PowerShell (not WSL)
Invoke-WebRequest -Uri http://192.168.49.2:30080 -UseBasicParsing
```

**For .NET Core developers:**
This demonstrates how external clients (browsers, mobile apps, other services outside the cluster) can access your ASP.NET Core API.

---

## Step 8: Compare ClusterIP vs NodePort Access

### Internal Access (Both Services Work)

Create a test pod if not already present:
```bash
kubectl run test-pod --image=busybox:latest --command -- sleep 3600
```

Test both services from inside the cluster:
```bash
# Access via ClusterIP service (internal only)
kubectl exec -it test-pod -- wget -qO- http://svc-demo:80

# Access via NodePort service (also works internally)
kubectl exec -it test-pod -- wget -qO- http://svc-node:80
```

**Both should work!** NodePort includes ClusterIP functionality.

### External Access (Only NodePort Works)
```bash
# This works - NodePort service from outside cluster
curl http://$(minikube ip):30080

# This would NOT work - ClusterIP service from outside cluster
# curl http://10.96.123.45:80  # Cannot reach cluster IP from host
```

**Key takeaway:** NodePort = ClusterIP + External Access

---

## Step 9: View and Edit the Service

### Export Service Configuration
```bash
kubectl get svc svc-node -o yaml
```

**Full YAML output:**
```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"name":"svc-node","namespace":"default"},...}
  creationTimestamp: "2025-10-29T10:30:00Z"
  name: svc-node
  namespace: default
  resourceVersion: "12345"
  uid: abc-123-def-456
spec:
  clusterIP: 10.96.234.56
  clusterIPs:
  - 10.96.234.56
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 30080
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: svc-demo
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```

### Export to a New File
```bash
kubectl get svc svc-node -o yaml > svc-node-backup.yaml
```

### Edit the Service (Live)
```bash
kubectl edit svc svc-node
```

This opens the service in your default editor. Changes are applied immediately upon saving.

**For .NET Core developers:**
This is useful for quick fixes, but in production, you should always update your YAML file and use `kubectl apply` for version control and GitOps practices.

---

## Step 10: Understand Port Forwarding vs NodePort

### Port Forwarding (Development/Debugging)
```bash
# Forward ClusterIP service to localhost
kubectl port-forward svc/svc-demo 8080:80
```

Then access at: `http://localhost:8080`

**Use case:** Quick local testing, debugging specific Pods

### NodePort (External Access)
Already configured at: `http://$(minikube ip):30080`

**Use case:** Expose service to other machines on the network, integration testing

**Comparison:**

| Method | Scope | Stability | Use Case |
|--------|-------|-----------|----------|
| Port Forward | Local machine only | Temporary (ends when command stops) | Debugging, local dev |
| NodePort | Network accessible | Persistent | Testing, simple external access |

---

## Step 11: Advanced Testing

### Test with Different Tools

**Using wget:**
```bash
wget -qO- http://$(minikube ip):30080
```

**Using curl with headers:**
```bash
curl -v http://$(minikube ip):30080
```

**Using HTTPie (if installed):**
```bash
http http://$(minikube ip):30080
```

### Test Load Balancing with Timing
```bash
for i in {1..20}; do
  echo "Request $i at $(date +%H:%M:%S)"
  curl -s http://$(minikube ip):30080
  sleep 1
done
```

### Monitor Service Endpoints in Real-Time
```bash
# In one terminal, watch endpoints
watch -n 1 kubectl get endpoints svc-node

# In another terminal, make requests
while true; do curl http://$(minikube ip):30080; sleep 1; done
```

---

## Step 12: Modify and Reapply

### Update the YAML File

Edit `nodeport-svc.yaml` to add labels:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-node
  labels:
    environment: development
    managed-by: kubectl
spec:
  type: NodePort
  selector:
    app: svc-demo
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

### Reapply the Configuration
```bash
kubectl apply -f nodeport-svc.yaml
```

**Expected output:**
```
service/svc-node configured
```

**Note:** "configured" instead of "created" because the service already exists.

### Verify the Update
```bash
kubectl describe svc svc-node | grep Labels
```

**Expected output:**
```
Labels:       environment=development
              managed-by=kubectl
```

---

## Real-World .NET Core Example

### Complete YAML for ASP.NET Core API with NodePort

**File: aspnet-api-nodeport.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapi
  labels:
    app: myapi
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapi
  template:
    metadata:
      labels:
        app: myapi
    spec:
      containers:
      - name: myapi
        image: myregistry.azurecr.io/myapi:v1.0
        ports:
        - containerPort: 5000
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: "Development"
        - name: ASPNETCORE_URLS
          value: "http://+:5000"
---
apiVersion: v1
kind: Service
metadata:
  name: myapi-service
  labels:
    app: myapi
spec:
  type: NodePort
  selector:
    app: myapi
  ports:
    - port: 8080
      targetPort: 5000
      nodePort: 30100
```

**Access the API:**
```bash
# Health check endpoint
curl http://$(minikube ip):30100/health

# API endpoint
curl http://$(minikube ip):30100/api/users

# Swagger UI
# Open browser: http://192.168.49.2:30100/swagger
```

### ASP.NET Core Startup Configuration

**Program.cs:**
```csharp
var builder = WebApplication.CreateBuilder(args);

// Configure Kestrel to listen on all interfaces
builder.WebHost.ConfigureKestrel(options =>
{
    options.ListenAnyIP(5000); // Listen on port 5000
});

var app = builder.Build();

// Health check endpoint (important for Kubernetes probes)
app.MapGet("/health", () => Results.Ok(new { status = "healthy" }));

app.Run();
```

---

## Troubleshooting

### Cannot Access via Browser

**Check Minikube is running:**
```bash
minikube status
```

**Get correct IP and port:**
```bash
minikube service svc-node --url
```

**Test from WSL first:**
```bash
curl http://$(minikube ip):30080
```

**Check Windows firewall** if accessing from Windows browser.

### Service Not Found

**Verify service exists:**
```bash
kubectl get svc svc-node
```

**Check YAML was applied:**
```bash
kubectl get svc svc-node -o yaml
```

### Wrong Port or 404 Error

**Verify endpoints:**
```bash
kubectl get endpoints svc-node
```

**Check Pod ports:**
```bash
kubectl get pods -l app=svc-demo -o jsonpath='{.items[*].spec.containers[*].ports[*].containerPort}'
```

### NodePort Out of Range Error

**Error:** "provided port is not in the valid range (30000-32767)"

**Solution:** Change `nodePort` value in YAML to be within range:
```yaml
ports:
  - port: 80
    targetPort: 80
    nodePort: 30080  # Must be 30000-32767
```

### Connection Refused

**Check Pods are running:**
```bash
kubectl get pods -l app=svc-demo
```

**Test Pod directly:**
```bash
POD_NAME=$(kubectl get pods -l app=svc-demo -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD_NAME -- curl localhost:80
```

---

## Cleanup

### Delete NodePort Service Only
```bash
kubectl delete -f nodeport-svc.yaml
# or
kubectl delete svc svc-node
```

### Delete Test Pod
```bash
kubectl delete pod test-pod
```

### Keep Deployment for Further Tasks
```bash
# Verify deployment is still running
kubectl get deployment svc-demo
kubectl get pods -l app=svc-demo
```

### Delete Everything (If Needed)
```bash
kubectl delete deployment svc-demo
kubectl delete svc svc-demo svc-node
kubectl delete pod test-pod
```

---

## Key Takeaways for .NET Core Developers

1. **Declarative Configuration**
   - YAML files define desired state
   - Version controlled infrastructure (GitOps ready)
   - Similar to ARM templates or Terraform for Azure

2. **NodePort = ClusterIP + External Access**
   - Internal cluster DNS still works
   - Additional external access via Node IP
   - Not recommended for production (use LoadBalancer or Ingress)

3. **Port Mapping**
   - NodePort (30000-32767) → Service Port → Container Port
   - Example: 30080 → 8080 → 5000 (ASP.NET Core)

4. **Load Balancing Included**
   - Automatic round-robin across Pods
   - Works for both internal and external traffic
   - No additional configuration needed

5. **Development vs Production**
   - NodePort: Good for development/testing
   - Production: Use Ingress Controller or LoadBalancer
   - Minikube: NodePort is the practical way to test

6. **Infrastructure as Code Benefits**
   - Repeatable deployments
   - Easy rollback (`kubectl apply -f old-version.yaml`)
   - Self-documenting infrastructure
   - CI/CD integration friendly

---

## Comparison: kubectl expose vs YAML

### Imperative (kubectl expose)
```bash
kubectl expose deployment svc-demo --port=80 --target-port=80 --type=NodePort --node-port=30080
```

**Pros:** Quick, simple
**Cons:** Not version controlled, hard to reproduce

### Declarative (YAML)
```yaml
# nodeport-svc.yaml
apiVersion: v1
kind: Service
...
```
```bash
kubectl apply -f nodeport-svc.yaml
```

**Pros:** Version controlled, reproducible, GitOps ready
**Cons:** More verbose, requires file management

**Best practice:** Always use YAML for production and any resources you want to track in version control.

---

## Next Steps

- Learn about Ingress Controllers for production-grade external access
- Explore LoadBalancer services (on cloud providers)
- Implement health checks and readiness probes for your ASP.NET Core apps
- Set up Kubernetes secrets for connection strings and API keys
- Create ConfigMaps for application settings
- Deploy a complete ASP.NET Core microservices application

---

## Summary

You have successfully:
- Created a NodePort Service using declarative YAML configuration
- Understood the three-tier port mapping (NodePort → Service Port → Target Port)
- Accessed the service externally from WSL, Windows, and browser
- Verified load balancing works across multiple Pods
- Compared ClusterIP (internal) vs NodePort (external) access patterns
- Learned Infrastructure as Code practices with Kubernetes YAML
- Applied real-world knowledge for exposing ASP.NET Core applications

The NodePort Service is now providing external access to your deployment, making it accessible from outside the Kubernetes cluster!