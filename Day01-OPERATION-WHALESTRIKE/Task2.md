# .NET Core Task: Install kubectl in WSL

## Task 2: Install kubectl (20 minutes)

### Objective
Install **kubectl** in Windows Subsystem for Linux (WSL) to work with Docker CLI and verify the installation.

### Steps

1. **Open WSL Terminal**
   - Launch your preferred WSL distribution (Ubuntu recommended) in **VS Code** or terminal.

2. **Install kubectl**
   - Download the latest release:
     ```bash
     curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
     ```
   - Make the binary executable:
     ```bash
     chmod +x kubectl
     ```
   - Move the binary to your PATH:
     ```bash
     sudo mv kubectl /usr/local/bin/
     ```

3. **Verify Installation**
   - Check kubectl version:
     ```bash
     kubectl version --client
     ```
   - You should see the client version output, confirming kubectl is installed.

    - Output:
        ```
        Client Version: v1.34.1
        Kustomize Version: v5.7.1
        ```

### Estimated Time
20 minutes

### Notes
- Ensure that Docker is already installed in WSL for kubectl to interact with containers if needed.
- Keep kubectl updated regularly using the same download method.
