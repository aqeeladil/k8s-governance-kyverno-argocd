# Kubernetes Governance Using Kyverno and Argo CD
# Kubernetes Governance with Kyverno and Argo CD: Automating Cluster Security Policies

## Architecture

On a very high level, A DevOps Engineer will write the required Kyverno Policy custom resource and commits it to a Git repository. Argo CD which is pre configured with `auto-sync` to watch for resources in the git repo, deploys the Kyverno Policies on to the Kubernetes cluster.

![Screenshot 2023-02-19 at 12 40 48 PM](https://user-images.githubusercontent.com/43399466/219934201-b542599a-7f8a-4b72-a1bf-5db6ba1bfade.png)

## 1. Launch an EC2 Ubuntu Instance
- Configure Security Group:
    - SSH: Port 22
    - HTTP: Port 80
    - Kubernetes API: Port 6443
    - Argo CD: Port 8080
- Connect to the Instance:
    `ssh -i your-key.pem ubuntu@<EC2-Public-IP>`

## 2. Update the System and Install Required Tools:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl apt-transport-https

# Install Docker
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

## 3. Set Up a Minikube Cluster
```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube /usr/local/bin/

# Start Minikube Cluster:
minikube start --driver=docker --cpus=2 --memory=4096

# Verify Cluster Status:
kubectl get nodes
kubectl get pods -A
```

## 4. Install Kyverno
```bash
# There are two easy ways to install kyverno: (Helm) or (kubernetes YAML manifest).

# Using Kubernetes manifest yaml files
kubectl apply -f https://github.com/kyverno/kyverno/releases/latest/download/install.yaml

# Verify
kubectl get pods -n kyverno
```

## 5. Apply and Validate the Kyverno Policy: 
- Enforce Resource Requests and Limits
```bash
kubectl apply -f examples/enforce-pod-requests-limits.yml

# Check Applied Policy:
kubectl get clusterpolicy

# Test the Policy: Try Creating a Non-Compliant Deployment.
# The deployment will fail because resource limits are not defined.
kubectl create deployment nginx --image=nginx

# Test with a Compliant Deployment.
kubectl apply -f nginx-deployment.yaml
kubectl get pods

# View Kyverno Logs:
kubectl logs -n kyverno -l app.kubernetes.io/name=kyverno

# Verify Compliance Actions in Audit Logs:
kubectl get events --all-namespaces

# Look for `Validation failed` or `Enforced` messages.
```

## 6. Integrate Argo CD with Kyverno
```bash
# Store Kyverno policies in a GitHub repository. Use Argo CD to monitor the repository and sync changes. Every policy change will automatically apply to the Kubernetes cluster.

# Automation: No manual intervention.
# Ensure uniform policy enforcement across clusters.
# Scalability: Easily manage thousands of policies.

# There are three ways to install Argo CD (kubernetes YAML mainfests, Helm Charts, ArgoCD Opeartor)

# Using Kubernetes manifest yaml files
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Port Forward Argo CD UI (Access via Browser):
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Login to Argo CD: Default username: admin
# Retrieve default password:
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode

# Push your Kyverno policies to a GitHub repository
# Add the GitHub Repo in Argo CD:
argocd app create kyverno-policies \
  --repo <your-github-repo> \
  --path policies \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace kyverno

# Sync Argo CD with the cluster
argocd app sync kyverno-policies

# Verify Sync Status
argocd app list
```




