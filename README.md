# vProfile Application: Advanced Migration to AWS Managed Services (PaaS)

📌 **Project Overview**

This project represents an advanced migration of the **vProfile Java Stack** from a traditional Lift & Shift model to a modern, scalable architecture. By offloading management to **AWS Managed Services** (RDS, MQ, ElastiCache), we achieve higher availability, automated patching, and lower operational overhead. The application is hosted using **AWS Elastic Beanstalk**, providing a seamless PaaS (Platform as a Service) experience.

---

## 🏗️ Architecture

The application is deployed using a modern multi-tier architecture designed for high availability and security:

* **Web Tier:** Amazon CloudFront & Route 53 for global delivery and DNS management.
* **Application Tier:** AWS Elastic Beanstalk (Running Apache Tomcat) with Auto Scaling and Application Load Balancer (ALB).
* **Backend Tier (Managed Services):**
    * **Amazon RDS:** MySQL Database.
    * **Amazon ElastiCache:** Memcached for session caching.
    * **Amazon MQ:** RabbitMQ for asynchronous messaging.
* **Networking:** Public and Private Subnets across multiple Availability Zones.

### Architecture Diagram
![Architecture Diagram](./Diagrams/awsrefactoring.png)

---

## 🛠️ Tech Stack & Tools

* **Cloud Provider:** AWS (Elastic Beanstalk, RDS, Amazon MQ, ElastiCache, CloudFront, Route 53)
* **Application Framework:** Java Spring Boot (vProfile)
* **Build Tool:** Maven
* **Security:** SSL/TLS, Security Group Referencing, IAM
* **Database & Middleware:** MySQL, RabbitMQ (AMQPS), Memcached

---

## 🚀 Key Implementation Details

### 1. Secure Messaging (AMQPS)
Unlike standard deployments, I enforced encryption in transit for the message broker. By using **Port 5671 (AMQPS)** instead of 5672, I ensured that all communication between the application and Amazon MQ is encrypted via SSL/TLS.

### 2. Advanced Security & Isolation
Implemented the **Principle of Least Privilege** using Security Group Referencing:
* **Backend Services:** Configured to only allow inbound traffic from the **Elastic Beanstalk Security Group ID**.
* **Self-Referencing Rule:** Added a rule to the backend SG allowing internal communication between data services for maintenance and integration.
* **SSL Termination:** Handled at the Load Balancer level for HTTPS traffic.

### 3. Database Provisioning & Migration
I performed a manual data migration by launching a temporary **EC2 Bastion Host**. This host was used to securely tunnel into the private RDS instance to restore the `db_backup.sql`, ensuring that the database remains private and never exposed to the public internet.

---

## 🚦 How to Access the App

The application is served globally through Amazon CloudFront:
1.  **DNS:** Navigate to your custom domain (configured via **Route 53**).
2.  **Traffic Flow:** The request is routed through **CloudFront (HTTPS)** to the Elastic Beanstalk Load Balancer.
3.  **Live Environment:** Access via: `https://app.yourdomain.com` *(Replace with your actual domain)*.

---

## 📂 Project Structure

* **/src:** Application source code and configuration.
* **/Diagrams:** Architecture diagrams and network flow charts.
* **pom.xml:** Maven configuration for building the WAR artifact.

### 📖 Detailed Documentation
For more technical details, please refer to:
* [Infrastructure Details](./INFRASTRUCTURE_DETAILS.md) - Deep dive into Security Groups and Ports.
* [DNS & SSL Setup](./DNS_SSL_SETUP.md) - Route 53 & SSL Certificate Configuration Guide.

---

## ⚙️ How to Run
1.  **Clone the repo:** `git clone https://github.com/SalmaEasa/vprofile-repo-name.git`
2.  **Build the Artifact:** Run `mvn clean install` to generate the `.war` file.
3.  **Configure Environment:** Update `src/main/resources/application.properties` with your AWS Managed Endpoints.
4.  **Deploy:** Upload the `.war` file to your AWS Elastic Beanstalk environment.