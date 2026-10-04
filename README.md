# Nutanix AHV / CVM CLI – Practical Operations Notes

A practical reference guide for **Nutanix AHV, CVM, AOS, NCC, NCLI, ACLI, ECLI and troubleshooting commands**.

This document is intended for:

- Nutanix L1/L2 administrators
- Infrastructure engineers
- AHV administrators
- Nutanix troubleshooting
- Daily operations
- Interview preparation
- Personal learning and revision

> **Note:** Exact command availability and syntax can vary depending on the Nutanix AOS/NCC version. Always verify commands in a lab or against the applicable Nutanix documentation before using them in production.

---

## 📚 Table of Contents

- [1. Nutanix Architecture](#1-nutanix-architecture)
- [2. Basic Terminology](#2-basic-terminology)
- [3. Check Linux OS Information](#3-check-linux-os-information)
- [4. Check Cluster Status](#4-check-cluster-status)
- [5. NCC Health Checks](#5-ncc-health-checks)
- [6. ECLI Task Management](#6-ecli-task-management)
- [7. Host IPs](#7-host-ips)
- [8. CVM IPs](#8-cvm-ips)
- [9. IPMI/BMC IPs](#9-ipmibmc-ips)
- [10. allssh](#10-allssh)
- [11. hostssh](#11-hostssh)
- [12. Nutanix User Password](#12-nutanix-user-password)
- [13. AHV Host Root Password](#13-ahv-host-root-password)
- [14. NCLI](#14-ncli)
- [15. ACLI](#15-acli)
- [16. NCLI vs ACLI vs ECLI](#16-ncli-vs-acli-vs-ecli)
- [17. Agent VM](#17-agent-vm)
- [18. Change AHV Hostname](#18-change-ahv-hostname)
- [19. Nutanix Logs](#19-nutanix-logs)
- [20. Panacea](#20-panacea)
- [21. Logbay](#21-logbay)
- [22. Command Cheat Sheet](#22-command-cheat-sheet)
- [23. Troubleshooting Flow](#23-troubleshooting-flow)
- [24. Quick Memory Guide](#24-quick-memory-guide)

---

# 1. Nutanix Architecture

A basic Nutanix AHV cluster looks like:

```text
                         NUTANIX CLUSTER
                               |
              +----------------+----------------+
              |                |                |
            AHV-01           AHV-02           AHV-03
              |                |                |
            CVM-01           CVM-02           CVM-03
              |                |                |
              +----------------+----------------+
                               |
                       Nutanix Services
```

### Important relationship

```text
Physical Server
      |
      +-- AHV Host
      |
      +-- CVM
      |
      +-- Guest VMs
```

The **CVM** provides Nutanix cluster/storage management functionality while **AHV** provides the virtualization layer.

---

# 2. Basic Terminology

| Term | Meaning |
|---|---|
| AHV | Nutanix Acropolis Hypervisor |
| CVM | Controller Virtual Machine |
| AOS | Nutanix Acropolis Operating System |
| NCC | Nutanix Cluster Check |
| NCLI | Nutanix Command Line Interface |
| ACLI | Acropolis CLI |
| ECLI | Nutanix task/operation CLI |
| IPMI/BMC | Out-of-band hardware management interface |
| Prism | Nutanix management interface |
| VM | Virtual Machine |

---

# 3. Check Linux OS Information

## Command

```bash
cat /etc/os-release
```

## Purpose

Displays Linux OS information for the **current system**.

For example:

- Linux distribution
- OS version
- Version ID
- Operating system information

## Important

Do **not** treat this command alone as the exact AHV/AOS version check.

Use appropriate Nutanix/Prism version information when you need the exact:

- AOS version
- AHV version
- NCC version

---

# 4. Check Cluster Status

## Command

```bash
cluster status
```

## Purpose

Checks the status of Nutanix cluster services.

## Use when

- Cluster has an issue
- CVM services need verification
- After reboot
- After maintenance
- Before/after upgrades
- Troubleshooting cluster availability

## Basic approach

```text
Cluster Issue
     |
     v
cluster status
     |
     +-- Services UP
     |      |
     |      +-- Continue investigation
     |
     +-- Services DOWN
            |
            +-- Investigate affected service
```

---

# 5. NCC Health Checks

## What is NCC?

**NCC = Nutanix Cluster Check**

NCC provides health checks for the Nutanix environment.

---

## 5.1 Run all NCC health checks

```bash
ncc health_checks run_all
```

### Purpose

Runs the available NCC health checks.

### Use before

- AOS upgrade
- Cluster maintenance
- Hardware maintenance
- Troubleshooting
- General health validation

### Memory

```text
NCC
 |
 +-- Cluster Health
```

---

## 5.2 Hardware Health Checks

Example syntax:

```bash
ncc health_checks hardware_checks run_all
```

### Purpose

Runs hardware-related health checks.

Potential areas include:

- Hardware components
- Node health
- Hardware configuration
- Hardware-related alerts

> **Note:** NCC check names can vary between releases. If this exact check is not available, use the NCC help/list commands applicable to your installed version.

---

## 5.3 Disk Health Checks

Example syntax:

```bash
ncc health_checks disk_checks run_all
```

### Purpose

Runs disk-related health checks.

Useful for investigating:

- Disk failures
- Disk health
- Storage-related warnings
- Disk configuration problems

> **Note:** Exact NCC check/plugin names may vary by AOS/NCC version.

---

# 6. ECLI Task Management

## Check incomplete tasks

```bash
ecli task.list include_completed=false
```

### Purpose

Displays tasks that have not completed.

Useful for checking:

- VM operations
- Storage operations
- Image operations
- Migration operations
- Other background operations

---

## Include completed tasks

```bash
ecli task.list include_completed=true
```

### Memory

```text
false → Show incomplete tasks
true  → Include completed tasks
```


```bash
include_completed=false
```

✅ Correct

---

# 7. Host IPs

## Command

```bash
hostips
```

### Purpose

Displays IP information for AHV hosts.

Example:

```text
AHV-01 → 10.x.x.11
AHV-02 → 10.x.x.12
AHV-03 → 10.x.x.13
```

### Useful for

- Identifying AHV hosts
- Troubleshooting host connectivity
- Using `hostssh`
- Verifying cluster node information

---

# 8. CVM IPs

## Command

```bash
svmips
```

### Purpose

Displays CVM/SVM IP information.

Example:

```text
CVM-01 → 10.x.x.21
CVM-02 → 10.x.x.22
CVM-03 → 10.x.x.23
```

### Remember

```text
svmips  → CVM IPs
hostips → AHV Host IPs
```

---

# 9. IPMI/BMC IPs

## Command

```bash
ipmiips
```

### Purpose

Displays IPMI/BMC management IP information for physical hosts.

## Used for

- Hardware management
- Remote console
- Power operations
- Hardware troubleshooting
- Out-of-band management

### Concept

```text
Physical Server
      |
      +-- AHV
      |
      +-- CVM
      |
      +-- IPMI/BMC
             |
             +-- Hardware Management
```

---

# 10. allssh

## Command

```bash
allssh 'date'
```

### Purpose

Runs a command across the CVMs.

Example:

```bash
allssh 'hostname'
```

Conceptually:

```text
CVM-01 → hostname
CVM-02 → hostname
CVM-03 → hostname
```

## Useful for

- Checking time
- Checking hostname
- Comparing configurations
- Running diagnostics
- Checking files across CVMs

### Important

```text
allssh
  ↓
CVMs
```

---

# 11. hostssh

## Command

```bash
hostssh 'date'
```

### Purpose

Runs a command across AHV hosts.

Example:

```bash
hostssh 'hostname'
```

Conceptually:

```text
AHV-01 → hostname
AHV-02 → hostname
AHV-03 → hostname
```

### Important

```text
hostssh
   ↓
AHV Hosts
```

---

# 12. Nutanix User Password

## Command

```bash
sudo passwd Nutanix
```

### Purpose

Changes the password of the `Nutanix` user on the **current system/CVM**.


```text
sudo passwd Nutanix
→ Change Nutanix user's password on the current CVM.
```

For production environments, follow the organization's approved Nutanix password-management procedure.

---

# 13. AHV Host Root Password

You may see commands such as:

```bash
hostssh "sudo passwd root"
```

`hostssh` means:

```text
Execute command across AHV hosts
```

However, interactive password changes through a multi-host SSH wrapper should **not be treated as a standard password-management procedure**.

Follow the organization's approved Nutanix/AHV access-management process.

### Learning point

```text
hostssh
   |
   +-- Execute commands on AHV hosts
```

---

# 14. NCLI

## Command

```bash
ncli
```

### NCLI

**NCLI = Nutanix Command Line Interface**

### Purpose

Used for Nutanix infrastructure and cluster administration/querying.

Depending on the AOS release, it can be used for areas such as:

- Cluster
- Hosts
- Storage
- Containers
- Networks
- Alerts
- DNS
- NTP
- Protection
- Configuration

### Memory

```text
NCLI
  ↓
Nutanix Infrastructure
```

---

# 15. ACLI

## Command

```bash
acli
```

### ACLI

**ACLI = Acropolis CLI**

### Purpose

Primarily used for AHV/VM administration.

Common areas include:

- VM management
- VM power operations
- VM CPU/RAM configuration
- VM disks
- VM NICs
- Images
- Networks
- Subnets
- Snapshots

### Memory

```text
ACLI
  ↓
AHV
  ↓
VM / Network / Image
```

---

# 16. NCLI vs ACLI vs ECLI

This is an important Nutanix concept.

| CLI | Primary purpose |
|---|---|
| `ncli` | Nutanix cluster/infrastructure administration |
| `acli` | AHV and VM administration |
| `ecli` | Task/operation information |
| `ncc` | Cluster health checks |
| `allssh` | Execute commands across CVMs |
| `hostssh` | Execute commands across AHV hosts |

### Easy memory trick

```text
NCLI → Nutanix Infrastructure
ACLI → AHV / VM
ECLI → Tasks / Operations
NCC  → Health
allssh  → CVMs
hostssh → AHV Hosts
```

---

# 17. Agent VM

## Important clarification

An Agent VM is a **Nutanix internal/system VM associated with cluster functionality**.

It should be distinguished from normal workload VMs such as:

- Application VM
- Database VM
- Web server VM
- Middleware VM

### Important

Nutanix system VMs should not be treated like ordinary customer workload VMs.

---

# 18. Change AHV Hostname

## Command

```bash
change_ahv_hostname --host_ip=10.134.4.112 --host_name=SHIV-NTNX-DC01
```

### Purpose

Changes the hostname of an AHV host.

### Execution context

This command is run from the **CVM**.

Conceptually:

```text
CVM
 |
 | change_ahv_hostname
 |
 v
AHV Host
 |
 +-- New hostname
```

### Example

```bash
change_ahv_hostname \
--host_ip=10.134.4.112 \
--host_name=SHIV-NTNX-DC01
```

### Use when

- Correcting hostname
- Standardizing hostnames
- Preparing environment for operations/documentation

Always verify the hostname after making changes.

---

# 19. Nutanix Logs

## Log directory

```bash
cd /home/nutanix/data/logs
```

### Important

Use lowercase:

```text
/home/nutanix/data/logs
```

Linux is case-sensitive.

---

## List logs

```bash
ls
```

## List logs by modification time

```bash
ls -ltr
```

## View last 100 lines

```bash
tail -100 <logfile>
```

## Follow log in real time

```bash
tail -f <logfile>
```

## Search for errors

```bash
grep -i error <logfile>
```

## Search for warnings

```bash
grep -i warning <logfile>
```

### Troubleshooting flow

```text
Problem
   |
   v
cd /home/nutanix/data/logs
   |
   v
ls -ltr
   |
   v
Identify relevant log
   |
   +-- tail
   |
   +-- grep
   |
   v
Analyze error
```

---

# 20. Panacea

## Command

```bash
panacea_cli show_leaders
```

### Purpose

Displays leader information associated with Panacea services.

Nutanix uses distributed services across CVMs, and some services use leader-election mechanisms.

Conceptually:

```text
CVM-01 ─┐
CVM-02 ─┼── Distributed Service
CVM-03 ─┘
              |
              +-- Leader
              +-- Other Members
```

### Use when

Investigating distributed service/leader-related issues.

---

# 21. Logbay

## Command

```bash
logbay collect -s <IP>
```

Example:

```bash
logbay collect -s 10.x.x.x
```

### Purpose

Collects diagnostic information/logs for troubleshooting.

### Useful for

- Nutanix support cases
- Troubleshooting
- Diagnostic collection
- Escalation to Nutanix support

### Important

The exact Logbay options can vary depending on the installed version.

Check:

```bash
logbay --help
```

before using it in production.

---

# 22. Command Cheat Sheet

| Command | Purpose | Context |
|---|---|---|
| `cat /etc/os-release` | Linux OS information | Current system |
| `cluster status` | Check cluster services | CVM |
| `ncc health_checks run_all` | Run NCC health checks | CVM |
| `ncc ... hardware_checks ...` | Hardware health checks | CVM |
| `ncc ... disk_checks ...` | Disk health checks | CVM |
| `ecli task.list include_completed=false` | Show incomplete tasks | CVM |
| `hostips` | Show AHV host IPs | CVM |
| `svmips` | Show CVM IPs | CVM |
| `ipmiips` | Show IPMI/BMC IPs | CVM |
| `allssh 'date'` | Run command on CVMs | CVM |
| `hostssh 'date'` | Run command on AHV hosts | CVM |
| `sudo passwd Nutanix` | Change local Nutanix-user password | Current system |
| `ncli` | Cluster/infrastructure administration | CVM |
| `acli` | AHV/VM administration | CVM |
| `change_ahv_hostname` | Change AHV hostname | CVM |
| `cd /home/nutanix/data/logs` | Access Nutanix logs | CVM |
| `panacea_cli show_leaders` | Show Panacea leaders | CVM |
| `logbay collect -s <IP>` | Collect diagnostic logs | Version dependent |

---

# 23. Practical Troubleshooting Flow

When a customer reports:

> **"Nutanix cluster is having an issue."**

Use a logical troubleshooting approach.

```text
                     ISSUE REPORTED
                           |
                           v
                  +------------------+
                  | 1. Cluster Status |
                  +------------------+
                           |
                    cluster status
                           |
                           v
                  +------------------+
                  | 2. NCC Health    |
                  +------------------+
                           |
               ncc health_checks run_all
                           |
                           v
                  +------------------+
                  | 3. Active Tasks  |
                  +------------------+
                           |
          ecli task.list include_completed=false
                           |
                           v
                  +------------------+
                  | 4. Node/IP Info  |
                  +------------------+
                           |
              +------------+------------+
              |            |            |
           hostips       svmips      ipmiips
              |            |            |
              +------------+------------+
                           |
                           v
                  +------------------+
                  | 5. Check Logs    |
                  +------------------+
                           |
               /home/nutanix/data/logs
                           |
                           v
                  +------------------+
                  | 6. Service Info  |
                  +------------------+
                           |
              panacea_cli show_leaders
                           |
                           v
                  +------------------+
                  | 7. Collect Logs  |
                  +------------------+
                           |
                  logbay collect -s IP
                           |
                           v
                     ROOT CAUSE
```

---

# 24. Quick Memory Guide

## Health

```bash
cluster status
```

**Question:** Are Nutanix services healthy?

---

## NCC

```bash
ncc health_checks run_all
```

**Question:** Does the cluster pass health checks?

---

## Tasks

```bash
ecli task.list include_completed=false
```

**Question:** What operations are currently running?

---

## AHV IPs

```bash
hostips
```

**Question:** What are my AHV host IPs?

---

## CVM IPs

```bash
svmips
```

**Question:** What are my CVM IPs?

---

## IPMI

```bash
ipmiips
```

**Question:** What are my hardware management IPs?

---

## All CVMs

```bash
allssh '<command>'
```

**Question:** How do I run a command across all CVMs?

---

## All AHV Hosts

```bash
hostssh '<command>'
```

**Question:** How do I run a command across all AHV hosts?

---

## Infrastructure

```bash
ncli
```

**Think:**

> Nutanix infrastructure / cluster administration

---

## VM Management

```bash
acli
```

**Think:**

> AHV / VM / Network / Image

---

## Logs

```bash
cd /home/nutanix/data/logs
```

**Think:**

> Investigate the actual problem.

---

## Log Collection

```bash
logbay collect -s <IP>
```

**Think:**

> Collect diagnostics for troubleshooting/support.

---

#  Final Memory Map

```text
                         NUTANIX
                            |
             +--------------+--------------+
             |                             |
            AHV                           CVM
             |                             |
       Virtualization                Nutanix Services
             |                             |
             |              +--------------+--------------+
             |              |              |              |
             |             NCLI           ACLI           ECLI
             |              |              |              |
             |         Infrastructure      VM            Tasks
             |              |            Management
             |              |
             +--------------+
                            |
                      Troubleshooting
                            |
       +--------------------+--------------------+
       |                    |                    |
    cluster               NCC                  Logs
    status             health_checks       /home/nutanix/
                                            data/logs
