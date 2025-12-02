# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GOAD (Game Of Active Directory) is a pentest Active Directory lab project designed to provide vulnerable AD environments for practicing attack techniques. It supports multiple lab configurations (GOAD, GOAD-Light, SCCM, NHA, MINILAB) and multiple infrastructure providers (VirtualBox, VMware, Proxmox, Azure, AWS, Ludus).

**IMPORTANT**: This is intentionally vulnerable infrastructure for security research and penetration testing practice. Never deploy on the internet or reuse these configurations in production environments.

## Commands

### Setup and Installation

```bash
# Initial setup (creates Python venv in ~/.goad/.venv and installs dependencies)
./goad.sh

# Check dependencies before installation
./goad.sh -t check -l GOAD -p virtualbox

# Install a specific lab with provider
./goad.sh -t install -l GOAD -p virtualbox

# Install with specific IP range
./goad.sh -t install -l GOAD -p virtualbox -ip 192.168.10

# Install with extensions
./goad.sh -t install -l GOAD -p virtualbox -e elk -e guacamole
```

### Interactive Console Mode

```bash
# Enter interactive mode (recommended)
./goad.sh

# Common interactive commands:
check                              # Verify dependencies
install                            # Create and provision lab with current settings
set_lab <lab>                      # Set lab (GOAD/GOAD-Light/SCCM/NHA/MINILAB)
set_provider <provider>            # Set provider (virtualbox/vmware/aws/azure/proxmox/ludus)
set_ip_range <range>               # Set IP range (e.g., 192.168.56)
list                               # List lab instances
load <instance_id>                 # Load a specific instance
status                             # Show lab status
start/stop                         # Control lab VMs
provision <playbook>               # Run specific Ansible playbook
provision_lab                      # Run full lab provisioning
```

### Instance Management

```bash
# Run provisioning only on existing instance
./goad.sh -t install -i <instance_id> -a

# Run specific playbook on instance
./goad.sh -t install -i <instance_id> -r <playbook.yml>

# Other tasks
./goad.sh -t start/stop/restart/destroy/status/snapshot/reset -i <instance_id>
```

### Development with Ansible

```bash
# Run Ansible playbooks directly (from ansible/ directory)
cd ansible
ansible-playbook -i ../workspace/<instance_id>/inventory ../ansible/<playbook>.yml

# Install Ansible requirements
ansible-galaxy install -r requirements.yml
```

## Architecture

### High-Level Structure

GOAD uses a factory pattern with pluggable providers and provisioners:

1. **Lab Manager** (goad/lab_manager.py): Central orchestrator managing labs, instances, and current settings
2. **Provider Layer**: Creates VMs via Vagrant (VirtualBox/VMware) or Terraform (Azure/AWS/Proxmox) or Ludus API
3. **Provisioner Layer**: Configures VMs via Ansible (local/remote/runner/docker/vm modes)
4. **Instance System**: Each installation creates a workspace instance with provider-specific files

### Key Components

**Main Entry Point**: `goad.py` - Python cmd.Cmd interactive console that parses arguments and delegates to LabManager

**Provider Factory** (goad/provider/provider_factory.py):
- Providers dynamically imported based on Dependencies flags
- Vagrant providers: VirtualboxProvider, VmwareProvider, VmwareEsxiProvider
- Terraform providers: AzureProvider, AwsProvider, ProxmoxProvider
- Ludus provider: LudusProvider

**Provisioner Factory** (goad/provisioner/provisioner_factory.py):
- `local`: Run ansible via subprocess (default for VirtualBox/VMware/Proxmox)
- `runner`: Run ansible via ansible-runner library
- `remote`: Run ansible through SSH jumpbox (default for Azure/AWS)
- `docker`: Run ansible in Docker container
- `vm`: Run ansible on local VM jumpbox

**Lab Definitions** (ad/\<lab\>/):
- `data/config.json`: Lab configuration (domains, users, computers, vulnerabilities)
- `data/inventory`: Main Ansible inventory (hosts and groups)
- `providers/<provider>/`: Provider-specific files (Vagrantfile, Terraform .tf files, inventory overrides)
- `files/`: Lab-specific files to deploy

