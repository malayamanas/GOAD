# OCSF-SPLUNK Lab - Complete Installation Guide

This guide provides complete step-by-step instructions for installing and configuring the OCSF-SPLUNK lab on macOS, Linux, and Windows.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [macOS Installation](#macos-installation)
3. [Linux Installation](#linux-installation)
4. [Windows Installation](#windows-installation)
5. [Testing & Verification](#testing--verification)
6. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### System Requirements

- **CPU**: 8+ vCPUs (10+ recommended)
- **RAM**: 13 GB minimum (16 GB+ recommended)
- **Disk Space**: 120 GB minimum (150-200 GB recommended)
- **Internet**: Required for downloading boxes (~41 GB) and packages

### Software Requirements

- Python 3.8 or higher (Python 3.11+ recommended)
- Vagrant 2.0+
- VirtualBox 6.0+ OR VMware Fusion/Workstation
- Ansible 2.12+ (installed automatically via GOAD)
- Git (to clone GOAD repository)

---

## macOS Installation

### Step 1: Install Homebrew (if not already installed)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Step 2: Install Required Software

```bash
# Install Vagrant
brew install --cask vagrant

# Install VirtualBox (free option)
brew install --cask virtualbox

# OR install VMware Fusion (paid, better performance)
brew install --cask vmware-fusion
brew install --cask vagrant-vmware-utility  # Required for VMware
```

**Important for VirtualBox users**: After installation, you may need to:
1. Go to **System Settings > Privacy & Security**
2. Click **Allow** for Oracle kernel extension
3. **Reboot** your Mac

### Step 3: Install Vagrant Plugins

```bash
# Required plugins for all providers
vagrant plugin install vagrant-reload
vagrant plugin install winrm
vagrant plugin install winrm-fs
vagrant plugin install winrm-elevated

# For VirtualBox users
vagrant plugin install vagrant-vbguest

# For VMware users (requires license purchase)
vagrant plugin install vagrant-vmware-desktop
```

### Step 4: Clone GOAD Repository

```bash
cd ~/
git clone https://github.com/Orange-Cyberdefense/GOAD.git
cd GOAD
```

### Step 5: Install Python Dependencies and Ansible

The GOAD script will automatically create a Python virtual environment and install all dependencies:

```bash
# First run - sets up everything
./goad.sh
```

This will:
- Create virtual environment at `~/.goad/.venv`
- Install Python dependencies (rich, psutil, Jinja2, PyYAML)
- Install Ansible (version based on Python version)
- Install required Ansible collections

### Step 6: Install Ansible Collections

If the automatic installation didn't complete, install manually:

```bash
# Activate the virtual environment
source ~/.goad/.venv/bin/activate

# Install Ansible collections
cd ~/GOAD/ansible
ansible-galaxy collection install ansible.windows --ignore-certs
ansible-galaxy collection install community.general --ignore-certs
ansible-galaxy collection install community.windows --ignore-certs
```

**Note**: The `--ignore-certs` flag is needed on macOS due to SSL certificate issues with Python 3.13+.

### Step 7: Verify Installation

```bash
cd ~/GOAD

# Check with VirtualBox
./goad.sh -t check -l OCSF-SPLUNK -p virtualbox

# OR check with VMware
./goad.sh -t check -l OCSF-SPLUNK -p vmware
```

**Expected output - all checks should show `[+]`:**
```
[+] vagrant found in PATH
[+] ansible-playbook found in PATH
[+] Ansible galaxy collection ansible.windows is installed
[+] Ansible galaxy collection community.general is installed
[+] Ansible galaxy collection community.windows is installed
[+] vagrant plugin vagrant-reload is installed
[+] VBoxManage found in PATH (or vmrun for VMware)
[+] vagrant plugin vagrant-vbguest is installed
```

---

## Linux Installation

### Step 1: Install System Dependencies

#### Ubuntu/Debian

```bash
# Update package list
sudo apt update

# Install dependencies
sudo apt install -y wget curl git python3 python3-pip python3-venv \
    sshpass lftp rsync openssh-client

# Install VirtualBox
sudo apt install -y virtualbox

# Install Vagrant
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install -y vagrant
```

#### Fedora/RHEL/CentOS

```bash
# Install dependencies
sudo dnf install -y wget curl git python3 python3-pip \
    sshpass rsync openssh-clients

# Install VirtualBox
sudo dnf install -y VirtualBox

# Install Vagrant
sudo dnf install -y dnf-plugins-core
sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
sudo dnf install -y vagrant
```

### Step 2: Install Vagrant Plugins

```bash
vagrant plugin install vagrant-reload
vagrant plugin install vagrant-vbguest
vagrant plugin install winrm
vagrant plugin install winrm-fs
vagrant plugin install winrm-elevated
```

### Step 3: Clone GOAD and Setup

```bash
cd ~/
git clone https://github.com/Orange-Cyberdefense/GOAD.git
cd GOAD

# Run setup script
./goad.sh
```

### Step 4: Install Ansible Collections

```bash
cd ~/GOAD/ansible
ansible-galaxy collection install ansible.windows
ansible-galaxy collection install community.general
ansible-galaxy collection install community.windows
```

### Step 5: Verify Installation

```bash
cd ~/GOAD
./goad.sh -t check -l OCSF-SPLUNK -p virtualbox
```

---

## Windows Installation

### Step 1: Install WSL2 (Windows Subsystem for Linux)

```powershell
# Run in PowerShell as Administrator
wsl --install -d Ubuntu-22.04
```

Reboot your computer after installation.

### Step 2: Install VirtualBox or VMware

- **VirtualBox**: Download from https://www.virtualbox.org/wiki/Downloads
- **VMware Workstation**: Download from VMware website (requires license)

### Step 3: Install Vagrant

Download and install from: https://www.vagrantup.com/downloads

### Step 4: Setup in WSL2

Open Ubuntu WSL2 terminal and follow the [Linux Installation](#linux-installation) steps.

---

## Testing & Verification

### Pre-Installation Checks

Run the check command to verify all dependencies:

```bash
cd ~/GOAD

# For VirtualBox
./goad.sh -t check -l OCSF-SPLUNK -p virtualbox

# For VMware
./goad.sh -t check -l OCSF-SPLUNK -p vmware
```

### Verify Individual Components

#### Check Vagrant

```bash
vagrant --version
# Expected: Vagrant 2.4.x or higher

vagrant plugin list
# Expected output should include:
# - vagrant-reload
# - vagrant-vbguest (for VirtualBox)
# - vagrant-vmware-desktop (for VMware)
# - winrm
# - winrm-fs
# - winrm-elevated
```

#### Check VirtualBox

```bash
VBoxManage --version
# Expected: 7.x.x or 6.x.x
```

#### Check VMware (if using)

```bash
vmrun -T fusion list  # macOS
vmrun -T ws list      # Linux/Windows
# Should run without errors
```

#### Check Python and Ansible

```bash
source ~/.goad/.venv/bin/activate

python --version
# Expected: Python 3.8.x or higher

ansible --version
# Expected: ansible [core 2.12.x] or [core 2.18.x]

ansible-galaxy collection list
# Should show:
# - ansible.windows
# - community.general
# - community.windows
```

### Installation Test

Perform a test installation:

```bash
cd ~/GOAD

# Using VirtualBox
./goad.sh -t install -l OCSF-SPLUNK -p virtualbox

# OR using VMware
./goad.sh -t install -l OCSF-SPLUNK -p vmware
```

**What happens during installation:**

1. **Instance Creation** (~1 minute)
   - Creates workspace directory: `~/GOAD/workspace/<instance_id>/`
   - Generates Vagrantfile and inventory files

2. **Box Downloads** (~20-60 minutes depending on internet speed)
   - Ubuntu 22.04: ~1.5 GB download, ~4 GB extracted
   - Windows 11: ~7-9 GB download, ~25 GB extracted
   - Kali Linux: ~3-4 GB download, ~12 GB extracted

3. **VM Provisioning** (~10-20 minutes)
   - Creates and starts 5 VMs
   - Configures network interfaces
   - Sets up basic OS configuration

4. **Ansible Provisioning** (~5-15 minutes)
   - Runs build.yml playbook
   - Runs security.yml playbook
   - Configures firewall and basic settings

**Total Time**: 30-90 minutes (first run)

### Post-Installation Verification

#### Check VM Status

```bash
cd ~/GOAD
./goad.sh -t status -l OCSF-SPLUNK -p virtualbox
```

Expected output:
```
SPLUNK-IDX    running (virtualbox)
SPLUNK-HF     running (virtualbox)
WS-WIN        running (virtualbox)
WS-LINUX      running (virtualbox)
KALI          running (virtualbox)
```

#### SSH into Linux VMs

```bash
cd ~/GOAD/workspace/<instance_id>

# Test SSH to each Linux VM
vagrant ssh SPLUNK-IDX
# Once logged in, verify:
hostname  # Should show: splunk-idx
exit

vagrant ssh SPLUNK-HF
vagrant ssh WS-LINUX
vagrant ssh KALI
```

#### RDP to Windows VM

**From macOS/Linux:**
```bash
# Install RDP client if needed
# macOS: Microsoft Remote Desktop from App Store
# Linux: sudo apt install remmina

# Connect to WS-WIN
# IP: 192.168.56.34
# Username: vagrant
# Password: vagrant
```

**From Windows:**
```cmd
mstsc /v:192.168.56.34
```

#### Check Network Connectivity

```bash
# SSH to any VM and ping others
vagrant ssh SPLUNK-IDX

# Ping other VMs
ping -c 3 192.168.56.33  # SPLUNK-HF
ping -c 3 192.168.56.34  # WS-WIN
ping -c 3 192.168.56.35  # WS-LINUX
ping -c 3 192.168.56.36  # KALI
```

### Verification Checklist

Use this checklist to confirm successful installation:

- [ ] All 5 VMs are running
- [ ] Can SSH to all Linux VMs (SPLUNK-IDX, SPLUNK-HF, WS-LINUX, KALI)
- [ ] Can RDP to Windows VM (WS-WIN)
- [ ] VMs can ping each other
- [ ] VMs have internet connectivity (test with `ping 8.8.8.8`)
- [ ] Sufficient disk space remaining (check with `df -h`)

---

## Troubleshooting

### Common Issues

#### Issue 1: SSL Certificate Errors (macOS)

**Symptoms:**
```
ERROR! Unknown error when attempting to call Galaxy at 'https://galaxy.ansible.com/api/': <urlopen error [SSL: CERTIFICATE_VERIFY_FAILED]
```

**Solution:**
```bash
source ~/.goad/.venv/bin/activate
ansible-galaxy collection install ansible.windows --ignore-certs
ansible-galaxy collection install community.general --ignore-certs
ansible-galaxy collection install community.windows --ignore-certs
```

#### Issue 2: VirtualBox Kernel Extension Blocked (macOS)

**Symptoms:**
```
VBoxManage: error: VirtualBox kernel modules are not loaded
```

**Solution:**
1. Go to **System Settings > Privacy & Security**
2. Scroll down and click **Allow** next to Oracle message
3. Reboot your Mac
4. Try again

#### Issue 3: Vagrant Plugin Installation Fails

**Symptoms:**
```
ERROR: Failed to build gem native extension
```

**Solution:**
```bash
# Install Ruby development tools
# macOS:
xcode-select --install

# Ubuntu/Debian:
sudo apt install build-essential ruby-dev

# Then retry plugin installation
vagrant plugin install <plugin-name>
```

#### Issue 4: Out of Disk Space

**Symptoms:**
```
No space left on device
```

**Solution:**
```bash
# Check available space
df -h

# Clean up old Vagrant boxes
vagrant box prune

# Remove unused Docker images (if you have Docker)
docker system prune -a

# On macOS, empty trash
rm -rf ~/.Trash/*
```

#### Issue 5: VM Won't Start

**Symptoms:**
```
VBoxManage: error: Failed to create the host-only adapter
```

**Solution:**
```bash
# macOS: Recreate network interfaces
sudo /Library/Application\ Support/VirtualBox/LaunchDaemons/VirtualBoxStartup.sh restart

# Linux: Reload VirtualBox kernel modules
sudo modprobe vboxdrv
sudo modprobe vboxnetadp
sudo modprobe vboxnetflt
```

#### Issue 6: Windows VM Timeout

**Symptoms:**
```
Timed out while waiting for the machine to boot
```

**Solution:**
```bash
# Increase timeout in Vagrantfile
# Edit: ~/GOAD/workspace/<instance_id>/Vagrantfile
# Add: config.vm.boot_timeout = 600

# Or increase VM resources
# Edit the Vagrantfile and increase :mem value
```

#### Issue 7: Ansible Playbook Failures

**Symptoms:**
```
TASK [xxx] **** FAILED
```

**Solution:**
```bash
# Re-run specific playbook
cd ~/GOAD
./goad.sh

# In interactive mode:
load <instance_id>
provision build.yml
```

### Getting Help

If you encounter issues not covered here:

1. **Check GOAD Documentation**: https://orange-cyberdefense.github.io/GOAD/
2. **GOAD GitHub Issues**: https://github.com/Orange-Cyberdefense/GOAD/issues
3. **GOAD Troubleshooting Guide**: https://orange-cyberdefense.github.io/GOAD/troobleshoot/

### Useful Commands for Debugging

```bash
# Check GOAD logs
tail -f ~/.goad/.venv/logs/*.log

# Check Vagrant logs
cd ~/GOAD/workspace/<instance_id>
vagrant up --debug

# Check VM console (VirtualBox)
VBoxManage showvminfo <VM_NAME> | grep "State"

# Check Ansible inventory
cat ~/GOAD/workspace/<instance_id>/inventory

# Test Ansible connectivity
cd ~/GOAD/ansible
ansible all -i ../workspace/<instance_id>/inventory -m ping
```

---

## Next Steps After Installation

Once your lab is successfully installed and verified:

1. **Install Splunk Software** (not included by default)
   - Download Splunk Enterprise for SPLUNK-IDX
   - Download Splunk Heavy Forwarder for SPLUNK-HF
   - Download Splunk Universal Forwarder for WS-WIN and WS-LINUX

2. **Configure OCSF Data Collection**
   - Set up OCSF data sources
   - Configure normalization pipelines
   - Create custom schemas

3. **Create Snapshots** (recommended)
   ```bash
   ./goad.sh -t snapshot -l OCSF-SPLUNK -p virtualbox
   ```

4. **Practice Security Monitoring**
   - Generate sample attacks from KALI
   - Monitor logs in Splunk
   - Create detection rules
   - Build dashboards

For detailed Splunk installation and OCSF configuration, see the main README.md.
