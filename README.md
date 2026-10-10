# Digital Architecture Evolution & Automation (DAEA)

> **Engineering Scalable, Secure, Resilient, and Automated AWS Cloud Infrastructure**

## Overview
Digital Architecture Evolution & Automation (DAEA) documents the progressive transition of a traditional three-tier AWS deployment into a high-availability, self-healing, automated enterprise cloud platform.

## Architecture Evolution
* **V1.0 - Foundation**: 3-Tier VPC with Public ALB, Nginx Web tier, Python App tier, and MariaDB.
* **V2.0 - Network Segmentation & CI/CD**: Isolated `DevOps-VPC` connected via non-overlapping VPC Peering (`pcx-046d26afb8efdb7fe`), validated by automated Jenkins pipelines.
* **V3.0 - Automation & Pipeline as Code**: Version-controlled Jenkinsfile in GitHub, scheduled health monitoring, and alerting hooks.
* **V4.0 (Planned) - High Availability**: Decoupled Web & App Auto Scaling Groups with an Internal ALB.

## Verified Results
* **ALB Health**: HTTP 200 OK via Public Load Balancer.
* **Peering Route**: Private ingress to App Server `10.0.4.162:8080` without internet exposure.
* **Full Data Flow**: Real-time MariaDB record retrieval verified end-to-end in 5.4 seconds.
