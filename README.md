# Secure Multi-VPC Peering Architecture with Bastion Host on AWS

## 📌 Project Overview
This project demonstrates a production-grade networking infrastructure deployed on AWS, establishing a secure private connection between two isolated Virtual Private Clouds (VPCs) using **VPC Peering**. The architecture mimics a real-world corporate cloud environment where a primary staging network needs to securely communicate with an internal database or services network without exposing critical workloads to the public internet.

By utilizing a **Bastion Host (Jump Box)** in the public subnet, secure administrative access is provided to underlying isolated instances, while explicit Route Tables and Security Group policies govern the traffic flow.

---

## 🏗️ Architecture Design & Specifications

The deployment spans across the `eu-north-1` (Stockholm) region utilizing Free-Tier eligible compute resources:

### 1. VPC-1 (Production Network)
- **IPv4 CIDR Block:** `10.0.0.0/16`
- **Public Subnet:** `10.0.1.0/24` (Attached to an Internet Gateway for external SSH access)
- **Private Subnet:** `10.0.2.0/24` (Isolated local tier)

### 2. VPC-2 (Internal Services Network)
- **IPv4 CIDR Block:** `192.168.0.0/16` (Non-overlapping boundary)
- **Private Subnet:** `192.168.1.0/24` (Fully isolated; no Internet Gateway attached)

### 3. Compute Resources (EC2)
- **Bastion Host:** `t3.micro` instance deployed in VPC-1 Public Subnet (Assigned Public IPv4).
- **Private Instances:** Deployed inside VPC-1 and VPC-2 private subnets respectively with restricted inbound interfaces.

---

## 🛠️ Step-by-Step Implementation Flow

### Phase 1: Network Provisioning
1. Created **VPC-1** (`10.0.0.0/16`) using the automated wizard to instantly generate standard public/private subnet groupings, an Internet Gateway, and baseline route tables.
2. Created **VPC-2** (`192.168.0.0/16`) explicitly stripping out public subnets and edge gateways to ensure full perimeter isolation.

### Phase 2: Compute Deployment & Windows Security Handshake
1. Launched the target Linux EC2 instances mapping them to their designated subnets with shared cryptographic keypair files (`ewb-project-key.pem`).
2. Resolved high-privilege permission leakage flags on the local Windows OS by strictly hardening key boundaries via:
   ```cmd
   icacls "ewb-project-key.pem" /inheritance:r /grant:r "%username%:R"
   ```

### Phase 3: Establishing the Peering Link & Route Propagation
1. Generated a **VPC Peering Connection** request setting VPC-1 as the Requester and VPC-2 as the Accepter, instantly executing an internal handshake to activate the route tunnel.
2. Modified the route tracking parameters inside both local routing tables to enforce strict cross-VPC forwarding criteria:
   - Added destination route `192.168.0.0/16` pointing to the Peering Target ID inside VPC-1's Public Route Table.
   - Added destination route `10.0.0.0/16` pointing to the Peering Target ID inside VPC-2's Private Route Table.

### Phase 4: Firewall Configuration & Inter-VPC Connectivity Validation
1. Modified the active Inbound Security Group rules for VPC-2 (`launch-wizard-3`) to open explicit paths authorizing **All ICMP - IPv4** protocols globally or from the VPC-1 range.
2. Securely tunneled into the Bastion Host via SSH, and executed a live validation check against the hidden remote nodes:
   ```bash
   ping 192.168.133.73
   ```
3. Successfully confirmed 0% packet loss metrics with stable bidirectional routing latencies over the private backbone network.

---

## 📈 Key Technical Takeaways & Troubleshooting
- **Boundary Hardening:** Gained deep operational experience managing system barriers and working safely within Root credentials.
- **Firewall Filtering:** Understood that cross-VPC routes fail quietly unless underlying security group filters are symmetrically opened to receive ICMP ping traffic.
- **Windows Key Security:** Mastered local terminal access security controls over SSH access keys on local execution environments.

---
*Developed as part of the hands-on cloud engineering tracking assignments at **EWB Tech Academy**.*
