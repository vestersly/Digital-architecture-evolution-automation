# Digital Architecture Evolution & Automation (DAEA)
### Engineering Scalable, Secure, Resilient, and Automated AWS Cloud Infrastructure

[![AWS](https://img.shields.io/badge/AWS-us--east--1-orange?logo=amazon-aws)](https://aws.amazon.com/)
[![CI/CD](https://img.shields.io/badge/CI%2FCD-Jenkins-red?logo=jenkins)](https://www.jenkins.io/)
[![Web%20Proxy](https://img.shields.io/badge/Proxy-Nginx-green?logo=nginx)](https://nginx.org/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## 📌 Executive Overview
**Digital Architecture Evolution & Automation (DAEA)** documents the progressive modernization of an enterprise AWS three-tier architecture—transitioning from a static single-instance deployment into a decoupled, highly available, and automated cloud infrastructure validated via continuous CI/CD health auditing.

📄 **[Read the Full Engineering Project Report (PDF)](docs/DAEA_Final_Project_Report_Revised.pdf)**

---

## 📐 Architecture Evolution

- **V1.0 — Baseline Foundation:** Monolithic three-tier VPC featuring an internet-facing Application Load Balancer (ALB), Nginx presentation tier, Python application runtime, and a dedicated MariaDB database subnet[cite: 27].
- **V2.0 — Network Segmentation & VPC Peering:** Isolated administrative workflows into a separate `DevOps-VPC` (`10.10.0.0/16`) linked to `3Tier-VPC` (`10.0.0.0/16`) via bidirectional AWS VPC Peering (`pcx-046d26afb8efdb7fe`) with dedicated route table propagation[cite: 56].
- **V3.0 — Pipeline-as-Code & Automated Auditing:** Version-controlled declarative `Jenkinsfile` executing multi-stage endpoint validation across public and private ingress routes.
- **V4.0 — High Availability & Decoupled Fleet:** Decoupled the application tier by deploying an **Internal ALB (`alb-internal-app`)**, custom Golden AMI (`ami-daea-app-v1`), and an **Auto Scaling Group (`daea-app-asg`)** spanning `us-east-1a` and `us-east-1b` with automated instance refresh and target connection draining[cite: 29, 37, 38, 41, 46].

---

## 🏗️ Architecture & Request Flow

```text
[ INTERNET TRAFFIC ]
         │
         ▼ (HTTP :80)
┌─────────────────────────────────────────────────────────────┐
│ Public ALB (alb-three-tier)                                 │
└──────────────┬───────────────────────────────┬──────────────┘
               │                               │
               ▼                               ▼
┌─────────────────────────────┐ ┌─────────────────────────────┐
│ Nginx Reverse Proxy (Tier 1)│ │ Nginx Reverse Proxy (Tier 1)│
│ Subnet: PUB-SN-A            │ │ Subnet: PUB-SN-B            │
└──────────────┬──────────────┘ └──────────────┬──────────────┘
               │                               │
               └───────────────┬───────────────┘
                               ▼ (HTTP :8080 - Internal)
┌─────────────────────────────────────────────────────────────┐
│ Internal ALB (alb-internal-app)                             │
└──────────────┬───────────────────────────────┬──────────────┘
               │                               │
               ▼                               ▼
┌─────────────────────────────┐ ┌─────────────────────────────┐
│ App Fleet (Tier 2 - ASG)    │ │ App Fleet (Tier 2 - ASG)    │
│ Subnet: Privat-SN-A         │ │ Subnet: Privat-SN-B         │
└──────────────┬──────────────┘ └──────────────┬──────────────┘
               │                               │
               └───────────────┬───────────────┘
                               ▼ (TCP :3306 - Prepared)
┌─────────────────────────────────────────────────────────────┐
│ MariaDB Database Tier (Tier 3 - Host: 10.0.6.23)            │
│ Subnet: Privat-SN-C                                         │
└─────────────────────────────────────────────────────────────┘
                               ▲
                               │ VPC Peering (pcx-046d26afb8efdb7fe)
┌──────────────────────────────┴──────────────────────────────┐
│ DevOps-VPC (CIDR: 10.10.0.0/16)                             │
│ Jenkins Controller running automated pipeline checks        │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔬 Measured & Verified Results

- **Public Ingress & Routing:** HTTP 200 OK verified across the Public ALB, with Nginx successfully serving root presentation assets (`/`) and proxying API traffic (`/api/`) across the internal boundary.
- **Private Transit over VPC Peering:** Verified bi-directional private routing from the DevOps network directly to the internal ALB hostname without exposing backend compute nodes to public ingress.
- **Fleet Self-Healing & Instance Replacement:** Terminated an active application node during testing; the Auto Scaling Group launched a replacement instance, successfully registered it with `tg-internal-app`, and returned 20/20 consecutive HTTP 200 responses during post-recovery sampling[cite: 58].
- **Rolling Instance Refresh:** Initiated an automated rolling update with 100% minimum healthy capacity, observing active target draining on legacy instances while new nodes passed health checks[cite: 46, 48].
- **Automated CI/CD Quality Gate:** Jenkins declarative pipeline completed 100% green execution across all verification stages (Build #74).

---

## 🔍 Technical Boundaries & Honest Engineering Notes

As documented in the project report[cite: 92]:
- **Application vs. Database Layer:** The Python application handler verified network pathing and returned structured JSON telemetry containing database host details[cite: 35]. Active SQL transaction execution against MariaDB was not asserted in the automated test suite[cite: 92, 95].
- **Availability Bounds:** While compute-tier node replacement was verified, whole-system high availability across database and web layers was not implemented in this phase[cite: 92, 97].
- **Resource Decommissioning:** All infrastructure—including Auto Scaling Groups, Load Balancers, Target Groups, Route Tables, and Elastic IPs—was systematically torn down to ensure an active $0.00 ongoing run rate[cite: 2, 3, 50].

---

## 🛠️ Technology Stack
- **Cloud Platform:** Amazon Web Services (VPC, Subnets, Route Tables, VPC Peering, ALB, ASG, Launch Templates, Golden AMI, EC2, CloudWatch)[cite: 37, 41, 56, 91]
- **Automation & CI/CD:** Jenkins Declarative Pipelines, GitHub SCM, Bash Shell Scripting, AWS CLI v2, PowerShell[cite: 17, 26, 51, 90]
- **Web & Operating Systems:** Nginx Reverse Proxy, Amazon Linux 2023, Systemd, Python 3[cite: 21, 35, 48]