Code Name: OPERATION PODSTORM

Objective: Understand what Pods are in Kubernetes and create your first Pod using a YAML file within 3 hours.

---------------------
Task 1: Understand What a Pod Is (30 minutes)
- Research: "What is a Pod in Kubernetes?"
- Grasp key ideas:
  • A Pod is the smallest deployable unit in Kubernetes.
  • A Pod can contain one or more containers sharing storage and network.
  • Containers inside a Pod share the same IP address.
- Optional: Watch a 5-10 minute YouTube explanation on Pods.

---------------------
Task 2: Inspect an Existing Pod Created by a Deployment (30 minutes)
- If you have Minikube or a cluster running, create a sample deployment:
  kubectl create deployment nginx-deploy --image=nginx
- View Pod details:
  kubectl get pods
  kubectl describe pod <pod-name>
- Observe: Status, containers, events, IP, node name.

---------------------
Task 3: Create Your First Pod Using YAML (45 minutes)
- Create a file named pod-demo.yaml with the following content:

  apiVersion: v1
  kind: Pod
  metadata:
    name: demo-pod
  spec:
    containers:
      - name: demo-container
        image: nginx

- Apply the YAML:
  kubectl apply -f pod-demo.yaml
- Confirm Pod is running:
  kubectl get pods

---------------------
Task 4: Explore & Troubleshoot Your Pod (30 minutes)
- Describe Pod and check details:
  kubectl describe pod demo-pod
- Get Pod logs:
  kubectl logs demo-pod
- Enter inside Pod shell (if needed):
  kubectl exec -it demo-pod -- /bin/bash
- Make notes on container status, restart counts, and events.

---------------------
Task 5: Delete and Recreate With Modifications (45 minutes)
- Delete Pod:
  kubectl delete pod demo-pod
- Modify YAML: change image to httpd or alpine and add a command or environment variable:

  apiVersion: v1
  kind: Pod
  metadata:
    name: demo-pod-v2
  spec:
    containers:
      - name: demo-container
        image: httpd
        env:
          - name: APP_ENV
            value: "test"

- Apply again:
  kubectl apply -f pod-demo.yaml (or new filename)
- Confirm behavior and check logs.

---------------------
Mission Complete:
Once done, you will understand what Pods are, how they function, and how to create/manage them using YAML.
End of OPERATION PODSTORM.
