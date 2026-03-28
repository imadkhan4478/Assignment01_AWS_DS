# 🚀 UniEvent System — Scalable AWS Architecture

## 📌 Project Overview
Designed and deployed a production-style cloud architecture for the UniEvent System using AWS. 

The system is built to handle real-world traffic with high availability, fault tolerance, and secure network isolation. This project simulates how modern backend systems are deployed in industry environments.

---

## 🧠 Key Highlights
* **Multi-AZ deployment** for zero single point of failure
* **Fully private backend infrastructure**
* **Intelligent traffic routing** via Load Balancer
* **Auto Scaling** with self-healing capability
* **Secure access** using IAM + SSM (no SSH keys required)

---

## 🏗️ Architecture

*(Replace the path below with your actual diagram image file)*
![Architecture Diagram](screenshots/architecture-diagram.png)

> Users → Internet → ALB → Target Group → EC2 Instances (Private Subnets)
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↑
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Auto Scaling Group

---

## ⚙️ Tech Stack

| Layer | Service Used |
| :--- | :--- |
| **Networking** | VPC, Subnets, Route Tables |
| **Compute** | EC2 (Amazon Linux 2023) |
| **Load Balancing** | Application Load Balancer |
| **Scaling** | Auto Scaling Group |
| **Security** | Security Groups, IAM |
| **Access** | AWS Systems Manager (SSM) |

---

## 🧱 Infrastructure Breakdown

### 🔹 VPC Design
* **Custom VPC:** `UniEvent-vpc`
* **Region:** `eu-north-1`
* **Subnet Strategy:** Spread across 2 Availability Zones for resilience ✔️

| Subnet Type | Purpose |
| :--- | :--- |
| **Public** | ALB, NAT Gateway |
| **Private** | EC2 Instances |

### 🌐 Internet Connectivity
* **Internet Gateway:** Enables public access to the ALB.
* **NAT Gateway:** Allows private instances to install packages and fetch updates while blocking inbound traffic.

### 🧭 Routing Logic

| Route Table | Destination | Target |
| :--- | :--- | :--- |
| **Public** | `0.0.0.0/0` | IGW |
| **Private** | `0.0.0.0/0` | NAT |

---

## 🔐 Security Design
* **ALB Security Group:** Allows HTTP (80) from anywhere.
* **EC2 Security Group:** Allows HTTP (80) *only* from ALB.
* ✔️ *Result: Backend is completely hidden from public access.*

---

## ⚙️ Compute Automation
* **Launch Template:** `UniEvent-Template`
* **OS:** Amazon Linux 2023
* **Permissions:** IAM Role attached
* **Auto-configured via User Data:**

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>UniEvent System - GIKI</h1>" > /var/www/html/index.html

⚖️ Load Balancing & 🔄 Auto Scaling
Application Load Balancer
Public-facing entry point that distributes traffic evenly.

Target Group: * Health check path: /

Success codes: 200–499

✔️ Ensures only healthy instances receive traffic.

Auto Scaling Group & Self-Healing
Desired Capacity: 2 instances (Integrated with Target Group).

Behavior: Detects unhealthy instances → Terminates automatically → Launches replacement instances.

✔️ System recovers without manual intervention.

📊 Real Engineering Decisions
Why Private Subnets? Prevents direct attacks on backend servers.

Why a NAT Gateway? Allows instances to download updates without exposing them to the internet.

Why an ALB instead of direct EC2 access? Ensures proper load distribution, health monitoring, and fault isolation.

Why SSM instead of SSH? Eliminates SSH key management and provides more secure, auditable access control.

💸 Cost Considerations
The NAT Gateway is the most expensive component in this setup. For learning or non-production environments, costs can be optimized by:

Using a NAT Instance instead of a Managed NAT Gateway.

Scheduling infrastructure shutdowns during off-hours.

🚧 Future Improvements
[ ] HTTPS implementation using AWS Certificate Manager (ACM).

[ ] CI/CD pipeline integration (GitHub Actions + AWS).

[ ] RDS integration for a robust database layer.

[ ] CloudWatch monitoring and advanced alerting.

[ ] WAF (Web Application Firewall) for enhanced edge security.

📷 Screenshots
(Upload your screenshots to a screenshots folder in your repository, and these links will display them automatically)

🎯 What This Project Demonstrates
Cloud architecture design thinking

Strong understanding of AWS networking

Security-first mindset

Production-level deployment practices
