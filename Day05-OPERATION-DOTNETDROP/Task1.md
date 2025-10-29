# Minikube .NET Core Developer Task

## Prerequisites Checklist
- ✓ Docker installed in WSL
- ✓ kubectl installed in WSL
- ✓ Minikube installed in WSL
- [ ] .NET SDK installed

## Task 1: Create a Simple .NET Web API (30 minutes)

### Step 1: Verify .NET SDK Installation
```bash
dotnet --version
```
**Expected Output:** Version number (e.g., 8.0.xxx or 9.0.xxx)

If not installed, download from: https://dotnet.microsoft.com/download

### Step 2: Create New Web API Project
```bash
dotnet new webapi -n MinikubeDemoAPI
```

### Step 3: Navigate to Project Folder
```bash
cd MinikubeDemoAPI
```

### Step 4: Run Application Locally
```bash
dotnet run
```
**Expected Output:** 
- Application starts on http://localhost:5xxx
- Swagger UI available at http://localhost:5xxx/swagger

**Test the API:**
- Press `Ctrl+C` to stop after verification
- Or open browser to http://localhost:5xxx/swagger to explore endpoints

### Step 5 (Optional): Customize WeatherForecastController

Open `Controllers/WeatherForecastController.cs` and modify the Get method:
```csharp
[HttpGet(Name = "GetWeatherForecast")]
public IEnumerable<WeatherForecast> Get()
{
    return Enumerable.Range(1, 5).Select(index => new WeatherForecast
    {
        Date = DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
        TemperatureC = Random.Shared.Next(-20, 55),
        Summary = $"Custom Weather Data - Day {index}"
    })
    .ToArray();
}
```

Or add a simple endpoint:
```csharp
[HttpGet("hello")]
public IActionResult GetHello()
{
    return Ok(new { message = "Hello from Minikube Demo API!", timestamp = DateTime.UtcNow });
}
```

### Step 6: Test Custom Changes
```bash
dotnet run
```

Navigate to:
- http://localhost:5xxx/WeatherForecast
- http://localhost:5xxx/hello (if you added the custom endpoint)

## Troubleshooting

### Issue: `dotnet: command not found`
**Solution:** Install .NET SDK:
```bash
wget https://dot.net/v1/dotnet-install.sh -O dotnet-install.sh
chmod +x dotnet-install.sh
./dotnet-install.sh --channel 8.0
export PATH=$PATH:$HOME/.dotnet
```

### Issue: Port already in use
**Solution:** 
- Stop the running process or specify a different port
- Edit `Properties/launchSettings.json` to change port numbers

### Issue: Browser can't access localhost
**Solution:** 
- Use WSL IP address instead: `ip addr show eth0`
- Or configure port forwarding from Windows to WSL

## Next Steps (Preparation for Containerization)
1. Note the working endpoints
2. Prepare to create a Dockerfile
3. Build Docker image
4. Deploy to Minikube cluster

## Verification Checklist
- [ ] .NET SDK version confirmed
- [ ] Project created successfully
- [ ] Application runs locally without errors
- [ ] Can access Swagger UI
- [ ] WeatherForecast endpoint returns data
- [ ] (Optional) Custom endpoint working