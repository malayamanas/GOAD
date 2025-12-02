# Guacamole Setup Guide for OCSF-SPLUNK Lab

This guide provides detailed instructions for installing Guacamole and manually configuring connections to all VMs in the OCSF-SPLUNK lab.

## Table of Contents

1. [Install Guacamole Extension](#install-guacamole-extension)
2. [Access Guacamole Web Interface](#access-guacamole-web-interface)
3. [Configure Connections](#configure-connections)
4. [Testing Connections](#testing-connections)
5. [Advanced Configuration](#advanced-configuration)
6. [Troubleshooting](#troubleshooting)

---

## Install Guacamole Extension

### Prerequisites

- OCSF-SPLUNK lab must be installed and VMs running
- At least 1 GB additional RAM available
- 5 GB additional disk space

### Step 1: Load Your Instance

```bash
cd ~/GOAD
./goad.sh

# In the interactive console:
list
```

You should see your OCSF-SPLUNK instance. Note the **Instance ID** (looks like `78fed0-ocsf-splunk-vmware`).

### Step 2: Load the Instance

```bash
# In the GOAD console:
load <instance-id>
# Example: load 78fed0-ocsf-splunk-vmware
```

### Step 3: Install Guacamole Extension

```bash
# In the GOAD console:
install_extension guacamole
```

**What this does:**
1. Creates a new Ubuntu 22.04 VM named GUACAMOLE
2. Assigns IP: 192.168.56.52
3. Installs Apache Guacamole (web-based remote desktop gateway)
4. Installs MySQL database for connection storage
5. Installs Apache Tomcat as the application server

**Installation time**: 10-20 minutes

**Expected output:**
```
[*] Start install extension
[*] Instance vagrantfile created
[*] Launch providing
...
[*] Provision extension done in XX:XX:XX
```

### Step 4: Verify Guacamole VM is Running

```bash
# In the GOAD console:
status
```

You should see:
```
GUACAMOLE    running (vmware)
SPLUNK-IDX   running (vmware)
SPLUNK-HF    running (vmware)
WS-WIN       running (vmware)
WS-LINUX     running (vmware)
KALI         running (vmware)
```

---

## Access Guacamole Web Interface

### Step 1: Open Web Browser

Open your web browser and navigate to:

```
http://192.168.56.52:8080/guacamole
```

**Note**: If you changed your IP range during installation, use:
```
http://<your-ip-range>.52:8080/guacamole
```

### Step 2: Login

**Default Credentials:**
- Username: `guacadmin`
- Password: `ohmygoadchangeme`

**If you customized the password:**
- Check `/Users/apple/GOAD/guacamole.yml` for your `guacadmin_password`

### Step 3: Change Default Password (Recommended)

1. After logging in, click **guacadmin** (top right)
2. Click **Settings**
3. Click **Preferences**
4. Under **Change Password**:
   - Old password: `ohmygoadchangeme`
   - New password: (your secure password)
   - Confirm password: (your secure password)
5. Click **Update Password**

---

## Configure Connections

### Overview

You'll create 5 connections for the OCSF-SPLUNK lab:

| VM Name | IP | Protocol | Port | Purpose |
|---------|-----|----------|------|---------|
| WS-WIN | 192.168.56.34 | RDP | 3389 | Windows 11 Workstation |
| SPLUNK-IDX | 192.168.56.32 | SSH | 22 | Splunk Indexer & Search Head |
| SPLUNK-HF | 192.168.56.33 | SSH | 22 | Splunk Heavy Forwarder |
| WS-LINUX | 192.168.56.35 | SSH | 22 | Linux Workstation |
| KALI | 192.168.56.36 | SSH | 22 | Kali Linux Security Tools |

---

### Connection 1: WS-WIN (Windows 11 - RDP)

#### Step 1: Navigate to Connections

1. Click **guacadmin** (top right)
2. Click **Settings**
3. Click **Connections** tab
4. Click **New Connection**

#### Step 2: Edit Connection - General Settings

**Name:** `WS-WIN - Windows 11 Workstation`

**Location:** `ROOT` (default)

**Protocol:** `RDP` (select from dropdown)

#### Step 3: Edit Connection - Parameters

**Network Section:**

| Field | Value |
|-------|-------|
| Hostname | `192.168.56.34` |
| Port | `3389` |

**Authentication Section:**

| Field | Value |
|-------|-------|
| Username | `vagrant` |
| Password | `vagrant` |
| Domain | (leave empty) |

**Display Section:**

| Field | Value | Notes |
|-------|-------|-------|
| Color depth | `True color (24-bit)` | Best quality |
| Width | `1920` | Or your preferred width |
| Height | `1080` | Or your preferred height |
| DPI | `96` | Standard |

**Clipboard:**

| Field | Value |
|-------|-------|
| Clipboard encoding | `UTF-8` |

**Enable clipboard** ✓ (check this box)

**Performance Section (Recommended Settings):**

| Field | Value |
|-------|-------|
| Console audio | Check ✓ |
| Desktop composition | Uncheck ☐ |
| Font smoothing | Uncheck ☐ |
| Wallpaper | Uncheck ☐ |

#### Step 4: Save Connection

Click **Save** at the bottom of the page.

---

### Connection 2: SPLUNK-IDX (Ubuntu 22.04 - SSH)

#### Step 1: Create New Connection

1. Click **Settings**
2. Click **Connections** tab
3. Click **New Connection**

#### Step 2: Edit Connection - General Settings

**Name:** `SPLUNK-IDX - Splunk Indexer & Search Head`

**Location:** `ROOT`

**Protocol:** `SSH`

#### Step 3: Edit Connection - Parameters

**Network Section:**

| Field | Value |
|-------|-------|
| Hostname | `192.168.56.32` |
| Port | `22` |

**Authentication Section:**

| Field | Value |
|-------|-------|
| Username | `vagrant` |
| Password | `vagrant` |

**Terminal Section:**

| Field | Value |
|-------|-------|
| Color scheme | `Gray on black` (or your preference) |
| Font name | `monospace` |
| Font size | `12` |
| Max scrollback size | `1000` |

**Clipboard:**

Enable clipboard ✓

**SFTP Section (Optional but Recommended):**

| Field | Value |
|-------|-------|
| Enable SFTP | Check ✓ |
| Hostname | `192.168.56.32` |
| Port | `22` |
| Username | `vagrant` |
| Password | `vagrant` |
| Root directory | `/home/vagrant` |

**This enables file upload/download via Guacamole**

#### Step 4: Save Connection

Click **Save**

---

### Connection 3: SPLUNK-HF (Ubuntu 22.04 - SSH)

#### Step 1: Create New Connection

Follow the same process as SPLUNK-IDX.

#### Step 2: Configuration

**Name:** `SPLUNK-HF - Splunk Heavy Forwarder`

**Protocol:** `SSH`

**Network:**
- Hostname: `192.168.56.33`
- Port: `22`

**Authentication:**
- Username: `vagrant`
- Password: `vagrant`

**Terminal:**
- Color scheme: `Gray on black`
- Font name: `monospace`
- Font size: `12`

**Clipboard:** Enable ✓

**SFTP (Optional):**
- Enable SFTP: ✓
- Hostname: `192.168.56.33`
- Port: `22`
- Username: `vagrant`
- Password: `vagrant`

Click **Save**

---

### Connection 4: WS-LINUX (Ubuntu 22.04 - SSH)

#### Step 1: Create New Connection

Same process as above.

#### Step 2: Configuration

**Name:** `WS-LINUX - Linux Workstation`

**Protocol:** `SSH`

**Network:**
- Hostname: `192.168.56.35`
- Port: `22`

**Authentication:**
- Username: `vagrant`
- Password: `vagrant`

**Terminal:**
- Color scheme: `Gray on black`
- Font name: `monospace`
- Font size: `12`

**Clipboard:** Enable ✓

**SFTP (Optional):**
- Enable SFTP: ✓
- Hostname: `192.168.56.35`
- Port: `22`
- Username: `vagrant`
- Password: `vagrant`

Click **Save**

---

### Connection 5: KALI (Kali Linux - SSH)

#### Step 1: Create New Connection

Same process as above.

#### Step 2: Configuration

**Name:** `KALI - Security Testing Platform`

**Protocol:** `SSH`

**Network:**
- Hostname: `192.168.56.36`
- Port: `22`

**Authentication:**
- Username: `vagrant`
- Password: `vagrant`

**Terminal:**
- Color scheme: `Green on black` (classic hacker terminal 😎)
- Font name: `monospace`
- Font size: `12`

**Clipboard:** Enable ✓

**SFTP (Optional):**
- Enable SFTP: ✓
- Hostname: `192.168.56.36`
- Port: `22`
- Username: `vagrant`
- Password: `vagrant`

Click **Save**

---

## Testing Connections

### Step 1: Return to Home

Click **guacadmin** (top right) → **Home**

### Step 2: View All Connections

You should now see 5 connections:

```
┌─────────────────────────────────────────────┐
│ WS-WIN - Windows 11 Workstation         RDP │
│ SPLUNK-IDX - Splunk Indexer & Search... SSH │
│ SPLUNK-HF - Splunk Heavy Forwarder      SSH │
│ WS-LINUX - Linux Workstation            SSH │
│ KALI - Security Testing Platform        SSH │
└─────────────────────────────────────────────┘
```

### Step 3: Test Each Connection

#### Test WS-WIN (RDP)

1. Click on **WS-WIN**
2. You should see Windows 11 desktop loading
3. Wait for desktop to appear (may take 10-30 seconds)
4. You should be logged in as **vagrant**

**If connection fails:**
- Verify WS-WIN is running: `./goad.sh -t status`
- Check RDP is enabled on Windows
- Try reconnecting

#### Test SPLUNK-IDX (SSH)

1. Click on **SPLUNK-IDX**
2. You should see a terminal prompt:
   ```
   vagrant@splunk-idx:~$
   ```
3. Test basic commands:
   ```bash
   hostname
   # Should show: splunk-idx

   ip addr show
   # Should show: 192.168.56.32

   df -h
   # Check disk space
   ```

#### Test Other SSH Connections

Repeat the same process for:
- **SPLUNK-HF** (should show `vagrant@splunk-hf:~$`)
- **WS-LINUX** (should show `vagrant@ws-linux:~$`)
- **KALI** (should show `vagrant@kali:~$`)

### Step 4: Test SFTP File Transfer (Optional)

If you enabled SFTP:

1. Connect to any Linux VM
2. Press **Ctrl+Alt+Shift** to open Guacamole menu
3. Click **Devices**
4. You should see a file browser showing `/home/vagrant`
5. You can upload/download files by dragging them

---

## Advanced Configuration

### Creating Connection Groups

To organize connections better:

1. **Settings** → **Connections**
2. Click **New Connection Group**
3. Name: `OCSF-SPLUNK Lab`
4. Click **Save**
5. Edit each connection and change **Location** to `OCSF-SPLUNK Lab`

### Creating Additional Users

If you want to create additional Guacamole users:

1. **Settings** → **Users**
2. Click **New User**
3. Fill in:
   - Username: (e.g., `analyst`)
   - Password: (secure password)
4. Click **Permissions** tab
5. Grant access to specific connections
6. Click **Save**

### SSH Key Authentication (More Secure)

Instead of using passwords, you can use SSH keys:

1. Generate SSH key on Guacamole VM:
   ```bash
   vagrant ssh GUACAMOLE
   ssh-keygen -t rsa -b 4096
   ```

2. Copy public key to target VMs:
   ```bash
   ssh-copy-id vagrant@192.168.56.32
   ssh-copy-id vagrant@192.168.56.33
   ssh-copy-id vagrant@192.168.56.35
   ssh-copy-id vagrant@192.168.56.36
   ```

3. In Guacamole connection settings:
   - Remove password
   - Set **Private key** to the contents of `~/.ssh/id_rsa`

### Custom Color Schemes

For SSH connections, you can customize terminal colors:

**Available color schemes:**
- Black on white
- Gray on black
- Green on black (Matrix style)
- White on black
- Custom (define your own)

### Recording Sessions (Optional)

You can record all sessions:

1. Edit connection → **Screen Recording**
2. Enable recording path: `/var/lib/guacamole/recordings`
3. Recording name: `${GUAC_USERNAME}-${GUAC_DATE}-${GUAC_TIME}`
4. Enable **Create recording path** ✓

---

## Troubleshooting

### Issue 1: Cannot Access Guacamole (http://192.168.56.52:8080/guacamole)

**Symptoms:**
- Browser shows "Cannot connect to site"
- Connection timeout

**Solutions:**

1. **Check if Guacamole VM is running:**
   ```bash
   ./goad.sh
   load <instance-id>
   status
   ```

2. **Verify IP address:**
   ```bash
   vagrant ssh GUACAMOLE
   ip addr show
   # Should show 192.168.56.52
   ```

3. **Check Tomcat service:**
   ```bash
   vagrant ssh GUACAMOLE
   sudo systemctl status tomcat9
   # Should show "active (running)"
   ```

4. **Restart Tomcat if needed:**
   ```bash
   vagrant ssh GUACAMOLE
   sudo systemctl restart tomcat9
   ```

5. **Check firewall:**
   ```bash
   vagrant ssh GUACAMOLE
   sudo ufw status
   # Should be inactive or allow 8080
   ```

### Issue 2: Login Failed - Invalid Credentials

**Solutions:**

1. **Check password in config:**
   ```bash
   cat ~/GOAD/guacamole.yml | grep guacadmin_password
   ```

2. **Reset password:**
   ```bash
   vagrant ssh GUACAMOLE
   sudo mysql guacamole

   # In MySQL:
   UPDATE guacamole_user SET password_hash = UNHEX(SHA2('newpassword', 256)) WHERE username = 'guacadmin';
   exit
   ```

### Issue 3: Connection to VM Failed

**For RDP (WS-WIN):**

1. **Verify VM is running:**
   ```bash
   ./goad.sh -t status
   ```

2. **Check RDP is enabled:**
   ```bash
   vagrant ssh WS-WIN
   # Then check RDP settings in Windows
   ```

3. **Test RDP directly:**
   ```bash
   # From your Mac
   mstsc /v:192.168.56.34
   # Or use Microsoft Remote Desktop app
   ```

**For SSH (Linux VMs):**

1. **Test SSH directly:**
   ```bash
   ssh vagrant@192.168.56.32
   # Password: vagrant
   ```

2. **Check SSH service:**
   ```bash
   vagrant ssh SPLUNK-IDX
   sudo systemctl status ssh
   ```

3. **Verify network connectivity:**
   ```bash
   vagrant ssh GUACAMOLE
   ping 192.168.56.32
   ssh -v vagrant@192.168.56.32
   ```

### Issue 4: Black Screen on RDP Connection

**Symptoms:**
- Connection succeeds but shows only black screen

**Solutions:**

1. **Disconnect and reconnect**
2. **Change color depth** to 16-bit or 8-bit
3. **Disable desktop composition** in connection settings
4. **Increase connection timeout**

### Issue 5: SFTP Not Working

**Solutions:**

1. **Check SFTP is enabled** in connection settings
2. **Verify SSH access** works first
3. **Check permissions:**
   ```bash
   vagrant ssh SPLUNK-IDX
   ls -la /home/vagrant
   ```

### Issue 6: Clipboard Not Working

**Solutions:**

1. **Verify clipboard is enabled** in connection settings
2. **Use Guacamole menu**: Press **Ctrl+Alt+Shift**
3. **Copy text** through Guacamole menu → Text Input

### Issue 7: Slow Performance

**Solutions:**

1. **Reduce color depth** (use 16-bit instead of 24-bit)
2. **Disable wallpaper, font smoothing** in RDP settings
3. **Lower resolution** (use 1280x720 instead of 1920x1080)
4. **Check network latency:**
   ```bash
   ping 192.168.56.52
   ```
5. **Increase Guacamole VM resources:**
   - Edit Vagrantfile to increase RAM/CPU for GUACAMOLE VM

---

## Keyboard Shortcuts

While connected to a VM:

| Shortcut | Action |
|----------|--------|
| **Ctrl+Alt+Shift** | Open Guacamole menu |
| **Ctrl+C** / **Ctrl+V** | Copy/Paste (in SSH) |
| **Ctrl+Alt+Del** | Send Ctrl+Alt+Del to Windows VM |

**In Guacamole Menu:**

- **Clipboard**: View/edit clipboard content
- **Devices**: Access SFTP file browser
- **Settings**: Adjust scaling, input method
- **Disconnect**: Close connection

---

## Best Practices

1. **Change default passwords** immediately after setup
2. **Use SSH keys** instead of passwords for Linux VMs
3. **Enable session recording** for audit trails
4. **Create separate users** for different team members
5. **Use connection groups** to organize VMs
6. **Enable SFTP** for easy file transfer
7. **Regular backups** of Guacamole database:
   ```bash
   vagrant ssh GUACAMOLE
   mysqldump -u root -p guacamole > guacamole_backup.sql
   ```
8. **Monitor resource usage** on Guacamole VM
9. **Update Guacamole** regularly for security patches

---

## Quick Reference Card

### Access URLs

| Service | URL |
|---------|-----|
| Guacamole | http://192.168.56.52:8080/guacamole |
| Splunk Web | http://192.168.56.32:8000 |

### Default Credentials

| System | Username | Password |
|--------|----------|----------|
| Guacamole | guacadmin | ohmygoadchangeme |
| All VMs | vagrant | vagrant |
| Splunk | admin | changeme |

### VM IP Addresses

| VM | IP |
|----|-----|
| GUACAMOLE | 192.168.56.52 |
| SPLUNK-IDX | 192.168.56.32 |
| SPLUNK-HF | 192.168.56.33 |
| WS-WIN | 192.168.56.34 |
| WS-LINUX | 192.168.56.35 |
| KALI | 192.168.56.36 |

---

## Additional Resources

- **Apache Guacamole Documentation**: https://guacamole.apache.org/doc/gug/
- **GOAD Guacamole Extension**: https://orange-cyberdefense.github.io/GOAD/extensions/guacamole/
- **RDP Optimization**: https://guacamole.apache.org/doc/gug/configuring-guacamole.html#rdp
- **SSH Configuration**: https://guacamole.apache.org/doc/gug/configuring-guacamole.html#ssh

---

## Summary

You've successfully configured Guacamole with all 5 OCSF-SPLUNK lab VMs! You can now:

✅ Access all VMs through a single web interface
✅ Use RDP for Windows 11 workstation
✅ Use SSH for all Linux VMs (Splunk servers, workstation, Kali)
✅ Transfer files via SFTP
✅ Copy/paste between VMs and your host
✅ Access the lab from anywhere with a web browser

**Next Steps:**
1. Install Splunk software on the VMs
2. Configure OCSF data collection
3. Start security monitoring and testing!
