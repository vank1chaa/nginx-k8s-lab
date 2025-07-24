NGINX on Kubernetes with CI/CD  

Project Overview:  

This project demonstrates a full CI/CD pipeline for deploying an NGINX web application to a self-hosted Kubernetes cluster, using GitHub Actions, Docker, Ingress, autoscaling, and readiness/liveness probes.  
________________________________________  
Completed Features:  

1	Create GitHub repository  
2	Register GitHub Actions self-hosted runner  
3	Deploy self-hosted Kubernetes cluster with Minikube  
4	Create CI/CD GitHub Actions workflow for Docker build & push  
5	Deploy NGINX Ingress controller  
6	Deploy NGINX image via CI/CD + Expose with Ingress (TLS/SSL)  
7	Configure readinessProbe  
8	Configure livenessProbe  
9	Configure HPA (Horizontal Pod Autoscaler) based on CPU  
10  Code Quality – SonarCloud Integration  
________________________________________  
Directory Structure:  

nginx-k8s-lab/  
├── .github/workflows/docker-build.yml   # CI workflow to build & push image  
├── Dockerfile                           # Dockerfile with static index.html  
├── index.html                           # Hello World HTML  
├── k8s/  
│   ├── nginx-deployment.yaml           # NGINX Deployment + HPA + Probes  
│   ├── nginx-service.yaml              # ClusterIP Service  
│   └── ingress.yaml                    # Ingress with SSL (uses correct ingressClassName)  
├── tls/                                 # Self-signed TLS cert & key  
│   ├── tls.crt  
│   └── tls.key  
└── README.md  

________________________________________  
Prerequisites:  

•	GitHub account  
•	DockerHub account  
•	Self-hosted Linux machine (used Minikube on openSUSE SLES 15 SP6)  
•	Kubernetes CLI (kubectl)  
•	Helm  
•	Docker  

________________________________________
1. Clone Repo  
```bash
git clone git@github.com:vank1chaa/nginx-k8s-lab.git ;cd nginx-k8s-lab  
```
2. Configure Docker & GitHub Secrets  
Create GitHub repo secrets:  
•	DOCKER_USERNAME
•	DOCKER_PASSWORD  
These are used in the GitHub Actions workflow.  

3. Register GitHub Runner  
Download and configure self-hosted runner:  
From your GitHub repo > Settings > Actions > Runners  
./config.sh --url https://github.com/vank1chaa/nginx-k8s-lab --token <TOKEN>  
./run.sh  

4. Deploy Minikube Cluster  
```bash
minikube start --driver=docker  
```
5. Enable Ingress Addon (Minikube built-in)  
Important: Do not use both Helm and the Minikube addon. This project uses the Minikube ingress addon, which avoids Helm-related TLS issues.  
```bash
minikube addons enable ingress  
```

6. Deploy NGINX Application  
```bash
kubectl apply -f k8s/  
```
7. Create TLS Certificate and Secret  
```bash
mkdir -p tls
```  
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \  
  -keyout tls/tls.key \  
  -out tls/tls.crt \  
  -subj "/CN=nginx.test" \  
  -addext "subjectAltName=DNS:nginx.test"  
```
```bash
kubectl delete secret nginx-tls --ignore-not-found
```
```bash
kubectl create secret tls nginx-tls \  
  --cert=./tls/tls.crt \  
  --key=./tls/tls.key  
  ```
8. Update /etc/hosts to Access Ingress  
Reminder: Ensure Minikube is running before executing this step, as the IP address depends on the active Minikube instance.  
```bash
sudo bash -c "echo \"$(minikube ip) nginx.test\" >> /etc/hosts"  
```  
9. Enable Metrics Server  
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
  ```bash
kubectl patch deployment metrics-server -n kube-system \  
  --type=json \  
  -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'  
```
Wait a few seconds, then verify:   
```bash
kubectl top nodes
```
```bash  
kubectl top pods  
```
10. Apply HPA  
```bash
kubectl autoscale deployment nginx-deployment \  
  --cpu-percent=50 \  
  --min=2 \  
  --max=5
```
```bash  
kubectl get hpa  
```
________________________________________  
Testing  
•	Access service via curl -k https://nginx.test  
•	Apply load with hey or ab to trigger HPA  
```bash
hey -n 100 -c 10 https://nginx.test  
for i in {1..10}; do curl -s -k https://nginx.test | grep "Hello"; done
```
________________________________________
Code Quality – SonarCloud Integration

This project uses **SonarCloud** to automatically analyze code quality on every push to the `main` branch.

Аnalysis includes:
- Security, reliability, and maintainability ratings  
- Code duplication detection  
- Static issue detection via GitHub Actions workflow  

**How it works:**
- GitHub Actions triggers SonarCloud analysis on every `push` or `pull request` to `main`.
- Workflow file: `.github/workflows/docker-build.yml`

Results available at:  
[https://sonarcloud.io/project/overview?id=vank1chaa_nginx-k8s-lab](https://sonarcloud.io/project/overview?id=vank1chaa_nginx-k8s-lab)

---

To simulate or demonstrate a working CI scan:

```bash
echo "# dummy change" >> README.md
git add README.md
git commit -m "Test SonarCloud CI"
git push origin main
```  
# dummy change
