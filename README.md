# devops-k3s-assessment
QUES 1.Explain briefly why this setup is suitable (or not) for an early-stage startup.
ANS. Why it’s suitable
]•	Low cost & simplicity: A single-node k3s cluster has minimal resource overhead and runs well on a small VM, keeping infrastructure costs low.
•	Fast to iterate: Helm-based deployments and a lightweight Kubernetes distribution enable quick changes, rapid testing, and easy rollbacks.
•	Production-like environment: k3s is CNCF-conformant, so developers get Kubernetes semantics early without the complexity of a full multi-node cluster.

Limitations to note
•	Single point of failure: One node means no high availability; a VM outage takes everything down.
•	Limited scalability: NodePort access and single-node capacity won’t handle high traffic or heavy workloads.


QUES 2.) Part 5: Ownership Questions
 
Answer clearly and concretely:
1. Production Readiness
- Top 5 risks before going live
- First 2 things you would fix
- 1. Production Readiness

ANS. Top 5 Risks Before Going Live
	1.	Single Point of Failure
	•	The cluster is single-node; any VM, disk, or OS failure will cause complete downtime.
	2.	No High Availability or Auto-Scaling
	•	No redundancy for pods, control plane, or workloads.
	•	Traffic spikes could overwhelm the node.
	3.	Security Exposure
	•	NodePort access exposes services directly.
	•	No TLS, WAF, or strict NetworkPolicies in place.
	4.	Weak Persistence & Backup Strategy
	•	Local-path storage ties data to one node.
	•	No automated backups for application state or data.
	5.	Limited Observability & Alerting
	•	Lack of structured logs, metrics, and alerts makes failures harder to detect and diagnose.

First 2 Things I Would Fix
	1.	Secure & Stabilize Access
	•	Replace NodePort with Ingress (Traefik/Nginx) behind HTTPS (cert-manager + Let’s Encrypt).
	•	Add basic authentication and restrict network access.
	•	This immediately reduces attack surface and improves reliability.
	2.	Add Monitoring, Logging & Backups
	•	Install Prometheus + Grafana for metrics and alerts.
	•	Centralize logs (e.g., Loki or CloudWatch).
	•	Implement automated backups for persistent volumes and cluster stat

QUES )2. Failure Scenario
- Traffic spikes 10x and node goes down at 2 AM
- What breaks first?
- How do you recover?
- What do you change the next day?

ANS. Traffic spikes 10× and the single node goes down at 2 AM

What Breaks First?
	1.	The Node Itself
	•	CPU / memory exhaustion or disk I/O saturation causes kubelet or the OS to crash.
	•	Since this is a single-node k3s cluster, the control plane and workloads go down together.
	2.	All Application Services
	•	Open WebUI, Redis, Ollama, pipelines — everything becomes unavailable.
	•	No pod rescheduling is possible without another node.

How Do You Recover?

Immediate (Reactive) Response
	1.	Restore Node Availability
	•	Restart the VM via cloud console if unreachable.
	•	Verify OS, disk, and network health.
	2.	Bring k3s Back Online
	•	Ensure k3s service is running.
	•	Confirm node status: kubectl get nodes.

What Do You Change the Next Day?

1. Eliminate the Single Point of Failure (Highest Priority)
	•	Move to a multi-node cluster (minimum 2 workers + control plane).
	•	Or migrate to a managed Kubernetes service.
	•	Enable pod replication and basic auto-scaling.

2. Add Traffic Protection
	•	Introduce an Ingress controller with rate limiting.
	•	Add request timeouts and circuit breakers at the application layer.
	•	Prevent traffic spikes from crashing the node.

3. Improve Observability & Alerting
	•	Add uptime alerts, CPU/memory thresholds, and pod crash alerts.
	•	Ensure the on-call engineer is notified immediately at 2 AM — not at 9 AM.

QUES) 3. Security & Secrets
- How do you manage secrets?
- What must never be in Git?
- What should be rotated?

ANS )3. Security & Secrets

How Do You Manage Secrets?

Early stage (current setup)
	•	Use Kubernetes Secrets for runtime configuration.
	•	Secrets are mounted as environment variables or files, never hardcoded.
	•	Access is scoped via namespace isolation and RBAC.

As we mature
	•	Move secrets to a centralized secrets manager (e.g., Vault / cloud-native secret manager).
	•	Use short-lived credentials and dynamic secrets where possible.

What Must Never Be in Git?

Absolutely never commit:
	•	API keys (OpenAI, cloud providers, third-party services)
	•	Database usernames & passwords
	•	Private SSH keys or .pem files
	
Rule of thumb:
If losing it would require rotating credentials immediately, it does not belong in Git.

What Should Be Rotated?

High-priority (rotate regularly):
	•	API keys and tokens
	•	Database credentials
	•	Redis / internal service passwords
After any incident:
	•	SSH keys
	•	CI/CD secrets
	•	Service account tokens

QUES) 4. Backups & Recovery
- What data must be backed up?
- Backup frequency?
- How do you test recovery?

ANS) Backups & Recovery

What Data Must Be Backed Up?

Highest priority (cannot be recreated):
	•	Application data
	•	Open WebUI user data, chat history, settings (PVC-backed data)
	•	Datastores
	•	Redis data (if used beyond cache)

Lower priority (recreatable, but useful):
	•	Container images (if not stored in a registry)
	•	Node configuration (cloud-init, firewall rules)

Rule: If losing it causes user data loss or prolonged downtime, it must be backed up.

Backup Frequency?

Data backups
	•	Daily full backups of persistent volumes
	•	Incremental backups if supported (hourly or every few hours)

Configuration backups
	•	On every change (Git already handles this for manifests)
	•	Secrets: versioned in a secrets manager, not Git

