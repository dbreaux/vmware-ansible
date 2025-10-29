# VMware Ansible Automation - Setup Guide

## Step-by-Step Setup Instructions

### 1. Prerequisites Installation

#### Install Ansible
**On macOS:**
```bash
brew install ansible



** On Ubuntu/Debian**
sudo apt update
sudo apt install ansible


**On RHEL/CentOS/Fedora:
sudo dnf install ansible



#### Install Python Dependencies
# For REST API functionality
pip3 install aiohttp pyvmomi

# If using system Python on Ubuntu/Debian
sudo apt install python3-aiohttp python3-pyvmomi

# If using system Python on Fedora/RHEL
sudo dnf install python3-aiohttp python3-pyvmomi


#### Install Ansible Collections
ansible-galaxy collection install vmware.vmware_rest
ansible-galaxy collection install community.vmware


2. Project Setup

Clone and Configure

# Clone the repository
git clone https://github.com/yourusername/vmware-ansible-automation.git
cd vmware-ansible-automation

# Copy configuration templates
cp inventory.yml.example inventory.yml
cp group_vars/vcenters.yml.example group_vars/vcenters.yml


Edit Configuration Files

inventory.yml:
all:
  children:
    vcenters:
      hosts:
        vcenter.yourdomain.com:  # Your vCenter FQDN
          vcenter_username: administrator@vsphere.local


group_vars/vcenters.yml:
vcenter_passwords:
  vcenter.yourdomain.com: YourActualPassword123!


create_vm_from_powered_off_vm.yml (vars section):
vars:
  datacenter_name: "Production"     # Your datacenter name
  cluster_name: "MainCluster"      # Your cluster name


3. Ansible Vault Configuration

Encrypt Password File
# Encrypt the password file
ansible-vault encrypt group_vars/vcenters.yml

# You'll be prompted to create a vault password
New Vault password: 
Confirm New Vault password:


4. Testing Your Setup

Test Connectivity
# Simple connectivity test
ansible vcenters -m ping --ask-vault-pass


Test vCenter Access
# Run a simple info gathering playbook
ansible-playbook -c local --ask-vault-pass << EOF
---
- hosts: vcenters
  gather_facts: no
  tasks:
    - name: Test vCenter connection
      vmware.vmware_rest.vcenter_datacenter_info:
        vcenter_hostname: "{{ inventory_hostname }}"
        vcenter_username: "{{ vcenter_username }}"
        vcenter_password: "{{ vcenter_passwords[inventory_hostname] }}"
        vcenter_validate_certs: no
      delegate_to: localhost
EOF


5. First VM Deployment
# Run the main playbook
ansible-playbook create_vm_from_template.yml --ask-vault-pass

# Follow the interactive prompts:
# 1. Enter VM name
# 2. Select template from the list
# 3. Confirm deployment details
# 4. Wait for completion


6. Troubleshooting

Common Issues and Solutions

"Template not found" Error:

    Ensure templates are powered off
    Check template names for exact spelling
    Verify templates are in the correct datacenter


Authentication Errors:

    Verify vCenter credentials
    Check vault password
    Test network connectivity to vCenter


PyVmomi JSON Errors:

    These are cosmetic and don't affect functionality
    VMs are still created successfully
    Consider using stdout_callback = minimal in ansible.cfg


Permission Errors:

    nsure vCenter user has proper privileges
    Required permissions: VM creation, datastore access, cluster access

Debug Mode
# Run with maximum verbosity
ansible-playbook create_vm_from_template.yml --ask-vault-pass -vvv



