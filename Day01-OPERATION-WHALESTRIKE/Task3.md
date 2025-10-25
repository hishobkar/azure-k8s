# .NET Core Task: Install Minikube in WSL

## Task 3: Install Minikube (25 minutes)

### Objective
Install **Minikube** in Windows Subsystem for Linux (WSL) using VS Code and verify the installation.

### Steps

1. **Open WSL Terminal**
   - Launch your preferred WSL distribution (Ubuntu recommended) in **VS Code** or terminal.

2. **Install Minikube**
   - Download the latest Minikube binary:
     ```bash
     curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
     ```
   - Make the binary executable:
     ```bash
     chmod +x minikube-linux-amd64
     ```
   - Move it to a directory in your PATH:
     ```bash
     sudo mv minikube-linux-amd64 /usr/local/bin/minikube
     ```

3. **Verify Installation**
   - Check Minikube version:
     ```bash
     minikube version
     ```
   - You should see the Minikube version output, confirming the installation is successful.

    - Output:
        ```
        minikube version: v1.37.0
        commit: 457800f4cfff9c12cc87ec9eb8f4cdd57b25047f3
        ```

### Estimated Time
25 minutes

### Notes
- Ensure Docker and kubectl are already installed in WSL before running Minikube.
- Minikube requires virtualization support (Hyper-V, VirtualBox, or WSL2 backend).
- You can start Minikube with Docker driver using:
  ```bash
  minikube start --driver=docker
