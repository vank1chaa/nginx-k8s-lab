NGINX on Kubernetes with CI/CD

Project Overview

This project demonstrates a full CI/CD pipeline for deploying an NGINX web application to a self-hosted Kubernetes cluster, using GitHub Actions, Docker, Ingress, autoscaling, and readiness/liveness probes.

________________________________________
📁 Directory Structure

nginx-k8s-lab/

├── .github/workflows/docker-build.yml   # CI workflow to build & push image       
├── Dockerfile                           # Dockerfile with static index.html    
├── index.html                           # Hello World HTML    
├── k8s/    
│   ├── nginx-deployment.yaml           # NGINX Deployment + HPA + Probes    
│   ├── nginx-service.yaml              # ClusterIP Service    
│   └── ingress.yaml                    # Ingress with SSL    
├── tls/                                 # Self-signed TLS cert & key    
│   ├── tls.crt    
│   └── tls.key    
└── README.md    

________________________________________

⚙️ Prerequisites
•	GitHub account

•	DockerHub account

•	Self-hosted Linux machine (used Minikube on openSUSE SLES 15 SP6)

•	Kubernetes CLI (kubectl)

•	Helm

•	Docker

________________________________________

🚀 Steps to Reproduce from Scratch

🛠️ 1. Clone Repo
git clone git@github.com:ivankanev/nginx-k8s.git
cd nginx-k8s

🐳 2. Configure Docker & GitHub Secrets
Create GitHub repo secrets: - DOCKER_USERNAME: your Docker Hub username - DOCKER_PASSWORD: your Docker Hub token/password
These are used in the GitHub Actions workflow.

⚙️ 3. Register GitHub Runner
Download and configure self-hosted runner:
# From your GitHub repo > Settings > Actions > Runners
./config.sh --url https://github.com/ivankanev/nginx-k8s --token <TOKEN>
./run.sh

🧱 4. Deploy Minikube Cluster
minikube start --driver=docker

🧰 5. Install Ingress Controller
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx

🧪 6. Deploy NGINX Application
kubectl apply -f k8s/

🔐 7. Create TLS Secret
kubectl create secret tls nginx-tls \
  --cert=tls/tls.crt \
  --key=tls/tls.key

📈 8. Enable Metrics Server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

📊 9. Apply HPA
kubectl get hpa
# Should show target 50%, 2-5 pods scaling
________________________________________

🧪 Testing
•	Access service via curl -k https://nginx.test
•	Apply load with hey or ab to trigger HPA
