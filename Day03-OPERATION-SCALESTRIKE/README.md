Code Name: OPERATION SCALESTRIKE

Objective: Understand Kubernetes Deployments and ReplicaSets, and learn how to scale Pods within 3 hours.

---------------------
Task 1: Understand Deployments & ReplicaSets Conceptually (30 minutes)
- Research the following:
  • What is a Deployment in Kubernetes?
  • What is a ReplicaSet, and how is it related to a Deployment?
  • How Deployments ensure desired Pod count.
- Note: A Deployment manages ReplicaSets, which in turn manage Pods.

---------------------
Task 2: Create a Deployment and Observe the ReplicaSet (30 minutes)
- Create a Deployment:
  kubectl create deployment web-deploy --image=nginx
- Verify Deployment and ReplicaSet:
  kubectl get deployments
  kubectl get rs
  kubectl get pods
- Observe that the ReplicaSet is automatically created by the Deployment.

---------------------
Task 3: Scale the Deployment to Increase Pod Count (45 minutes)
- Scale to 3 replicas:
  kubectl scale deployment web-deploy --replicas=3
- Verify:
  kubectl get deployments
  kubectl get rs
  kubectl get pods
- Make notes on how the ReplicaSet changes.

---------------------
Task 4: Create Deployment Using YAML (45 minutes)
- Create a file named deployment-demo.yaml with:

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

- Apply and verify:
  kubectl apply -f deployment-demo.yaml
  kubectl get deployments
  kubectl get rs
  kubectl get pods

---------------------
Task 5: Update Deployment & Observe Rolling Update (30 minutes)
- Update image:
  kubectl set image deployment/app-deploy myapp-container=nginx:alpine
- Monitor rollout:
  kubectl rollout status deployment/app-deploy
- View history:
  kubectl rollout history deployment/app-deploy
- Rollback (optional):
  kubectl rollout undo deployment/app-deploy

---------------------
Mission Complete:
You now understand Deployments, ReplicaSets, and how to scale Pods and perform rolling updates.
End of OPERATION SCALESTRIKE.
