# Task 2: Dockerize the .NET Application (45 minutes)

## Prerequisites
- Completed Task 1 (MinikubeDemoAPI project created and tested)
- Currently in the `MinikubeDemoAPI` project root directory
- Minikube running (`minikube status` should show "Running")

## Step 1: Create Dockerfile

Create a new file named `Dockerfile` (no extension) in the project root directory:
```bash
touch Dockerfile
```

Open the file with your preferred editor (nano, vim, or VS Code):
```bash
nano Dockerfile
# OR
code Dockerfile
```

Add the following content:
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app

FROM base AS final
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "MinikubeDemoAPI.dll"]
```

Save and close the file (`Ctrl+X`, then `Y`, then `Enter` if using nano)

## Step 2: Understanding the Dockerfile

**Multi-stage Build Explanation:**

- **Stage 1 (base):** Uses lightweight runtime image, sets working directory, exposes port 80
- **Stage 2 (build):** Uses full SDK image, copies source code, compiles and publishes the application
- **Stage 3 (final):** Uses base runtime image, copies compiled output from build stage, sets entry point

**Benefits:**
- Smaller final image size (no SDK bloat)
- Faster deployment
- Better security (only runtime dependencies included)

## Step 3: Configure Minikube Docker Environment

Point your Docker CLI to use Minikube's Docker daemon (this ensures images are built inside Minikube):
```bash
eval $(minikube docker-env)
```

**Important Notes:**
- This command sets environment variables for the current terminal session only
- You'll need to run this in each new terminal session
- Images built with this configuration exist only in Minikube's Docker daemon

**Verify Docker is pointing to Minikube:**
```bash
docker context ls
# OR check Docker info
docker info | grep -i "Name:"
```

## Step 4: Build the Docker Image

Build the image with a tag:
```bash
docker build -t minikube-demo-api:latest .
```

**Expected Output:**
```
[+] Building 45.2s (14/14) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 284B
 => [internal] load .dockerignore
 => [internal] load metadata for mcr.microsoft.com/dotnet/sdk:8.0
 => [internal] load metadata for mcr.microsoft.com/dotnet/aspnet:8.0
 ...
 => => naming to docker.io/library/minikube-demo-api:latest
```

**Build Time:** Usually 2-5 minutes on first build (downloads base images)

## Step 5: Verify Image Creation

List Docker images to confirm:
```bash
docker images
```

**Expected Output:**
```
REPOSITORY                            TAG       IMAGE ID       CREATED          SIZE
minikube-demo-api                     latest    abc123def456   30 seconds ago   220MB
mcr.microsoft.com/dotnet/aspnet       8.0       ...            ...              ...
mcr.microsoft.com/dotnet/sdk          8.0       ...            ...              ...
```

Look for `minikube-demo-api` with tag `latest`

## Step 6: Test Docker Container Locally (Optional but Recommended)

Run the container to verify it works:
```bash
docker run -d -p 8080:80 --name test-api minikube-demo-api:latest
```

Test the API:
```bash
curl http://localhost:8080/WeatherForecast
# OR
curl http://localhost:8080/hello
```

**Expected Output:** JSON response with weather data or custom message

Stop and remove the test container:
```bash
docker stop test-api
docker rm test-api
```

## Step 7: Add .dockerignore File (Best Practice)

Create `.dockerignore` file in project root:
```bash
nano .dockerignore
```

Add these entries to exclude unnecessary files:
```
bin/
obj/
*.user
*.suo
.vs/
.vscode/
*.log
Dockerfile
.dockerignore
README.md
.git/
.gitignore
```

This reduces build context size and speeds up builds.

## Troubleshooting

### Issue: `eval $(minikube docker-env)` doesn't work
**Solution:** 
```bash
# Check if Minikube is running
minikube status

# Start Minikube if stopped
minikube start

# Retry the eval command
eval $(minikube docker-env)
```

### Issue: Build fails with "No such file or directory"
**Solution:** 
- Ensure you're in the `MinikubeDemoAPI` directory (where .csproj file exists)
- Check Dockerfile is in the same directory: `ls -la Dockerfile`

### Issue: Build fails with SDK/runtime version mismatch
**Solution:** 
- Check your .NET version: `dotnet --version`
- Update Dockerfile to match your SDK version (e.g., use `7.0` or `9.0` instead of `8.0`)
- Ensure both FROM statements use compatible versions

### Issue: "docker build" shows old cached image
**Solution:** 
```bash
# Build without cache
docker build --no-cache -t minikube-demo-api:latest .
```

### Issue: Image not showing in `docker images`
**Solution:** 
```bash
# Verify you're using Minikube's Docker
echo $DOCKER_HOST

# Re-run eval command
eval $(minikube docker-env)

# List images again
docker images | grep minikube-demo-api
```

### Issue: WSL memory issues during build
**Solution:** 
```bash
# Check available memory
free -h

# Restart Docker if needed
docker system prune -a
```

## Alternative: Build for Different .NET Versions

**For .NET 7:**
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
# ... rest same, change SDK to 7.0
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
```

**For .NET 9:**
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS base
# ... rest same, change SDK to 9.0
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
```

## Reset Docker Environment (When Done with Minikube)

To point Docker back to your local Docker daemon:
```bash
eval $(minikube docker-env -u)
```

## Verification Checklist

- [ ] Dockerfile created in project root
- [ ] Minikube Docker environment configured (`eval` command executed)
- [ ] Docker image built successfully
- [ ] Image appears in `docker images` output
- [ ] Image name is `minikube-demo-api:latest`
- [ ] (Optional) Container tested locally and responds correctly
- [ ] .dockerignore file created

## Key Takeaways

1. **Multi-stage builds** keep images small and secure
2. **Minikube Docker daemon** allows building images directly in the cluster
3. **Image versioning** (using tags like `latest`) is important for deployments
4. Images built with `eval $(minikube docker-env)` only exist in Minikube

## Next Steps

- Create Kubernetes deployment manifests
- Deploy the containerized application to Minikube
- Expose the service and test from outside the cluster