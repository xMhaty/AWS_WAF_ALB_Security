# 🛡️ AWS Project: Setting Up ALB with AWS WAF to Block SQL Injection, Geo Location and Query String

This project introduces the use of an **Application Load Balancer (ALB)** to distribute traffic across two EC2 instances with advanced security features using **AWS WAF**.

---
## 🌐 Why This Project?
This project is designed to simulate a real-world scenario where application security and scalability are top priorities. It allows you to learn how to deploy an **AWS WAF Web ACL** to block requests based on geolocation, SQL injection attempts, and specific query strings. By configuring two EC2 instances and placing them behind an Application Load Balancer, you not only gain insights into scalable architectures but also understand how AWS WAF interacts with Elastic Load Balancing to protect against common web exploits. AWS WAF enables the creation of custom rules to filter traffic and block malicious actors while following a cost-effective, pay-as-you-go pricing model.

---
## 🗺️ Architecture Diagram

<img width="4093" height="2106" alt="image" src="https://github.com/user-attachments/assets/d97a86f4-b9f7-43b3-93ab-d1b9c29c212c" />

---

## 🧱 Key AWS Services Used

- **Amazon EC2** – To host the web servers.
- **Application Load Balancer (ALB)** – To distribute incoming traffic across the two EC2 instances.
- **AWS WAF** – To apply web access control rules and block malicious traffic.
- **Security Groups** – To control inbound and outbound traffic to the instances and load balancer.
- **Target Groups** – To register EC2 instances for load balancing.

---
## 🛠️ Deployment Steps

### ✅ Step 1 –  Creating a Security Group for the Load Balancer

Staying with the default VPC, I created a security group `MyWebserverSG` with the following inbound rules:

| Type   | Protocol | Port Range | Source    |
|--------|----------|------------|-----------|
| HTTP   | TCP      | 80         | 0.0.0.0/0 |
| HTTPS  | TCP      | 443        | 0.0.0.0/0 |
| SSH    | TCP      | 22         | 0.0.0.0/0 |

### ✅ Step 2 – Launch EC2 Instances

I launched two EC2 instances: `MyEC2Server1` and `MyEC2Server2`. They share the following configurations:
- AMI Amazon Linux 2
- Instance Type: t2.micro
- Key Pair: myKey
- Auto-assign Public IP: Enabled
- Security Group: `MyWebserverSG`

With the following user data scripts:

**MyEC2Server1**

```bash
#!/bin/bash
sudo su
yum update -y
yum install httpd -y
systemctl start httpd
systemctl enable httpd
echo "<html><h1> Welcome to Nidhal's Server 1 </h1></html>" >> /var/www/html/index.html
````

**MyEC2Server2**

````bash
#!/bin/bash
sudo su
yum update -y
yum install httpd -y
systemctl start httpd
systemctl enable httpd
echo "<html><h1> Welcome to Nidhal's Server 2 </h1></html>" >> /var/www/html/index.html
````

### ✅ Step 3 – Creating the Load Balancer

First, I created a **Target Group** named `MyWAFTargetGroup` and registered both EC2 instances as targets.

Then, I created an **Application Load Balancer** named `MyWAFLoadBalancer` with the following settings:
- **Scheme**: Internet-facing  
- **Security Group**: `MyWebserverSG`  
- **Listener**: HTTP (Port 80) forwarding traffic to `MyWAFTargetGroup`  

<img width="1634" height="480" alt="25" src="https://github.com/user-attachments/assets/c0e0198d-0882-4acb-b2be-013170e85b8f" />


### ✅ Step 4 - Testing the Load Balancer

Using the DNS name of the load balancer, I tested round-robin access between the two servers via a browser.

<img width="1918" height="178" alt="28" src="https://github.com/user-attachments/assets/a9f4c3f5-6ad4-44a1-a96d-547d359cafe5" />

✅ As seen above, traffic is successfully balanced between both EC2 instances.

**SQL innjection test :** 
Example: `http://<ELB DNS>/product?item=securitynumber'+OR+1=1--`

<img width="1919" height="251" alt="29" src="https://github.com/user-attachments/assets/e566949c-cbf0-444c-94e4-99dcedf83ba2" />
⚠️ At this stage, the request goes through because no WAF rules are applied yet.

**Query Strings test :** 
Example: `http://<ELB DNS>/?admin=123456`

<img width="1917" height="162" alt="image" src="https://github.com/user-attachments/assets/e12c6204-a48b-4cee-af87-38c4898e7419" />
⚠️ This request is also allowed before WAF is configured.

### ✅ Step 5 -  Creating AWS WAF Web ACL

I created a Web ACL named `MyWAFWebAcl` in the **US East (N. Virginia)** region.

I added the following **managed rule groups**:
- 🗺️ **GeoLocationRestriction** – to restrict traffic from outside Algeria  
- 🔐 **QueryStringRestriction** – to block specific patterns in query strings 
- 💣 **AWS SQL Database Rule Group** – to detect and block SQL injection attempts
  
<img width="1101" height="343" alt="image" src="https://github.com/user-attachments/assets/1cf4da58-c4bb-4a05-b840-0d94ed631174" />

### ✅ Step 6 -  Test Load Balancer with WAF Rules

I re-tested the application with SQL injection and query string payloads.

**SQL innjection test :** 

<img width="1919" height="232" alt="35" src="https://github.com/user-attachments/assets/256b58f7-c9db-425c-9a3e-af834257a5d0" />
✅ WAF blocks the SQL injection with a **403 Forbidden** response, confirming the rule works.

**Query Strings test :** 

<img width="1919" height="283" alt="34" src="https://github.com/user-attachments/assets/0292a7d1-2f25-4807-9ed1-eeb1634a91e1" />
✅ Query strings are now blocked, and the WAF correctly denies access.

✅ I successfully configured an **Application Load Balancer with AWS WAF** to restrict traffic based on **geolocation** (allowing only Algeria) and **protect against SQL injection and malicious query strings**. This architecture simulates a secure and scalable cloud-based web application environment.

---

### ✍️ Author

Made with 💻 by **Nidhal Labri**  
🔗 [LinkedIn](https://www.linkedin.com/in/YOUR-USERNAME)
