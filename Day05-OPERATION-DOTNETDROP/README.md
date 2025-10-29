Code Name: OPERATION DOTNETDROP
Objective: Build, containerize, and deploy a simple .NET Web API to Minikube within 3 hours.

---------------------
Task 1: Create a Simple .NET Web API (30 minutes)
- Ensure .NET SDK is installed (dotnet --version).
- Create a new API project:
  dotnet new webapi -n MinikubeDemoAPI
- Navigate into the folder:
  cd MinikubeDemoAPI
- Run locally to confirm:
  dotnet run
- Optional: Modify WeatherForecastController to return a custom message.

---------------------
Task 2: Dockerize the .NET Application (45 minutes)
- Add a Dockerfile in the project root:

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

- Build the image (using Minikube Docker daemon):
  eval $(minikube docker-env)
  docker build -t minikube-demo-api:latest .

- Confirm image exists:
  docker images

---------------------
Task 3: Create Deployment and Service YAML Files (45 minutes)
1) deployment.yaml:

  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: dotnet-deploy
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: dotnetapi
    template:
      metadata:
        labels:
          app: dotnetapi
      spec:
        containers:
          - name: dotnet-container
            image: minikube-demo-api:latest
            ports:
              - containerPort: 80

2) service.yaml:

  apiVersion: v1
  kind: Service
  metadata:
    name: dotnet-service
  spec:
    type: NodePort
    selector:
      app: dotnetapi
    ports:
      - port: 80
        targetPort: 80
        nodePort: 30081

- Apply:
  kubectl apply -f deployment.yaml
  kubectl apply -f service.yaml

---------------------
Task 4: Access the API via Minikube (30 minutes)
- Get service details:
  kubectl get svc
- Open in browser:
  minikube service dotnet-service --url
- Test endpoint, e.g. /weatherforecast

---------------------
Task 5: Scale, Update & Cleanup (30 minutes)
- Scale to 2 replicas:
  kubectl scale deployment dotnet-deploy --replicas=2
- Verify:
  kubectl get pods
- Cleanup when done:
  kubectl delete svc dotnet-service
  kubectl delete deployment dotnet-deploy

---------------------
Mission Complete:
You have successfully built, containerized, and deployed a .NET Web API to Minikube and tested scaling.
End of OPERATION DOTNETDROP.
