# Task 4: Access the API via Minikube (30 minutes)

## Prerequisites
- Completed Task 3 (Deployment and Service applied successfully)
- Minikube running with your application deployed
- kubectl configured and working
- Pods are in "Running" status

## Step 1: Verify Current Deployment Status

Check if pods are running:
```bash
kubectl get pods
```

**Expected Output:**
```
NAME                             READY   STATUS    RESTARTS   AGE
dotnet-deploy-xxxxxxxxxx-xxxxx   1/1     Running   0          5m
```

**Status Meanings:**
- **Running:** Pod is healthy and ready
- **Pending:** Pod is being scheduled
- **CrashLoopBackOff:** Pod keeps failing (check logs)
- **ImagePullBackOff:** Can't pull image (use imagePullPolicy: Never)

If pod is not running, troubleshoot before proceeding:
```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

## Step 2: Get Service Details

View all services in the cluster:
```bash
kubectl get svc
```

**Expected Output:**
```
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
dotnet-service   NodePort    10.96.123.45     <none>        80:30081/TCP   3m
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP        5d
```

**Understanding the Output:**
- **NAME:** Service name (`dotnet-service`)
- **TYPE:** NodePort (accessible externally)
- **CLUSTER-IP:** Internal cluster IP (10.96.x.x)
- **EXTERNAL-IP:** `<none>` for Minikube NodePort services
- **PORT(S):** Internal:External mapping (80:30081)
- **AGE:** Time since service creation

Get detailed service information:
```bash
kubectl get svc dotnet-service
```

For more details including endpoints:
```bash
kubectl describe svc dotnet-service
```

**Expected Output:**
```
Name:                     dotnet-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=dotnetapi
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.123.45
IPs:                      10.96.123.45
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30081/TCP
Endpoints:                172.17.0.3:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

**Key Fields:**
- **Selector:** Shows which pods this service targets
- **Endpoints:** Actual pod IPs that receive traffic
- **NodePort:** External port number (30081)

## Step 3: Get Minikube IP Address

Find the IP address of your Minikube node:
```bash
minikube ip
```

**Example Output:**
```
192.168.49.2
```

**Note:** This IP address is specific to your WSL/Minikube installation and may differ from the example.

Store it in a variable for convenience (optional):
```bash
MINIKUBE_IP=$(minikube ip)
echo $MINIKUBE_IP
```

## Step 4: Get Service URL Using Minikube Command

Minikube provides a convenient command to get the service URL:
```bash
minikube service dotnet-service --url
```

**Expected Output:**
```
http://192.168.49.2:30081
```

This command shows the complete URL to access your service.

To see all services with their URLs:
```bash
minikube service list
```

**Expected Output:**
```
|----------------------|------------------|--------------|-----------------------------|
|      NAMESPACE       |       NAME       | TARGET PORT  |             URL             |
|----------------------|------------------|--------------|-----------------------------|
| default              | dotnet-service   |           80 | http://192.168.49.2:30081   |
| default              | kubernetes       | No node port |                             |
| kube-system          | kube-dns         | No node port |                             |
|----------------------|------------------|--------------|-----------------------------|
```

## Step 5: Test API Using curl (Command Line)

Test the WeatherForecast endpoint:
```bash
# Method 1: Using Minikube IP and NodePort
curl http://$(minikube ip):30081/WeatherForecast

# Method 2: Using the full URL from minikube service command
curl $(minikube service dotnet-service --url)/WeatherForecast

# Method 3: Direct URL (replace with your actual IP)
curl http://192.168.49.2:30081/WeatherForecast
```

**Expected Output (Example):**
```json
[
  {
    "date": "2025-10-31",
    "temperatureC": 32,
    "temperatureF": 89,
    "summary": "Warm"
  },
  {
    "date": "2025-11-01",
    "temperatureC": -5,
    "temperatureF": 24,
    "summary": "Chilly"
  }
]
```

Test with formatted output (pretty-print JSON):
```bash
curl -s http://$(minikube ip):30081/WeatherForecast | jq .
```

**Note:** If `jq` is not installed:
```bash
sudo apt-get update
sudo apt-get install jq -y
```

## Step 6: Test Custom Endpoint (If Created in Task 1)

If you added a custom `/hello` endpoint:
```bash
curl http://$(minikube ip):30081/hello
```

**Expected Output:**
```json
{
  "message": "Hello from Minikube Demo API!",
  "timestamp": "2025-10-30T10:30:45.123Z"
}
```

## Step 7: Access Swagger UI (If Available)

Check if Swagger is accessible:
```bash
# Get the base URL
minikube service dotnet-service --url
```

Open in browser (from Windows, not WSL terminal):
```
http://192.168.49.2:30081/swagger
```

**Note:** .NET 6+ Web API templates include Swagger by default in development mode.

If Swagger doesn't load, check if it's enabled in `Program.cs`:
```csharp
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
```

