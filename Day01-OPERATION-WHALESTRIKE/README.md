Code Name: OPERATION WHALESTRIKE

Objective: Understand and practically set up Docker, kubectl, and Minikube within 3 hours.

---------------------
Task 1: Install Docker (45 minutes)
- Go to the official Docker website and download Docker Desktop.
- Install and launch Docker.
- Verify installation by running:
  docker --version
  docker run hello-world

---------------------
Task 2: Install kubectl (20 minutes)
- Download kubectl using the official Kubernetes documentation instructions.
- Verify installation by running:
  kubectl version --client

---------------------
Task 3: Install Minikube (25 minutes)
- Download Minikube from the official website.
- Install it and verify using:
  minikube version

---------------------
Task 4: Launch Your First Local Kubernetes Cluster (30 minutes)
- Start Minikube:
  minikube start
- Check cluster status:
  kubectl get nodes
- Stop the cluster when done:
  minikube stop

---------------------
Task 5: Deploy a Sample App to Understand Cluster Workflow (1 hour)
- Create a simple deployment:
  kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
- Expose it:
  kubectl expose deployment hello-minikube --type=NodePort --port=8080
- Get service URL:
  minikube service hello-minikube --url
- Open in browser to confirm it works.
- Delete everything:
  kubectl delete service hello-minikube
  kubectl delete deployment hello-minikube

---------------------
Mission Complete:
Once all tasks are completed, you will have a basic functional understanding of Docker, kubectl, and Minikube usage.
End of OPERATION WHALESTRIKE.
