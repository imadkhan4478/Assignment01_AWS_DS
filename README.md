📘 Step-by-Step Deployment Guide: UniEvent System AWS Infrastructure
Project: UniEvent System (GIKI)
Author: Imad Khan
Region: eu-north-1 (Stockholm)

This document outlines the step-by-step creation of a highly available, secure, and auto-scaling web architecture on Amazon Web Services (AWS).

Step 1: Laying the Network Foundation (VPC & Subnets)
Before launching any servers, we created an isolated virtual network to ensure complete control over our environment's security.

Virtual Private Cloud (VPC): We created UniEvent-vpc to act as the foundational network boundary.

Subnet Tiering: To ensure high availability across multiple data centers, we divided the VPC into four subnets across two Availability Zones (eu-north-1a and eu-north-1b):

2 Public Subnets: To host public-facing resources (Load Balancer, NAT Gateway).

2 Private Subnets: To securely host our backend EC2 compute instances.

<img width="1899" height="656" alt="image" src="https://github.com/user-attachments/assets/5d317805-114e-49e3-a0bb-13dcebda4d31" />


Step 2: Configuring Internet Access (IGW & NAT Gateway)
Our network needed to allow public traffic to our website, while keeping our backend servers hidden but able to download software updates.

Internet Gateway (IGW): We created UniEvent-igw and attached it to the VPC. This is the main "front door" for public internet traffic.

NAT Gateway: We deployed a Regional NAT Gateway (UniEvent-NAT) inside the Public Subnet. This acts as a secure, one-way bridge, allowing our private EC2 instances to reach out to the internet to download the Apache web server, without allowing the public internet to initiate connections to them.

<img width="1867" height="854" alt="image" src="https://github.com/user-attachments/assets/81621cbf-1ed1-4301-b2ed-05531bb17fe3" />


Step 3: Directing the Traffic (Route Tables)
With the gateways built, we configured the "GPS" of our network to ensure traffic flowed to the correct destinations.

Public Route Table: We mapped all outbound traffic (0.0.0.0/0) to the Internet Gateway. We associated this table with our Public Subnets.

Private Route Tables: We mapped all outbound traffic (0.0.0.0/0) to the NAT Gateway. We associated these tables with our Private Subnets, ensuring our secure servers could successfully download the httpd packages.

<img width="1919" height="732" alt="image" src="https://github.com/user-attachments/assets/6a41961f-e48a-4b8e-b42c-695805b9c550" />

Step 4: Establishing Security Boundaries (Security Groups)
To enforce the principle of least privilege, we created strict virtual firewalls to act as "bouncers" for our resources.

ALB-SG (Load Balancer Security Group): Configured to allow inbound HTTP (Port 80) traffic from anywhere (0.0.0.0/0).

EC2-SG (Backend Security Group): Configured to allow inbound HTTP (Port 80) traffic ONLY from the ALB-SG. This ensures that no one can bypass the Load Balancer to directly access the backend servers.

<img width="1914" height="807" alt="image" src="https://github.com/user-attachments/assets/378df29d-80cb-47fb-a1e6-95911ec9518f" />


Step 5: Server Blueprint & Permissions (Launch Template & IAM)
We automated the server creation process so that any new server would automatically configure itself to host the UniEvent website.

IAM Role: We created UniEvent-SSM-Role with the AmazonSSMManagedInstanceCore policy. This grants us secure, keyless terminal access to the servers for troubleshooting.

Launch Template (UniEvent-Template): We selected the Amazon Linux 2023 OS, attached our IAM Role and Security Group, and added the following automated User Data script:

Bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Welcome to UniEvent System - GIKI</h1><p>Secure Backend Servers are Running!</p>" > /var/www/html/index.html
<img width="1862" height="841" alt="image" src="https://github.com/user-attachments/assets/4df453ad-016c-4c4d-a420-3254b1001b6b" />


Step 6: Load Balancing & Health Checks (Target Group & ALB)
To distribute incoming traffic evenly and monitor server health, we implemented an Application Load Balancer.

Target Group (UniEvent-TG): We configured this to ping the root path (/) of our servers on Port 80. We expanded the success codes to 200-499 to ensure smooth health check transitions.

Application Load Balancer (UniEvent-ALB): We deployed the ALB into the Public Subnets. It listens on Port 80 and actively forwards traffic only to instances marked as "Healthy" in the Target Group.

<img width="1904" height="855" alt="image" src="https://github.com/user-attachments/assets/263f40e4-141e-4a54-920c-c89952093449" />


Step 7: High Availability & Self-Healing (Auto Scaling Group)
Finally, we wrapped our backend instances in an Auto Scaling Group to guarantee continuous uptime.

Auto Scaling Group (UniEvent-ASG): Linked to our Launch Template and Target Group, we configured the ASG to maintain a desired capacity of 2 instances.

Self-Healing Mechanism: If an instance fails a health check or experiences a network anomaly (such as the initial 502 Bad Gateway errors during network configuration), the ASG automatically drains, terminates, and replaces the instance with a fresh, correctly configured server.


