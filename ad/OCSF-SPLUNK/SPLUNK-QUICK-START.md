# Splunk Quick Start Guide

## Installation Command

To install all Splunk components on the OCSF-SPLUNK lab:

```bash
cd ~/GOAD
./goad.sh

# In GOAD console:
load <your-instance-id>
provision splunk.yml
```

## What Gets Installed

| VM | Component | Purpose |
|----|-----------|---------|
| SPLUNK-IDX | Splunk Enterprise 9.3.2 | Indexer + Search Head |
| SPLUNK-HF | Splunk Enterprise 9.3.2 | Heavy Forwarder |
| WS-WIN | Universal Forwarder 9.3.2 | Windows log collection |
| WS-LINUX | Universal Forwarder 9.3.2 | Linux log collection |
| KALI | None | Security testing (no forwarder) |

## Access After Installation

- **Splunk Web UI**: http://192.168.56.32:8000
  - Username: `admin`
  - Password: `changeme`

- **Heavy Forwarder UI**: http://192.168.56.33:8000
  - Username: `admin`
  - Password: `changeme`

## Data Flow

```
WS-WIN ──┐
WS-LINUX─┼──→ SPLUNK-HF ──→ SPLUNK-IDX
         │     (Port 9997)    (Port 9997)

KALI (standalone - no forwarder, used for security testing)
```

## Verify Installation

Search for data in Splunk Web:

```spl
index=* | stats count by index, host
```

You should see data from 4 hosts (splunk-idx, splunk-hf, ws-win, ws-linux) in various indexes (windows, linux, security, ocsf).

**Note:** KALI does not have Splunk Universal Forwarder installed - it's kept clean for security testing purposes.

## For More Details

See [SPLUNK-INSTALLATION.md](SPLUNK-INSTALLATION.md) for:
- Architecture details
- Manual installation steps
- Troubleshooting guide
- Configuration reference
