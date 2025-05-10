 AWS Route 53 - Complete Guide with Hands-On

  What is Route 53?

- **Amazon Route 53** is a scalable and highly available **Domain Name System (DNS) web service**.
    
- It is used to:
    
    - Register domain names
        
    - Route traffic to AWS resources or other servers
        
    - Check the health of resources
        

## 🌐 Key Concepts in DNS

### 📌 1. Domain

- A human-readable name (e.g., `example.com`) that maps to an IP address.
    

### 📌 2. DNS

- DNS (Domain Name System) translates domain names into IP addresses.
    

### 📌 3. Name Server (NS)

- Servers that host DNS records for a domain.
    
- Route 53 provides 4 NS when you create a hosted zone.
    

### 📌 4. Hosted Zone

- A container for records that belong to a domain (like a DNS database).
    
- Two types:
    
    - **Public Hosted Zone**: For domains on the internet
        
    - **Private Hosted Zone**: For domains in your VPC
        

---

## 🛠️ Hands-On: Simulate DNS Using `/etc/hosts`

### ✅ Step 1: Pick a dummy domain (e.g., `mypractice.com`)

### ✅ Step 2: Edit `/etc/hosts` on your EC2 instance

    sudo nano /etc/hosts

Add this line:

     <your-ec2-public-ip> mypractice.com

Save & exit.

### ✅ Step 3: Serve a test page

    echo '<h1>Welcome to mypractice.com - It works!</h1>' | sudo tee    /var/www/html/index.html

### ✅ Step 4: Test it

     curl http://mypractice.com

## 🧪 Route 53 Record Types

### 1. A Record (Address Record)

- Maps domain → IPv4 address

    Name: mypractice.com
    Type: A
    Value: <EC2 Public IP>

2. CNAME Record (Canonical Name)

   Maps one domain → another domain

     Name: www.mypractice.com
     Type: CNAME
    Value: mypractice.com

3. MX Record (Mail Exchange)

   Used for email delivery

     Name: mypractice.com
     Type: MX
     Value: 10 mail.mypractice.com

4. TXT Record

   Used for domain verification / SPF records

     Name: mypractice.com
    Type: TXT
    Value: "v=spf1 include:_spf.google.com ~all"

5. NS Record (Name Servers)

    - Auto-created when hosted zone is made
    
    - Points to AWS name servers

