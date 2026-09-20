# Day 01 Key Takeaways: Setting Up a Kubernetes Cluster

## 1. Choose How to Run Kubernetes

| Tool / Service | What It Does | Best For |
|---|---|---|
| `kubeadm` | Bootstraps a real Kubernetes cluster on servers you manage | Self-managed and production-style clusters |
| Rancher | Web UI to deploy and manage many clusters | Managing multiple clusters |
| Minikube | Runs a single-node cluster on your laptop | Learning and testing |
| `kind` | Runs each Kubernetes node as a Docker container | Fast local clusters, CI pipelines |
| Amazon EKS | Managed Kubernetes on AWS | Production on AWS |
| Azure AKS | Managed Kubernetes on Azure | Production on Azure |
| Google GKE | Managed Kubernetes on Google Cloud | Production on Google Cloud |

With managed services (EKS, AKS, GKE) the cloud provider runs the control plane for you. With `kubeadm`, Minikube and `kind`, you run it yourself.

## 2. Check the Kubernetes Version

```bash
kubectl version            # Show client (kubectl) and server (cluster) versions
kubectl version --client   # Show only the kubectl version; works without a cluster
```

For release notes and supported versions, see the official docs at [kubernetes.io](https://kubernetes.io).

> If no cluster exists yet, plain `kubectl version` prints the client version and then a connection error for the server. That is expected.

## 3. Prepare the Machine

Start from a fresh Ubuntu installation. Older Docker or Kubernetes installs can leave conflicting packages, configs and ports behind, so a clean machine avoids hard-to-debug problems.

## 4. Install and Configure Docker

Docker is the foundation. `kind`, and many other setups, need it running before anything else.

### Step 1: Refresh the Package Index

```bash
sudo apt-get update
```

Downloads the latest list of available packages and versions from Ubuntu's repositories. It installs nothing itself, but it makes sure the next step gets current versions.

### Step 2: Install Docker

```bash
sudo apt install docker.io
```

Installs Docker Engine from Ubuntu's repository. `sudo` runs it with administrator rights, and `docker.io` is Ubuntu's package name for Docker.

### Step 3: Confirm the Installation

```bash
docker --version
```

Prints the installed Docker version. If you see a version number, the install worked.

### Step 4: Allow Your User to Run Docker Without sudo

```bash
sudo usermod -aG docker $USER
```

**Breaking it down:**

| Part | Meaning |
|---|---|
| `usermod` | Modify a user account |
| `-a` | Append: add to the group without removing existing groups |
| `-G docker` | The group to add, here `docker` |
| `$USER` | Your current username, filled in automatically |

Without this step, every Docker command needs `sudo`.

### Step 5: Activate the New Group in This Terminal

```bash
newgrp docker
```

Group changes normally apply only after you log out and back in. `newgrp` starts a shell with the new group active, so you can continue immediately. Logging out and back in also works.

### Step 6: Test Docker

```bash
docker ps
```

Lists running containers. An empty table with headers (`CONTAINER ID`, `IMAGE`, and so on) means Docker is working and your permissions are correct. If you see "permission denied", repeat Steps 4 and 5.

## Quick Flow

> Choose a tool → Fresh Ubuntu → Install Docker → Add user to docker group → Verify → Install kind/kubectl → Create cluster

Docker is the prerequisite for everything else. Once `docker ps` works, you can continue with the `kind`, `kubectl` and Helm installation and cluster creation covered in the [full command reference](#full-command-reference) below.

> **Warning:** Group membership in `docker` is effectively root-level access on that machine, so only add users you trust.

## Full Command Reference

### 1. Verify Docker

`kind` runs Kubernetes nodes as Docker containers, so Docker must be working first.

```bash
docker --version        # Show the installed Docker version
docker version          # Show client and server details (confirms the daemon is running)
docker ps               # List running containers (an empty list is fine)
```

| Command | Purpose |
|---|---|
| `docker --version` | Show the installed Docker version |
| `docker version` | Show client and server details (confirms the daemon is running) |
| `docker ps` | List running containers (an empty list is fine) |

### 2. Install kind (Kubernetes IN Docker)

```bash
# Download the kind binary, only if the machine is x86_64
[ "$(uname -m)" = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.32.0/kind-linux-amd64

# Make it executable and move it onto the PATH
chmod +x ./kind && sudo mv ./kind /usr/local/bin/kind

# Confirm the installation
kind --version
```

### 3. Install kubectl (the Kubernetes CLI)

```bash
# Download the latest stable kubectl release
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make it executable and move it onto the PATH
chmod +x ./kubectl && sudo mv ./kubectl /usr/local/bin/kubectl

# Confirm the installation
kubectl version
```

### 4. Install Helm (the Kubernetes package manager)

```bash
# Download and run the official Helm 3 installer script
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### 5. Create the Working Directory

```bash
mkdir simple-k8s        # Folder to hold all your YAML files
cd simple-k8s           # Move into it
```

### 6. Create the Cluster

```bash
nano kind-config.yml                                  # Write the cluster definition (cluster name: devboard)
kind create cluster --config kind-config.yml          # Build the cluster from that file
kubectl cluster-info --context kind-devboard          # Show the API server address for this cluster
kubectl get nodes                                     # List nodes; they should show STATUS "Ready"
```

**Optional:** look inside the node, which is just a Docker container.

```bash
docker ps                                       # Find the node container's name or ID
docker exec -it devboard-control-plane bash     # Open a shell inside the node
```

### 7. Explore the Empty Cluster

```bash
kubectl get pods        # Pods in the "default" namespace (none yet)
kubectl get ns          # All namespaces (default, kube-system, etc.)
```

### 8. Deploy a Test nginx Pod

```bash
nano namespace.yml                       # Define a namespace (yours is named ngnix-ns)
kubectl apply -f namespace.yml           # Create the namespace
kubectl get namespaces                   # Confirm it exists

nano pod.yml                             # Define the nginx pod
kubectl apply -f pod.yml                 # Create the pod

kubectl get pods -n ngnix-ns             # Check pod status in that namespace
kubectl get pods -n ngnix-ns -o wide     # Same, plus pod IP and the node it runs on
```

### 9. Reach nginx From Your Machine

```bash
# Forward local port 8080 to port 80 of the nginx pod
kubectl port-forward pod/nginx -n ngnix-ns 8080:80

# Same, but reachable from other machines on your network too
kubectl port-forward pod/nginx -n ngnix-ns 8081:80 --address 0.0.0.0
```

Then open `http://localhost:8080`. Press `Ctrl+C` to stop forwarding.

### 10. Deploy the DevBoard Frontend

```bash
nano devboard-ns.yml                                  # Define the devboard-ns namespace
kubectl apply -f devboard-ns.yml                      # Create it

nano devboard-frontend-pod.yml                        # Define the frontend pod
kubectl apply -f devboard-frontend-pod.yml            # Create the pod

kubectl get pods -n devboard-ns                       # Verify it is Running

# Forward local port 8082 to container port 4173, exposed on all interfaces
kubectl port-forward pod/devboard-frontend -n devboard-ns 8082:4173 --address 0.0.0.0
```

Then open `http://localhost:8082`.

### Command Summary

| Step | Command | Purpose |
|---|---|---|
| 1 | `docker --version` | Show installed Docker version |
| 1 | `docker version` | Confirm Docker daemon is running |
| 1 | `docker ps` | List running containers |
| 2 | `curl -Lo ./kind ...` | Download the `kind` binary |
| 2 | `chmod +x ./kind && sudo mv ./kind /usr/local/bin/kind` | Install `kind` onto the PATH |
| 2 | `kind --version` | Confirm `kind` installation |
| 3 | `curl -LO ".../kubectl"` | Download the latest `kubectl` |
| 3 | `chmod +x ./kubectl && sudo mv ./kubectl /usr/local/bin/kubectl` | Install `kubectl` onto the PATH |
| 3 | `kubectl version` | Confirm `kubectl` installation |
| 4 | `curl .../get-helm-3 \| bash` | Install Helm 3 |
| 6 | `kind create cluster --config kind-config.yml` | Build the cluster from a config file |
| 6 | `kubectl cluster-info --context kind-devboard` | Show the API server address |
| 6 | `kubectl get nodes` | List cluster nodes and their status |
| 6 | `docker exec -it devboard-control-plane bash` | Shell into the node container |
| 7 | `kubectl get pods` | List pods in the `default` namespace |
| 7 | `kubectl get ns` | List all namespaces |
| 8 | `kubectl apply -f namespace.yml` | Create a namespace |
| 8 | `kubectl apply -f pod.yml` | Create a pod |
| 8 | `kubectl get pods -n ngnix-ns -o wide` | Check pod status, IP and node |
| 9 | `kubectl port-forward pod/nginx -n ngnix-ns 8080:80` | Forward a local port to the pod |
| 10 | `kubectl apply -f devboard-ns.yml` | Create the DevBoard namespace |
| 10 | `kubectl apply -f devboard-frontend-pod.yml` | Deploy the DevBoard frontend pod |
| 10 | `kubectl port-forward pod/devboard-frontend -n devboard-ns 8082:4173 --address 0.0.0.0` | Expose the frontend on all interfaces |
