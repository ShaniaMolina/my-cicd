# 🚀 Static Website Deployment with GitHub Actions, Docker, and EC2

## 📌 Overview
This project demonstrates how to deploy a **static website** on an **AWS EC2 instance** using **Docker** and an automated **CI/CD pipeline with GitHub Actions**.  

The pipeline builds a Docker image for the static website, pushes it to **Docker Hub**, and deploys the latest version automatically to the EC2 server over SSH.

---

## 🛠️ Project Steps

1. **Provision EC2 Instance**
   - Launched an **Ubuntu 22.04 EC2 instance**.  
   - Configured the **Security Group** to allow:
     - **Port 22 (SSH)** for remote access  
     - **Port 80 (HTTP)** for serving the website  

2. **Install Docker on EC2**
   - Installed Docker to run containers on the remote host.  

3. **Prepare Static Website**
   - Pulled an existing static website.  
   - Pushed the website files into a **GitHub repository**.  

4. **Configure SSH Connectivity**
   - Generated and configured an **SSH key pair**.  
   - Set up **GitHub Actions secrets** for secure access to EC2 and Docker Hub.  

5. **Set up CI/CD Pipeline**
   - Automated workflow that:
     - Builds a Docker image for the static site  
     - Pushes the image to Docker Hub  
     - SSHs into EC2 and deploys the container automatically  

---

## 🔑 Required GitHub Secrets

Set the following secrets in your **GitHub repository** under **Settings > Secrets and variables > Actions**:

- `DOCKERHUB_USERNAME` → Your Docker Hub username  
- `DOCKERHUB_PASSWORD` → Your Docker Hub password or access token  
- `EC2_HOST` → Public IP or hostname of your EC2 instance  
- `EC2_SSH_KEY` → Private SSH key for EC2 access  

---

## 🚀 How It Works

1. Push code changes to the `main` branch  
2. GitHub Actions builds a Docker image and pushes it to Docker Hub  
3. The pipeline connects via SSH to EC2  
4. EC2 pulls the latest Docker image  
5. A container with Apache serving the static website is started on **port 80**  

---

## ✅ Outcome

- Fully automated deployment of a static website to **AWS EC2**  
- Continuous Delivery powered by **GitHub Actions**  
- Infrastructure using **Docker** + **Apache**  
- Zero manual deployment effort after setup  

---

## 💡 Recommendations

- **Use a non-root user on EC2** for better security instead of connecting as `root`.  
- **Enable HTTPS** using a reverse proxy (e.g., Nginx with Let’s Encrypt or AWS Load Balancer + ACM).  
- **Add monitoring/logging** with CloudWatch or Prometheus to track uptime and container health.  
- **Use Docker Compose or ECS/EKS** if scaling to multiple services or containers.  
- **Automate EC2 setup** with Infrastructure as Code (Terraform/CloudFormation) for reproducibility.  

---

## 📂 Code Samples

### GitHub Actions Workflow (`.github/workflows/deploy.yml`)

```yaml
name: Build and Deploy Static Website to EC2

on:
  push:
    branches:
      - main  # Triggers when you push to main branch

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: 🧾 Checkout latest code
      uses: actions/checkout@v3

    - name: 🔐 Log in to Docker Hub
      run: echo "${{ secrets.DOCKERHUB_PASSWORD }}" | docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin

    - name: 🛠️ Build Docker image with latest static site
      run: docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/static-website:latest .

    - name: 📤 Push image to Docker Hub
      run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/static-website:latest

    - name: 🚀 SSH and deploy on EC2
      uses: appleboy/ssh-action@v1.0.3
      with:
        host: ${{ secrets.EC2_HOST }}
        username: root
        key: ${{ secrets.EC2_SSH_KEY }}
        script: |
          docker pull ${{ secrets.DOCKERHUB_USERNAME }}/static-website:latest
          docker stop static-website || true
          docker rm static-website || true
          docker run -d --name static-website -p 80:80 ${{ secrets.DOCKERHUB_USERNAME }}/static-website:latest
```
