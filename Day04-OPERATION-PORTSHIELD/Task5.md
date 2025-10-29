# Task 5: Compare, Delete & Recreate for Practice

**Duration:** 30 minutes  
**Environment:** WSL with Docker, kubectl, and Minikube installed  
**Target Audience:** .NET Core Developer

---

## Objective

Solidify understanding of Kubernetes Services by comparing ClusterIP and NodePort behaviors, practicing deletion and recreation of services, and verifying load balancing functionality. This hands-on practice reinforces muscle memory for common Kubernetes operations.

---

## Prerequisites

- Completed Tasks 2, 3, and 4
- Deployment `svc-demo` with 2 replicas is running
- Both ClusterIP (`svc-demo`) and NodePort (`svc-node`) services exist
- Minikube is running in WSL

**Verify prerequisites:**
```bash
# Check deployment
kubectl get deployment svc-demo

# Check Pods (should be 2)
kubectl get pods -l app=svc-demo

# Check existing services
kubectl get svc
```

**Expected services:**
```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        5d
svc-demo     ClusterIP   10.96.123.45    <none>        80/TCP         1h
svc-node     NodePort    10.96.234.56    <none>        80:30080/TCP   30m
```

---

## Part 1: Compare ClusterIP vs NodePort Behaviors (15 minutes)

### Comparison Exercise 1: Access Patterns

#### Test ClusterIP Service (Internal Only)

**Create test pod if not exists:**
```bash
kubectl run test-pod --image=busybox:latest --command -- sleep 3600
```

**Wait for Pod to be ready:**
```bash
kubectl get pod test-pod
```

**Test internal access:**
```bash
# Access ClusterIP service from inside cluster
kubectl exec -it test-pod -- wget -qO- http://svc-demo:80
```

**Expected:** Successfully returns response from one of the Pods.

**Try external access (this will fail):**
```bash
# Get ClusterIP
CLUSTER_IP=$(kubectl get svc svc-demo -o jsonpath='{.spec.clusterIP}')
echo $CLUSTER_IP

# Try to access from WSL host (should fail/timeout)
curl --connect-timeout 5 http://$CLUSTER_IP:80
```

**Expected:** Connection timeout or failure. ClusterIP is not accessible from outside the cluster.

#### Test NodePort Service (Internal + External)

**Test internal access:**
```bash
# Access NodePort service from inside cluster (works like ClusterIP)
kubectl exec -it test-pod -- wget -qO- http://svc-node:80
```

**Expected:** Successfully returns response.

**Test external access:**
```bash
# Access from WSL host (works!)
curl http://$(minikube ip):30080
```

**Expected:** Successfully returns response. NodePort is accessible externally.

### Comparison Exercise 2: Service Configuration

**View ClusterIP configuration:**
```bash
kubectl get svc svc-demo -o yaml | grep -A 10 "spec:"
```

**Key fields to observe:**
```yaml
spec:
  clusterIP: 10.96.123.45
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: svc-demo
  type: ClusterIP
```

**View NodePort configuration:**
```bash
kubectl get svc svc-node -o yaml | grep -A 15 "spec:"
```

**Key fields to observe:**
```yaml
spec:
  clusterIP: 10.96.234.56
  ports:
  - nodePort: 30080
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: svc-demo
  type: NodePort
```

**Key differences:**
- NodePort has `nodePort` field (30080)
- Both have `clusterIP` (NodePort includes ClusterIP functionality)
- Both target same Pods via `selector: app=svc-demo`

### Comparison Exercise 3: Endpoints

**Check both services target same Pods:**
```bash
# ClusterIP endpoints
kubectl get endpoints svc-demo

# NodePort endpoints
kubectl get endpoints svc-node
```

**Expected output (both should show same Pod IPs):**
```
NAME       ENDPOINTS                         AGE
svc-demo   172.17.0.3:80,172.17.0.4:80      1h
svc-node   172.17.0.3:80,172.17.0.4:80      30m
```