For production, you may need to enable it explicitly.

## Step 8: Open Service in Browser (Alternative Method)

Minikube can automatically open the service in your default browser:
```bash
minikube service dotnet-service
```

**Expected Output:**
```
|-----------|----------------|-------------|---------------------------|
| NAMESPACE |      NAME      | TARGET PORT |            URL            |
|-----------|----------------|-------------|---------------------------|
| default   | dotnet-service |          80 | http://192.168.49.2:30081 |
|-----------|----------------|-------------|---------------------------|
🎉  Opening service default/dotnet-service in default browser...
```

**Important for WSL Users:**
- This command may not automatically open a browser in WSL
- Copy the URL and paste it into your Windows browser manually
- Or use the `--url` flag to just get the URL without opening

## Step 9: Test API with Different HTTP Methods

**GET Request (retrieve data):**
```bash
curl -X GET http://$(minikube ip):30081/WeatherForecast
```

**GET with headers:**
```bash
curl -v http://$(minikube ip):30081/WeatherForecast
```

**Test specific endpoint with parameters (if you added custom endpoints):**
```bash
# Example: If you have /weatherforecast/{id}
curl http://$(minikube ip):30081/WeatherForecast/1
```

## Step 10: Monitor API Requests in Real-Time

View pod logs while making requests:
```bash
# In one terminal, watch logs
kubectl logs -f <pod-name>

# In another terminal, make requests
curl http://$(minikube ip):30081/WeatherForecast
```

Watch logs from all pods with the app label:
```bash
kubectl logs -f -l app=dotnetapi
```

## Step 11: Test API Performance

Send multiple requests to test:
```bash
# Send 10 requests
for i in {1..10}; do
  curl -s http://$(minikube ip):30081/WeatherForecast | jq -r '.[0].date'
done
```

Measure response time:
```bash
time curl -s http://$(minikube ip):30081/WeatherForecast > /dev/null
```

## Step 12: Access from Windows Browser

Since you're using WSL, access the API from Windows browser:

1. Get the URL:
```bash
minikube service dotnet-service --url
```

2. Copy the URL (e.g., `http://192.168.49.2:30081`)

3. Open in Windows browser:
   - Chrome, Edge, or Firefox
   - Navigate to: `http://192.168.49.2:30081/WeatherForecast`
   - Navigate to: `http://192.168.49.2:30081/swagger` (for Swagger UI)

## Step 13: Test API with Postman (Optional)

If you have Postman installed on Windows:

1. **GET Request:**
   - Method: GET
   - URL: `http://192.168.49.2:30081/WeatherForecast`
   - Send request

2. **View Response:**
   - Check Status: 200 OK
   - View JSON response
   - Examine headers

3. **Save to Collection:**
   - Save requests for future testing

## Step 14: Verify Service Endpoints

Check which pod IPs the service is routing to:
```bash
kubectl get endpoints dotnet-service
```

**Expected Output:**
```
NAME             ENDPOINTS         AGE
dotnet-service   172.17.0.3:80     10m
```

If you have multiple replicas:
```bash
kubectl scale deployment dotnet-deploy --replicas=3
kubectl get endpoints dotnet-service
```

**Expected Output:**
```
NAME             ENDPOINTS                              AGE
dotnet-service   172.17.0.3:80,172.17.0.4:80,172.17.0.5:80   12m
```

## Step 15: Test Service Load Balancing (If Multiple Replicas)

Scale to multiple replicas:
```bash
kubectl scale deployment dotnet-deploy --replicas=3
kubectl get pods -o wide
```

Make multiple requests and check which pod handles them:
```bash
# Watch logs from all pods
kubectl logs -f -l app=dotnetapi --all-containers=true --prefix=true

# In another terminal, send requests
for i in {1..20}; do
  curl -s http://$(minikube ip):30081/WeatherForecast > /dev/null
  echo "Request $i sent"
  sleep 1
done
```

You should see requests distributed across different pods.

## Troubleshooting

### Issue: "Connection refused" or "No route to host"

**Solution:**
```bash
# Check if Minikube is running
minikube status

# Verify service exists and has endpoints
kubectl get svc dotnet-service
kubectl get endpoints dotnet-service

# Check pod status
kubectl get pods

# Verify pod logs
kubectl logs <pod-name>
```

### Issue: API returns 404 Not Found

**Solution:**
```bash
# Verify the endpoint path
curl -v http://$(minikube ip):30081/WeatherForecast

# Check available routes (if Swagger is enabled)
curl http://$(minikube ip):30081/swagger/v1/swagger.json | jq .

# Verify pod logs for routing issues
kubectl logs <pod-name>
```

### Issue: Browser can't access from Windows

**Solution:**
```bash
# Check WSL and Windows networking
# From WSL, get Minikube IP
minikube ip

# From Windows PowerShell, test connectivity
Test-NetConnection -ComputerName <minikube-ip> -Port 30081

# Alternative: Use port forwarding
kubectl port-forward service/dotnet-service 8080:80
# Then access from Windows: http://localhost:8080
```

