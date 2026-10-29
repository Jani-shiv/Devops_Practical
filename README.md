# DevOps Practical Labs Guide

A comprehensive, production-grade repository containing verified, step-by-step documentation and runnable source code for DevOps CI/CD pipelines, containerization with Docker, GitHub Actions, Jenkins, and AWS cloud deployments.

---

## Practical Index

| # | Practical Guide | Core Tech Stack | Description |
|---|---|---|---|
| 1 | [Practical 1: Manual Static Web on Ubuntu VM](q1-deploy-static-website-manual-vm-ubuntu.md) | Ubuntu, Nginx | Safe, dedicated site root deployment with network setup explanation |
| 2 | [Practical 2: Automated Static Web on Ubuntu VM](q2-deploy-static-website-autoscript-vm-ubuntu.md) | Bash, Nginx | Idempotent bash script with syntax validation & HTTP healthcheck |
| 3 | [Practical 3: Manual Static Web on AWS EC2](q3-deploy-static-website-manual-aws-ec2-ubuntu.md) | AWS EC2, Nginx | Security-hardened EC2 instance configuration with dedicated virtual hosts |
| 4 | [Practical 4: Automated Static Web on AWS EC2](q4-deploy-static-website-autoscript-aws-ec2-ubuntu.md) | Bash, AWS EC2 | Automated bash script with dynamic IP detection and health verification |
| 5 | [Practical 5: Static Web via EC2 User Data](q5-deploy-static-website-userdata-aws-ec2-ubuntu.md) | AWS Cloud-Init, Nginx | Zero-SSH launch bootstrap with cloud-init log troubleshooting guide |
| 6 | [Practical 6: GitHub Actions Self-Hosted Runner on EC2](q6-github-actions-self-hosted-runner-aws-ec2-ubuntu.md) | GitHub Actions, Systemd | Dynamic runner setup, persistent service configuration, security policies |
| 7 | [Practical 7: GitHub Actions CI/CD to EC2 (Nginx)](q7-hosting-static-website-aws-ec2-github-actions-nginx.md) | GitHub Actions, SCP, Nginx | Structured `src/*` transfers and non-destructive virtual hosts |
| 8 | [Practical 8: GitHub Actions CI/CD to EC2 (Apache2)](q8-hosting-static-website-aws-ec2-github-actions-apache2.md) | GitHub Actions, SCP, Apache2 | Dedicated Apache VirtualHost with automated configuration validation |
| 9 | [Practical 9: Portfolio Container on Docker Hub](q9-create-portfolio-container-upload-docker-hub.md) | Docker, Nginx, Docker Hub | Production Alpine image, multi-tag builds (`v1.0.0` & `latest`), PAT login |
| 10 | [Practical 10: Docker CI/CD to EC2 (Nginx)](q10-cicd-pipeline-github-actions-docker-dockerhub-aws-ec2-nginx.md) | Docker, GitHub Actions, EC2 | Immutable Git SHA image tagging, daemon permissions, container healthcheck |
| 11 | [Practical 11: Docker CI/CD to EC2 (Apache2)](q11-cicd-pipeline-github-actions-docker-dockerhub-aws-ec2-apache2.md) | Docker, Apache2, EC2 | Commit SHA tagging, automated container replacement, status verification |
| 12 | [Practical 12: Full Stack CI/CD (MERN + Docker + EC2)](q12-fullstack-cicd-react-nodejs-mongodb-docker-github-actions-ec2.md) | React, Node.js, Mongo, Compose | Full source code, Nginx reverse proxy, internal database, health checks |
| 13 | [Practical 13: Jenkins on Windows Local PC](q13-jenkins-installation-job-setup-windows-local-pc.md) | Jenkins LTS, Windows, JDK 17 | JDK LTS verification, service accounts, Git paths, Freestyle jobs |
| 14 | [Practical 14: Jenkins CI/CD on AWS EC2](q14-jenkins-deployment-aws-ec2-cicd-pipeline.md) | Jenkins, Docker, GitHub, EC2 | Declarative Jenkinsfile, Docker socket permissions, webhook vs polling |

---

## Key Best Practices Applied
- **Non-Destructive Deployments**: All static site exercises configure dedicated virtual hosts (e.g. `/var/www/my-site`) without deleting `/var/www/html/*`.
- **Security Group Hardening**: Inbound SSH (Port 22) is restricted to specific IP addresses rather than global `0.0.0.0/0`.
- **Immutable Container Tagging**: CI/CD pipelines tag images with the short Git commit SHA for traceability and deterministic rollback.
- **Health Verification**: All pipelines and scripts validate container status and HTTP response codes before signaling successful deployment.
- **Isolated Databases**: Practical 12 keeps MongoDB internal to the Docker bridge network without exposing sensitive database ports to the internet.
