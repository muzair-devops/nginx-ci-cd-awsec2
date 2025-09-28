
# CICD using Github Actions and AWS EC2

This documentation provides step-by-step instructions to create and configure an **Ubuntu 24.04 EC2 instance** on AWS using the CLI.

---

## Step 1: Create EC2 machine
 [AWS CLI](./AWS_CLI.md)


## Step 2. Install Docker and Docker Compose on EC2

SSH into the instance:

```bash
ssh -i my-keypair.pem ubuntu@<EC2_PUBLIC_IP>
```

Install Docker:

```bash
sudo apt update -y
sudo apt install -y docker.io
sudo systemctl enable docker
sudo usermod -aG docker ubuntu
```

Install Docker Compose:

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)"   -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

---

## Step 3. Create Nginx Project with Docker Compose

On your local machine or EC2 (or locally first, then push to GitHub):

**Dockerfile**

```dockerfile
FROM nginx:alpine
COPY ./nginx.conf /etc/nginx/nginx.conf
```

**docker-compose.yml**

```yaml
services:
  web:
    build: .
    ports:
      - "8080:80"
```

**index.html**

```html
<!DOCTYPE html>
<html>
<head>
  <title>My CI/CD Nginx App</title>
</head>
<body>
  <h1>Hello from Nginx CI/CD Pipeline!</h1>
  <p>This page is deployed automatically with GitHub Actions + Docker Compose.</p>
</body>
</html>

```

---

## Step 4. Setup SSH Keys for GitHub Actions

1. Generate a new keypair **inside EC2**:

```bash
ssh-keygen -t ed25519 -C "github-ci" -f ~/.ssh/github_ci_key -N ""
```

2. Add the public key to authorized keys:

```bash
cat ~/.ssh/github_ci_key.pub >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

3. Print the private key:

```bash
cat ~/.ssh/github_ci_key
```

4. Copy the whole output (`-----BEGIN ... END-----`) into your GitHub repository secrets:

- `EC2_HOST` → your EC2 public IP  
- `EC2_USER` → `ubuntu`  
- `EC2_SSH_KEY` → the private key  

<img width="1162" height="691" alt="image" src="https://github.com/user-attachments/assets/bb83ebbf-2b79-4359-9dd5-96577778e826" />


---

## Step 5. Create GitHub Actions Workflow

In your repository, create `.github/workflows/deploy.yml`:

```yaml
name: Deploy App to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Deploy to EC2 via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd /home/ubuntu/nginx-ci-cd-awsec2
            git pull origin main
            docker-compose down
            docker-compose up -d --build
```

---

## Step 6. Test Deployment

1. Push code to `main` branch.  
2. GitHub Actions will trigger automatically.  
3. Check logs in **GitHub → Actions** tab.  
4. Visit your EC2 Public IP in a browser:  

```
http://<EC2_PUBLIC_IP>:8080
```

You should see:
<img width="2516" height="1430" alt="image" src="https://github.com/user-attachments/assets/ed57771d-2d19-447c-8517-584cef02f007" />


---

## ✅ Summary

- EC2 created with AWS CLI.  
- Docker + Docker Compose installed.  
- Nginx project configured with Docker.  
- SSH keypair generated for GitHub Actions.  
- CI/CD workflow added to deploy automatically.  

---

## 🔧 Troubleshooting

### 1. Permission Denied (publickey)
- Ensure the private key in GitHub Secrets matches the **public key** in `/home/ubuntu/.ssh/authorized_keys`.  
- Check permissions:  
  ```bash
  chmod 700 ~/.ssh
  chmod 600 ~/.ssh/authorized_keys
  ```
- Verify connection manually:  
  ```bash
  ssh -i ~/.ssh/github_ci_key ubuntu@<EC2_PUBLIC_IP>
  ```


### 4. GitHub Workflow Fails
- Ensure `appleboy/ssh-action` version matches latest.  
- Check that secrets are set correctly:  
  - `EC2_HOST` → public IP  
  - `EC2_USER` → `ubuntu`  
  - `EC2_SSH_KEY` → private key  

### 4. Firewall/Security Group Issues
- Ensure EC2 **Security Group** allows inbound traffic on port `22` (SSH) and `8080ZZZ` (HTTP).

