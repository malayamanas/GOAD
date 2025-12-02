# Splunk Installation Guide for OCSF-SPLUNK Lab

This guide explains how to install and configure Splunk Enterprise, Splunk Heavy Forwarder, and Splunk Universal Forwarders across all VMs in the OCSF-SPLUNK lab using Ansible automation.

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Prerequisites](#prerequisites)
3. [Installation Methods](#installation-methods)
4. [Verification](#verification)
5. [Accessing Splunk](#accessing-splunk)
6. [Data Flow](#data-flow)
7. [Troubleshooting](#troubleshooting)

---

## Architecture Overview

The OCSF-SPLUNK lab implements a complete Splunk deployment with the following components:

### Splunk Components

| Component | VM | Role | IP Address | Port |
|-----------|-----|------|------------|------|
| **Splunk Enterprise** | SPLUNK-IDX | Indexer + Search Head | 192.168.56.32 | 8000 (Web), 9997 (Receiver) |
| **Splunk Heavy Forwarder** | SPLUNK-HF | Data Processing & Routing | 192.168.56.33 | 8000 (Web), 9997 (Receiver) |
| **Universal Forwarder** | WS-WIN | Windows Log Collection | 192.168.56.34 | N/A |
| **Universal Forwarder** | WS-LINUX | Linux Log Collection | 192.168.56.35 | N/A |
| **No Forwarder** | KALI | Security Testing Platform | 192.168.56.36 | N/A |

### Data Flow

```
┌─────────────┐
│   WS-WIN    │ (Universal Forwarder)
│ Windows 11  │ Collects: Security, System, Application logs
└──────┬──────┘
       │
       │ Port 9997
       ↓
┌─────────────┐
│  WS-LINUX   │ (Universal Forwarder)
│ Ubuntu 22.04│ Collects: syslog, auth.log, audit.log
└──────┬──────┘
       │
       │ Port 9997
       ↓
┌─────────────┐
│    KALI     │ (No Forwarder)
│ Kali Linux  │ Security Testing Platform
└─────────────┘ Used for penetration testing
                NOT forwarding logs to Splunk

                                     ┌─────────────┐
                                     │  SPLUNK-HF  │ (Heavy Forwarder)
                                     │ Ubuntu 22.04│ Processes & Routes data
                                     └──────┬──────┘
                                            │
                                            │ Port 9997
                                            ↓
                                     ┌─────────────┐
                                     │ SPLUNK-IDX  │ (Indexer + Search Head)
                                     │ Ubuntu 22.04│ Indexes & Searches data
                                     └─────────────┘
                                     Web UI: http://192.168.56.32:8000
```

### Splunk Indexes

The following custom indexes are automatically created:

- **ocsf** - OCSF-formatted security data
- **windows** - Windows event logs
- **linux** - Linux system logs
- **security** - Security-related logs from all systems

---

## Prerequisites

### 1. Lab Must Be Running

Ensure all VMs are up and running:

```bash
cd ~/GOAD
./goad.sh -i <instance-id> -t status -l OCSF-SPLUNK -p <provider>
```

All VMs should show `running` status.

### 2. Basic Provisioning Complete

The `build.yml` and `security.yml` playbooks should have been run:

```bash
./goad.sh -i <instance-id> -t provision -l OCSF-SPLUNK -p <provider>
```

### 3. Network Connectivity

Verify VMs can communicate:

```bash
# SSH to any VM and ping others
vagrant ssh SPLUNK-IDX
ping -c 3 192.168.56.33  # SPLUNK-HF
ping -c 3 192.168.56.34  # WS-WIN
ping -c 3 192.168.56.35  # WS-LINUX
ping -c 3 192.168.56.36  # KALI
```

---

## Installation Methods

### Method 1: Automated Installation (Recommended)

Install all Splunk components across all VMs with a single command:

```bash
cd ~/GOAD
./goad.sh

# In GOAD console:
load <instance-id>
provision splunk.yml
```

**What this does:**
- Installs Splunk Enterprise on SPLUNK-IDX
- Installs Splunk Heavy Forwarder on SPLUNK-HF
- Installs Splunk Universal Forwarder on WS-LINUX and KALI
- Installs Splunk Universal Forwarder on WS-WIN
- Configures data forwarding: Endpoints → Heavy Forwarder → Indexer
- Creates custom indexes (ocsf, windows, linux, security)
- Configures log collection inputs
- Starts all services

**Installation time:** 15-30 minutes (depends on download speeds)

### Method 2: Component-by-Component Installation

Install specific components using tags:

#### Install Splunk Enterprise only:
```bash
cd ~/GOAD/ansible
ansible-playbook -i ../workspace/<instance-id>/inventory splunk.yml --tags splunk_enterprise
```

#### Install Splunk Heavy Forwarder only:
```bash
ansible-playbook -i ../workspace/<instance-id>/inventory splunk.yml --tags splunk_heavy_forwarder
```

#### Install Universal Forwarders (Linux) only:
```bash
ansible-playbook -i ../workspace/<instance-id>/inventory splunk.yml --tags splunk_universal_forwarder_linux
```

#### Install Universal Forwarder (Windows) only:
```bash
ansible-playbook -i ../workspace/<instance-id>/inventory splunk.yml --tags splunk_universal_forwarder_windows
```

### Method 3: Manual Installation

If automated installation fails, you can install manually:

#### SPLUNK-IDX (Manual)

```bash
vagrant ssh SPLUNK-IDX

# Download Splunk Enterprise
wget -O splunk-9.3.2-d8bb32809498-linux-2.6-amd64.deb \
  'https://download.splunk.com/products/splunk/releases/9.3.2/linux/splunk-9.3.2-d8bb32809498-linux-2.6-amd64.deb'

# Install
sudo dpkg -i splunk-9.3.2-d8bb32809498-linux-2.6-amd64.deb

# Start and accept license
sudo /opt/splunk/bin/splunk start --accept-license --answer-yes --seed-passwd changeme

# Enable boot-start
sudo /opt/splunk/bin/splunk enable boot-start -user splunk

# Configure to receive data on port 9997
sudo /opt/splunk/bin/splunk enable listen 9997 -auth admin:changeme
```

#### SPLUNK-HF (Manual)

```bash
vagrant ssh SPLUNK-HF

# Download and install Splunk Enterprise (same package as indexer)
wget -O splunk-9.3.2-d8bb32809498-linux-2.6-amd64.deb \
  'https://download.splunk.com/products/splunk/releases/9.3.2/linux/splunk-9.3.2-d8bb32809498-linux-2.6-amd64.deb'
sudo dpkg -i splunk-9.3.2-d8bb32809498-linux-2.6-amd64.deb

# Start and configure
sudo /opt/splunk/bin/splunk start --accept-license --answer-yes --seed-passwd changeme
sudo /opt/splunk/bin/splunk enable boot-start -user splunk

# Configure forwarding to indexer
sudo /opt/splunk/bin/splunk add forward-server 192.168.56.32:9997 -auth admin:changeme

# Enable receiving from Universal Forwarders
sudo /opt/splunk/bin/splunk enable listen 9997 -auth admin:changeme
```

#### WS-LINUX and KALI (Manual)

```bash
vagrant ssh WS-LINUX  # or KALI

# Download Universal Forwarder
wget -O splunkforwarder-9.3.2-d8bb32809498-linux-2.6-amd64.deb \
  'https://download.splunk.com/products/universalforwarder/releases/9.3.2/linux/splunkforwarder-9.3.2-d8bb32809498-linux-2.6-amd64.deb'

# Install
sudo dpkg -i splunkforwarder-9.3.2-d8bb32809498-linux-2.6-amd64.deb

# Start and configure
sudo /opt/splunkforwarder/bin/splunk start --accept-license --answer-yes --seed-passwd changeme
sudo /opt/splunkforwarder/bin/splunk enable boot-start -user splunk

# Configure forwarding to Heavy Forwarder
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.56.33:9997 -auth admin:changeme

# Add log file monitoring
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog -index linux -auth admin:changeme
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log -index security -auth admin:changeme
```

#### WS-WIN (Manual)

```powershell
# RDP to WS-WIN (192.168.56.34)

# Download Universal Forwarder
# Visit: https://www.splunk.com/en_us/download/universal-forwarder.html
# Or use PowerShell:
Invoke-WebRequest -Uri "https://download.splunk.com/products/universalforwarder/releases/9.3.2/windows/splunkforwarder-9.3.2-d8bb32809498-x64-release.msi" `
  -OutFile "C:\temp\splunkforwarder.msi"

# Install silently
msiexec.exe /i C:\temp\splunkforwarder.msi AGREETOLICENSE=Yes LAUNCHSPLUNK=0 SERVICESTARTTYPE=auto `
  SPLUNKUSERNAME=admin SPLUNKPASSWORD=changeme /quiet

# Configure forwarding
cd "C:\Program Files\SplunkUniversalForwarder\bin"
.\splunk.exe add forward-server 192.168.56.33:9997 -auth admin:changeme

# Add Windows Event Log monitoring
.\splunk.exe add monitor "Security" -index security -auth admin:changeme
.\splunk.exe add monitor "System" -index windows -auth admin:changeme
.\splunk.exe add monitor "Application" -index windows -auth admin:changeme

# Start service
.\splunk.exe start
```

---

## Verification

### 1. Check Service Status

#### On Linux VMs (SPLUNK-IDX, SPLUNK-HF, WS-LINUX, KALI):

```bash
# Check if Splunk service is running
sudo systemctl status Splunkd
# or
sudo /opt/splunk/bin/splunk status        # for Enterprise
sudo /opt/splunkforwarder/bin/splunk status  # for Universal Forwarder
```

#### On Windows (WS-WIN):

```powershell
Get-Service SplunkForwarder
```

### 2. Check Web Interfaces

Open in your browser:

- **Splunk Indexer**: http://192.168.56.32:8000
  - Username: `admin`
  - Password: `changeme`

- **Splunk Heavy Forwarder**: http://192.168.56.33:8000
  - Username: `admin`
  - Password: `changeme`

### 3. Verify Data Flow

In Splunk Web (http://192.168.56.32:8000):

#### Check if data is being received:

```spl
index=* | stats count by index, host
```

You should see data from all hosts (splunk-idx, splunk-hf, ws-win, ws-linux, kali).

#### Check Windows logs:

```spl
index=windows OR index=security host=ws-win
| head 20
```

#### Check Linux logs:

```spl
index=linux OR index=security host=ws-linux OR host=kali
| head 20
```

### 4. Verify Forwarder Connections

In Splunk Web (http://192.168.56.32:8000):

Go to **Settings → Forwarding and receiving → Receive data**

You should see connections from:
- 192.168.56.33 (SPLUNK-HF)

In Heavy Forwarder Web (http://192.168.56.33:8000):

Go to **Settings → Forwarding and receiving → Receive data**

You should see connections from:
- 192.168.56.34 (WS-WIN)
- 192.168.56.35 (WS-LINUX)
- 192.168.56.36 (KALI)

---

## Accessing Splunk

### Web Access

| Component | URL | Credentials |
|-----------|-----|-------------|
| Splunk Enterprise | http://192.168.56.32:8000 | admin / changeme |
| Heavy Forwarder | http://192.168.56.33:8000 | admin / changeme |

### SSH Access to Splunk CLI

```bash
# SPLUNK-IDX
vagrant ssh SPLUNK-IDX
sudo -u splunk /opt/splunk/bin/splunk help

# SPLUNK-HF
vagrant ssh SPLUNK-HF
sudo -u splunk /opt/splunk/bin/splunk help

# Universal Forwarders
vagrant ssh WS-LINUX
sudo -u splunk /opt/splunkforwarder/bin/splunk help
```

### Common Splunk CLI Commands

```bash
# Check Splunk status
sudo /opt/splunk/bin/splunk status

# Restart Splunk
sudo /opt/splunk/bin/splunk restart

# List all indexes
sudo /opt/splunk/bin/splunk list index -auth admin:changeme

# List forward servers
sudo /opt/splunk/bin/splunk list forward-server -auth admin:changeme

# List monitored inputs
sudo /opt/splunk/bin/splunk list monitor -auth admin:changeme

# Search from CLI
sudo /opt/splunk/bin/splunk search "index=linux" -auth admin:changeme
```

---

## Data Flow

### Architecture Summary

1. **Universal Forwarders** collect raw logs from endpoints:
   - WS-WIN: Windows Event Logs (Security, System, Application)
   - WS-LINUX: Syslog, auth.log, audit.log
   - KALI: System logs

2. **Heavy Forwarder** receives data from Universal Forwarders:
   - Processes and normalizes data
   - Routes to appropriate indexes
   - Forwards to Indexer

3. **Indexer** receives data from Heavy Forwarder:
   - Indexes data for fast searching
   - Provides Search Head functionality
   - Stores data in custom indexes (ocsf, windows, linux, security)

### Port Usage

| Source | Destination | Port | Protocol | Purpose |
|--------|-------------|------|----------|---------|
| WS-WIN | SPLUNK-HF | 9997 | TCP | Forward logs |
| WS-LINUX | SPLUNK-HF | 9997 | TCP | Forward logs |
| KALI | SPLUNK-HF | 9997 | TCP | Forward logs |
| SPLUNK-HF | SPLUNK-IDX | 9997 | TCP | Forward processed logs |
| Browser | SPLUNK-IDX | 8000 | HTTP | Web interface |
| Browser | SPLUNK-HF | 8000 | HTTP | Web interface |

---

## Troubleshooting

### Issue 1: Splunk Won't Start

**Symptoms:**
```
Splunk status: not running
```

**Solutions:**

```bash
# Check logs
sudo tail -f /opt/splunk/var/log/splunk/splunkd.log

# Common issues:
# 1. Port already in use
sudo netstat -tulpn | grep 8000

# 2. Insufficient disk space
df -h

# 3. Permission issues
sudo chown -R splunk:splunk /opt/splunk
```

### Issue 2: No Data Appearing in Indexer

**Symptoms:**
- Search returns no results
- No data in indexes

**Solutions:**

```bash
# 1. Check forwarder connection on SPLUNK-HF
vagrant ssh SPLUNK-HF
sudo /opt/splunk/bin/splunk list forward-server -auth admin:changeme

# Should show: 192.168.56.32:9997 (Active)

# 2. Check if indexer is listening
vagrant ssh SPLUNK-IDX
sudo netstat -tulpn | grep 9997

# 3. Check firewall (should be disabled in lab)
sudo ufw status

# 4. Test connectivity
ping 192.168.56.32
telnet 192.168.56.32 9997
```

### Issue 3: Universal Forwarder Not Sending Data

**Symptoms:**
- Forwarder shows as connected but no data

**Solutions:**

```bash
# Check what's being monitored
sudo /opt/splunkforwarder/bin/splunk list monitor -auth admin:changeme

# Check forwarder logs
sudo tail -f /opt/splunkforwarder/var/log/splunk/splunkd.log

# Verify log files exist and are readable
ls -la /var/log/syslog /var/log/auth.log

# Check forward-server connection
sudo /opt/splunkforwarder/bin/splunk list forward-server -auth admin:changeme
```

### Issue 4: Windows Forwarder Installation Fails

**Symptoms:**
- MSI installation fails
- Service won't start

**Solutions:**

```powershell
# Check if service exists
Get-Service SplunkForwarder

# Check Windows Event Logs
Get-EventLog -LogName Application -Source Splunk* -Newest 20

# Verify installation path
Test-Path "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe"

# Reinstall with logging
msiexec.exe /i splunkforwarder.msi /L*V install.log AGREETOLICENSE=Yes
```

### Issue 5: Downloads Failing (403 Forbidden)

**Symptoms:**
```
Error downloading Splunk package: HTTP 403
```

**Cause:** Splunk requires authentication for downloads.

**Solutions:**

**Option 1: Download manually to your local machine, then upload to VM**

```bash
# On your local machine:
# 1. Visit https://www.splunk.com/en_us/download.html
# 2. Create free account and download packages
# 3. Upload to VMs:

# For Linux VMs
scp -P 2232 splunk-9.3.2-*.deb vagrant@localhost:/tmp/

# For Windows
# Use RDP to copy files
```

**Option 2: Use wget with Splunk authentication (if you have credentials)**

```bash
wget --auth-no-challenge --http-user=<username> --http-password=<password> \
  "https://download.splunk.com/products/splunk/releases/9.3.2/linux/splunk-9.3.2-d8bb32809498-linux-2.6-amd64.deb"
```

### Issue 6: Ansible Playbook Fails

**Symptoms:**
- `splunk.yml` playbook fails during execution

**Solutions:**

```bash
# Check Ansible connectivity
cd ~/GOAD/ansible
ansible all -i ../workspace/<instance-id>/inventory -m ping

# Run with verbose output
ansible-playbook -i ../workspace/<instance-id>/inventory splunk.yml -vvv

# Run specific failed task
ansible-playbook -i ../workspace/<instance-id>/inventory splunk.yml --start-at-task="<task-name>"
```

---

## Next Steps

After successful installation:

1. **Configure OCSF Data Sources** - Set up OCSF schema normalization
2. **Create Dashboards** - Build monitoring dashboards in Splunk Web
3. **Set Up Alerts** - Configure alerting for security events
4. **Test Attack Scenarios** - Use KALI to generate sample attacks and detect in Splunk
5. **Install Splunk Apps** - Add apps like Splunk Security Essentials, CIM, etc.

---

## Additional Resources

- [Splunk Documentation](https://docs.splunk.com/Documentation/Splunk/latest)
- [Splunk Universal Forwarder Manual](https://docs.splunk.com/Documentation/Forwarder/latest)
- [OCSF Schema](https://schema.ocsf.io/)
- [GOAD Documentation](https://orange-cyberdefense.github.io/GOAD/)

---

## Configuration Files Reference

All Splunk configuration files created by Ansible:

### SPLUNK-IDX
- `/opt/splunk/etc/system/local/web.conf` - Web interface settings
- `/opt/splunk/etc/system/local/inputs.conf` - Receiving configuration
- `/opt/splunk/etc/system/local/outputs.conf` - Forwarding (disabled)
- `/opt/splunk/etc/system/local/indexes.conf` - Custom indexes
- `/opt/splunk/etc/system/local/server.conf` - Server settings

### SPLUNK-HF
- `/opt/splunk/etc/system/local/web.conf` - Web interface settings
- `/opt/splunk/etc/system/local/inputs.conf` - Receiving from UFs
- `/opt/splunk/etc/system/local/outputs.conf` - Forwarding to indexer
- `/opt/splunk/etc/system/local/server.conf` - Server settings

### Universal Forwarders (Linux)
- `/opt/splunkforwarder/etc/system/local/outputs.conf` - Forward to HF
- `/opt/splunkforwarder/etc/system/local/inputs.conf` - Log monitoring

### Universal Forwarder (Windows)
- `C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf`
- `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`