**Key observation:** Both services route to the same Pods. Multiple services can target the same set of Pods.

### Comparison Exercise 4: Load Balancing Test

**Test ClusterIP load balancing:**
```bash
echo "Testing ClusterIP Service:"
for i in {1..6}; do
  echo "Request $i:"
  kubectl exec -it test-pod -- wget -qO- http://svc-demo:80 2>/dev/null
done
```

**Test NodePort load balancing:**
```bash
echo "Testing NodePort Service:"
for i in {1..6}; do
  echo "Request $i:"
  curl -s http://$(minikube ip):30080
done
```

**Expected:** Both services distribute traffic across both Pods (round-robin or random distribution).

### Summary Table: ClusterIP vs NodePort

| Feature | ClusterIP | NodePort |
|---------|-----------|----------|
| **Type** | ClusterIP | NodePort |
| **Internal Access** | ✓ Yes | ✓ Yes (includes ClusterIP) |
| **External Access** | ✗ No | ✓ Yes |
| **Access URL (Internal)** | `http://svc-demo:80` | `http://svc-node:80` |
| **Access URL (External)** | N/A | `http://<NodeIP>:30080` |
| **Port Configuration** | port, targetPort | port, targetPort, nodePort |
| **Use Case** | Microservice communication | External testing/development |
| **Production Ready** | Yes (internal services) | No (use Ingress/LoadBalancer) |
| **Load Balancing** | ✓ Automatic | ✓ Automatic |
| **DNS Resolution** | ✓ Yes | ✓ Yes |

**For .NET Core developers:**
- **ClusterIP:** Your backend API, database, message queue services
- **NodePort:** Exposing API during local development/testing
- **Production:** Use Ingress Controller (similar to Azure Application Gateway)

---

## Part 2: Delete Services (5 minutes)

### View Current Services
```bash
kubectl get svc
```

**Note the services you're about to delete:**
```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
svc-demo     ClusterIP   10.96.123.45    <none>        80/TCP         1h
svc-node     NodePort    10.96.234.56    <none>        80:30080/TCP   30m
```

### Delete Both Services

**Single command (delete multiple services):**
```bash
kubectl delete svc svc-demo svc-node
```

**Expected output:**
```
service "svc-demo" deleted
service "svc-node" deleted
```

**Alternative (delete one by one):**
```bash
kubectl delete svc svc-demo
kubectl delete svc svc-node
```

**Alternative (delete using label selector):**
```bash
# If services had labels, you could delete by label
kubectl delete svc -l app=svc-demo
```

**Alternative (delete from YAML file):**
```bash
# If you have the YAML file
kubectl delete -f nodeport-svc.yaml
```

### Verify Deletion
```bash
kubectl get svc
```

**Expected output (only default kubernetes service remains):**
```
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5d
```

### Verify Pods Are Still Running

**Important:** Deleting services does NOT delete Pods or Deployments!
```bash
kubectl get pods -l app=svc-demo
```

**Expected output (Pods still running):**
```
NAME                        READY   STATUS    RESTARTS   AGE
svc-demo-7d4b8c9f5d-abc12   1/1     Running   0          2h
svc-demo-7d4b8c9f5d-xyz89   1/1     Running   0          2h
```

**Key observation:** Services and Pods/Deployments are independent. Deleting a service doesn't affect the Pods.

### Verify External Access is Lost

**Try to access (should fail now):**
```bash
curl --connect-timeout 5 http://$(minikube ip):30080
```

**Expected:** Connection refused or timeout. The service no longer exists.

**For .NET Core developers:**
This is like removing the load balancer or reverse proxy in front of your application. The application still runs, but there's no routing to it.

---

## Part 3: Recreate Services Without Notes (10 minutes)

**Challenge:** Recreate both services from memory. Try not to look at previous tasks!

### Recreate ClusterIP Service

**Method 1: Imperative command**
```bash
kubectl expose deployment svc-demo --port=80 --target-port=80 --type=ClusterIP
```

