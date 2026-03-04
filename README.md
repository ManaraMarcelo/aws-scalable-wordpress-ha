<p align="center">
  <img src="https://github.com/ManaraMarcelo/aws-scalable-wordpress-ha/blob/main/IMAGES/logo_wordpress_aws.png" width="600">
</p>

<h1 align="center">AWS Scalable Web Infrastructure: High-Availability WordPress Deployment</h1>

<p align="center">
  Automated deployment with high scalability, data persistence, security, and load balancing using AWS, Docker, WordPress, and MySQL.
</p>

<br>

<div align="center">
  <a href="https://skillicons.dev">
    <img src="https://skillicons.dev/icons?i=aws,docker,linux,wordpress,mysql" alt="Technologies" />
  </a>
</div>

---

### 💡 Project Overview

**The Problem:** Traditional web hosting often fails under variable traffic loads and lacks data persistence when instances are terminated. Manually managing servers, databases, and file synchronization is inefficient and prone to downtime.

**The Solution:** A fully decoupled and elastic architecture on AWS. By separating the application layer (EC2), the database (RDS), and the file system (EFS), we ensure that the WordPress site can scale horizontally via Auto Scaling while maintaining a single, consistent state across all nodes.

---

## 📌 Table of Contents
1. [Features](#-features)
2. [Service Infrastructure](#-service-infrastructure)
3. [Configuration Steps](#-configuration-steps)
    * [1. VPC Creation](#1-vpc-creation)
    * [2. Security Group Configuration](#2-security-group-configuration)
    * [3. File System (EFS) Setup](#3-file-system-efs-setup)
    * [4. Managed Database (RDS) Setup](#4-managed-database-rds-setup)
    * [5. Base EC2 & User Data Testing](#5-base-ec2--user-data-testing)
    * [6. Launch Template](#6-launch-template)
    * [7. Target Group](#7-target-group)
    * [8. Application Load Balancer](#8-application-load-balancer)
    * [9. Auto Scaling Group](#9-auto-scaling-group)
4. [Final Results](#-final-results)
5. [Docker & User Data](#-docker--user-data)
6. [Security Considerations](#-security-considerations)
7. [Contact](#-contact)

---

## ✅ Features

- **Elastic Environment:** Horizontal scaling with Auto Scaling Group.
- **File Persistence:** Shared persistent storage using **Amazon EFS**.
- **Managed Database:** High-performance persistence with **Amazon RDS (MySQL)**.
- **Traffic Distribution:** Seamless load balancing via **Elastic Load Balancer (ALB)**.
- **Zero-Touch Deployment:** Automated setup via **User Data** initialization scripts.
- **Granular Security:** Orchestrated Security Groups following the principle of least privilege.

## 📁 Service Infrastructure

- **Custom VPC**
  - 2 Public Subnets (EC2 + Load Balancer)
  - 2 Private Subnets (EFS + RDS)
- **Amazon EC2**
  - Docker Compose running WordPress.
  - Launch Template for automated scaling.
- **Amazon RDS (MySQL)**
  - Managed database for WordPress core data.
- **Amazon EFS**
  - Shared persistent storage for `/wp-content`.
- **Elastic Load Balancer (ALB)**
  - Single entry point for external web traffic.
- **Auto Scaling Group**
  - Dynamic fleet management (Min: 1, Desired: 2, Max: 3).

## ⚙️ Configuration Steps

### 1. VPC Creation
![vpcroutes](https://github.com/ManaraMarcelo/aws-scalable-wordpress-ha/blob/main/IMAGES/vpc01.png)

- Configured public subnets for instances and Load Balancer.
- Configured private subnets for EFS and RDS.

### 2. Security Group Configuration
- **ALB SG:**
    - Inbound: All traffic -> 0.0.0.0/0
- **EC2 SG:**
    - Inbound SSH (22) -> Developer IP only.
    - Inbound HTTP (80) -> Restricted to **ALB SG**.
    - Inbound NFS (2049) -> Restricted to **EFS SG**.
- **RDS SG:**
    - Inbound MySQL (3306) -> Restricted to **EC2 SG** only.
- **EFS SG:**
    - Inbound NFS (2049) -> Restricted to **EC2 SG**.

### 3. File System (EFS) Setup
- Named and initialized within the custom VPC.
![efsName](https://github.com/ManaraMarcelo/aws-scalable-wordpress-ha/blob/main/IMAGES/efsName.png)
- Mount targets configured for private subnets using the **EFS SG**.
![efssubnets](https://github.com/ManaraMarcelo/aws-scalable-wordpress-ha/blob/main/IMAGES/efs02.png)

### 4. Managed Database (RDS) Setup
- **Engine:** MySQL (Free Tier template).
- **Connectivity:** Deployed in private subnets with access restricted to the EC2 security group.
![t3microDatabase](https://github.com/ManaraMarcelo/aws-scalable-wordpress-ha/blob/main/IMAGES/databaset3micro.png)
![vpcdatabase](https://github.com/ManaraMarcelo/aws-scalable-wordpress-ha/blob/main/IMAGES/databasevpc.png)
![databaseSG](https://github.com/ManaraMarcelo/aws-scalable-wordpress-ha/blob/main/IMAGES/databaseSG.png)

### 5. Base EC2 & User Data Testing
The instances are provisioned using a script that:
1. Installs Docker/Containerd.
2. Mounts the EFS volume.
3. Launches the WordPress container with RDS environment variables.
4. Creates a non-root 'devuser' for security.
🔗 **Script Link:** [`userData.sh`](./userData.sh)

### 6. Launch Template
- Defines the AMI (Ubuntu), Key Pair, and the automated User Data script to be used by the Auto Scaling Group.

### 7. Target Group
- Configured with a specific Health Check path to ensure application availability:
```text
/wp-admin/images/wordpress-logo.svg

```

### 8. Application Load Balancer

* Acts as the entry point, distributing traffic across public subnets to the healthy instances in the Target Group.

### 9. Auto Scaling Group

* Dynamic fleet management integrated with the Load Balancer.
* **Desired Capacity:** 2 instances.

## 💪 Final Results

The project is fully autonomous. Traffic is routed through the **Load Balancer DNS**, ensuring access even if individual instances are terminated or replaced.

**Persistence Test:** 1. Upload an image via WordPress. 2. Terminate the active instance. 3. Wait for Auto Scaling to provision a new node. 4. Verify the image is still available via the new instance. (Success = EFS & RDS working correctly).

## 🐳 Docker & User Data

The User Data script automatically prepares the environment and mounts the network file system.
🔗 **Link to `user_data.sh`:** [user_data.sh](https://www.google.com/search?q=./userData.sh)

## 🔐 Security Considerations

* **Isolation:** No EC2 instance has direct public exposure; traffic is filtered through the ALB.
* **Least Privilege:** Security Groups act as virtual firewalls between layers (Web, DB, Storage).
* **Non-Root Access:** Implementation of `devuser` for Docker operations.
* **Data Integrity:** Shared EFS ensures no data loss during horizontal scaling.

---

## 🔗 Contact

**Marcelo Manara** [LinkedIn](https://www.linkedin.com/in/marcelo-manara) | [Portfolio](https://portifolio-peach-beta.vercel.app/)
