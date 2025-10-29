# Task 3: Create a ClusterIP Service

**Duration:** 45 minutes  
**Environment:** WSL with Docker, kubectl, and Minikube installed  
**Target Audience:** .NET Core Developer

---

## Objective

Create a ClusterIP Service to provide stable internal networking for the deployment created in Task 2. Test internal connectivity and observe load balancing across multiple Pod replicas.

---

## Prerequisites

- Completed Task 2 (Deployment `svc-demo` with 2 replicas is running)
- Minikube is running in WSL
- kubectl is configured and connected to cluster

**Verify prerequisites:**
```bash
# Check deployment exists
kubectl get deployment svc-demo

# Verify 2 Pods are running
kubectl get pods -l app=svc-demo

# Expected: 2 Pods in Running status
```

---

## Step 1: Create a ClusterIP Service

**Command:**
```bash
kubectl expose deployment svc-demo --port=80 --target-port=80 --type=ClusterIP
```

**What this does:**
- Creates a Service named `svc-demo` (same as deployment name)
- Type: `ClusterIP` (internal access only)
- Port: `80` (the port the Service listens on)
- Target Port: `80` (the port on the Pod containers)
- Automatically uses deployment's label selector (`app=svc-demo`)

**Expected output:**
```
service/svc-demo exposed
```

**For .NET Core developers:**
In a real scenario with ASP.NET Core API:
```bash
kubectl expose deployment myapi --port=8080 --target-port=5000 --type=ClusterIP
```
Where:
- `--port=8080`: Service listens on port 8080
- `--target-port=5000`: Your ASP.NET Core app runs on port 5000 in the container

---

## Step 2: Verify the Service

### View Services
```bash
kubectl get svc
```

**Expected output:**
```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   5d
svc-demo     ClusterIP   10.96.123.45    <none>        80/TCP    10s
```

