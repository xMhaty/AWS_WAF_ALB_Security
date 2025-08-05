# 🛡️ AWS Project: Blocking Web Traffic with AWS WAF and Security Group ✅

This project demonstrates how to configure **AWS WAF** to block web traffic and secure web applications against common exploits like **SQL injection** and **cross-site scripting (XSS)**. It also includes setting up a **bastion host**, **web servers**, and an **Application Load Balancer (ALB)**.

---
## 🌐 Why This Project?
(complete here )

## 🗺️ Architecture Diagram

<img width="4093" height="2106" alt="image" src="https://github.com/user-attachments/assets/bb51448d-3c1e-42e7-91a1-d3f2f91620fc" />

---

## 🧱 Key AWS Services Used

(complete here)

---
## 🛠️ Deployment Steps

### ✅ Step 1 – : Creating a Security Group for the Load Balancer

Staying with the default VPC, I created a security groupe LoadBalancer-SG where the inound rules 
| Type               | Protocol | Port Range | Source      |
|--------------------|----------|------------|-------------|
| HTTP               | tcp      | 80         | 0.0.0.0/0   |

I also created a security groupe named `webserver-SG` where the inound rules :
| Type               | Protocol | Port Range | Source      |
|--------------------|----------|------------|-------------|
| HTTP               | tcp      | 80         | LoadBalancer-SG   |
| SSH                | TCP      | 22         | 0.0.0.0/0   |


### ✅ Step 2 – Creating the Web Servers

I launched 2 EC2 instances: `webserver-A` and `webserver-B` where they have some common Configurations:
- **AMI:** Amazon Linux 2
- **Instance Type:** t2.micro
- **Key Pair:** `myKey`
- **Auto-assign Public IP:** Enabled
- **Security Group:** `webserver-SG`

and with a kinda diffrent User Data:


| Server Name   | User Data Script |
|---------------|------------------|
| `webserver-A` | ```bash<br>#!/bin/bash<br>sudo su<br>yum update -y<br>yum install -y httpd<br>systemctl start httpd<br>systemctl enable httpd<br>echo "Welcome to Nidhal's Server 1" > /var/www/html/index.html<br>``` |
| `webserver-B` | ```bash<br>#!/bin/bash<br>sudo su<br>yum update -y<br>yum install -y httpd<br>systemctl start httpd<br>systemctl enable httpd<br>echo "Welcome to Nidhal's Server 2" > /var/www/html/index.html<br>``` |


For `webserver-A`:

```bash
#!/bin/bash
sudo su
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "Welcome to Nidhal's Server 1" > /var/www/html/index.html
```

For `webserver-B`:

```bash
#!/bin/bash
sudo su
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "Welcome to Nidhal's Server 2" > /var/www/html/index.html
```

---

## ✅ Task 4 – Creating the Load Balancer

### Create Target Group:

- **Name:** `web-server-TG`
- **Health Check Path:** `/index.html`
- **Register Targets:** Add both web servers

### Create Application Load Balancer:

- **Name:** `Web-server-LB`
- **Scheme:** Internet-facing
- **IP Type:** IPv4
- **Security Group:** `LoadBalancer-SG`
- **Listeners:** HTTP port 80 → `web-server-TG`

---

## ✅ Task 5 – Create a Bastion Host Server

1. Launch instance:

- **AMI:** Amazon Linux 2
- **Instance Type:** t2.micro
- **Key Pair:** `myKey`
- **Public IP:** Enabled
- **Security Group:** `bastion-SG`

Inbound Rule for `bastion-SG`:

| Type | Source |
|------|--------|
| SSH  | 0.0.0.0/0 |

---

## ✅ Task 6 – Testing the Load Balancer

- Get **DNS name** from `Web-server-LB`.
- Test in browser for **round-robin** response (A/B).
- SSH into **bastion-server** and test via `curl`.

---

## ✅ Task 7 – Restrict Access to Specific IPs

- Modify `LoadBalancer-SG` to allow only:
  - Your IP or Bastion IP instead of 0.0.0.0/0

- Confirm:
  - ❌ Access via browser is blocked.
  - ✅ `curl` from bastion-server still works.

---

## ✅ Task 8 – Create an IP Set

1. Go to **WAF & Shield → IP Sets → Create IP Set**
2. Configure:

- **Name:** `MyIPset`
- **Description:** IP set to block
- **Region:** US East (N. Virginia)
- **IP Address:** Your IP or Bastion IP/32

---

## ✅ Task 9 – Create a Web ACL

1. Navigate to **WAF dashboard → Web ACLs → Create Web ACL**

- **Name:** `MywebACL`
- **Region:** US East (N. Virginia)
- **Resource Type:** Regional
- **Associate ALB:** `Web-server-LB`

2. Add Rule:

- **Rule type:** IP set
- **Name:** `MywebACL-rule`
- **IP set:** `MyIPset`
- **Action:** Block

---

## ✅ Task 10 – Test the WAF

- From bastion-server, run:

```bash
curl <ALB-DNS-NAME>
```

- ✅ Expect **403 Forbidden** if IP is blocked.

---

## ✅ Task 11 – Unblock the IP

- Go to **WAF & Shield → IP Sets → MyIPset**
- Remove your IP from the list.
- Re-test access — it should now be allowed.

---

## 🎉 Conclusion

✅ Successfully blocked and managed web traffic using **AWS WAF** and **Security Groups** to protect web applications and enforce IP-based filtering.

---

### ✍️ Author

Made with 💻 by **Nidhal Labri**  
🔗 [LinkedIn](https://www.linkedin.com/in/YOUR-USERNAME)
