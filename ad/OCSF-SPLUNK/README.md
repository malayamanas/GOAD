# OCSF Lab for Splunk

## Description

The OCSF-SPLUNK lab is a specialized environment designed for learning and practicing Splunk deployment, OCSF (Open Cybersecurity Schema Framework) data collection, and security monitoring in a multi-platform environment.

## Lab Architecture

This lab consists of 5 virtual machines in a **standalone (non-domain)** configuration:

### Linux VMs
1. **SPLUNK-IDX** (Ubuntu 22.04) - Splunk Indexer & Search Head
   - Combined Indexer and Search Head deployment
   - IP: {{ip_range}}.32
   - Resources: 2 vCPUs, 4GB RAM
   - OS: Ubuntu 22.04 (minimal)

2. **SPLUNK-HF** (Ubuntu 22.04) - Splunk Heavy Forwarder
   - Heavy Forwarder for data processing and routing
   - IP: {{ip_range}}.33
   - Resources: 1 vCPU, 2GB RAM
   - OS: Ubuntu 22.04 (minimal)

3. **WS-LINUX** (Ubuntu 22.04) - Linux Workstation with Splunk Universal Forwarder
   - Endpoint for Linux log collection
   - IP: {{ip_range}}.35
   - Resources: 1 vCPU, 1GB RAM

4. **KALI** (Kali Linux) - Security Testing Platform
   - Pentesting and security analysis tools
   - IP: {{ip_range}}.36
   - Resources: 2 vCPUs, 2GB RAM

### Windows VMs
5. **WS-WIN** (Windows 11) - Windows Workstation with Splunk Universal Forwarder
   - Endpoint for Windows log collection
   - IP: {{ip_range}}.34
   - Resources: 2 vCPUs, 4GB RAM
   - Standalone (not domain-joined)

## Total Resource Requirements

### Compute & Memory
- **Minimum**: 8 vCPUs, 13GB RAM
- **Recommended**: 10+ vCPUs, 16GB+ RAM for better performance

### Storage
- **Minimum**: 120 GB free disk space
- **Recommended**: 150-200 GB free disk space

#### Storage Breakdown:
- **Vagrant Boxes** (downloaded once, shared across VMs):
  - Ubuntu 22.04: ~4 GB
  - Windows 11: ~25 GB
  - Kali Linux: ~12 GB
  - **Subtotal**: ~41 GB

- **VM Disks** (dynamically allocated):
  - SPLUNK-IDX: ~8 GB (grows with indexed data)
  - SPLUNK-HF: ~6 GB
  - WS-WIN: ~31 GB
  - WS-LINUX: ~5.5 GB
  - KALI: ~18 GB
  - **Subtotal**: ~68.5 GB

- **Overhead** (snapshots, metadata): ~10-20 GB

## System Access

### Linux Systems
- **SSH Access**: vagrant / vagrant (default)
- **Sudo Password**: vagrant

### Windows Systems
- **Local Admin**: vagrant / vagrant (default Vagrant credentials)
- **RDP Access**: Available on WS-WIN

### Splunk Access
- **Splunk Web**: http://{{ip_range}}.32:8000
- **Default Credentials**: admin / changeme (Splunk default - change on first login)

## Use Cases

This lab is ideal for:
- Learning Splunk deployment architecture (Indexer, Search Head, Heavy Forwarder, Universal Forwarder)
- Practicing OCSF data collection and normalization
- Testing cross-platform log collection (Windows + Linux)
- Security monitoring and threat hunting exercises
- Developing Splunk apps and dashboards
- Testing detection rules and use cases

## Installation

<<<<<<< HEAD
```bash
# Using VirtualBox
./goad.sh -t install -l OCSF-SPLUNK -p virtualbox

# Using VMware
**For detailed installation instructions, see [INSTALLATION.md](INSTALLATION.md)**

### Quick Start

```bash
# Check prerequisites
./goad.sh -t check -l OCSF-SPLUNK -p virtualbox

# Install the lab
./goad.sh -t install -l OCSF-SPLUNK -p virtualbox

# OR with VMware
./goad.sh -t install -l OCSF-SPLUNK -p vmware

# With custom IP range
./goad.sh -t install -l OCSF-SPLUNK -p virtualbox -ip 192.168.100
```

### Prerequisites

- **Vagrant** 2.0+ with plugins: vagrant-reload, winrm, winrm-fs, winrm-elevated
- **VirtualBox** 6.0+ OR **VMware Fusion/Workstation**
- **Ansible** 2.12+ (installed via GOAD)
- **Python** 3.8+ (3.11+ recommended)
- **Disk Space**: 120-150 GB free
- **RAM**: 13 GB minimum (16 GB+ recommended)
- **CPU**: 8+ vCPUs

## Notes

- **No Active Directory**: This is a standalone lab without domain services - focuses purely on Splunk infrastructure
- **Minimal Configuration**: All VMs use minimal resource allocations
- **Linux-based Splunk**: Splunk Indexer and Heavy Forwarder run on Ubuntu for reduced resource usage
- **Manual Splunk Installation**: Splunk software installation and configuration should be done via Ansible playbooks or manually
- **Cross-platform Log Collection**: Mix of Linux and Windows endpoints for comprehensive logging scenarios
- **Security Reduced**: This is a lab environment - security is intentionally reduced for learning purposes

## Future Enhancements

- Add Ansible playbooks for automated Splunk deployment
- Configure OCSF data sources and normalization
- Add pre-configured dashboards and alerts
- Include sample attack scenarios for detection testing