**Understanding the output:**
- **NAME:** Service name
- **TYPE:** ClusterIP (internal only)
- **CLUSTER-IP:** Virtual IP address assigned to the Service (stable, doesn't change)
- **EXTERNAL-IP:** None (ClusterIP doesn't expose externally)
- **PORT(S):** Service port (80/TCP)
- **AGE:** Time since Service was created

**Key observation:** The CLUSTER-IP (e.g., 10.96.123.45) is a stable virtual IP that won't change, unlike Pod IPs.

### Get Detailed Service Information
```bash
kubectl describe svc svc-demo
```

**Expected output:**
```
Name:              svc-demo
Namespace:         default
Labels:            app=svc-demo
Annotations:       <none>
Selector:          app=svc-demo
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.96.123.45
IPs:               10.96.123.45
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         172.17.0.3:80,172.17.0.4:80
Session Affinity:  None
Events:            <none>
```

**Important fields:**
- **Selector:** `app=svc-demo` (matches Pods with this label)
- **Endpoints:** IP addresses of the 2 Pods (these are the actual targets)
- **IP:** The stable ClusterIP address

### View Service Endpoints
```bash
kubectl get endpoints svc-demo
```

**Expected output:**
```
NAME       ENDPOINTS                         AGE
svc-demo   172.17.0.3:80,172.17.0.4:80      2m
```

**What this shows:**
- The Service is routing traffic to 2 Pod IPs
- Both Pods are healthy and ready to receive traffic
- If a Pod is deleted, it will be automatically removed from endpoints

---

## Step 3: Test Internal Access

ClusterIP Services are only accessible from within the cluster. To test, we'll create a temporary test Pod.

### Create a Test Pod
```bash
kubectl run test-pod --image=busybox:latest --command -- sleep 3600
```

**What this does:**
- Creates a Pod named `test-pod` using busybox image (lightweight Linux utilities)
- Runs `sleep 3600` to keep the Pod alive for 1 hour
- We'll use this Pod to test connectivity to the Service

**Expected output:**
```
pod/test-pod created
```

### Wait for Test Pod to be Ready
```bash
kubectl get pods test-pod
```

**Wait until STATUS shows "Running":**
```
NAME       READY   STATUS    RESTARTS   AGE
test-pod   1/1     Running   0          10s
```

### Test Service Connectivity Using ClusterIP
```bash
# Get the ClusterIP
kubectl get svc svc-demo -o jsonpath='{.spec.clusterIP}'

# Test using ClusterIP (replace with actual IP)
kubectl exec -it test-pod -- wget -qO- http://10.96.123.45:80
```

**Expected output:**
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
</html>
```

### Test Service Connectivity Using DNS (Recommended)
```bash
kubectl exec -it test-pod -- wget -qO- http://svc-demo:80
```

**Alternative commands:**
```bash
# Using the full DNS name
kubectl exec -it test-pod -- wget -qO- http://svc-demo.default.svc.cluster.local:80

# Shorter version (port 80 is default)
kubectl exec -it test-pod -- wget -qO- svc-demo
```

**Expected output:**
Same nginx HTML page as above.

**For .NET Core developers:**
This is equivalent to calling your API from another service:
```bash
kubectl exec -it test-pod -- wget -qO- http://myapi:8080/api/health
```

---

## Step 4: Observe Load Balancing

To see load balancing in action, we'll make multiple requests and identify which Pod handles each request.

### Add Custom Index Pages to Each Pod

First, let's modify each Pod to return a unique identifier:
```bash
# Get Pod names
POD1=$(kubectl get pods -l app=svc-demo -o jsonpath='{.items[0].metadata.name}')
POD2=$(kubectl get pods -l app=svc-demo -o jsonpath='{.items[1].metadata.name}')

# Add custom content to Pod 1
kubectl exec -it $POD1 -- /bin/sh -c "echo 'Response from Pod 1: $HOSTNAME' > /usr/share/nginx/html/index.html"

# Add custom content to Pod 2
kubectl exec -it $POD2 -- /bin/sh -c "echo 'Response from Pod 2: $HOSTNAME' > /usr/share/nginx/html/index.html"
```

### Test Load Balancing
```bash
# Make 10 requests and observe which Pod responds
for i in {1..10}; do
  echo "Request $i:"
  kubectl exec -it test-pod -- wget -qO- http://svc-demo
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

Request 4:
Response from Pod 2: svc-demo-7d4b8c9f5d-xyz89
...
```

**Observation:** Traffic is distributed across both Pods in a round-robin fashion (default load balancing algorithm).

### View Real-Time Traffic Distribution
```bash
# In one terminal, watch Pod logs for Pod 1
kubectl logs -f $POD1

# In another terminal, watch Pod logs for Pod 2
kubectl logs -f $POD2

# In a third terminal, make requests
kubectl exec -it test-pod -- /bin/sh -c "while true; do wget -qO- http://svc-demo; sleep 1; done"
```

Press `Ctrl+C` to stop.

---

## Step 5: Test Service Resilience

Demonstrate how the Service automatically handles Pod failures.

### View Current Endpoints
```bash
kubectl get endpoints svc-demo
```

**Output shows 2 endpoints:**
```
NAME       ENDPOINTS                         AGE
svc-demo   172.17.0.3:80,172.17.0.4:80      10m
```

### Delete One Pod
```bash
# Delete Pod 1
kubectl delete pod $POD1
```

### Check Endpoints Immediately
```bash
kubectl get endpoints svc-demo
```

**Initially, you'll see only 1 endpoint:**
```
NAME       ENDPOINTS          AGE
svc-demo   172.17.0.4:80      11m
```

### Watch Deployment Recreate the Pod
```bash
kubectl get pods -l app=svc-demo -w
```

**You'll see:**
- One Pod terminating
- A new Pod being created
- New Pod reaching Running status

### Verify Endpoints are Restored
```bash
kubectl get endpoints svc-demo
```

**Now 2 endpoints again, but Pod 1 has a new IP:**
```
NAME       ENDPOINTS                         AGE
svc-demo   172.17.0.4:80,172.17.0.5:80      12m
```

**Key takeaway:** The Service automatically updated its endpoints when the Pod was recreated with a new IP address. Consumers using the Service DNS name (`svc-demo`) don't need to know about this change!

---

## Step 6: Verify DNS Resolution

### Check Service DNS Name
```bash
kubectl exec -it test-pod -- nslookup svc-demo
```

**Expected output:**
```
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      svc-demo
Address 1: 10.96.123.45 svc-demo.default.svc.cluster.local
```

### Test Different DNS Formats
```bash
# Short name (within same namespace)
kubectl exec -it test-pod -- wget -qO- http://svc-demo

# With namespace
kubectl exec -it test-pod -- wget -qO- http://svc-demo.default

# Fully qualified domain name (FQDN)
kubectl exec -it test-pod -- wget -qO- http://svc-demo.default.svc.cluster.local
```

All three formats should work.

**For .NET Core developers:**
In your appsettings.json or environment variables:
```json
{
  "ApiSettings": {
    "DatabaseUrl": "http://postgres-service:5432",
    "CacheUrl": "http://redis-service:6379"
  }
}
```

---

## Step 7: View Service Configuration (YAML)

### Export Service Configuration
```bash
kubectl get svc svc-demo -o yaml
```

**Key sections in the output:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-demo
  namespace: default
spec:
  clusterIP: 10.96.123.45
  clusterIPs:
  - 10.96.123.45
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: svc-demo
  type: ClusterIP
```

**Understanding the YAML:**
- **clusterIP:** The stable virtual IP
- **ports:** Service port and target port on Pods
- **selector:** Label used to find target Pods
- **type:** ClusterIP (internal only)

---

## Step 8: Additional Testing Commands

### Test from Within a Pod (Interactive Shell)
```bash
kubectl exec -it test-pod -- /bin/sh

# Inside the Pod shell:
wget -qO- http://svc-demo
ping svc-demo
nslookup svc-demo
exit
```

### Check Service Port Forwarding (Alternative Testing)
```bash
# Forward Service port to localhost (for testing purposes)
kubectl port-forward svc/svc-demo 8080:80
```

Then in another terminal or browser:
```bash
curl http://localhost:8080
```

Press `Ctrl+C` to stop port forwarding.

---

## Understanding ClusterIP for .NET Core Developers

### Scenario: Microservices Architecture

Imagine you have:
- **Frontend Service:** ASP.NET Core Blazor/Razor Pages
- **API Service:** ASP.NET Core Web API
- **Database:** PostgreSQL

**Without Services:**
```
Frontend → Need to know API Pod IPs (172.17.0.3, 172.17.0.4, ...)
Problems: IPs change, manual load balancing, service discovery issues
```

**With ClusterIP Service:**
```
Frontend → http://api-service:8080 → Service → Load balances to API Pods
Database → postgres-service:5432 → Service → Postgres Pod
```

**Configuration in ASP.NET Core:**
```csharp
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=postgres-service;Port=5432;Database=mydb"
  },
  "ExternalServices": {
    "ApiBaseUrl": "http://api-service:8080"
  }
}

// HttpClient configuration
services.AddHttpClient("ApiClient", client =>
{
    client.BaseAddress = new Uri("http://api-service:8080");
});
```

---

## Troubleshooting

### Service Not Responding
```bash
# Check if Pods are running
kubectl get pods -l app=svc-demo

# Check Service endpoints
kubectl get endpoints svc-demo

# Verify Pod labels match Service selector
kubectl get pods --show-labels
```

### DNS Resolution Issues
```bash
# Check CoreDNS is running
kubectl get pods -n kube-system | grep coredns

# Test DNS from test pod
kubectl exec -it test-pod -- nslookup kubernetes.default
```

### Cannot Create Test Pod
```bash
# Check Pod status
kubectl describe pod test-pod

# View logs if Pod is failing
kubectl logs test-pod
```

### Connection Timeout
```bash
# Verify target port matches container port
kubectl describe pod <pod-name> | grep Port

# Check if nginx is running in Pods
kubectl exec -it <pod-name> -- curl localhost:80
```

---

## Cleanup

### Delete Test Pod
```bash
kubectl delete pod test-pod
```

### Delete Service (if needed for next task)
```bash
kubectl delete svc svc-demo
```

**Note:** Keep the deployment for the next task (NodePort Service).

---

## Key Takeaways for .NET Core Developers

1. **ClusterIP provides stable internal networking**
   - Service DNS name never changes
   - Abstracts away Pod IP changes
   - Works like an internal load balancer

2. **Service discovery via DNS**
   - Use service name instead of IP addresses
   - Format: `service-name.namespace.svc.cluster.local`
   - Short name works within same namespace

3. **Automatic load balancing**
   - Round-robin by default
   - Only routes to healthy Pods
   - No configuration needed

4. **Self-healing**
   - Service automatically updates endpoints when Pods change
   - Consumers don't need to know about Pod lifecycle

5. **Similar to internal APIs in .NET**
   - Like using HttpClient with a constant base URL
   - Backend service instances can scale without affecting clients
   - Comparable to Azure Internal Load Balancer or Service Fabric naming

---

## Real-World .NET Core Example

**Deployment YAML for ASP.NET Core API:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapi
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
        image: myregistry/myapi:latest
        ports:
        - containerPort: 5000
---
apiVersion: v1
kind: Service
metadata:
  name: myapi-service
spec:
  type: ClusterIP
  selector:
    app: myapi
  ports:
  - port: 8080
    targetPort: 5000
```

**Accessing from another service:**
```csharp
var response = await httpClient.GetAsync("http://myapi-service:8080/api/users");
```

---

## Next Steps

- Create a NodePort Service to expose the application externally
- Compare ClusterIP vs NodePort behavior
- Understand when to use each Service type
- Learn about LoadBalancer and Ingress for production scenarios

---

## Summary

You have successfully:
- Created a ClusterIP Service for internal networking
- Verified the Service is routing traffic to both Pods
- Tested connectivity using Service DNS name
- Observed automatic load balancing across replicas
- Demonstrated Service resilience when Pods are recreated
- Understood how Services provide stable networking for microservices

The ClusterIP Service is now providing reliable, load-balanced access to your deployment within the cluster!