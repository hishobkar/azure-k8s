# Kubernetes Deployments & ReplicaSets - Task 1

## Objective
Understand the conceptual relationship between **Deployments** and **ReplicaSets** in Kubernetes, and how they ensure the desired number of running Pods.

---

## What is a Deployment in Kubernetes?

A **Deployment** is a higher-level Kubernetes object used to manage and automate the lifecycle of applications. It provides declarative updates for Pods and ReplicaSets.

### Key Features:
- Manages application rollout and rollback.
- Ensures the desired state is always maintained.
- Provides versioning and update strategies.
- Enables scaling up/down application instances.

In practice, you define a Deployment with a desired Pod template, and Kubernetes ensures that the correct number of Pods are running at all times.

---

## What is a ReplicaSet?

A **ReplicaSet** ensures that a specified number of identical Pods (replicas) are running at any given time.

### Responsibilities of a ReplicaSet:
- Monitors Pods.
- Ensures the number of Pods matches the desired count.
- Creates new Pods if some are deleted or fail.

---

## Relationship Between Deployment and ReplicaSet

- A **Deployment manages one or more ReplicaSets**.
- When a Deployment is applied, it creates a ReplicaSet to manage the Pods.
- During updates (e.g., a new image version), the Deployment creates a new ReplicaSet for the updated Pods.
- The old ReplicaSet may be retained for rollback purposes.

### Hierarchy:
```
Deployment → ReplicaSet → Pods
```

---

## How Deployments Ensure Desired Pod Count

1. The Deployment specifies the number of replicas (e.g., `replicas: 3`).
2. The Deployment creates a ReplicaSet with that replica count.
3. The ReplicaSet continuously monitors the number of running Pods.
4. If a Pod fails or is deleted, the ReplicaSet automatically recreates it to match the desired state.
5. If scaling occurs (`kubectl scale deployment ...`), the Deployment updates the ReplicaSet accordingly.

---

## Suggested Time: 30 Minutes
- Read official Kubernetes documentation.
- Understand the flow: **Deployment → ReplicaSet → Pod management**.
- Summarize key concepts and relationships.

---

## Note
A **Deployment manages ReplicaSets**, which in turn **manage Pods** to ensure the correct number of application instances are running.

