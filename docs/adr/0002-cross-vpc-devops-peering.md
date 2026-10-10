# ADR 0002: Cross-VPC DevOps Isolation via VPC Peering

## Status
Accepted / Implemented (V2.0)

## Context
Running CI/CD automation inside the production application VPC mixes administrative privileges with application workloads.

## Decision
Provisioned a dedicated DevOps-VPC (10.10.0.0/16) housing Jenkins, interconnected with 3Tier-VPC through VPC Peering (pcx-046d26afb8efdb7fe) with strictly scoped routing and security groups.
