# 🛍️ Retail Store Microservices Project on AWS

## 📍 Table of Contents
1. [Introduction](#1-introduction)  
2. [Application Overview](#2-application-overview)  
3. [Infrastructure Design Principles](#3-infrastructure-design-principles)  
4. [Overall Architecture](#4-overall-architecture)  
5. [CloudFormation Deployment - Region 1](#5-cloudformation-deployment---region-1)  
6. [Terraform Deployment - Region 2](#6-terraform-deployment---region-2)  
7. [Global Traffic Management (Route 53)](#7-global-traffic-management-route-53)  
8. [CI/CD Pipeline Setup](#8-cicd-pipeline-setup)  
9. [Monitoring & Alerting](#9-monitoring--alerting)  
10. [Security Best Practices](#10-security-best-practices)  
11. [Author](#12-author)  
12. [Final Notes](#13-final-notes)  

---

## 1. Introduction

This project implements a **retail store microservices architecture** deployed across two AWS regions with full **CI/CD automation**, **observability**, and **disaster recovery**.

It uses:
- Amazon EKS for orchestration  
- Amazon RDS MySQL and DynamoDB for databases  
- Route 53 for DNS-based global failover  
- CodePipeline and CodeBuild for CI/CD  

---

## 2. Application Overview

**GitHub Repos:**
- CloudFormation_Infra: `https://github.com/Bhargavkulla/aws_project_info.git`
- Terraform_Infra: `https://github.com/Bhargavkulla/terraform_infra.git`
- Frontend_and_Backend: `https://github.com/Bhargavkulla/AWS_CAPSTONE_FINAL_PROJECT.git`

**Tech Stack:**
- **Frontend**: Java (Spring Boot UI)
- **Cart Service**: Java (Spring Boot) + DynamoDB
- **Catalog Service**: Go + MySQL

**Infrastructure Tools:**
- CloudFormation (Region 1 - us-east-1)
- Terraform (Region 2 - us-west-1)

---

## 3. Infrastructure Design Principles

| Goal              | Strategy                                                      |
|-------------------|---------------------------------------------------------------|
| High Availability | Multi-AZ subnets, Multi-region failover using Route 53        |
| Fault Tolerance   | Redundant EKS node groups, Route 53 health checks             |
| Scalability       | EKS Auto Scaling, Load Balancers                              |
| DR Readiness      | Active/passive regional setup with Route 53 + ALB failover    |

---

## 4. Overall Architecture

```
                         🌐 Internet
                              |
                         [ Route 53 ]
                      /                  \
         📍 Region 1 (us-east-1)       📍 Region 2 (us-west-1)
         ------------------------       ------------------------
         [ ALB ]                       [ ALB ]
            |                              |
         [ EKS ]                        [ EKS ]
            |                              |
   [ Java & Go Microservices ]   [ Java & Go Microservices ]
            |                              |
 [ RDS MySQL & DynamoDB ]       [ RDS MySQL & DynamoDB ]
```

---

## 5. CloudFormation Deployment --- Region 1

💡 **Template File**: `infras.yaml`

### 🔍 Key Resources

| Layer     | Resources                                         |
|-----------|--------------------------------------------------|
| Network   | VPC, Subnets, IGW, NAT Gateway, Route Tables     |
| Compute   | EKS Cluster with Managed Node Groups             |
| Database  | RDS MySQL, DynamoDB                              |
| IAM       | Roles for EKS, EC2, CloudWatch, CodeBuild        |
| Monitoring| CloudWatch Logs & Alarms, Fluent Bit, SNS        |

### 🛠 Deployment Steps

1. Commit `infras.yaml` to a GitHub repo
2. Create a CodePipeline:
   - Source: GitHub
   - Deploy: CloudFormation
3. Trigger pipeline to create infrastructure

---

## 6. Terraform Deployment --- Region 2

💡 **Terraform Stack Includes**: VPC, EKS, RDS

### 🛠 Deployment Steps

```bash
cd terraform/
terraform init
terraform plan
terraform apply
```

This provisions the same architecture in Region `us-west-1`.

---

## 7. Global Traffic Management (Route 53)

🧠 **Failover Setup**

| Record Type | Region     | Role      | Health Check |
|-------------|------------|-----------|--------------|
| A (Alias)   | us-east-1  | PRIMARY   | Enabled      |
| A (Alias)   | us-west-1  | SECONDARY | Yes          |

- If the primary ALB fails, Route 53 automatically routes traffic to the secondary region.

---

## 8. CI/CD Pipeline Setup

📦 **Tools Used**

| Tool                     | Purpose                                     |
|--------------------------|---------------------------------------------|
| CodePipeline             | Manages the pipeline flow                   |
| CodeBuild                | Builds, tests, deploys to EKS               |
| ECR                      | Stores Docker images                        |
| SonarQube                | Performs Java code quality analysis         |
| EventBridge + SNS + Lambda | Notifies on pipeline failure             |

### 📊 Pipeline Architecture

```
GitHub → CodeBuild → EKS → Route 53
    ↑
 Webhook Trigger
```

### 🧪 SonarQube
- Integrated into CodeBuild using environment variables
- Fails builds if code quality gates are not passed

---

## 9. Monitoring & Alerting

🔔 **CloudWatch Alarms**

| Metric         | Threshold | Action     |
|----------------|-----------|------------|
| RDS CPU        | >70%      | SNS Email  |
| Pod CPU (EKS)  | >80%      | SNS Email  |
| Pod Restarts   | >1/5min   | SNS Email  |

📊 **Prometheus + Grafana**
- Deployed via `kube-prometheus`
- Dashboards for pod, cluster, and service metrics
- AlertManager integrated for alerting

🪵 **Fluent Bit**
- Collects logs from EKS pods and ships to CloudWatch

---

## 10. Security Best Practices

- RDS is not publicly accessible
- IAM policies follow the **least privilege** model
- Network access controlled using **SGs** 

---

## 11. Author

**Bhargav Kulla**  
GitHub: [@Bhargavkulla](https://github.com/Bhargavkulla)

---

## 12. Final Notes

This project is a production-ready reference architecture that ensures:
- 🚀 High availability with multi-region deployment  
- 🔁 Disaster recovery with Route 53 DNS failover  
- 🧪 Continuous integration and delivery using AWS DevOps tools  
- 📊 Observability using Prometheus, Grafana, and CloudWatch  
- 🔒 Security and scalability using AWS best practices  

---

## Useful Commands

```bash
# Configure kubectl
aws eks update-kubeconfig --region <region> --name <cluster-name>

# Check pods and services
kubectl get nodes
kubectl get pods -A
kubectl get svc -A

```

