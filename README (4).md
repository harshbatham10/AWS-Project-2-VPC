# AWS Project 2 – Host a Web Server on EC2 (Ubuntu + Nginx) inside a Custom VPC

A beginner-friendly AWS project where I built my own network from scratch (VPC, subnet, internet gateway, route table, security group), launched an Ubuntu EC2 instance in it, installed **Nginx**, and served a web page that is reachable from the internet.

---

## 📌 Table of Contents
- [Project Overview](#-project-overview)
- [Architecture](#-architecture)
- [AWS Services Used](#-aws-services-used)
- [Resource Configuration](#-resource-configuration)
- [Step-by-Step Implementation](#-step-by-step-implementation)
- [Verification & Output](#-verification--output)
- [Security Notes](#-security-notes)
- [Key Learnings](#-key-learnings)
- [Cleanup](#-cleanup)

---

## 📖 Project Overview

| Item | Details |
|------|---------|
| **Goal** | Host a website on an EC2 instance using Nginx in a custom VPC |
| **Region** | US East (N. Virginia) – `us-east-1` |
| **OS** | Ubuntu Server |
| **Web Server** | Nginx |
| **Instance Type** | `t3.micro` |
| **Result** | Website accessible over the internet via the instance's public IP |

---

## 🏗 Architecture

```
                        Internet
                           │
                  ┌────────▼─────────┐
                  │ Internet Gateway │  (My-Internet-Gateway)
                  └────────┬─────────┘
                           │
        ┌──────────────────▼───────────────────────┐
        │ VPC: MY-VPC  (10.0.0.0/16)               │
        │                                          │
        │  Route Table: My-Route-Table             │
        │    10.0.0.0/16 → local                   │
        │    0.0.0.0/0   → Internet Gateway        │
        │                                          │
        │  ┌────────────────────────────────────┐  │
        │  │ Public-Subnet (10.0.0.0/20)        │  │
        │  │                                    │  │
        │  │  ┌──────────────────────────────┐  │  │
        │  │  │ EC2: My-Webserver (t3.micro) │  │  │
        │  │  │ Ubuntu + Nginx               │  │  │
        │  │  │ SG: MY-Security-Group        │  │  │
        │  │  └──────────────────────────────┘  │  │
        │  └────────────────────────────────────┘  │
        └──────────────────────────────────────────┘
```

---

## ☁️ AWS Services Used

- **Amazon VPC** – isolated virtual network
- **Subnet** – public subnet inside the VPC
- **Internet Gateway (IGW)** – connects the VPC to the internet
- **Route Table** – sends internet-bound traffic to the IGW
- **Security Group** – virtual firewall for the instance
- **Amazon EC2** – virtual server hosting the website

---

## ⚙️ Resource Configuration

| Resource | Name | Key Settings |
|----------|------|--------------|
| VPC | `MY-VPC` | CIDR `10.0.0.0/16`, DNS resolution enabled |
| Subnet | `Public-Subnet` | CIDR `10.0.0.0/20`, AZ `us-east-1a` |
| Internet Gateway | `My-Internet-Gateway` | Attached to `MY-VPC` |
| Route Table | `My-Route-Table` | `0.0.0.0/0` → Internet Gateway, associated with `Public-Subnet` |
| Security Group | `MY-Security-Group` | Inbound: HTTP (80), SSH (22) |
| EC2 Instance | `My-Webserver` | `t3.micro`, Ubuntu, public IPv4 enabled |

### Security Group – Inbound Rules

| Type | Protocol | Port | Purpose |
|------|----------|------|---------|
| HTTP | TCP | 80 | Allow visitors to open the website |
| SSH | TCP | 22 | Remote login to the server |

---

## 🛠 Step-by-Step Implementation

### 1. Create the VPC
Created `MY-VPC` with CIDR block `10.0.0.0/16`.

![image alt](https://github.com/harshbatham10/AWS-Project-2-VPC/blob/088f48051ed8832448fbae4bb21727e503d8d22d/Screenshot%20(46).png)

### 2. Create the Public Subnet
Created `Public-Subnet` (`10.0.0.0/20`) inside `MY-VPC`.

![image alt](https://github.com/harshbatham10/AWS-Project-2-VPC/blob/088f48051ed8832448fbae4bb21727e503d8d22d/Screenshot%20(47).png)

### 3. Create and Attach an Internet Gateway
Created `My-Internet-Gateway` and attached it to `MY-VPC` (state: **Attached**).

![image alt](https://github.com/harshbatham10/AWS-Project-2-VPC/blob/088f48051ed8832448fbae4bb21727e503d8d22d/Screenshot%20(48).png)

### 4. Create the Route Table and Add an Internet Route
Created `My-Route-Table` and added a route sending `0.0.0.0/0` to the Internet Gateway.

![image alt](https://github.com/harshbatham10/AWS-Project-2-VPC/blob/088f48051ed8832448fbae4bb21727e503d8d22d/Screenshot%20(49).png)

### 5. Associate the Route Table with the Subnet
Associated `My-Route-Table` with `Public-Subnet` so the subnet becomes truly public.

![image alt](https://github.com/harshbatham10/AWS-Project-2-VPC/blob/088f48051ed8832448fbae4bb21727e503d8d22d/Screenshot%20(52).png)

### 6. Create the Security Group
Created `MY-Security-Group` allowing HTTP (80) and SSH (22).

![image alt](https://github.com/harshbatham10/AWS-Project-2-VPC/blob/088f48051ed8832448fbae4bb21727e503d8d22d/Screenshot%20(51).png)

### 7. Launch the EC2 Instance
Launched `My-Webserver` (Ubuntu, `t3.micro`) in `Public-Subnet` with `MY-Security-Group` and a public IP. Status checks: **3/3 passed**.

![EC2 Instance](https://github.com/harshbatham10/AWS-Project-2-VPC/blob/088f48051ed8832448fbae4bb21727e503d8d22d/Screenshot%20(50).png)

### 8. Connect via SSH and Install Nginx
Connected to the instance over SSH, then ran:

```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

Nginx is **active (running)** and **enabled** (starts automatically on reboot).

![image alt](https://github.com/harshbatham10/AWS-Project-2-VPC/blob/088f48051ed8832448fbae4bb21727e503d8d22d/Screenshot%20(54).png)

### 9. Create the Web Page
Replaced the default page with a custom `index.html`:

```bash
sudo nano /var/www/html/index.html
```

```html
<!DOCTYPE html>
<html>
<head>
  <title>AWS Project 2</title>
</head>
<body>
  <h1>Welcome to My AWS Web Server</h1>
  <p>This website is hosted on Ubuntu EC2 using Nginx.</p>
  <h2>Project 2 Completed</h2>
</body>
</html>
```

---

## ✅ Verification & Output

Opened the instance's public IPv4 address in a browser (`http://<EC2-Public-IP>`) and the page loaded successfully.

![image alt](https://github.com/harshbatham10/AWS-Project-2-VPC/blob/088f48051ed8832448fbae4bb21727e503d8d22d/Screenshot%20(56).png)

> **Note:** The public IP changes if the instance is stopped and started. Use an **Elastic IP** for a fixed address.

---

## 🔒 Security Notes

This is a learning project. For a real deployment, I would:

- Restrict **SSH (22)** to **my own IP** instead of `0.0.0.0/0`
- Remove any unnecessary rules (such as a Custom TCP rule on port `0`)
- Add **HTTPS (443)** with a TLS certificate (e.g., Let's Encrypt or ACM + load balancer)
- Keep the OS updated (`sudo apt update && sudo apt upgrade`)
- Never commit `.pem` key files or credentials to GitHub

---

## 📚 Key Learnings

- A subnet is **public** only when its route table has a route to an **Internet Gateway**
- Security Groups are **stateful** and act as a firewall at the instance level
- How **VPC, subnet, IGW, route table, and security group** work together
- Installing and managing a service with `systemctl` (start / enable / status)
- Hosting a static website on Nginx running on EC2

---

## 🧹 Cleanup

To avoid unexpected charges, delete resources when done, in this order:

1. Terminate the EC2 instance
2. Delete the Security Group
3. Detach and delete the Internet Gateway
4. Delete the Subnet and Route Table
5. Delete the VPC

---

## 👤 Author

**Harsh**
AWS Cloud Practitioner / Fresher – learning by building hands-on projects.
