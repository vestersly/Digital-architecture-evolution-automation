# ADR 0001: Baseline Three-Tier Infrastructure Design

## Status
Accepted / Implemented (V1.0)

## Context
Needed a production foundation separating user-facing ingress, business computation, and persistent storage.

## Decision
Implemented a 3-tier architecture inside 3Tier-VPC (10.0.0.0/16) featuring an internet-facing ALB, Nginx reverse proxy, Python REST backend, and an isolated MariaDB database.
