##devops-k3s-assessment
Synthlane DevOps Assessment: K3s & Open WebUI Deployment
##Overview

This project demonstrates the setup of a single-node Kubernetes cluster using k3s, installation of Docker, deployment of Open WebUI using Helm, and exposure/debugging of services.
The goal is to showcase end-to-end DevOps ownership, including cluster setup, application deployment, service access, and troubleshooting.

##Part 1: Docker Installation & Validation

Docker is required as a container runtime and for local container validation.

##Step 1: Verify existing Docker installation
docker --version
docker compose version
docker run hello-world
docker ps

##Step 2: Update system and install required dependencies
sudo apt update && sudo apt upgrade -y
sudo apt install -y ca-certificates curl gnupg lsb-release apt-transport-https

##Step 3: Add Docker GPG key and repository
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

##Step 4: Install Docker Engine and plugins
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

##Step 5: Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker

##Step 6: Allow non-root Docker usage (IMPORTANT)
sudo usermod -aG docker $USER
newgrp docker

##Part 2: K3s Single-Node Kubernetes Cluster Setup
Step 1: Disable swap (required for Kubernetes)
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

Step 2: Install k3s
curl -sfL https://get.k3s.io | sh -

Step 3: Verify k3s service
sudo systemctl status k3s

Step 4: Validate cluster access
sudo k3s kubectl get nodes
kubectl get nodes
kubectl get pods -A


This confirms:

Control plane is running

Core system pods are healthy

Networking & DNS are functional

##Part 3: Helm Installation & Configuration
Step 1: Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash


Verify:

helm version

Step 2: Add Open WebUI Helm repository
helm repo add open-webui https://helm.openwebui.com/
helm repo update

Part 4: Open WebUI Deployment
Step 1: Create namespace
kubectl create namespace openwebui
kubectl get ns openwebui

##Step 2: Dry-run Helm installation (MANDATORY)
helm install webui open-webui/open-webui \
  --namespace openwebui \
  --set service.type=ClusterIP \
  --dry-run \
  --debug


This validates:

Helm chart rendering

Kubernetes manifests

No runtime changes applied

##Step 3: Deploy Open WebUI
helm install webui open-webui/open-webui \
  --namespace openwebui \
  --set service.type=ClusterIP

##Step 4: Validate deployment
kubectl get all -n openwebui
kubectl get pods -n openwebui
kubectl get svc -n openwebui

##Part 5: Service Access & Networking
Option 1: Port-forward via Service
kubectl port-forward svc/webui 8080:80 -n openwebui

Option 2: Port-forward via Pod
export LOCAL_PORT=8080
export POD_NAME=$(kubectl get pods -n openwebui -l "app.kubernetes.io/component=open-webui" -o jsonpath="{.items[0].metadata.name}")
export CONTAINER_PORT=$(kubectl get pod -n openwebui $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")

kubectl -n openwebui port-forward $POD_NAME $LOCAL_PORT:$CONTAINER_PORT

#Option 3: SSH Tunnel for Remote Server Access
ssh -L 8080:localhost:8080 root@91.98.157.19

#Option 4: Expose via NodePort (Debugging)
kubectl patch svc open-webui -n openwebui -p '{"spec":{"type":"NodePort"}}'
kubectl get svc -n openwebui

##Part 6: Helm & Kubernetes Debugging
Check active kubeconfig
kubectl config view --minify

##Explicit kubeconfig usage (k3s)
helm install webui open-webui/open-webui \
  --namespace openwebui \
  --set service.type=ClusterIP \
  --kubeconfig /etc/rancher/k3s/k3s.yaml

##Troubleshooting Commands Used
kubectl describe pod -n openwebui <pod-name>
kubectl get pods -n openwebui
kubectl get svc -n openwebui
