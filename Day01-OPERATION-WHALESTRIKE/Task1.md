# .NET Core Task: Docker Installation in WSL

## Task 1: Install Docker and Docker CLI (45 minutes)

### Objective
Install Docker and Docker CLI in Windows Subsystem for Linux (WSL) and verify the installation.

### Steps

1. **Visit Docker Official Website**
   - Navigate to the [Docker Official Website](https://www.docker.com/) to download Docker Desktop for Windows.

2. **Install Docker and Docker CLI**
   - Follow the installation instructions for **Docker Desktop**.
   - Ensure that the option **“Use WSL 2 based engine”** is enabled during installation.
   - Complete the installation process.

3. **Launch Docker in WSL using VS Code**
   - Open **VS Code**.
   - Install the **Remote - WSL** extension if not already installed.
   - Open your WSL terminal in VS Code.
   - Start Docker from the WSL terminal:
     ```bash
     sudo service docker start
     ```
     or use:
     ```bash
     docker --help
     ```

4. **Verify Docker Installation**
   - Check Docker version:
     ```bash
     docker --version
     ```
   - Run a test container:
     ```bash
     docker run hello-world
     ```
   - You should see a message confirming that Docker is installed and working correctly.

### Estimated Time
45 minutes

### Notes
- Ensure that WSL 2 is installed and set as default.
- Make sure your user has permissions to run Docker commands without `sudo`.
