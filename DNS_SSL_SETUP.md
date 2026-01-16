# AWS Route 53 & SSL Certificate Configuration Guide

This document outlines the process of delegating DNS management from a registrar to AWS Route 53 and provisioning an SSL certificate via AWS Certificate Manager (ACM).

## 1. DNS Configuration (Route 53 & Namecheap Integration)

* **Hosted Zone Creation**: Created a Public Hosted Zone in **AWS Route 53** using the domain `example.site`.
* **Name Server Extraction**: Extracted the four unique **Name Server (NS)** records provided by the Route 53 Hosted Zone.
* **Registrar Update**: Logged into the **Namecheap** dashboard and updated the Nameserver settings from "BasicDNS" to **"Custom DNS"**.
* **Finalizing Delegation**: Inserted the four AWS NS records into Namecheap to delegate DNS authority to AWS.

## 2. SSL/TLS Certificate Provisioning (AWS ACM)

* **Certificate Request**: Requested a Public SSL certificate through **AWS Certificate Manager (ACM)**.
* **Domain Coverage**: Included both the root domain (`example.site`) and a wildcard domain (`*.example.site`) to ensure full coverage for all future sub-domains.
* **Validation**: Utilized **DNS Validation** as the verification method.
* **Automated DNS Record Injection**: Used the **"Create records in Route 53"** feature in ACM to automatically populate the required CNAME validation records into the Hosted Zone.
* **Verification**: Monitored the status until it reached **"Issued"**, confirming the domain is ready for HTTPS traffic.
