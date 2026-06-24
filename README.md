# AWS Production Architecture: Security Governance, Enterprise Observability, and FinOps

## 📌 Overview

This phase of the architecture focuses on transitioning a multi-tier AWS infrastructure into an enterprise-ready, production-grade deployment. The implementations emphasize three core pillars of the AWS Well-Architected Framework:
1. **Security & Governance:** Enforcing cryptographic isolation and organizational guardrails.
2. **Observability & Operational Excellence:** Building centralized telemetry dashboards and real-world incident response alerting mechanisms.
3. **Cost Optimization (FinOps):** Mapping financial models to infrastructure scaling lifecycles.

All administrative tasks and infrastructure provisions were executed following strict security compliance protocols using a dedicated, non-root Administrator IAM identity.

---

## 🧠 Core Technical Learning Objectives

### 1. Cryptographic Profiles & Key Management (AWS KMS)
* **AWS-Managed vs. Customer-Managed Keys (CMKs):** Evaluated architectural trade-offs. Deployed Customer-Managed Keys to gain granular control over key policies, rotation lifecycles, and cross-account access controls, bypassing the immutable limitations of default AWS-managed keys.
* **Envelope Encryption Patterns:** Mastered advanced data protection mechanics. The architecture leverages a KMS CMK to generate a plaintext and encrypted Data Key. The plaintext key encrypts raw storage blocks (EBS, RDS, S3) and is immediately destroyed from memory, while the encrypted data key is stored alongside the ciphertext.

### 2. Secret Lifecycle Management & Guardrails
* **Secrets Manager vs. SSM Parameter Store:** Implemented a dual-track configuration design. AWS Secrets Manager is utilized for high-value rotating credentials (database access tokens) due to its native API rotation support. SSM Parameter Store is utilized as a cost-effective, low-latency key-value store for static environment configurations.
* **Service Control Policies (SCPs):** Analyzed multi-account governance using AWS Organizations. SCPs establish absolute permission boundaries across the organizational tree, acting as an immutable guardrail that cannot be overridden by member account administrators or root users.

---

## 🛠️ Practical Tasks & Implementations Executed

### Task 1: Enterprise Observability Telemetry Pane
Designed and provisioned a centralized **CloudWatch Dashboard** named `Production-3Tier-Application-Telemetry`. The dashboard organizes real-time system performance data into a single pane of glass using five critical metric widgets:

* **Compute Tier:** Auto Scaling Group (ASG) `CPUUtilization` (tracked as a 5-minute average baseline).
* **Data Tier:** Amazon RDS `DatabaseConnections` (monitored to identify connection pooling bottlenecks).
* **Traffic Tier:** Application Load Balancer (ALB) `RequestCount` (tracks aggregate throughput trends).
* **Client-Side Edge Errors:** ALB `HTTPCode_Target_4XX_Count` (isolates broken endpoints or unauthorized scanning activities).
* **Server-Side Faults:** ALB `HTTPCode_Target_5XX_Count` (acts as an immediate indicator of backend runtime application crashes).

### Task 2: Incident Response & Automated Alerting
Configured an automated remediation and notification loop to catch architectural stress conditions:
1. **CloudWatch Alarm:** Initialized an alarm monitoring the ASG compute cluster. The state transitions to `ALARM` if the aggregate `CPUUtilization` exceeds **70% for a continuous 5-minute period**.
2. **Notification Broadcast:** Integrated the alarm state with an **Amazon SNS Topic** bound to an enterprise email distribution list.
3. **Chaos Validation Drill:** Tested end-to-end delivery by scaling down minimum group capacities and forcing load spikes, successfully validating the alerting pipeline upon receiving the delivery notification.

---

## 💰 FinOps Financial Projections (AWS Pricing Calculator)

To demonstrate cloud economics awareness, the entire 3-tier application stack was modeled under two distinct business cycles running 24/7.

| Infrastructure State | Operational Profile | Estimated Monthly Run Cost |
| :--- | :--- | :--- |
| **Peak / Production Mode** | Full Multi-AZ RDS Active, ALB Multi-Zone Routing, ASG fully scaled out with multiple computing nodes. | `$19.56 USD`
| **Lean / Scaled-Down Mode**| Off-peak/Dev cycle. ASG restricted to a minimum capacity of 1 baseline compute instance. | `$234.72 USD` 

> **FinOps Insight:** Scaling down the application's compute tier during off-peak periods or weekend development cycles yields a cost optimization savings of **60%** *[Calculate percentage change]* over standard continuous peak run times.

---