How Do You Test Recovery?

You don’t trust backups until restore works.
	1.	Regular Restore Drills
	•	Restore data into a new namespace or test cluster
	•	Verify application starts and data is intact
	2.	Automated Validation
	•	Smoke tests after restore (app loads, user data visible)
	•	Check data integrity (counts, timestamps)
	3.	Disaster Simulation
	•	Simulate node loss
	•	Rebuild cluster from scratch and restore backups


QUES)5. Cost Ownership (Hetzner)
- How do you keep infra costs low?
- What do you avoid early?
- When would you move away from k3s?

ANS) How Do You Keep Infra Costs Low?
	1.	Start Small and Right-Size
	•	Use a single, modest Hetzner VM sized to current traffic.
	•	Monitor CPU, memory, and disk and resize only when metrics justify it.
	2.	Lightweight Stack Choices
	•	k3s over full Kubernetes to reduce overhead.
	•	One cluster, one environment early on (prod-like, not multiple copies).
	3.	On-Demand Scaling vs Always-On Capacity
	•	Scale vertically (resize VM) before adding nodes.
	•	Run only essential services; remove unused components.


What Do You Avoid Early?
	1.	Managed Kubernetes Platforms
	•	High fixed costs and operational overhead before product-market fit.
	2.	Multi-Region / Multi-Cluster Setups
	•	Doubles or triples costs with little early benefit.
	3.	Over-Provisioning
	•	Large VMs “just in case”
	•	Multiple replicas for low-traffic services

When Would You Move Away from k3s?

You move away from k3s only when the business demands it, not “because it’s prod”.

Clear signals to move:
	1.	Availability Becomes a Business Requirement
	•	Downtime has revenue or contractual impact.
	•	You need multi-node, HA control planes.
	2.	Traffic & Data Growth
	•	Sustained high traffic requiring horizontal scaling.
	•	Network-backed storage and stronger guarantees are needed.
	3.	Operational Load Increases
	•	On-call fatigue from manual recovery.
	•	Need managed upgrades, backups, and SLAs.

QUES) Extra Credit
OIDC provider uses a self-signed certificate.
Explain how you would make the application trust a custom CA in a maintainable way.?

ANS) Trusting a Self-Signed OIDC Provider Certificate (Maintainable Approach)

Goal
Make the application trust a custom Certificate Authority (CA) used by an OIDC provider without hacks, without disabling TLS, and without code changes per environment.

The Right Principle (First)
Never disable TLS verification.
If the certificate is self-signed, add trust, don’t weaken security.


COMMANDS
1.docker --version
2.docker compose version
3.docker run hello-world
4.docker ps
5.sudo apt update && sudo apt upgrade -y
6.sudo apt install -y curl ca-certificates apt-transport-7.https gnupg lsb-release
8.sudo swapoff -a
9.sudo sed -i '/ swap / s/^/#/' /etc/fstab
10.curl -sfL https://get.k3s.io | sh -
11.sudo systemctl status k3s
12.sudo k3s kubectl get nodes
13.kubectl get nodes
14.kubectl get pods -A
15.Curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
16.helm repo add open-webui https://helm.openwebui.com/
17.helm repo update
18.kubectl create namespace openwebui
19.helm install webui open-webui/open-webui   --namespace openwebui   --set service.type=ClusterIP   --dry-run   --debug
19.kubectl get all -n openwebui
    20. vi values-oidc.yaml
21. kubectl get pods -n openwebui
22. kubectl get svc -n openwebui
23.  kubectl port-forward svc/webui 8080:80 -n openwebui
24.  kubectl get ns openwebui
25.helm install webui open-webui/open-webui   --namespace openwebui   --set service.type=ClusterIP
26. kubectl get ns openwebui
27root@vaibhavpanwar202:~# helm install webui open-webui/open-webui   --namespace openwebui   --set service.type=ClusterIP
28.helm install webui open-webui/open-webui   --namespace openwebui   --set service.type=ClusterIP
29. kubectl config view --minify
30.helm install webui open-webui/open-webui   --namespace openwebui   --set service.type=ClusterIP   --kubeconfig /etc/rancher/k3s/k3s.yaml
31.kubectl get pods -n openwebui
32.  kubectl get svc -n openwebui
33.export LOCAL_PORT=8080
34. export POD_NAME=$(kubectl get pods -n openwebui -l "app.kubernetes.io/component=open-webui" -o jsonpath="{.items[0].metadata.name}")
35.export CONTAINER_PORT=$(kubectl get pod -n openwebui $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
36. kubectl -n openwebui port-forward $POD_NAME $LOCAL_PORT:$CONTAINER_PORT
37.kubectl describe pod -n openwebui $POD_NAME
38.kubectl get pods -n openwebui
39.kubectl -n openwebui port-forward pod/open-webui-0 8080:8080
40.ssh -L 8080:localhost:8080 root@91.98.157.19
41.kubectl patch svc webui -n openwebui -p '{"spec":{"type":"NodePort"}}'
42.kubectl patch svc open-webui -n openwebui   -p '{"spec":{"type":"NodePort"}}'
43.kubectl get svc -n openwebui
44. sudo apt update
45. sudo apt upgrade -y
46.sudo apt install -y ca-certificates curl gnupg lsb-release
47.sudo mkdir -p /etc/apt/keyrings
48. curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
49. sudo chmod a+r /etc/apt/keyrings/docker.gpg
50.echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
51. https://download.docker.com/linux/ubuntu \
52.$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
53. sudo apt update
54.sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
55. sudo systemctl start docker
56.sudo systemctl enable docker
57.sudo systemctl status docker
58. sudo usermod -aG docker $USER
59.newgrp docker
