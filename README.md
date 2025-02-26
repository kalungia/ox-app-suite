# Ox App Suite v8 Installation Guide

This repository contains a step-by-step guide for installing Ox App Suite v8. The installation uses K3s for a lightweight Kubernetes distribution, Helm for package management, and additional tools like yq and Python to complete the setup.

## Prerequisites

- **Operating System:** Debian-based Linux (Ubuntu 22.04) with `sudo` privileges.

## Installation Steps

### 1. Update System & Install Dependencies

Update your system and install the necessary packages:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget gnupg2 software-properties-common apt-transport-https ca-certificates lsb-release
```

### 2. Install K3s

Install K3s, a lightweight Kubernetes distribution:

```bash
curl -sfL https://get.k3s.io | sh -
```

Then configure your shell to use the K3s configuration:

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
echo 'export KUBECONFIG=/etc/rancher/k3s/k3s.yaml' >> ~/.bashrc
source ~/.bashrc
```

### 3. Install Helm

Install Helm, which will be used to manage Kubernetes packages:

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### 4. Deploy MariaDB and Redis

Deploy MariaDB and Redis using Helm. (Remember to update the passwords for a production environment.)

```bash
helm install mariadb bitnami/mariadb --set auth.rootPassword="0717121315kA@123"
helm install redis bitnami/redis --set global.redis.password="0717121315kA@123"
```

> **Note:** The passwords used above are for demonstration purposes only. **Change these to secure values in your environment.**

### 5. Install yq

Download and install `yq` for YAML processing:

```bash
wget https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 -O /usr/local/bin/yq
chmod +x /usr/local/bin/yq
```

### 6. Clone the Operation Guides Repository

Clone the repository containing the operation guides:

```bash
cd ~
git clone https://gitlab.open-xchange.com/appsuite/operation-guides.git
cd operation-guides
```

### 7. Set Up Python Environment & Render Templates

Install the Python virtual environment package and create a virtual environment:

```bash
apt install python3.10-venv
python3 -mvenv v
v/bin/pip install --upgrade pip wheel
v/bin/pip install -r requirements.txt
```

Create a new Kubernetes namespace and render the installation templates:

```bash
kubectl create namespace as8
v/bin/python render.py
```

### 8. Run the Final Installation Script

Navigate to the rendered lab directory and execute the installation script:

```bash
cd rendered/lab
./install.sh
```

## Post-Installation

After running the installation steps, verify that all components are up and running by checking your Kubernetes pods and services. You can use:

```bash
kubectl get pods --namespace as8
```

## Troubleshooting

- **Permissions:** Ensure you run commands with the necessary privileges (e.g., using `sudo` where required).
- **Network Issues:** Verify your internet connection if any downloads fail.
- **Component-Specific Issues:** Consult the documentation for K3s, Helm, or other tools if you encounter errors.
