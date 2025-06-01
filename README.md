# AWS-SOCLab
Step-by-step guide to setting up a NAT Gateway in AWS
# Setting Up a NAT Gateway in AWS for a Private Subnet

## Why Do We Need a NAT Gateway?

When you deploy resources in a **private subnet** in AWS, those resources **cannot access the internet by default**. This means your EC2 instances won’t be able to:

- Download packages
- Get system updates
- Reach external APIs or services

To solve this, **AWS provides a NAT Gateway**, which allows outbound internet traffic from instances in the private subnet. **It’s one-way only**—the internet **cannot initiate** a connection **into** the private subnet.

## High-Level Architecture

- **VPC** with both **public** and **private** subnets
- An **Internet Gateway (IGW)** attached to the public subnet
- A **NAT Gateway** in the public subnet
- EC2 instances in the private subnet use the NAT Gateway for internet access

---

![AWS LAB drawio](https://github.com/user-attachments/assets/68513e1a-029e-4dc6-8304-db7e111c3c33)

---

## Step-by-Step Setup

### 1. Create a VPC

1. Go to **VPC > Create VPC**
2. Select **VPC only**
3. Name: `test-vpc`
4. CIDR Block: `12.0.0.0/16`
5. Click **Create VPC**

---

### 2. Create Subnets (Public and Private)

1. Go to **Subnets > Create Subnet**
2. Select VPC: `test-vpc`

**Public Subnet**
- Name: `test-subnet-public`
- AZ: `eu-central-1a` (or your region)
- CIDR Block: `12.0.0.0/24`

**Private Subnet**
- Name: `test-subnet-private`
- AZ: `eu-central-1a`
- CIDR Block: `12.0.1.0/24`

Click **Create subnet**

---

### 3. Create and Attach an Internet Gateway

1. Go to **Internet Gateway > Create internet gateway**
2. Name: `test-igw`
3. Click **Create**
4. Select it, click **Actions > Attach to VPC**, and choose `test-vpc`

---

### 4. Create Route Tables

#### Public Route Table
1. Go to **Route Tables > Create route table**
2. Name: `test-rt-public`
3. VPC: `test-vpc`
4. Click **Create**

**Edit Routes**
- Destination: `0.0.0.0/0`
- Target: **Internet Gateway** (`test-igw`)

**Edit Subnet Associations**
- Associate with: `test-subnet-public`

#### Private Route Table
1. Name: `test-rt-private`
2. VPC: `test-vpc`
3. Click **Create**

**Edit Subnet Associations**
- Associate with: `test-subnet-private`

*No route changes yet.*

---

### 5. Create a NAT Gateway

1. Go to **NAT Gateway > Create NAT Gateway**
2. Name: `test-nat-gateway`
3. Subnet: **Public subnet** (`test-subnet-public`)
4. Allocate a new **Elastic IP**
5. Click **Create NAT Gateway**

Wait until the state is **Available**

---

### 6. Update Private Route Table with NAT Gateway

1. Go to **Route Tables > test-rt-private**
2. Edit Routes:
   - Destination: `0.0.0.0/0`
   - Target: **NAT Gateway** (`test-nat-gateway`)
3. Save Changes

---

### 7. Launch EC2 Instances

**Public EC2 Instance**
- Launch into `test-subnet-public`
- Assign public IP

**Private EC2 Instance**
- Launch into `test-subnet-private`
- No public IP

---

### Test NAT Gateway

1. SSH into the **public EC2**
2. From there, connect to the **private EC2** (using private IP)
3. On the **private EC2**, run:
   ```bash
   curl https://www.google.com
   ```
   or try installing a package:
   ```bash
   sudo apt update
   ```

If successful, your **NAT Gateway is working**.
