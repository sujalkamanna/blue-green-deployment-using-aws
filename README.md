# 🚀 Blue-Green Deployment using AWS ALB, Target Groups, Docker Compose and Amazon ECR

This project demonstrates a production-style Blue-Green deployment strategy using AWS services and Docker containers.

The setup uses:

- Ubuntu 24 EC2 instances
- Amazon ECR for container image storage
- Docker + Docker Compose
- Application Load Balancer (ALB)
- Target Groups (TG)
- Apache web server running inside Docker containers
- Blue-Green deployment model

The purpose of this project is to achieve near-zero downtime deployments and easy rollback capabilities.

---

# 📌 Architecture
![Architecture Diagram](https://github.com/sujalkamanna/blue-green-deployment-using-aws/blob/main/archicture.png)


```text
                        Internet
                            │
                            ▼
                  +-----------------+
                  |       ALB       |
                  | Application LB  |
                  +-----------------+
                      │         │
                      │         │
              TG-Blue │         │ TG-Green
                      │         │
                      ▼         ▼
          +----------------+   +----------------+
          | EC2 Instance   |   | EC2 Instance   |
          | Ubuntu 24      |   | Ubuntu 24      |
          | Blue Version   |   | Green Version  |
          | Docker Compose |   | Docker Compose |
          | Apache         |   | Apache         |
          +----------------+   +----------------+
                     ▲                ▲
                     │                │
                     └────Pull Images─┘
                             │
                             ▼
                        Amazon ECR
```

---

# 🎯 Project Objectives

- Implement Blue-Green deployment strategy
- Reduce deployment downtime
- Provide quick rollback capability
- Store application images in Amazon ECR
- Use Docker Compose for container management
- Route traffic through ALB and Target Groups

---

# 🛠 Technologies Used

| Technology | Purpose |
|------------|----------|
| AWS EC2 | Application servers |
| Ubuntu 24 | Operating System |
| Docker | Containerization |
| Docker Compose | Container management |
| Amazon ECR | Image repository |
| ALB | Traffic routing |
| Target Groups | Environment management |
| Apache2 | Web server |
| HTML/CSS | Static website |

---

# 📁 Project Structure

```text
project/

📦blue green deployment
 ┣ 📜blue.html
 ┣ 📜blue_compose.yml
 ┣ 📜commands.txt
 ┣ 📜Dockerfile
 ┣ 📜green.html
 ┣ 📜green_compose.yml
 ┗ 📜green.html
 ┗ 📜README.md
```

---

# Step 1: Create Amazon ECR Repository

Go to:

AWS Console

→ Elastic Container Registry

→ Create Repository

Repository Name:

```bash
b_g_deployment
```

Example repository URI:

```bash
ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/b_g_deployment
```

---

# Step 2: Blue Website

Create:

blue-site/index.html

---

# Step 3: Green Website

Create:

green-site/index.html

---

# Step 4: Dockerfile

Create Dockerfile in both folders:

```dockerfile
FROM ubuntu:24.04

RUN apt update && \
    apt install apache2 -y

COPY index.html /var/www/html/index.html

EXPOSE 80

CMD ["apachectl","-D","FOREGROUND"]
```

---

# Step 5: Build Docker Images

Blue:

```bash
cd blue-site

docker build -t blue:v1 .
```

Green:

```bash
cd ../green-site

docker build -t green:v2 .
```

---

# Step 6: Authenticate Docker with ECR

```bash
aws ecr get-login-password --region ap-south-1 | \
docker login \
--username AWS \
--password-stdin ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com
```

---

# Step 7: Tag Images

Blue:

```bash
docker tag blue:v1 \
ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/b_g_deployment:blue-v1
```

Green:

```bash
docker tag green:v2 \
ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/b_g_deployment:green-v2
```

---

# Step 8: Push Images to ECR

```bash
docker push ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/b_g_deployment:blue-v1

docker push ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/b_g_deployment:green-v2
```

---

# Step 9: Launch EC2 Instances

Create:

Instance 1:

```text
Blue Server
Ubuntu 24
```

Instance 2:

```text
Green Server
Ubuntu 24
```

Security Group:

```text
SSH : 22
HTTP : 80
```

---

# Step 10: Install Docker on EC2

```bash
sudo apt update

sudo apt install docker.io docker-compose-v2 -y

sudo systemctl enable docker

sudo systemctl start docker

sudo usermod -aG docker ubuntu
```

---

# Step 11: Create Docker Compose File

Blue Instance:

```yaml
services:

 app:
   image: ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/b_g_deployment:blue-v1

   container_name: apache-blue

   ports:
      - "80:80"

   restart: always
```

Green Instance:

```yaml
services:

 app:
   image: ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/b_g_deployment:green-v2

   container_name: apache-green

   ports:
      - "80:80"

   restart: always
```

Run:

```bash
docker compose up -d
```

---

# Step 12: Create Target Groups

Create:

TG-Blue

```text
Protocol: HTTP
Port: 80
Health Check: /
```

Register:

```text
Blue EC2 Instance
```

Create:

TG-Green

```text
Protocol: HTTP
Port: 80
Health Check: /
```

Register:

```text
Green EC2 Instance
```

---

# Step 13: Create Application Load Balancer

Configuration:

```text
Name: app-alb

Scheme:
Internet Facing

Listener:
HTTP : 80

Forward to:
TG-Blue
```

---

# Testing Deployment

Open:

```bash
http://ALB-DNS-NAME
```

Expected:

```text
🚀 BLUE ENVIRONMENT
```

---

# Blue-Green Deployment Workflow

```text
Developer

    ↓

Build Docker Image

    ↓

Push Image → Amazon ECR

    ↓

Pull Image → Green Server

    ↓

Deploy Green Version

    ↓

Health Check Success

    ↓

Switch ALB Traffic

    ↓

Production Updated
```

---

# Rollback Procedure

If deployment fails:

```text
ALB

↓

Change Listener Rule

↓

TG-Green → TG-Blue
```

Traffic immediately returns to previous stable version.

---

# Future Improvements

- GitHub Actions CI/CD
- Automatic deployment pipeline
- HTTPS using ACM
- Route53 custom domain
- Monitoring with CloudWatch
- Auto Scaling Group support

---

# Author

Sujal Kamanna

DevOps | AWS | Docker | Cloud Engineering