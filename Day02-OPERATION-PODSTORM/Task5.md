# .NET Core Task: Delete and Recreate Pod with Modifications

## Task 5: Delete and Recreate With Modifications (45 minutes)

### Objective
Delete an existing Pod and recreate it with modified configurations such as a different image and environment variables. This task helps .NET Core developers understand how to iterate on application configurations in Kubernetes using YAML.

---

### Steps

1. **Delete the Existing Pod**
   - Run the command:
     ```bash
     kubectl delete pod demo-pod
     ```

2. **Modify YAML File**
   - Update your existing `pod-demo.yaml` or create a new file (e.g., `pod-demo-v2.yaml`) with the following content:
     ```yaml
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
     ```

3. **Apply the Modified YAML**
   - Run:
     ```bash
     kubectl apply -f pod-demo.yaml      # If overwritten
     ```
     OR
     ```bash
     kubectl apply -f pod-demo-v2.yaml   # If using a new file
     ```

4. **Confirm Pod Status**
   - Check if the Pod is running:
     ```bash
     kubectl get pods
     ```

5. **Inspect Pod Behavior**
   - View details:
     ```bash
     kubectl describe pod demo-pod-v2
     ```
   - Check logs:
     ```bash
     kubectl logs demo-pod-v2
     ```

6. **(Optional) Enter Pod Shell (if compatible)**
   - If using an image like `alpine`, run:
     ```bash
     kubectl exec -it demo-pod-v2 -- /bin/sh
     ```

---

### What to Observe

| Observation Area | What to Check |
|------------------|--------------|
| **Image change** | Is the new `httpd` or `alpine` image running correctly? |
| **Environment variable** | Is `APP_ENV=test` visible via logs or shell (`echo $APP_ENV`)? |
| **Pod behavior** | Any errors in pulling the image or running the container? |
| **Logs** | Do logs differ based on image selection? |

---

### Why This Matters for .NET Core Developers

- In a .NET Core Kubernetes deployment, you will frequently modify configuration values, environment variables, and container images during development and testing.
- YAML updates help simulate scenarios where `.NET Core` services run with different environments like `Development`, `Test`, or `Production`.

---

### Estimated Time
🕒 **45 minutes**

---

### Summary
In this task, you deleted an existing Pod and recreated it using a new configuration with a different image and environment variable. This exercise reinforces how iterative configuration changes are applied in Kubernetes, a critical workflow when deploying and updating .NET Core applications.
