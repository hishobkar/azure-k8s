# .NET Core Task: Understanding Kubernetes Pods

## Task 1: Understand What a Pod Is (30 minutes)

### Objective
Gain a clear understanding of what a **Pod** is in Kubernetes and why it is essential when working with containerized .NET Core applications in a Kubernetes environment.

### Research Topic
- Search for: **"What is a Pod in Kubernetes?"**
- Recommended documentation source: *Kubernetes Official Docs (kubernetes.io)*

### Key Concepts to Understand

✔ **A Pod is the smallest deployable unit in Kubernetes.**  
   - In Kubernetes, you don’t deploy containers directly. Instead, you deploy Pods, which act as wrappers around one or more containers.

✔ **A Pod can contain one or more containers that share the same environment.**  
   - Think of it like a group of containers that must work together closely (e.g., a .NET Core API container with a sidecar logging container).

✔ **Containers inside a Pod share:**
   - The **same network namespace** (including IP address and port space).  
   - The **same storage volumes**, allowing them to easily share data.

✔ **All containers in a Pod are scheduled together**, meaning they are always co-located on the same Kubernetes node.

### Why It Matters for .NET Core Developers
When deploying a .NET Core microservice application on Kubernetes, each microservice typically runs inside its own Pod. Sometimes, helper containers (like monitoring or caching sidecars) are added within the same Pod.

### Optional Learning (Recommended)
- Watch a 5–10 minute YouTube video explaining Kubernetes Pods.
  - Search: **"Kubernetes Pods explained for beginners"**
  - Focus on real-world examples involving microservices (such as .NET Core Web API).

### Estimated Time: 30 minutes

### Summary
A **Pod** is the fundamental building block in Kubernetes. It is a logical host for one or more tightly coupled containers that must share resources such as network and storage. Understanding Pods is critical before working with deployments, services, and scaling in Kubernetes for your .NET Core applications.
