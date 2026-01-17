# vProfile Cloud-Native Deployment: AWS Managed Services (PaaS)

This project represents an advanced migration of the **vProfile Java Stack** from a traditional Lift & Shift model to a modern, scalable architecture using **AWS Managed Services**. By offloading management to AWS (RDS, MQ, ElastiCache), we achieve higher availability and lower operational overhead.

## 🏗️ Architecture Diagram

Below is the technical architecture showing the interaction between the application tier and the managed data services.

![Architecture Diagram](./Diagrams/awsrefactoring.png)

🏗️ Architecture

The application is deployed using a modern multi-tier architecture:

Web Tier: Amazon CloudFront & Route 53 for global delivery and DNS management.

Application Tier: AWS Elastic Beanstalk (Running Apache Tomcat) with Auto Scaling and ALB.

Backend Tier (Managed Services):

Amazon RDS: MySQL Database.

Amazon ElastiCache: Memcached for session caching.

Amazon MQ: RabbitMQ for asynchronous messaging.

Networking: Public and Private Subnets across multiple Availability Zones.

🛠️ Tech Stack & Tools

Cloud Provider: AWS (Elastic Beanstalk, RDS, Amazon MQ, ElastiCache, CloudFront, Route 53)

Application: Java Spring Boot (vProfile)

Build Tool: Maven

Security: SSL/TLS, Security Group Referencing, IAM

Database & Middleware: MySQL, RabbitMQ (AMQPS), Memcached

🚀 Key Implementation Details

1. Secure Messaging (AMQPS)
Unlike standard deployments, I enforced encryption in transit for the message broker. By using Port 5671 (AMQPS) instead of 5672, I ensured that all communication between the application and Amazon MQ is encrypted via SSL/TLS.

2. Advanced Security & Isolation
Implemented the Principle of Least Privilege using Security Group Referencing:

Backend Services: Configured to only allow inbound traffic from the Elastic Beanstalk Security Group ID.

Self-Referencing Rule: Added a rule to the backend SG allowing internal communication between data services for maintenance and integration.

3. Database Provisioning & Migration
I performed a manual data migration by launching a temporary EC2 Bastion Host. This host was used to securely tunnel into the private RDS instance to restore the db_backup.sql, ensuring that the database remains private and never exposed to the public internet.

🚦 How to Access the App

The application is served globally through Amazon CloudFront:

Navigate to your custom domain (configured via Route 53).

The request is routed through CloudFront (HTTPS) to the Elastic Beanstalk Load Balancer.

Access the live environment via: https://app.yourdomain.com (Replace with your actual domain).

📂 Project Structure

/src: Application source code and configuration.

/Diagrams: Architecture diagrams and network flow charts.

pom.xml: Maven configuration for building the WAR artifact.

INFRASTRUCTURE_DETAILS.md: Deep dive into Route 53, CloudFront, and Security Groups.

DNS_SSL_SETUP.md: AWS Route 53 & SSL Certificate Configuration Guide


## 🔒 Security Architecture (Security-in-Depth)

The project implements a strict security model:
* **SSL Termination:** Handled at the Load Balancer level for HTTPS traffic.
* **Security Group Referencing:** The backend services (RDS, MQ, Cache) do not use IP-based rules. Instead, they reference the **Elastic Beanstalk Security Group ID**, ensuring only our app tier can communicate with the data layer.
* **Encryption in Transit:** All traffic to Amazon MQ is encrypted using **TLS** over port 5671.

---

## 🚀 How to Run
1.  **Clone the repo:** `git clone ...`
2.  **Build the Artifact:** `mvn clean install`
3.  **Configure Environment:** Update `application.properties` with the AWS Endpoints.
4.  **Deploy:** Upload the `.war` file to your Elastic Beanstalk environment.