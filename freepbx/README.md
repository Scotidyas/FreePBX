# FreePBX Proxmox Automation

This Ansible project automates the deployment of FreePBX VMs on Proxmox infrastructure across 8 different network ranges.

## Project Structure

```
freepbx/
├── ansible.cfg                    # Ansible configuration
├── requirements.txt               # Python dependencies
├── collections/requirements.yml   # Ansible collections
├── inventory/
│   ├── freepbx_inventory.yaml    # VM inventory with ranges 1-8
│   └── group_vars/
│       └── all.yml               # Global variables
├── playbooks/
│   ├── deploy_freepbx_vms.yaml   # Main deployment playbook
│   └── roles/
│       └── create_freepbx_vm/    # Role for VM creation
│           ├── meta/main.yml
│           ├── defaults/main.yml
│           ├── vars/main.yml
│           ├── tasks/main.yml
│           └── files/
│               └── configure_freepbx_ip.sh
└── README.md
```

## VM Configuration

### Naming Convention
- VM Names: `BT-freepbx-1` through `BT-freepbx-8`
- Template: `PBX-clone-for-testing`

### Network Configuration
- IP Range: `100.10x.1.42` (where x = range number 1-8)
- Gateway: `100.10x.1.1`
- Netmask: `255.255.255.0`
- DNS: `8.8.8.8`, `1.1.1.1`

### Node Assignment
- Template Node: `am-kvm-10`
- Range 1: `am-kvm-01` (bridge: `vmbr01`)
- Range 2: `am-kvm-02` (bridge: `vmbr02`)
- Range 3: `am-kvm-03` (bridge: `vmbr03`)
- Range 4: `am-kvm-04` (bridge: `vmbr04`)
- Range 5: `am-kvm-05` (bridge: `vmbr05`)
- Range 6: `am-kvm-06` (bridge: `vmbr06`)
- Range 7: `am-kvm-07` (bridge: `vmbr07`)
- Range 8: `am-kvm-08` (bridge: `vmbr08`)

### VM Specifications
- CPU: 4 cores
- RAM: 4GB
- Storage: `nfs-fast`
- Clone Type: Full clone
- Pool: `bt01` through `bt08`

## Prerequisites

1. **Proxmox Access**: Ensure you have the correct API token and credentials
2. **Template**: The `PBX-clone-for-testing` template must exist on `am-kvm-10`
3. **Dependencies**: Install required Python packages and Ansible collections

## Installation

1. **Install Python dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

2. **Install Ansible collections**:
   ```bash
   ansible-galaxy collection install -r collections/requirements.yml
   ```

3. **Install sshpass** (required for IP configuration script):
   ```bash
   # On Ubuntu/Debian
   sudo apt-get install sshpass
   
   # On macOS
   brew install sshpass
   ```

## Usage

### Deploy All FreePBX VMs
```bash
cd freepbx
ansible-playbook -i inventory/freepbx_inventory.yaml playbooks/deploy_freepbx_vms.yaml
```

### Deploy Specific Range
To deploy only a specific range, you can modify the `ranges_to_deploy` variable in the playbook or create a separate playbook for individual ranges.

## VM Access

After deployment, each FreePBX VM will be accessible via SSH:
- **Username**: `root`
- **Password**: `changeme`
- **Port**: `22`

Example access to Range 1 FreePBX:
```bash
ssh root@100.101.1.42
```

## Troubleshooting

### Common Issues

1. **SSH Connection Timeout**: The script waits up to 5 minutes for SSH to become available. If it times out, check if the VM started properly.

2. **IP Configuration Fails**: Ensure the FreePBX template has SSH enabled and the credentials are correct.

3. **Proxmox API Errors**: Verify your API token and that the template exists on the source node.

### Logs
Check Ansible output for detailed error messages. The IP configuration script provides verbose output for troubleshooting network setup.

## Security Notes

- Default credentials are used for initial setup
- Consider changing passwords after deployment
- Ensure proper firewall rules are in place
- The IP configuration script uses `sshpass` which stores passwords in plain text

## Customization

To modify VM specifications, update the variables in:
- `inventory/group_vars/all.yml` (global settings)
- `playbooks/roles/create_freepbx_vm/defaults/main.yml` (role defaults)

To add more ranges, update:
- `inventory/freepbx_inventory.yaml` (add new range groups)
- `playbooks/deploy_freepbx_vms.yaml` (add to `ranges_to_deploy` list)
