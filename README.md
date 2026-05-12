# 🐳 Auto Update Portfolio-pipeline

A static HTML portfolio page served by Nginx, containerized with Docker, deployed to a local Kubernetes cluster via Minikube, and automatically rebuilt and redeployed on every push through a GitHub Actions CI/CD pipeline.

> Push a change → pipeline builds a new image → Kubernetes rolls it out with zero downtime.

---

## ⚙️ How It Works

```
Edit index.html  →  git push
                          ↓
                  GitHub Actions triggers
                          ↓
                  Docker image built & tagged
                  (latest + commit SHA)
                          ↓
                  Image pushed to Docker Hub
                          ↓
                  deployment.yml updated with new SHA tag
                  (committed back to repo — GitOps)
                          ↓
                  kubectl apply → rolling update on Minikube
                  (zero downtime, 2 replicas)
```

---

## 🛠️ Tech Stack

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat-square&logo=html5&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=flat-square&logo=nginx&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat-square&logo=kubernetes&logoColor=white)
![Minikube](https://img.shields.io/badge/Minikube-F5C518?style=flat-square&logo=kubernetes&logoColor=black)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white)

---

## 📁 Project Structure

```
portfolio-devops/
├── site/
│   └── index.html               # Portfolio page (edit name, role, skills here)
├── Dockerfile                   # Nginx base image, serves site/
├── kubernetes/
│   ├── deployment.yml           # 2 replicas, rolling update strategy
│   └── service.yml              # NodePort — exposes app on port 30080
└── .github/
    └── workflows/
        └── pipeline.yml         # CI/CD pipeline (build → push → deploy)
```

---

## 🚀 Run Locally with Docker

```bash
# Build the image
docker build -t portfolio-devops .

# Run the container
docker run -d -p 8080:80 portfolio-devops

# Open in browser
http://localhost:8080
```

---

## ☸️ Deploy to Kubernetes (Minikube)

```bash
# Start the local cluster
minikube start

# Apply the manifests
kubectl apply -f kubernetes/deployment.yml
kubectl apply -f kubernetes/service.yml

# Verify pods are running
kubectl get pods
# portfolio-deployment-xxxx   1/1   Running   0
# portfolio-deployment-xxxx   1/1   Running   0

# Get the URL and open in browser
minikube service portfolio-service --url
```

The Deployment runs **2 replicas** — deleting one pod triggers Kubernetes to immediately restart it, demonstrating self-healing behaviour.

```bash
# Test self-healing
kubectl delete pod <pod-name>
kubectl get pods --watch   # watch it come back automatically
```

---

## 🔁 CI/CD Pipeline

The `pipeline.yml` workflow triggers on every push to `main` and runs 5 steps:

| Step | What It Does |
|---|---|
| **Checkout** | Clones the repo onto the GitHub-hosted runner |
| **Docker login** | Authenticates to Docker Hub using encrypted GitHub Secrets |
| **Build & push** | Builds the image, pushes two tags: `latest` and the git commit SHA |
| **Update manifest** | Rewrites `deployment.yml` with the exact commit SHA tag |
| **Commit back** | Pushes the updated manifest to the repo (GitOps pattern) |

**Why commit SHA tagging?** `latest` is mutable — you can't tell which code version is running. SHA tags give a direct 1:1 mapping between what's deployed and what's in Git.

**Why commit the manifest back?** The repo always reflects the actual deployed state. This is the GitOps pattern — the repo is the source of truth.

Credentials are stored as `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` in GitHub repository secrets — never hardcoded.

---

## 🔄 Applying a Pipeline Update Locally

After the pipeline runs and commits a new SHA tag to `deployment.yml`:

```bash
# Pull the updated manifest
git pull

# Point kubectl to Minikube's Docker environment
eval $(minikube docker-env)

# Pull the new image
docker pull basraramit/portfolio:latest

# Apply the updated manifest
kubectl apply -f kubernetes/deployment.yml

# Watch the rolling update
kubectl rollout status deployment/portfolio-deployment
# deployment "portfolio-deployment" successfully rolled out
```

Kubernetes replaces old pods one by one with the new version — zero downtime.

---

## 🐳 Docker Hub

```bash
docker pull basraramit/portfolio:latest
```

[View on Docker Hub →](https://hub.docker.com/r/basraramit/portfolio)

---

## 🙋 Author

**Ramit Basra** — [GitHub](https://github.com/basraramit) · [LinkedIn](https://linkedin.com/in/ramit-basra)
