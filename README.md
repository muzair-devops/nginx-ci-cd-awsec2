# 🚀 Nginx CI/CD with GitHub Actions (AWS EC2)

This project demonstrates a simple **CI/CD pipeline** using **GitHub Actions** with a **self-hosted runner**.  
The pipeline automatically builds and deploys an **Nginx container** serving a custom HTML page whenever changes are pushed to the `main` branch.

---

## 📂 Project Structure
```
nginx-ci-cd/
│── index.html           # Custom HTML page
│── Dockerfile           # Dockerfile to build Nginx image
│── docker-compose.yml   # Compose file to run container
│── .github/
│    └── workflows/
│         └── ci.yml     # GitHub Actions workflow
```

---

## 🛠️ Prerequisites
- GitHub account + repository
- Docker & Docker Compose installed on your server/local machine
- A **self-hosted GitHub Actions runner** configured for your repo  
  (Go to **Repo → Settings → Actions → Runners → New self-hosted runner**)

---

## ⚙️ Setup

### 1. Clone the repository
```bash
git clone https://github.com/<your-username>/nginx-ci-cd.git
cd nginx-ci-cd
```

### 2. Build & run locally (optional test)
```bash
docker-compose up -d --build
```
Visit: 👉 [http://localhost:8080](http://localhost:8080)

---

## 🤖 CI/CD Workflow

The pipeline is defined in `.github/workflows/ci.yml`:

- **Trigger**: On push to `main`
- **Runner**: Uses a self-hosted runner
- **Steps**:
  1. Checkout code
  2. Stop old container
  3. Rebuild & start Nginx with Docker Compose

---

### Example Workflow (`ci.yml`)
```yaml
name: CI/CD Nginx Pipeline

on:
  push:
    branches: ["main"]

jobs:
  deploy:
    runs-on: self-hosted

    steps:
      - name: Checkout repo
        uses: actions/checkout@v3

      - name: Build & Run Nginx with Docker Compose
        run: |
          docker-compose down
          docker-compose up -d --build
```

---

## 🚀 Deployment

1. Push changes to `main`:
   ```bash
   git add .
   git commit -m "Update HTML page"
   git push origin main
   ```

2. GitHub Actions triggers the workflow.

3. The self-hosted runner executes Docker Compose and redeploys Nginx with your updated HTML page.

4. Access your app at:  
   👉 http://localhost:8080

---

## 📌 Notes
- You can change the port mapping in `docker-compose.yml` if needed.
- To keep the runner alive, configure it as a **service** instead of running `./run.sh` manually.
- Extend this setup to run on a remote server (AWS EC2, DigitalOcean, etc.) for real deployments.

---

## 🏆 Result
Every commit to `main` → redeploys your **Nginx container with updated HTML** automatically 🎉
