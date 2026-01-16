# Infrastructure, Networking & Security Details

This document provides a deep dive into the networking setup and security configurations for the vProfile AWS deployment.

---

## 🌐 1. Domain Management & DNS (Route 53)

To provide a professional entry point for the application, I integrated **Amazon Route 53** with my custom domain.

### Steps Implemented:
* **Custom Nameservers:** I updated the Nameservers (NS) at my domain registrar to point to the four unique AWS Route 53 Nameservers provided in the Hosted Zone.
* **Public Hosted Zone:** Created a public hosted zone to manage DNS records globally.
* **Alias Records:** Instead of using standard A records with IPs, I used **Alias Records** to point my domain (e.g., `app.yourdomain.com`) directly to the **CloudFront Distribution**, ensuring lower latency and seamless integration.

---

## 🚀 2. Content Delivery (Amazon CloudFront)

To optimize performance and add an extra layer of security, I deployed a **CloudFront Distribution** as the Content Delivery Network (CDN).

### Configuration:
* **Origin:** The Application Load Balancer (ALB) created by Elastic Beanstalk.
* **SSL/TLS:** Configured to redirect all HTTP traffic to **HTTPS** for secure end-to-end communication.
* **Caching Policy:** Optimized to cache static assets while forwarding dynamic requests to the Tomcat backend.
* **Edge Locations:** This setup ensures that users receive the application content from the nearest AWS Edge location, significantly reducing TTFB (Time to First Byte).

---

## 🛡️ 3. Security Groups & Port Mapping

Security is implemented at the network level using **Stateful Firewalls (Security Groups)**. The architecture follows the **Principle of Least Privilege**.

### A. Application Tier (Elastic Beanstalk SG)
* **Inbound:** * Allow **Port 80/443** from the CloudFront IP ranges (or Public if testing).
* **Outbound:** * Allow all traffic to the Backend SG.

### B. Data Tier (Shared Backend SG)
This single Security Group protects **RDS, ElastiCache, and Amazon MQ**.

| Service | Protocol | Port | Source |
| :--- | :--- | :--- | :--- |
| **Amazon RDS** | MySQL | `3306` | EB-Security-Group-ID |
| **ElastiCache** | Memcached | `11211` | EB-Security-Group-ID |
| **Amazon MQ** | AMQPS (SSL) | `5671` | EB-Security-Group-ID |



---

## 🔐 4. Port 5671 vs 5672 (The Security Choice)

In this project, I specifically enforced **Port 5671** for RabbitMQ communication. 
* **Port 5672:** Standard AMQP (Unencrypted).
* **Port 5671:** AMQPS (Encrypted via TLS/SSL).

**Implementation Note:** Amazon MQ requires TLS for its managed service. Therefore, the Application SG was configured to egress traffic over 5671, and the MQ Inbound rule was strictly limited to this port to maintain a high security posture.

---

## 🛠 Troubleshooting Commands used for Verification
To verify the connectivity within the VPC from the EC2 instances, I used:
* `telnet <endpoint> 3306` (For RDS)
* `nc -zv <mq-endpoint> 5671` (For Amazon MQ SSL)


---

## 💾 5. Database Migration & Provisioning

To initialize the application data, I performed a manual migration of the database schema and data into the **Amazon RDS** instance.

### The Migration Process:
1.  **Bastion/Jump Host:** Since the RDS instance is in a private subnet and not publicly accessible, I launched a temporary **EC2 Instance** (Ubuntu) within the same VPC.
2.  **Security Group Tunneling:** I temporarily allowed inbound MySQL traffic (Port 3306) on the RDS Security Group from the EC2 Instance's Security Group.
3.  **Data Upload:**
    * Uploaded the `db_backup.sql` from the project repository to the EC2 instance.
    * Installed the MySQL client on the EC2 instance: `apt update && apt install mysql-client git -y`.
4.  **Database Restoration:** Executed the migration command to restore the backup into the managed RDS endpoint:
    ```bash
    mysql -h <rds-endpoint> -u <admin-user> -p <db_name> < db_backup.sql
    ```
5.  **Cleanup:** After the successful migration, the EC2 instance was terminated, and the temporary security group rule was removed to maintain maximum security.

---

## 🔐 6. Self-Referencing Security Group Rule

A unique feature of the **Backend Security Group** is the inclusion of a **Self-Referencing Inbound Rule**:
* **Rule:** Allow **All Traffic** where the **Source = Backend-SG-ID**.
* **Reasoning:** This allows all data tier services (RDS, MQ, ElastiCache) to communicate with each other if needed for internal maintenance, health checks, or potential future service integrations, without exposing them to the rest of the VPC.