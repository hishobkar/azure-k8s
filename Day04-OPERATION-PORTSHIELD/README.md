Code Name: OPERATION PORTSHIELD

Objective: Understand and work with Kubernetes Services, focusing on ClusterIP and NodePort types, within 3 hours.

---------------------
Task 1: Understand the Role of Services (30 minutes)
- Research and summarize:
  • What problem Services solve in Kubernetes.
  • What ClusterIP and NodePort Services are.
  • How Services provide stable networking to Pods.
- Write down differences:
  • ClusterIP → Internal access only.
  • NodePort → Accessible externally via <NodeIP>:<Port>.

---------------------
Task 2: Create a Deployment to Expose (30 minutes)
- Create a deployment:
  kubectl create deployment svc-demo --image=nginx
- Scale it to 2 replicas:
  kubectl scale deployment svc-demo --replicas=2
- Verify Pods:
  kubectl get pods -o wide

---------------------
Task 3: Create a ClusterIP Service (45 minutes)
- Expose the deployment:
  kubectl expose deployment svc-demo --port=80 --target-port=80 --type=ClusterIP
- Verify service:
  kubectl get svc
- Test internal access (if using Minikube):
  kubectl run test-pod --image=busybox:latest --command -- sleep 3600
  kubectl exec -it test-pod -- wget -qO- svc-demo
- Observe how traffic is load-balanced across Pods.

---------------------
Task 4: Create a NodePort Service via YAML (45 minutes)
- Create a file named nodeport-svc.yaml with:

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

- Apply and verify:
  kubectl apply -f nodeport-svc.yaml
  kubectl get svc
- Access using:
  minikube service svc-node --url
  or http://<NodeIP>:30080

---------------------
Task 5: Compare, Delete & Recreate for Practice (30 minutes)
- Compare behaviors of ClusterIP vs NodePort.
- Delete services:
  kubectl delete svc svc-demo svc-node
- Recreate both quickly without looking at notes.
- Note: Confirm service load balancer distributes traffic to both Pods again.

---------------------
Mission Complete:
You now understand how Kubernetes Services (ClusterIP and NodePort) enable stable networking and external/internal access to Pods.
End of OPERATION PORTSHIELD.
