Advanced AWS VPC Guide Detailed Concepts with Hands-On Steps

📘 Table of Contents

1. **Overview of VPC & Key Components**
    
2. **VPC Subnet Design (Public & Private)**
    
3. **Internet Gateway & NAT Gateway**
    
4. **Network ACLs (NACLs)**
    
5. **VPC Peering**
    
6. **VPC Endpoints (S3 & DynamoDB)**
    
7. **VPC Flow Logs & CloudWatch Monitoring**
    

---

## 1. 🌐 VPC Overview

A **Virtual Private Cloud (VPC)** is an isolated section of the AWS cloud where you can launch AWS resources in a virtual network you define.

### Key Components:

- CIDR Block
    
- Subnets (Public & Private)
    
- Route Tables
    
- Internet Gateway
    
- NAT Gateway
    
- Security Groups
    
- Network ACLs
    

---

## 2. 🧱 Subnet Design

### Public vs Private Subnet:

- **Public Subnet**: Has route to Internet Gateway.
    
- **Private Subnet**: No direct route to Internet.
    

### Hands-On:

1. Create a VPC with CIDR: `10.0.0.0/16`
    
2. Add subnets:
    
    - Public Subnet: `10.0.0.0/24`
        
    - Private Subnet: `10.0.1.0/24`
        
3. Attach IGW to VPC and update route table:
    
    - Destination: `0.0.0.0/0` → Target: Internet Gateway
        
4. Create a NAT Gateway in Public Subnet and update private subnet route table:
    
    - Destination: `0.0.0.0/0` → Target: NAT Gateway
        

---

## 3. 🌍 Internet Gateway & NAT Gateway

### IGW:

- Public access to EC2 via internet
    
- Associated with Public Subnet
    

### NAT Gateway:

- Allows Private Subnet EC2s to access internet for updates
    

---

## 4. 🛡️ Network ACLs (NACLs)

### Key Points:

- Stateless: Inbound and Outbound rules configured separately
    
- Rules evaluated by number (lowest wins)
    

### Hands-On:

1. Go to NACL → Create new → Associate with subnets
    
2. Allow rules:
    
    - Inbound: Allow HTTP, HTTPS, ICMP
        
    - Outbound: Allow All or specific as needed
        
3. Delete unnecessary Deny rules with lower priority
    

---

## 5. 🔁 VPC Peering

### Purpose:

- Enable traffic between two VPCs (same or different accounts)
    

### Pre-requisite:

- **Non-overlapping CIDR blocks**
    

### Hands-On:

1. Create two VPCs:
    
    - VPC-A: `10.0.0.0/16`
        
    - VPC-B: `10.1.0.0/16`
        
2. Create Peering Connection: VPC-A requester → VPC-B accepter
    
3. Update route tables in both VPCs:
    
    - VPC-A Subnet Route: `10.1.0.0/16` → Peering Connection
        
    - VPC-B Subnet Route: `10.0.0.0/16` → Peering Connection
        
4. Test connectivity using:
    
    - Ping (Enable ICMP in Security Group)
        
    - Session Manager (Preferred for Private EC2s)
        

---

## 6. 🔒 VPC Endpoints

### Gateway Endpoints (S3, DynamoDB):

- Private access to AWS services from within VPC
    
- No NAT or IGW required
    

### Hands-On:

1. Go to VPC → Endpoints → Create Endpoint
    
2. Service: `com.amazonaws.<region>.s3`
    
3. Type: Gateway
    
4. Route Table: Select private subnet's route table
    
5. Add IAM Role to EC2:
    
    - S3: AmazonS3FullAccess
        
    - DynamoDB: AmazonDynamoDBFullAccess
        
6. Test from EC2:
    
    - `aws s3 ls`
        
    - `aws dynamodb list-tables`
        

---

## 7. 📊 VPC Flow Logs

### Purpose:

- Monitor IP traffic in/out of network interfaces
    
- Send logs to CloudWatch or S3
    

### Hands-On:

1. Go to VPC → Your VPC → Flow Logs tab
    
2. Create Flow Log:
    
    - Filter: All / Accept / Reject
        
    - Destination: CloudWatch Logs
        
    - Log Group: `/vpc1-flow-logs`
        
    - Role: Create or use `VPCFlowLogs-Cloudwatch` role
        
3. View logs in CloudWatch Logs
    

### Query Examples:

Go to CloudWatch Logs Insights:

**Rejected Traffic**

  fields @timestamp, action, srcAddr, destAddr, srcPort, destPort
  | filter action = "REJECT"
  | sort @timestamp desc
  | limit 20

**Accepted Traffic** 

fields @timestamp, action, srcAddr, destAddr
| filter action = "ACCEPT"
| sort @timestamp desc
| limit 20

**Traffic Between Two IPs**

fields @timestamp, srcAddr, destAddr, action
| filter srcAddr = "10.0.0.98" and destAddr = "10.1.0.237"
| sort @timestamp desc
| limit 20

**Traffic for Specific Port (e.g., SSH)**

fields @timestamp, srcPort, destPort, action
| filter destPort = 22
| sort @timestamp desc
| limit 20

## ✅ Summary Checklist

|Task|Status|
|---|---|
|VPC with Public and Private Subnets|✅|
|Internet & NAT Gateway Configuration|✅|
|NACL with Correct Rules|✅|
|VPC Peering & Route Table Updates|✅|
|S3 & DynamoDB VPC Endpoints|✅|
|EC2 IAM Roles for Endpoint Access|✅|
|VPC Flow Logs with CloudWatch|✅|
|Logs Querying in Logs Insights|✅|

---
