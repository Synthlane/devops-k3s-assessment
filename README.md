# devops-k3s-assessment
1. I have created a user and give neccessary permissions
```
a. adduser atul
b. usermod -aG sudo atul
```
2. Steps to install docker.
```
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

usermod -aG docker atul
```
3.  Verify the docker
<img width="950" height="42" alt="Screenshot 2026-01-06 111417" src="https://github.com/user-attachments/assets/9ac2a3a2-23e0-4810-843c-08fbbab7f59f" />

<img width="951" height="561" alt="Screenshot 2026-01-06 111455" src="https://github.com/user-attachments/assets/d812af77-a69f-4866-b7a1-c509f632c41b" />

4. Steps to install k3s
```
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644
sudo systemctl status k3s
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```
<img width="947" height="85" alt="image" src="https://github.com/user-attachments/assets/8082df5b-037a-4764-8b8e-9efbd9bd1aa5" />
<img width="942" height="59" alt="image" src="https://github.com/user-attachments/assets/79eca4d3-7614-4bb2-ba1e-5e56e96c0333" />

5.  Application Deployment (Helm)
```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm repo add open-webui https://helm.openwebui.com/
helm repo update
 helm install webui open-webui/open-webui \
  --namespace openwebui \
  --set service.type=ClusterIP \
  --dry-run --debug
kubectl get all -n openwebui
```
<img width="942" height="500" alt="image" src="https://github.com/user-attachments/assets/a68b69e9-2dc7-4268-9dde-9f70a08be9f2" />

6. Authentication Configuration (OIDC)
```
nano values-oidc.yaml
helm upgrade webui open-webui/open-webui --namespace openwebui --values values-oidc.yaml
```
<img width="941" height="200" alt="image" src="https://github.com/user-attachments/assets/9a92b083-e20e-4917-97d0-53e09979094d" />

7.  Debugging (Intentional Failure)
```
1. I have applied the values-oidc.yaml I was unable to get any errors, however I can see that we require a domain name against which we can gnereate the certificate for SSL.
2. May be I'm missing something, please comment any.
```


