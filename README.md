🚀 Argo CD on Minikube (Windows / PowerShell)

This guide shows how to install and access Argo CD on a local Minikube cluster using PowerShell on Windows.

📌 Architecture Overview
Argo CD on Minikube — High Level Flow

┌───────────────────┐
│     Git Repo       │
│ (App Manifests)    │
└─────────┬─────────┘
          │ GitOps Pull
┌─────────▼─────────┐
│     Argo CD        │
│  (Reconciler)      │
└─────────┬─────────┘
          │ applies manifests
┌─────────▼─────────┐
│   Minikube Cluster │
│ (Deployments/SVCs) │
└────────────────────┘

🏁 1. Start Minikube

minikube start --driver=docker


🧩 2. (Optional) Enable useful Minikube addons

minikube addons enable ingress


📦 3. Create namespace & install Argo CD

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


⏳ 4. Wait for Argo CD pods to be ready

kubectl get pods -n argocd -w


🌐 5. Access the Argo CD UI

A — Port-forward (recommended)
kubectl port-forward svc/argocd-server -n argocd 8080:443


Open in your browser:
🔗 https://localhost:8080

B — NodePort (optional)
kubectl -n argocd patch svc argocd-server -p '{"spec": {"type": "NodePort"}}'
kubectl -n argocd get svc argocd-server


🔑 6. Get initial admin password

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"

Decode:
powershell -command "[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String('VXB3ZUFub3d2LWYyYnFqaA=='))"


Result:
Password: UpweAnowv-f2bqjh


🔐 7. Change admin password
argocd account update-password --account admin --current-password <old> --new-password <new>


💻 8. (Optional) Install Argo CD CLI
Download:
https://github.com/argoproj/argo-cd/releases/latest

Then login:

argocd login localhost:8080 --username admin --password <PASTE_PASSWORD> --insecure


📁 Example GitOps Application
Create a simple demo application that syncs a public repo.

app.yaml

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: demo-app
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/argoproj/argocd-example-apps
    path: guestbook
    targetRevision: HEAD

  destination:
    server: https://kubernetes.default.svc
    namespace: default

  syncPolicy:
    automated:
      prune: true
      selfHeal: true


Apply it:
kubectl apply -f app.yaml

Your app will appear in the Argo CD UI.

=================================================================================================

📝 Quick Reference (Copy/Paste)

# start minikube
minikube start --driver=docker

# install argocd
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# wait
kubectl get pods -n argocd -w

# access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# get password
$pw = kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($pw))

# CLI login
argocd login localhost:8080 --username admin --password <PASTE_PASSWORD> --insecure







