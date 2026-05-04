# Firewall Management Tool

## Overview

This playbook provides a centralized, automated solution for managing the `firewalld` service across multiple Linux hosts. It supports starting, stopping, or checking the status of the firewall service with comprehensive error handling, validation, and detailed status reporting.

## Features

- **OS Compatibility**: Supports RHEL 7/8/9 and CentOS 7/8/9
- **Three Operation Modes**:
  - `start`: Start and enable firewall service
  - `stop`: Stop and disable firewall service
  - `check`: Check current status (default, safe mode)
- **Automatic Package Installation**: Installs `firewalld` if not present
- **Comprehensive Error Handling**: Uses `block-rescue-always` for robust error handling
- **Detailed Status Reporting**: Provides formatted status reports for each host
- **Parameter Validation**: Validates input parameters before execution
- **Idempotent Operations**: Safe to run multiple times

## Files

- `firewall_management.yml` - Main playbook file
- `README.md` - This file

## Prerequisites

- Ansible 2.9 or later
- Ansible control node with network access to target hosts
- Target hosts: RHEL 7/8/9 or CentOS 7/8/9
- SSH access to target hosts with root privileges (or sudo access)

## Usage

### 1. Prepare Inventory File

Create an inventory file (e.g., `inventory`) with your target hosts:

```ini
[rhel7]
Test-RHEL-7.9-1
Test-RHEL-7.9-2
Test-RHEL-7.9-3

[rhel8]
TEST-RHEL-8.9-1

[rhel9]
ansible25.example.com

[all:children]
rhel7
rhel8
rhel9
```

**Important Notes**:

- This playbook supports RHEL 7/8/9 and CentOS 7/8/9 only
- You can organize hosts into groups as needed
- The playbook will validate OS compatibility before execution

### 2. Run the Playbook

#### Start Firewall Service

To start and enable the firewall service on all hosts:

```bash
ansible-playbook firewall_management.yml -i inventory -e "firewall_action=start"
```

#### Stop Firewall Service

To stop and disable the firewall service on all hosts:

```bash
ansible-playbook firewall_management.yml -i inventory -e "firewall_action=stop"
```

#### Check Firewall Status (Default)

To check the current status without making changes (safe default):

```bash
# Option 1: Explicitly specify check
ansible-playbook firewall_management.yml -i inventory -e "firewall_action=check"

# Option 2: Omit the parameter (check is the default)
ansible-playbook firewall_management.yml -i inventory
```

### 3. Verify Results

After execution, review the output for status reports:

```
+--------------------------------------------------+
|          Firewall Status Report                  |
+--------------------------------------------------+
|  Host: Test-RHEL-7.9-1
|  Action: Start
|  Service: firewalld
|  Current State: running
|  Enabled on Boot: enabled
|  Distribution: RedHat 7
+--------------------------------------------------+
```

## Playbook Structure

### Main Playbook (`firewall_management.yml`)

1. **Parameter Validation**: Validates `firewall_action` parameter
2. **OS Compatibility Check**: Ensures target hosts are supported OS versions
3. **Core Operations** (when action is start/stop):
   - Package installation (if needed)
   - Service management (start/stop and enable/disable)
   - Error handling with rollback capability
4. **Status Collection**: Gathers current service status using `service_facts`
5. **Status Reporting**: Displays formatted status report for each host

### Key Design Features

- **Block-Rescue-Always Structure**: Ensures proper error handling and cleanup
- **Idempotent Operations**: Safe to run multiple times without side effects
- **Comprehensive Logging**: Debug tasks after each major operation
- **Parameter Safety**: Default action is "check" to prevent accidental changes

## Customization

### Change Default Action

To change the default action, modify the `vars` section in the playbook:

```yaml
vars:
  firewall_action: "start"  # Change from "check" to "start" or "stop"
```

**Note**: It's recommended to keep the default as "check" for safety.

### Target Specific Host Groups

Use inventory groups to target specific hosts:

```bash
# Only manage RHEL 8 hosts
ansible-playbook firewall_management.yml -i inventory -e "firewall_action=start" --limit rhel8

# Only manage RHEL 7 hosts
ansible-playbook firewall_management.yml -i inventory -e "firewall_action=stop" --limit rhel7
```

## Best Practices

- **Test First**: Use `--check` mode to preview changes:

  ```bash
  ansible-playbook firewall_management.yml -i inventory -e "firewall_action=start" --check
  ```
- **Use Tags**: Use tags for selective execution:

  ```bash
  # Only run validation tasks
  ansible-playbook firewall_management.yml -i inventory --tags validation

  # Only run service management tasks
  ansible-playbook firewall_management.yml -i inventory -e "firewall_action=start" --tags service
  ```