**Expected output:**
```
service/svc-demo exposed
```

**Method 2: YAML (if you prefer declarative approach)**

Create file `clusterip-svc.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-demo
spec:
  type: ClusterIP
  selector:
    app: svc-demo
  ports:
    - port: 80
      targetPort: 80
```

Apply:
```bash
kubectl apply -f clusterip-svc.yaml
```

### Recreate NodePort Service

**Method 1: Imperative command**
```bash
kubectl expose deployment svc-demo --name=svc-node --port=80 --target-port=80 --type=NodePort --node-port=30080
```

**Note:** Use `--name=svc-node` because the deployment name is `svc-demo` but we want service named `svc-node`.

**Expected output:**
```
service/svc-node exposed
```

**Method 2: YAML (recommended approach)**

Create or reuse file `nodeport-svc.yaml`:
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

Apply:
```bash
kubectl apply -f nodeport-svc.yaml
```

### Verify Recreation
```bash
kubectl get svc
```

**Expected output:**
```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        5d
svc-demo     ClusterIP   10.97.45.67     <none>        80/TCP         30s
svc-node     NodePort    10.98.56.78     <none>        80:30080/TCP   20s
```

**Note:** ClusterIP addresses may be different from before (they're dynamically assigned), but functionality is the same.

### Quick Verification Commands
```bash
# Check endpoints are populated
kubectl get endpoints svc-demo
kubectl get endpoints svc-node

# Describe services
kubectl describe svc svc-demo
kubectl describe svc svc-node

# View in YAML format
kubectl get svc svc-demo -o yaml
kubectl get svc svc-node -o yaml
```

---

## Part 4: Confirm Load Balancing (5 minutes)

### Test ClusterIP Load Balancing

**Ensure test pod exists:**
```bash
kubectl get pod test-pod || kubectl run test-pod --image=busybox:latest --command -- sleep 3600
```

**Make multiple requests:**
```bash
echo "Testing ClusterIP load balancing:"
for i in {1..10}; do
  echo -n "Request $i: "
  kubectl exec -it test-pod -- wget -qO- http://svc-demo:80 2>/dev/null | head -1
done
```

**Expected:** Requests distributed across both Pods.

### Test NodePort Load Balancing

**Make multiple requests from WSL:**
```bash
echo "Testing NodePort load balancing:"
for i in {1..10}; do
  echo -n "Request $i: "
  curl -s http://$(minikube ip):30080 | head -1
done
```

**Expected:** Requests distributed across both Pods.

### Verify Both Endpoints Are Active

**Check endpoints:**
```bash
kubectl get endpoints svc-demo
kubectl get endpoints svc-node
```

**Expected (both should show 2 Pod IPs):**
```
NAME       ENDPOINTS                         AGE
svc-demo   172.17.0.3:80,172.17.0.4:80      2m
svc-node   172.17.0.3:80,172.17.0.4:80      2m
```

### Test with Enhanced Output

**Add custom responses to Pods (if not already done):**
```bash
POD1=$(kubectl get pods -l app=svc-demo -o jsonpath='{.items[0].metadata.name}')
POD2=$(kubectl get pods -l app=svc-demo -o jsonpath='{.items[1].metadata.name}')

kubectl exec $POD1 -- /bin/sh -c "echo 'Response from Pod 1: $HOSTNAME' > /usr/share/nginx/html/index.html"
kubectl exec $POD2 -- /bin/sh -c "echo 'Response from Pod 2: $HOSTNAME' > /usr/share/nginx/html/index.html"
```

**Test again to clearly see load balancing:**
```bash
echo "ClusterIP Service:"
for i in {1..6}; do
  kubectl exec -it test-pod -- wget -qO- http://svc-demo:80 2>/dev/null
done

echo -e "\nNodePort Service:"
for i in {1..6}; do
  curl -s http://$(minikube ip):30080
done
```

**Expected output (alternating between Pods):**
```
ClusterIP Service:
Response from Pod 1: svc-demo-7d4b8c9f5d-abc12
Response from Pod 2: svc-demo-7d4b8c9f5d-xyz89
Response from Pod 1: svc-demo-7d4b8c9f5d-abc12
Response from Pod 2: svc-demo-7d4b8c9f5d-xyz89
Response from Pod 1: svc-demo-7d4b8c9f5d-abc12
Response from Pod 2: svc-demo-7d4b8c9f5d-xyz89

NodePort Service:
Response from Pod 1: svc-demo-7d4b8c9f5d-abc12
Response from Pod 2: svc-demo-7d4b8c9f5d-xyz89
Response from Pod 1: svc-demo-7d4b8c9f5d-abc12
Response from Pod 2: svc-demo-7d4b8c9f5d-xyz89
Response from Pod 1: svc-demo-7d4b8c9f5d-abc12
Response from Pod 2: svc-demo-7d4b8c9f5d-xyz89
```

**Confirmation:** Load balancer is distributing traffic to both Pods successfully!

---

## Additional Practice Exercises

### Exercise 1: Delete and Recreate with Different Names

**Delete services:**
```bash
kubectl delete svc svc-demo svc-node
```

**Recreate with new names:**
```bash
# ClusterIP with name "backend"
kubectl expose deployment svc-demo --name=backend --port=80 --target-port=80 --type=ClusterIP

# NodePort with name "frontend"
kubectl expose deployment svc-demo --name=frontend --port=80 --target-port=80 --type=NodePort --node-port=30090
```

**Test:**
```bash
kubectl exec -it test-pod -- wget -qO- http://backend:80
curl http://$(minikube ip):30090
```

**Cleanup:**
```bash
kubectl delete svc backend frontend
```

### Exercise 2: Recreate with Different Ports

**Recreate NodePort on different port:**
```bash
kubectl expose deployment svc-demo --name=svc-node --port=8080 --target-port=80 --type=NodePort --node-port=30100
```

**Test (note the port 8080 for internal, 30100 for external):**
```bash
# Internal uses port 8080
kubectl exec -it test-pod -- wget -qO- http://svc-node:8080

# External uses NodePort 30100
curl http://$(minikube ip):30100
```

**Cleanup:**
```bash
kubectl delete svc svc-node
```

### Exercise 3: Practice Speed Recreation

**Time yourself recreating both services:**
```bash
# Start timer
time {
  kubectl delete svc svc-demo svc-node 2>/dev/null
  kubectl expose deployment svc-demo --port=80 --target-port=80 --type=ClusterIP
  kubectl expose deployment svc-demo --name=svc-node --port=80 --target-port=80 --type=NodePort --node-port=30080
  kubectl get svc
}
```

**Goal:** Complete in under 10 seconds.

---

## Common Mistakes and How to Avoid Them

### Mistake 1: Forgetting --name Flag
```bash
# Wrong: Creates service with name "svc-demo" (conflicts with ClusterIP)
kubectl expose deployment svc-demo --type=NodePort

# Correct: Specify different name
kubectl expose deployment svc-demo --name=svc-node --type=NodePort
```

### Mistake 2: Wrong Port Specification
```bash
# Wrong: targetPort doesn't match container port
kubectl expose deployment svc-demo --port=80 --target-port=8080

# Correct: Match container's actual port (nginx uses 80)
kubectl expose deployment svc-demo --port=80 --target-port=80
```

### Mistake 3: NodePort Outside Valid Range
```bash
# Wrong: Port 8080 is outside NodePort range
kubectl expose deployment svc-demo --type=NodePort --node-port=8080

# Correct: Use port in range 30000-32767
kubectl expose deployment svc-demo --type=NodePort --node-port=30080
```

### Mistake 4: Not Verifying Endpoints
```bash
# Always check endpoints after creating service
kubectl get endpoints svc-demo

# If empty, check selector matches Pod labels
kubectl get pods --show-labels
kubectl describe svc svc-demo | grep Selector
```

---

## Quick Reference Commands

### Service Creation
```bash
# ClusterIP (imperative)
kubectl expose deployment <name> --port=<port> --target-port=<port> --type=ClusterIP

# NodePort (imperative)
kubectl expose deployment <name> --name=<svc-name> --port=<port> --target-port=<port> --type=NodePort --node-port=<30000-32767>

# From YAML
kubectl apply -f service.yaml
```

### Service Management
```bash
# List services
kubectl get svc

# Detailed info
kubectl describe svc <name>

# YAML output
kubectl get svc <name> -o yaml

# Delete service
kubectl delete svc <name>

# Delete multiple
kubectl delete svc <name1> <name2>
```

### Verification
```bash
# Check endpoints
kubectl get endpoints <name>

# Test internal access
kubectl exec -it <pod> -- curl http://<service>:<port>

# Test external access (NodePort)
curl http://$(minikube ip):<nodePort>

# Get Minikube IP
minikube ip

# Get service URL (Minikube)
minikube service <name> --url
```

---

## For .NET Core Developers: Real-World Scenario

### Typical Microservices Setup

**Architecture:**
```
[API Gateway - NodePort 30080] → External traffic
    ↓
[API Service - ClusterIP] → Backend API
    ↓
[Database Service - ClusterIP] → PostgreSQL
```

**Quick recreation commands:**
```bash
# Assuming deployments: api-gateway, api-service, database

# API Gateway (external access)
kubectl expose deployment api-gateway --name=gateway --port=80 --target-port=5000 --type=NodePort --node-port=30080

# API Service (internal)
kubectl expose deployment api-service --name=api --port=8080 --target-port=5000 --type=ClusterIP

# Database (internal)
kubectl expose deployment database --name=postgres --port=5432 --target-port=5432 --type=ClusterIP
```

**Connection strings in appsettings.json:**
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=postgres;Port=5432;Database=mydb;Username=user;Password=pass"
  },
  "Services": {
    "ApiUrl": "http://api:8080"
  }
}
```

---

## Self-Assessment Checklist

After completing this task, you should be able to:

- [ ] Explain the difference between ClusterIP and NodePort
- [ ] Create a ClusterIP service using imperative command
- [ ] Create a NodePort service using imperative command
- [ ] Create services using YAML manifests
- [ ] Delete one or multiple services
- [ ] Verify service endpoints are correct
- [ ] Test internal service connectivity
- [ ] Test external service connectivity (NodePort)
- [ ] Confirm load balancing across multiple Pods
- [ ] Understand that services and Pods are independent
- [ ] Recreate services quickly without documentation

**If you can do all of the above, you've mastered basic Kubernetes Services!**

---

## Cleanup

### Delete Test Pod
```bash
kubectl delete pod test-pod
```

### Keep Services for Next Tasks
```bash
# Verify both services exist
kubectl get svc
```

**Expected:**
```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
svc-demo     ClusterIP   10.97.45.67     <none>        80/TCP         10m
svc-node     NodePort    10.98.56.78     <none>        80:30080/TCP   10m
```

### Complete Cleanup (If Finishing All Tasks)
```bash
kubectl delete deployment svc-demo
kubectl delete svc svc-demo svc-node
kubectl delete pod test-pod
```

---

## Summary

You have successfully:
- Compared ClusterIP and NodePort services side-by-side
- Understood access patterns (internal vs external)
- Deleted services and verified Pods remain unaffected
- Recreated both services from memory (imperative and/or declarative)
- Confirmed load balancing works correctly across both Pods
- Practiced common Kubernetes service operations
- Built muscle memory for quick service creation and management

**Key insight for .NET Core developers:**
Services in Kubernetes are like load balancers and service discovery mechanisms combined. They provide stable endpoints for your microservices, similar to Azure Load Balancer or Service Fabric naming, but with simpler configuration and automatic management.

You now have hands-on experience creating, managing, and understanding Kubernetes Services!