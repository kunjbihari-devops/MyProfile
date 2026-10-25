# 🚀 Enterprise 100+ Application Migration Runbook & Cutover Playbook

> **Author**: Enterprise Cloud & DevSecOps Practice Lead  
> **Target Scope**: 100+ Heterogeneous Enterprise Applications (Dev, UAT, Production)  
> **Topology**: Hybrid On-Premises Data Center ➔ Multi-Subscription Azure Landing Zones  
> **Non-Functional Mandates**: Zero Data Loss, High Availability, Compliant, Cost-Optimized, Governed  

---

## 📑 Table of Contents
1. [Migration Strategy & The 6 R's Rationalization](#1-migration-strategy--the-6-rs-rationalization)
2. [End-to-End Discovery & Assessment Phase](#2-end-to-end-discovery--assessment-phase)
3. [Pre-Migration Checklist](#3-pre-migration-checklist)
4. [T-Minus Cutover Execution Timeline](#4-t-minus-cutover-execution-timeline)
5. [Automated Rollback Playbook](#5-automated-rollback-playbook)
6. [Post-Migration & 14-Day Hypercare Protocol](#6-post-migration--14-day-hypercare-protocol)

---

## 1. Migration Strategy & The 6 R's Rationalization

Every workload in the enterprise application portfolio was classified according to the 6 R's framework to balance speed to value, business risk, and long-term architectural health:

| Rationalization Pattern | Target Workload Profile | Target Azure Services | Percentage of Portfolio | Key Rationale |
| :--- | :--- | :--- | :--- | :--- |
| **Rehost (Lift & Shift)** | COTS software, complex legacy Windows/Linux monoliths with rigid dependencies | Azure VMs, VMSS, Azure Disk Storage | 35% | Speed of data center exit; minimal code touch required. |
| **Replatform (Lift & Reshape)** | Standard web apps, internal portals, relational databases (SQL Server, Oracle) | Azure App Service, Azure SQL Managed Instance, Azure Database for PostgreSQL | 40% | Offload OS patching, automated backups, automated HA without rewriting code. |
| **Refactor / Re-architect** | Core differentiator customer-facing APIs, microservices architectures | Azure Kubernetes Service (AKS), Azure Container Apps, Cosmos DB | 15% | High elasticity, containerized CI/CD, microservice scale. |
| **Repurchase (Drop & Shop)** | Legacy exchange servers, self-hosted bug trackers, legacy HR tools | Microsoft 365, SaaS Jira/ServiceNow, Workday | 5% | Focus engineering bandwidth on core revenue systems. |
| **Retain** | Applications slated for decommission within 6-12 months | On-premises (isolated VLAN) | 3% | Avoid wasted migration investment on depreciating software. |
| **Retire** | Zombie servers, dev environments for defunct initiatives, orphaned reporting tools | Decommissioned | 2% | Instant license and hosting cost reduction. |

---

## 2. End-to-End Discovery & Assessment Phase

### A. Automated Tooling Matrix
- **Azure Migrate Appliance**: Deployed on-premises Hyper-V / VMware clusters for agentless dependency mapping, performance profiling (CPU, RAM, IOPS, Network Throughput).
- **Service Map / Log Analytics**: Network dependency mapping discovering hidden cross-application API calls, hardcoded IP configurations, and database dependencies.
- **Data Migration Assistant (DMA)**: Database compatibility assessments detecting deprecated features and breaking syntax before migrating to Azure SQL.

### B. Wave Planning & Grouping Rules
1. **Wave 0 (Foundation)**: Hub VNet, Azure Firewall, ExpressRoute, Entra ID PIM, Core Shared Log Analytics.
2. **Wave 1 (Low-Risk Internal Services)**: Internal wikis, dev/testing clusters, batch reporting tools (Validating pipeline & cutover mechanics).
3. **Wave 2 (Business Enablers - Dev & UAT)**: Staging tiers for customer-facing systems, shared middleware, messaging queues.
4. **Wave 3 (Tier-2 Production Workloads)**: Standard enterprise apps with scheduled weekend maintenance windows.
5. **Wave 4 (Tier-1 Mission-Critical Systems)**: High-transaction databases, core financial ledgers, 24/7 public APIs with near-zero downtime cutovers.

---

## 3. Pre-Migration Checklist

```
[ ] 1. Network & Routing Verification
    ├── ExpressRoute private peering operational with BGP routes propagated.
    ├── Spoke VNet peering established with Gateway Transit enabled.
    ├── UDR route tables attached to spoke subnets with 0.0.0.0/0 pointing to Azure Firewall IP.
    └── Azure Private DNS Zones linked to all Spoke VNets; private endpoint FQDNs resolve internally.

[ ] 2. Governance & Security Clearance
    ├── Target Subscriptions placed in designated Management Groups (Prod/UAT).
    ├── Azure Policy initiatives applied (CIS Benchmarks, Tagging, Allowed Regions).
    ├── Microsoft Defender for Cloud agents auto-provisioned on target resources.
    └── Just-in-Time (JIT) access policies established via Entra ID PIM for cutover engineers.

[ ] 3. Data Replication & Synchronization
    ├── Initial seed replication completed via Azure Site Recovery (ASR) / Azure DMS.
    ├── Delta replication lag monitored; RPO verified to be under 60 seconds.
    └── Target disk encryption keys (CMEK) validated in Azure Key Vault.

[ ] 4. Operational & CAB Approvals
    ├── Change Advisory Board (CAB) approval confirmed with Change Ticket #.
    ├── Application Owner, Security Lead, and Executive Sponsor formal sign-off recorded.
    ├── Customer/User notification dispatched 7 days and 24 hours prior to maintenance window.
    └── DNS Time-to-Live (TTL) reduced to 300 seconds (5 minutes) across external & internal DNS.
```

---

## 4. T-Minus Cutover Execution Timeline

```
T-Minus 4h ─── Pre-flight Bridge Opens (All Leads, SRE, DBAs, App Owners)
T-Minus 2h ─── Final Validation of Target Landing Zone Health & Network Connectivity
T-Minus 1h ─── Source Application Enters Maintenance Mode (Read-Only)
T-00:00    ─── MAINTENANCE WINDOW INITIATED
T + 00:15  ─── Stop Source Services; Flush Connection Pools; Final DB Delta Sync
T + 00:30  ─── Data Integrity Verification (Checksum comparison & row count checks)
T + 00:45  ─── Promote Target Database to Primary Read-Write
T + 01:00  ─── Start Azure Application Compute (VMSS / App Services / AKS Pods)
T + 01:15  ─── Internal Technical Smoke Tests (Health endpoints, auth, DB read/write)
T + 01:30  ─── DNS Cutover: Point Traffic to Azure Application Gateway / Front Door
T + 01:45  ─── Business & UAT Validation (Application Owner live test cases)
T + 02:00  ─── Sign-off Received; Maintenance Window Officially Closed
```

---

## 5. Automated Rollback Playbook

If critical blockers arise during cutover that cannot be resolved within 45 minutes of the maintenance window:

1. **Rollback Decision Trigger**: Any fatal data corruption, unresolvable latency spike (> 300% baseline), or primary transaction failure.
2. **Step 1 - Traffic Reversion**: Immediately point DNS / Azure Front Door back to On-Premises VIPs (takes < 5 minutes due to prior TTL reduction).
3. **Step 2 - Target Isolation**: Stop target Azure compute services to prevent ghost writes.
4. **Step 3 - Source Re-Enablement**: Remove read-only lock on on-premises database and restart source applications.
5. **Step 4 - Stakeholder Notification**: Issue incident broadcast with root-cause analysis schedule.

---

## 6. Post-Migration & 14-Day Hypercare Protocol

- **War Room Support**: Dedicated 24/7 high-priority bridge staffed by Cloud Architects, Network Engineers, and Application Leads for the first 72 hours.
- **Log Analytics Telemetry Monitoring**: Continuous inspection of HTTP 5xx errors, database connection pool exhaustion, and CPU/memory throttling.
- **Daily Performance & Health Sign-Off**: 09:00 AM daily executive standup reviewing incident count, response times, and system stability.
- **FinOps Optimization Pass**:
  - Decommission source on-premises workloads and release hypervisor storage.
  - Review initial 14-day CPU/RAM utilization and apply 1-3 year Azure Reserved Instances (RIs) or Savings Plans.
  - Delete unattached migration staging disks and snapshot replicas.
