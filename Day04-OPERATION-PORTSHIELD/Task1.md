# Task 1: Understand the Role of Services in Kubernetes

**Duration:** 30 minutes  
**Environment:** WSL with Docker, kubectl, and Minikube installed  
**Target Audience:** .NET Core Developer

---

## Objective

Research and understand Kubernetes Services, their purpose, types (ClusterIP and NodePort), and how they provide stable networking to Pods.

---

## Part 1: Research and Summarize

### 1. What problem do Services solve in Kubernetes?

**Problems without Services:**
- Pods are ephemeral (can be created, destroyed, or rescheduled)
- Pod IP addresses change when Pods restart or are recreated
- No built-in load balancing across multiple Pod replicas
- Difficult to discover and connect to Pods reliably

**How Services solve these problems:**
- Provide a stable IP address and DNS name for accessing Pods
- Abstract away the dynamic nature of Pod IPs
- Enable load balancing across multiple Pod replicas
- Act as a service discovery mechanism
- Decouple consumers from Pod lifecycle changes

**Real-world analogy for .NET developers:**
Think of a Service like a reverse proxy (similar to IIS or nginx) that sits in front of your ASP.NET Core application instances. Even if you scale up/down or restart your app instances, the proxy's endpoint remains constant for clients.

---

### 2. What are ClusterIP and NodePort Services?

#### ClusterIP Service
- **Default Service type**
- Creates a virtual IP address accessible only within the cluster
- Internal networking only - cannot be accessed from outside the cluster
- Use case: Internal microservice communication (e.g., API → Database, Frontend → Backend API)

#### NodePort Service
- Exposes the Service on each Node's IP at a static port (30000-32767 range by default)
- Makes the Service accessible from outside the cluster
- Automatically creates a ClusterIP Service as well
- Access pattern: `<NodeIP>:<NodePort>`
- Use case: Development/testing, or when you need simple external access without a LoadBalancer

---

### 3. How Services provide stable networking to Pods

**Selector-based Pod discovery:**
- Services use label selectors to identify target Pods
- Example: `app: myapp` selector matches all Pods with that label
- Dynamic membership - Pods are automatically added/removed based on labels

**Stable endpoint:**
- Service gets a cluster-internal DNS name: `<service-name>.<namespace>.svc.cluster.local`
- Example: `myapi-service.default.svc.cluster.local`
- IP address remains stable even when backend Pods change

**Load balancing:**
- Traffic is distributed across all healthy Pods matching the selector
- Uses round-robin by default
- Only routes to ready Pods (based on readiness probes)

**Connection flow example:**
```
Client Request → Service (stable IP/DNS) → Selector matches Pods → 
Load balances → Forwards to healthy Pod → Pod handles request
```

---

## Part 2: Key Differences

| Aspect | ClusterIP | NodePort |
|--------|-----------|----------|
| **Accessibility** | Internal only (within cluster) | External (from outside cluster) |
| **IP Address** | Virtual IP within cluster network | Uses Node's IP address |
| **Port Range** | Standard service ports | 30000-32767 (by default) |
| **Use Case** | Microservice-to-microservice | Development, testing, simple external access |
| **DNS** | `<service>.default.svc.cluster.local` | Same + accessible via `<NodeIP>:<NodePort>` |
| **Typical Scenario** | Database, internal API, message queue | Exposing API during development |
| **Production Ready** | Yes (for internal services) | Not recommended (use LoadBalancer/Ingress) |

---

## Summary for .NET Core Developers

**Think of Services as:**
- **ClusterIP** = Private endpoints for internal microservices (like calling `http://api-service` from another container)
- **NodePort** = Public endpoints during dev/test (like exposing your ASP.NET Core API on `http://localhost:30080`)

**Key Takeaways:**
1. Services provide stable networking despite Pod IP changes
2. ClusterIP keeps traffic internal - perfect for backend APIs and databases
3. NodePort opens external access - useful for development but not ideal for production
4. Services use label selectors to dynamically track Pods
5. Built-in load balancing across Pod replicas

---

## Hands-On Verification Commands

After completing research, verify your understanding:
```bash
# Start Minikube
minikube start

# Check service types available
kubectl explain service.spec.type

# View existing services
kubectl get services

# Get detailed service info
kubectl describe service <service-name>

# Check service endpoints (which Pods it routes to)
kubectl get endpoints <service-name>
```

---

## Next Steps

1. Create sample ClusterIP Service for an ASP.NET Core API
2. Test internal connectivity between Pods
3. Create NodePort Service to expose API externally
4. Access the API from your host machine via NodePort
5. Compare behavior and use cases

---

## Additional Resources

- Kubernetes Documentation: https://kubernetes.io/docs/concepts/services-networking/service/
- .NET Microservices Guide: https://learn.microsoft.com/en-us/dotnet/architecture/microservices/