### Issue: Minikube IP not accessible

**Solution:**
```bash
# Restart Minikube
minikube stop
minikube start

# Reapply manifests
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Use alternative access method
kubectl port-forward service/dotnet-service 8080:80
```

### Issue: "minikube service" command hangs

**Solution:**
```bash
# Use --url flag instead
minikube service dotnet-service --url

# Or manually construct URL
echo "http://$(minikube ip):30081"
```

### Issue: Slow response times

**Solution:**
```bash
# Check resource usage
kubectl top pods
kubectl top nodes

# Check if pods are being throttled
kubectl describe pod <pod-name> | grep -i cpu
kubectl describe pod <pod-name> | grep -i memory

# Scale up resources or replicas
kubectl scale deployment dotnet-deploy --replicas=2
```

## Alternative Access Methods

### Method 1: Port Forwarding (Recommended for Development)

Forward local port to service:
```bash
kubectl port-forward service/dotnet-service 8080:80
```

Access from Windows browser or WSL:
```
http://localhost:8080/WeatherForecast
```

**Advantages:**
- Works reliably in WSL
- No need to remember Minikube IP
- Easier for local development

### Method 2: Kubectl Proxy

Start proxy server:
```bash
kubectl proxy --port=8001
```

Access via proxy:
```bash
curl http://localhost:8001/api/v1/namespaces/default/services/dotnet-service:80/proxy/WeatherForecast
```

### Method 3: Create LoadBalancer Service (Requires Minikube Tunnel)

Edit service.yaml to change type to LoadBalancer:
```yaml
spec:
  type: LoadBalancer
```

Apply and run tunnel:
```bash
kubectl apply -f service.yaml
minikube tunnel
```

**Note:** Requires running `minikube tunnel` in a separate terminal with sudo privileges.

## Testing Different Scenarios

### Test with Invalid Endpoint
```bash
curl http://$(minikube ip):30081/nonexistent
```

**Expected:** 404 Not Found

### Test with Different HTTP Headers
```bash
curl -H "Accept: application/json" \
     -H "User-Agent: TestClient/1.0" \
     http://$(minikube ip):30081/WeatherForecast
```

### Test Response Time
```bash
curl -w "\nTime: %{time_total}s\n" \
     -o /dev/null -s \
     http://$(minikube ip):30081/WeatherForecast
```

### Continuous Testing Script

Create a simple test script:
```bash
#!/bin/bash
URL=$(minikube service dotnet-service --url)
ENDPOINT="/WeatherForecast"

echo "Testing API at: $URL$ENDPOINT"
echo "Press Ctrl+C to stop"

while true; do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$URL$ENDPOINT")
  if [ $STATUS -eq 200 ]; then
    echo "✓ $(date +%T) - Status: $STATUS - OK"
  else
    echo "✗ $(date +%T) - Status: $STATUS - ERROR"
  fi
  sleep 5
done
```

Save as `test-api.sh`, make executable, and run:
```bash
chmod +x test-api.sh
./test-api.sh
```

## Verification Checklist

- [ ] Service details retrieved successfully (`kubectl get svc`)
- [ ] Minikube IP obtained (`minikube ip`)
- [ ] Service URL retrieved (`minikube service dotnet-service --url`)
- [ ] WeatherForecast endpoint returns JSON data via curl
- [ ] Custom endpoints accessible (if created)
- [ ] Swagger UI loads in browser (if enabled)
- [ ] API accessible from Windows browser
- [ ] Pod logs show incoming requests
- [ ] Service endpoints show correct pod IPs
- [ ] Multiple replicas distribute load (if scaled)

## Useful Commands Summary
```bash
# Get service URL
minikube service dotnet-service --url

# Test endpoint
curl http://$(minikube ip):30081/WeatherForecast

# Watch logs
kubectl logs -f -l app=dotnetapi

# Port forward (alternative access)
kubectl port-forward service/dotnet-service 8080:80

# Check service endpoints
kubectl get endpoints dotnet-service

# Scale replicas
kubectl scale deployment dotnet-deploy --replicas=3

# Get all service info
kubectl describe svc dotnet-service
```

## Key Takeaways

1. **NodePort services** expose applications on a specific port (30000-32767)
2. **Minikube IP** is required to access services from outside the cluster
3. **minikube service** command simplifies getting service URLs
4. **Port forwarding** is often more convenient for local development in WSL
5. **Service endpoints** show which pods receive traffic
6. **Load balancing** happens automatically with multiple pod replicas

## Next Steps

- Monitor application performance with kubectl top
- Implement ingress for better routing
- Add ConfigMaps for environment-specific configuration
- Set up persistent storage with PersistentVolumes
- Deploy to cloud Kubernetes (AKS, EKS, GKE)
- Implement CI/CD pipeline for automated deployments