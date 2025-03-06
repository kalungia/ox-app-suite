# Ox App Suite v8 Installation Guide

This repository contains a step-by-step guide for installing Ox App Suite v8. The installation uses K3s for a lightweight Kubernetes distribution, Helm for package management, and additional tools like yq and Python to complete the setup.

## Prerequisites

- **Tested On:** Ubuntu 22.04 LTS

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
helm install mariadb bitnami/mariadb --set auth.rootPassword="YourPassword"
helm install redis bitnami/redis --set global.redis.password="YourPassword"
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

Before running the installation script, you can change the default hostname `as8.lab.test` to your own domain name in the `values.yml or lab.yml` file.

Navigate to the rendered lab directory and execute the installation script:

```bash
cd rendered/lab
./install.sh
```
## Accessing the Web Application

The default configuration of the lab produces a service for web UI access of type NodePort. This was chosen for its universal availability. For other service type options, see the advanced documentation, section "Accessing the Web Application".

```bash
kubectl get service -n as8 istio-ingressgateway
```

Example output:

```bash
NAME                   TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)                                      AGE
istio-ingressgateway   NodePort   10.233.13.12   <none>        15021:30021/TCP,80:30080/TCP,443:30443/TCP   18m
```

To connect to App Suite via this service, two more things need to be done:

1. The default lab configuration configures a hostname of `as8.lab.test` (via the `as_hostname` key in `lab.yml`). This is not a valid publicly resolvable DNS name, so you need to add a local `/etc/hosts` entry for that purpose. (Windows: the hosts file location is something like `C:\Windows\system32\drivers\etc\hosts`.)

2. The example assumes a `10.50.2.89` IP for your k8s node (or, one of your k8s nodes):

```bash
10.50.2.89 as8.lab.test
```

## Post-Installation

After running the installation steps, verify that all components are up and running by checking your Kubernetes pods and services. You can use:

```bash
kubectl get pods --namespace as8
```

```bash
kubectl expose deployment as8 --type=LoadBalancer --name=as8
kubectl expose deployment keycloak --type=LoadBalancer --name=keycloak
```

## Troubleshooting

- **Permissions:** Ensure you run commands with the necessary privileges (e.g., using `sudo` where required).
- **Network Issues:** Verify your internet connection if any downloads fail.
- **Component-Specific Issues:** Consult the documentation for K3s, Helm, or other tools if you encounter errors.
