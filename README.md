# UniEvent System - AWS Infrastructure ☁️

This repository contains the infrastructure configuration, deployment documentation, and data automation scripts for the **UniEvent System (GIKI)**. The project demonstrates a highly available, fault-tolerant, and secure cloud architecture deployed on Amazon Web Services (AWS).



## 🏗️ Architecture Features

* **Custom VPC:** Deployed with Public and Private Subnets across multiple Availability Zones (`eu-north-1a`, `eu-north-1b`) for high availability.
* **Security First:** EC2 backend servers reside strictly in Private Subnets. All public web traffic is routed securely through an Application Load Balancer (ALB).
* **Auto Scaling:** An Auto Scaling Group (ASG) monitors server health and dynamically provisions or terminates instances based on load and Target Group health checks.
* **Secure Egress:** A Regional NAT Gateway provides private instances with secure, outbound-only internet access for software updates and API calls.
* **Automated Bootstrapping:** EC2 instances are provisioned using a Launch Template with a custom User Data script to automatically install and configure the Apache web server.

## ⚙️ Deployment Configuration

### 1. Networking (VPC)
* **Internet Gateway:** `UniEvent-igw`
* **NAT Gateway:** `UniEvent-NAT` (Regional, Public Subnet)
* **Route Tables:** 1 Public Route Table (IGW targeting), 2 Private Route Tables (NAT Gateway targeting).

### 2. Compute & Scaling (EC2)
* **OS:** Amazon Linux 2023
* **Launch Template:** `UniEvent-Template`
* **User Data Script:**
  ```bash
  #!/bin/bash
  yum update -y
  yum install -y httpd
  systemctl start httpd
  systemctl enable httpd
  echo "<h1>Welcome to UniEvent System - GIKI</h1><p>Secure Backend Servers are Running!</p>" > /var/www/html/index.html