**Extensions** (extensions/\<extension\>/):
- Optional add-ons (ELK, Exchange, Guacamole, Wazuh, WS01)
- Each has ansible/install.yml playbook and optional data/config.json

**Workspace** (workspace/\<instance_id\>/):
- Created per installation
- Contains provider files, generated inventory, and state

**Playbooks** (ansible/):
- Executed sequentially per playbooks.yml
- Each lab has specific playbook list (e.g., build.yml → ad-servers.yml → ad-parent_domain.yml → ...)
- Retry mechanism: 3 attempts per playbook before failure

### Data Flow

1. User runs `./goad.sh -t install -l GOAD -p virtualbox`
2. Config loads from globalsettings.ini + args
3. LabManager creates LabInstance in workspace/
4. Provider (VirtualboxProvider) generates Vagrantfile and runs `vagrant up`
5. For cloud providers: Provisioner syncs source to jumpbox and installs ansible there
6. Provisioner runs playbooks from playbooks.yml sequentially
7. Each playbook loads data from ad/\<lab\>/data/config.json
8. Ansible inventory merges: lab inventory → provider inventory → extension inventory → globalsettings.ini

### Inventory Override Order

Ansible variables are overridden in this order (last wins):
1. `ad/<lab>/data/inventory` (lab defaults)
2. `workspace/<instance_id>/inventory` (provider-specific, from `ad/<lab>/providers/<provider>/inventory`)
3. `workspace/<instance_id>/inventory_<extension>` (extension-specific)
4. `globalsettings.ini` (user global settings)

### Dependencies System

`goad/dependencies.py` manages conditional imports based on Python environment:
- Checks for provider libraries (vagrant, terraform, azure, aws, proxmox)
- Checks for provisioner dependencies (ansible, ansible-runner, docker)
- Enables/disables features with flags like `Dependencies.vmware_enabled`

## Code Patterns

### Adding a New Lab

1. Create `ad/<LAB_NAME>/` with structure matching TEMPLATE
2. Add config.json with AD structure
3. Create provider directories with inventory and provider files
4. Add entry to playbooks.yml with playbook sequence
5. Register in goad/labs.py

### Adding a New Provider

1. Create provider class in `goad/provider/<type>/<provider>.py`
2. Inherit from base Provider class
3. Implement: install(), start(), stop(), destroy(), status(), get_ip_range()
4. Add to provider_factory.py with dependency check
5. Create provider inventory template in each lab's `providers/<provider>/` directory

### Adding a New Extension

1. Create `extensions/<extension>/` directory
2. Add `ansible/install.yml` playbook
3. Add `ansible.cfg` with roles_path to main roles
4. Optional: data/config.json, inventory, provider-specific files
5. Register in lab's extension list

### Provisioning Flow

All playbooks follow this pattern:
1. Load data.yml (reads ad/\<lab\>/data/config.json)
2. Execute role-based tasks
3. Extensions load their own config.json in install.yml

Playbooks are in ansible/ directory and called sequentially per playbooks.yml.

## Important Notes

### Python Version Handling

- Python >= 3.8 required
- Python < 3.11: uses requirements.yml (ansible-core 2.12.6)
- Python >= 3.11: uses requirements_311.yml (ansible-core 2.18.0)
- Virtual env created in ~/.goad/.venv by goad.sh

### WinRM and Ansible

- Default: uses SSL (port 5986)
- Override in globalsettings.ini if needed:
  ```
  ansible_winrm_transport=basic
  ansible_port=5985
  ```

### Jumpbox Usage

Azure and AWS providers use jumpbox pattern:
- Source synced to jumpbox via rsync
- Ansible runs from jumpbox to reach private VMs
- prepare_jumpbox command installs dependencies on jumpbox
- sync_source_jumpbox command syncs code changes

### Testing Changes

When modifying provisioning:
1. Test on MINILAB (2 VMs) or GOAD-Light (3 VMs) first
2. Use `provision <playbook>` to test individual playbooks
3. Use `provision_lab_from <playbook>` to resume from specific point
4. Create snapshots before major changes

### File Locations

- Main Python code: goad/
- Ansible playbooks: ansible/
- Ansible roles: ansible/roles/
- Lab definitions: ad/
- Extensions: extensions/
- Packer templates: packer/
- Documentation: docs/mkdocs/
- Workspace instances: workspace/ (generated)
