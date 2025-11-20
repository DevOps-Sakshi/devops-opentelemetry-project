📘 DevOps Opentelemetry Microservices Project

This project demonstrates a small-scale DevOps + Cloud + Containers workflow using:

Three Dockerized microservices (Ad, Product Catalog, Recommendation)

Infrastructure as Code using Terraform

Modular Terraform setup (Backend, VPC, EKS)

Deployments targeting AWS EKS

Basic Opentelemetry-based microservice structure (from the reference demo project)

This repository is structured to showcase real-world DevOps skills such as Dockerization, IaC, version control, and modular cloud deployments.

📂 Project Structure
DEVOPS-OPENTELEMETRY-PROJECT/
│
├── microservices/
│   ├── ad/
│   │   └── Dockerfile
│   │       
│   ├── product-catalog/
│   │   └── Dockerfile
│   │
│   └── recommendation/
│       └── Dockerfile
│
├── terraform/
│   ├── backend/          # S3 + DynamoDB for Terraform remote state
│   │   └── main.tf
│   │
│   ├── modules/
│   │   ├── eks/          # EKS module
│   │   │   ├── main.tf
│   │   │   ├── outputs.tf
│   │   │   └── variables.tf
│   │   │
│   │   └── vpc/          # VPC Module
│   │       ├── main.tf
│   │       ├── outputs.tf
│   │       └── variables.tf
│   │
│   ├── main.tf           # root Terraform config
│   ├── outputs.tf
│   └── variables.tf
│
└── README.md

🧩 Microservices Included

This project contains 3 microservices taken from the AWS Opentelemetry demo architecture.

1️⃣ Ad Service

A lightweight recommendation generator for ads.

2️⃣ Product Catalog Service

Returns mock products and handles product endpoints.

3️⃣ Recommendation Service

Processes recommendations based on product data.

Each microservice contains:

✔ Its own Dockerfile
✔ Can be built and run independently
✔ Designed to later deploy on Kubernetes (EKS)

🐳 Docker Instructions
Build Images
docker build -t sakshi1729/ad-service:v1
docker build -t sakshi1729/product-catalog:v1
docker build -t sakshi1729/recommendtion-service:v1

Run Containers 
docker run sakshi1729/ad-service:v1
docker run sakshi1729/product-catalog:v1
docker run sakshi1729/recommendtion-service:v1

☁️ Terraform Infrastructure

The Terraform folder manages AWS infrastructure in three layers:

🗂 1. Backend (Remote State)

Stores Terraform state safely using:

Amazon S3 (state file)

Amazon DynamoDB (state locking)

🌐 2. VPC Module

Creates:

VPC

Private & Public Subnets

NAT Gateway & Internet Gateway

Route Tables

Tags for EKS compatibility

🧵 3. EKS Module

Deploys:

Amazon EKS Cluster

IAM roles

Managed Node Group

Cluster authentication

🚀 Terraform Commands
Initialize
terraform init

Preview
terraform plan

Deploy
terraform apply

Destroy
terraform destroy

🔧 Tools & Technologies Used

Docker

Terraform

Kubernetes

AWS EKS

AWS cli

EC2

S3 + DynamoDB

VS Code

Git & GitHub

Linux Commands

🏗️ How This Project Was Built (Step-by-Step)

1️⃣ Created EC2 instance
2️⃣ Installed Docker, kubectl, Terraform
3️⃣ Cloned the Opentelemetry demo app
4️⃣ Built and fixed Docker-compose issues
5️⃣ Identified disk space issue → expanded EC2 volume
6️⃣ Rebuilt containers and services started successfully
7️⃣ Allowed inbound SG rules → accessed app via browser
8️⃣ Built 3 microservices manually with Dockerfiles
9️⃣ Pushed images to Docker Hub
🔟 Wrote Terraform backend, VPC, EKS module code
1️⃣1️⃣ Initialized and deployed using Terraform

🎯 Purpose of This Project

This repository is designed to:

Demonstrate real DevOps skills

Showcase microservices architecture

Build AWS infrastructure using Terraform

Practice Docker + Kubernetes deployment workflow

Share a structured project on GitHub & LinkedIn

👤 Author

Sakshi — DevOps & Cloud Enthusiast
Working on microservices, Docker, Terraform, AWS, and Kubernetes.
Project inspired by Abhishek Veermalla (Udemy)