- **Check Status First**: Always check status before making changes:

  ```bash
  ansible-playbook firewall_management.yml -i inventory
  ```
- **Maintenance Windows**: Schedule firewall changes during maintenance windows
- **Documentation**: Document firewall state changes in change management systems

## Troubleshooting

### Playbook fails with "Unsupported operating system"

- Verify target host OS is RHEL 7/8/9 or CentOS 7/8/9
- Check `ansible_distribution` and `ansible_distribution_major_version` facts
- Remove any unsupported hosts from the inventory

### "Invalid firewall_action value"

- Ensure `firewall_action` is one of: `start`, `stop`, `check`
- Check for typos in the command line parameter
- Verify the parameter is passed correctly: `-e "firewall_action=start"`

### "Failed to start/stop firewall service"

- **Check package installation:**

  ```bash
  yum install -y firewalld
  ```
- **Check service status manually:**

  ```bash
  systemctl status firewalld
  ```
- **Check system logs:**

  ```bash
  journalctl -u firewalld
  ```
- **Verify service is not masked:**

  ```bash
  systemctl is-enabled firewalld
  systemctl unmask firewalld  # If masked
  ```

### Service shows incorrect status in report

- The playbook uses `service_facts` to gather status
- If status seems incorrect, verify manually:

  ```bash
  systemctl status firewalld
  systemctl is-enabled firewalld
  ```
- Re-run the playbook with `firewall_action=check` to refresh status

### Package installation fails

- **Check network connectivity:**

  ```bash
  ping yum repository server
  ```
- **Check YUM/DNF repositories:**

  ```bash
  yum repolist enabled
  ```
- **Install manually and re-run:**

  ```bash
  yum install -y firewalld
  ```

## Example Output

### Starting Firewall

```
PLAY [Firewall Management Tool] ********************************************

TASK [Validate firewall_action parameter] ********************************
ok: [Test-RHEL-7.9-1] => {
    "msg": "Parameter validation passed: firewall_action='start'"
}

TASK [Validate OS compatibility (RHEL/CentOS 7/8/9 only)] ***************
ok: [Test-RHEL-7.9-1] => {
    "msg": "OS compatibility check passed: RedHat 7"
}

TASK [Start firewall service] **********************************************
ok: [Test-RHEL-7.9-1] => (item=Ensure firewalld package is installed)
changed: [Test-RHEL-7.9-1] => (item=Start and enable firewalld service)

TASK [Collect service facts for status reporting] **************************
ok: [Test-RHEL-7.9-1]

TASK [Display current firewall service status] ****************************
ok: [Test-RHEL-7.9-1] => {
    "msg": [
        "+--------------------------------------------------+",
        "|          Firewall Status Report                  |",
        "+--------------------------------------------------+",
        "|  Host: Test-RHEL-7.9-1",
        "|  Action: Start",
        "|  Service: firewalld",
        "|  Current State: running",
        "|  Enabled on Boot: enabled",
        "|  Distribution: RedHat 7",
        "+--------------------------------------------------+"
    ]
}

PLAY RECAP ******************************************************************
Test-RHEL-7.9-1            : ok=8    changed=1    failed=0
```

### Checking Status (Default)

```
PLAY [Firewall Management Tool] ********************************************

TASK [Validate firewall_action parameter] ********************************
ok: [Test-RHEL-7.9-1] => {
    "msg": "Parameter validation passed: firewall_action='check'"
}

TASK [Collect service facts for status reporting] **************************
ok: [Test-RHEL-7.9-1]

TASK [Display current firewall service status] ****************************
ok: [Test-RHEL-7.9-1] => {
    "msg": [
        "+--------------------------------------------------+",
        "|          Firewall Status Report                  |",
        "+--------------------------------------------------+",
        "|  Host: Test-RHEL-7.9-1",
        "|  Action: Check",
        "|  Service: firewalld",
        "|  Current State: running",
        "|  Enabled on Boot: enabled",
        "|  Distribution: RedHat 7",
        "+--------------------------------------------------+"
    ]
}

PLAY RECAP ******************************************************************
Test-RHEL-7.9-1            : ok=6    changed=0    failed=0
```

## Security Considerations

- **Root Access Required**: This playbook requires root privileges to manage system services
- **Firewall Impact**: Stopping the firewall service will disable network protection
- **Production Use**: Always test in non-production environments first
- **Change Management**: Document all firewall state changes
- **Audit Trail**: Review playbook execution logs for compliance

## Author

GCG AAP SSA Team + v3.01 20260217

## License

📜 License Type: End User License Agreement (EULA)
🔒 Authorization: Subscription-based License
