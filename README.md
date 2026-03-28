# 🚀 UniEvent System — Scalable AWS Architecture

![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![Security](https://img.shields.io/badge/Security-Strict-success?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Deployed-success?style=for-the-badge)

## 📌 Project Overview
Designed and deployed a production-style cloud architecture for the UniEvent System using AWS. 

The system is built to handle real-world traffic with high availability, fault tolerance, and secure network isolation. This project simulates how modern backend systems are deployed in industry environments, demonstrating a secure, scalable foundation.

---

## 🧠 Key Highlights
* **Multi-AZ deployment** for zero single point of failure.
* **Fully private backend infrastructure**.
* **Intelligent traffic routing** via Load Balancer.
* **Auto Scaling** with self-healing capability.
* **Secure access** using IAM + SSM (no SSH keys required).

---

## 🏗️ Architecture

![Architecture Diagram]

> Users → Internet → ALB → Target Group → EC2 Instances (Private Subnets)
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↑
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Auto Scaling Group

---

## ⚙️ Tech Stack

| Layer | Service Used |
| :--- | :--- |
| **Networking** | VPC, Subnets, Route Tables, IGW, NAT Gateway |
| **Compute** | EC2 (Amazon Linux 2023) |
| **Load Balancing** | Application Load Balancer |
| **Scaling** | Auto Scaling Group |
| **Security** | Security Groups, IAM |
| **Access** | AWS Systems Manager (SSM) |

---

## 📘 Step-by-Step Deployment & Infrastructure Breakdown

### Step 1: Laying the Network Foundation (VPC & Subnets)
Created an isolated virtual network (`UniEvent-vpc` in `eu-north-1`) to ensure complete control over the environment's security. To ensure high availability, the VPC was divided into four subnets across two Availability Zones:
* **2 Public Subnets:** To host public-facing resources.
* **2 Private Subnets:** To securely host backend EC2 instances.

![VPC and Subnets Configuration]
<img width="1909" height="861" alt="image" src="https://github.com/user-attachments/assets/8ebf51e5-294c-46d9-bad6-2573133e7978" />


### Step 2: Configuring Internet Access (IGW & NAT Gateway)
Configured gateways to allow public traffic to the website while keeping backend servers hidden.
* **Internet Gateway:** Enables public access to the ALB.
* **NAT Gateway:** A Regional NAT Gateway allows private instances to install packages and fetch updates while strictly blocking inbound traffic.

![Gateways Configuration]
<img width="1894" height="848" alt="image" src="https://github.com/user-attachments/assets/f33326d4-4ab1-4b6b-b902-b1ea519f2cbb" />


### Step 3: Directing the Traffic (Routing Logic)
Configured the route tables to ensure traffic flowed to the correct destinations securely.

| Route Table | Destination | Target |
| :--- | :--- | :--- |
| **Public** | `0.0.0.0/0` | Internet Gateway (IGW) |
| **Private** | `0.0.0.0/0` | NAT Gateway |

![Route Tables]

<img width="1899" height="802" alt="image" src="https://github.com/user-attachments/assets/7e76f370-0fb5-4920-b13b-0a4a7b12e779" />


### Step 4: Establishing Security Boundaries (Security Groups)
Enforced the principle of least privilege using strict virtual firewalls.
* **ALB Security Group:** Allows `HTTP (80)` from anywhere.
* **EC2 Security Group:** Allows `HTTP (80)` *only* from the ALB Security Group. 
* *Result: The backend is completely hidden from direct public access.*

![Security Groups]

<img width="1896" height="857" alt="image" src="https://github.com/user-attachments/assets/57e27bf0-cc93-4fbc-b705-fbfdafcb1d3f" />


### Step 5: Compute Automation (Launch Template)
Automated the server creation process so any new server automatically configures itself to host the UniEvent website. 
* **Launch Template:** `UniEvent-Template` (Amazon Linux 2023).
* **Permissions:** IAM Role (`AmazonSSMManagedInstanceCore`) attached for keyless SSM access.
* **User Data Script:**

<img width="1901" height="858" alt="image" src="https://github.com/user-attachments/assets/80411673-c9bd-4753-8029-950f09828ae8" />

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>UniEvent System - GIKI</h1>" > /var/www/html/index.html
```
******Step 6: Load Balancing & Health Checks******

<img width="1871" height="820" alt="image" src="https://github.com/user-attachments/assets/073d39aa-d3ca-48fc-9a8f-c4701ce93c35" />

Implemented an Application Load Balancer to distribute incoming traffic evenly and monitor server health.

Target Group: Pings the root path (/). Success codes set to 200–499.

ALB: Listens on Port 80 and actively forwards traffic only to "Healthy" instances.

******Step 7: Auto Scaling & Self-Healing******

<img width="1909" height="820" alt="image" src="https://github.com/user-attachments/assets/6c41a3fb-4f8d-41b9-a5e2-c842f7ba9007" />

Wrapped the backend instances in an Auto Scaling Group to guarantee continuous uptime.

Desired Capacity: 2 instances.

Behavior: Detects unhealthy instances → Terminates automatically → Launches replacement instances. The system recovers without manual intervention.

🎉 Final Result
The fully deployed and load-balanced UniEvent System landing page.

📊 Real Engineering Decisions
Why Private Subnets? Prevents direct attacks on backend servers.

Why a NAT Gateway? Allows instances to download updates without exposing them to the internet.

Why an ALB instead of direct EC2 access? Ensures proper load distribution, health monitoring, and fault isolation.

Why SSM instead of SSH? Eliminates SSH key management and provides more secure, auditable access control.

🚧 Future Improvements
[ ] HTTPS implementation using AWS Certificate Manager (ACM).

[ ] CI/CD pipeline integration (GitHub Actions + AWS).

[ ] RDS integration for a robust database layer.

[ ] CloudWatch monitoring and advanced alerting.
