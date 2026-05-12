# Automated RHEL Security Patch Installation Based on CVE ID

## Overview

This playbook automates the installation of security patches on RHEL/CentOS systems based on CVE (Common Vulnerabilities and Exposures) ID. It provides a complete, enterprise-grade solution for querying and applying security patches across your infrastructure, eliminating manual patch management steps and ensuring consistent, secure system updates.

The playbook queries Red Hat security advisories for a specific CVE ID, identifies affected packages, and optionally applies the necessary security patches. It also checks if a system reboot is required after patching.

## Features

- **Complete Automation**: End-to-end CVE patch deployment from query to installation
- **OS Compatibility**: Supports RHEL 7/8/9 and CentOS 7/8/9 with automatic validation
- **Flexible Execution**: Query-only mode or full patch installation
- **Reboot Detection**: Automatically detects if system reboot is required after patching
- **Robust Error Handling**: Comprehensive error handling with block-rescue structures ensures execution continues even if individual tasks fail
- **Idempotent Design**: Safe to run multiple times without side effects
- **Comprehensive Validation**: Input parameter validation prevents configuration errors
- **Repository Validation**: Validates YUM/DNF repository configuration before patching

## Prerequisites

### ⚠️ CRITICAL: Repository Configuration Required

**This playbook assumes that YUM/DNF repositories are already properly configured on all target hosts.**

Before running this playbook, ensure:

1. **Red Hat Subscription Management (RHSM) is configured**:
   ```bash
   # Verify subscription status
   subscription-manager status
   
   # If not registered, register the system:
   subscription-manager register --username <username> --password <password>
   
   # Attach a subscription:
   subscription-manager attach --auto
   ```

2. **YUM/DNF repositories are enabled and accessible**:
   ```bash
   # For RHEL 7
   yum repolist
   
   # For RHEL 8/9
   dnf repolist
   
   # Verify repository metadata is current
   yum clean all && yum makecache
   # or
   dnf clean all && dnf makecache
   ```

3. **Required repositories are enabled**:
   - Base repository (rhel-7-server-rpms, rhel-8-for-x86_64-baseos-rpms, etc.)
   - Updates repository (rhel-7-server-rpms, rhel-8-for-x86_64-appstream-rpms, etc.)
   - Optional repository (if needed for specific packages)

4. **Network connectivity** to Red Hat Content Delivery Network (CDN) or Satellite Server:
   ```bash
   # Test connectivity
   ping subscription.rhsm.redhat.com
   # or your Satellite server
   ```

5. **For RHEL 7**: Ensure `yum-plugin-security` can be installed (will be installed automatically if missing)

6. **For RHEL 8/9**: Ensure `dnf` and `updateinfo` command are available

### General Prerequisites

- Ansible 2.9 or later
- Ansible control node with network access to target hosts
- Target hosts: RHEL 7/8/9 or CentOS 7/8/9
- SSH access to target hosts (password or key-based authentication)
- Sudo/root privileges on target hosts
- Valid Red Hat subscription (for RHEL systems)
- Internet access or configured Satellite Server for package downloads

## Files

- `rhel_patch_install_site.yml` - Main patch installation playbook
- `inventory` - Host inventory file
- `ansible.cfg` - Ansible configuration file
- `README.md` - This file

## Usage

### 1. Prepare Inventory File

Edit the `inventory` file to define your target hosts. The playbook uses the `patch_servers` group:

```ini
[patch_servers]
# Define target hosts for RHEL patch installation here
# Example:
# rhel-server-01 ansible_host=192.168.1.100
# rhel-server-02 ansible_host=192.168.1.101

# Using the standard inventory format:
[rhel7]
Test-RHEL-7.9-1
Test-RHEL-7.9-2
Test-RHEL-7.9-3

[rhel8]
TEST-RHEL-8.9-1

[rhel9]
ansible25.example.com

# Group all RHEL hosts as patch_servers
[patch_servers:children]
rhel7
rhel8
rhel9
```

### 2. Verify Repository Configuration

**Before running the playbook, verify repository configuration on target hosts:**

```bash
# Test repository access on a target host
ansible patch_servers -i inventory -m shell -a "yum repolist"  # RHEL 7
ansible patch_servers -i inventory -m shell -a "dnf repolist"  # RHEL 8/9

# Verify subscription status
ansible patch_servers -i inventory -m shell -a "subscription-manager status"
```

### 3. Run the Playbook

#### Query CVE Information Only (No Patch Installation)

```bash
# Query CVE information without applying patches
ansible-playbook rhel_patch_install_site.yml -i inventory \
  -e "cve_id=CVE-2023-42753" \
  -e "apply_patch=no"
```

#### Install Patches for a Specific CVE

```bash
# Install patches for a specific CVE
ansible-playbook rhel_patch_install_site.yml -i inventory \
  -e "cve_id=CVE-2023-42753" \
  -e "apply_patch=yes"
```

#### Dry Run (Check Mode)

```bash
# Preview what would be done (check mode)
ansible-playbook rhel_patch_install_site.yml -i inventory \
  -e "cve_id=CVE-2023-42753" \
  -e "apply_patch=yes" \
  --check
```

#### Patch Specific Host or Group

```bash
# Patch only RHEL 9 hosts
ansible-playbook rhel_patch_install_site.yml -i inventory \
  -e "cve_id=CVE-2023-42753" \
  -e "apply_patch=yes" \
  --limit rhel9

# Patch a specific host
ansible-playbook rhel_patch_install_site.yml -i inventory \
  -e "cve_id=CVE-2023-42753" \
  -e "apply_patch=yes" \
  --limit Test-RHEL-7.9-1
```

#### Advanced Usage

```bash
# Run with increased parallelism for bulk execution
ansible-playbook rhel_patch_install_site.yml -i inventory \
  -e "cve_id=CVE-2023-42753" \
  -e "apply_patch=yes" \
  --forks 10

# Run with verbose output for troubleshooting
ansible-playbook rhel_patch_install_site.yml -i inventory \
  -e "cve_id=CVE-2023-42753" \
  -e "apply_patch=yes" \
  -v
```

### 4. Verify Patch Installation

After successful execution, verify the patches were applied:

```bash
# Check installed packages related to CVE
ssh <hostname> "rpm -qa | grep <package-name>"

# Verify patch was applied (check package version)
ssh <hostname> "rpm -q <package-name>"

# Check if reboot is required
ssh <hostname> "needs-restarting -r"
```

## What Gets Configured

The playbook performs the following tasks:

1. **OS Compatibility Validation**: Verifies RHEL/CentOS 7/8/9 compatibility
2. **Input Parameter Validation**: Validates CVE ID format and apply_patch parameter
3. **Repository Validation**: Checks if YUM/DNF repositories are accessible
4. **Security Plugin Installation**: Installs `yum-plugin-security` for RHEL 7
5. **CVE Query**: Queries Red Hat security advisories for the specified CVE ID
6. **Patch Application**: Applies security patches for affected packages (if apply_patch=yes)
7. **Reboot Detection**: Checks if system reboot is required after patching
8. **Summary Report**: Displays comprehensive summary of patch installation status

## Configuration Details

### CVE ID Format

The CVE ID must follow the standard format:
- Format: `CVE-YYYY-NNNN` or `CVE-YYYY-NNNNN`
- Example: `CVE-2023-42753`
- YYYY: 4-digit year
- NNNN/NNNNN: 4-5 digit vulnerability number

### Apply Patch Parameter

- **yes**: Apply security patches for the CVE (default)
- **no**: Only query CVE information, do not apply patches

### Package Manager Selection

The playbook automatically selects the appropriate package manager:
- **RHEL 7**: Uses `yum` with `yum-plugin-security`
- **RHEL 8/9**: Uses `dnf` with built-in `updateinfo` command

## Troubleshooting

### Repository Issues

#### No Repositories Available

```bash
# Check repository status
yum repolist  # RHEL 7
dnf repolist  # RHEL 8/9

# If no repositories, register and attach subscription
subscription-manager register --username <username> --password <password>
subscription-manager attach --auto
subscription-manager repos --enable rhel-7-server-rpms  # Adjust for your version
```

#### Repository Metadata Outdated

```bash
# Refresh repository metadata
yum clean all && yum makecache  # RHEL 7
dnf clean all && dnf makecache  # RHEL 8/9
```

### CVE Query Issues

#### "No such CVE" Error

- Verify the CVE ID format is correct
- Check if the CVE affects RHEL (some CVEs may not have RHEL patches)
- Ensure repository metadata is current

#### "No updates available" Message

- The CVE may already be patched on the system
- The CVE may not affect installed packages
- Repository metadata may need to be refreshed

### Patch Installation Issues

#### Package Installation Fails

```bash
# Check repository configuration
yum repolist  # RHEL 7
dnf repolist  # RHEL 8/9

# Check for conflicting packages
yum check  # RHEL 7
dnf check  # RHEL 8/9

# View detailed error messages
yum update --cve CVE-XXXX-XXXX -v  # RHEL 7
dnf update --cve CVE-XXXX-XXXX -v  # RHEL 8/9
```

#### Insufficient Disk Space

```bash
# Check available disk space
df -h

# Clean package cache if needed
yum clean all  # RHEL 7
dnf clean all  # RHEL 8/9
```

### Reboot Detection Issues

#### needs-restarting Command Not Available

The playbook will automatically install `yum-utils` (RHEL 7) or `dnf-utils` (RHEL 8/9) if missing. If installation fails, install manually:

```bash
# RHEL 7
yum install -y yum-utils

# RHEL 8/9
dnf install -y dnf-utils
```

## Best Practices

1. **Test First**: Always query CVE information first before applying patches:
   ```bash
   ansible-playbook rhel_patch_install_site.yml -i inventory \
     -e "cve_id=CVE-XXXX-XXXX" \
     -e "apply_patch=no"
   ```

2. **Repository Maintenance**: Regularly update repository metadata:
   ```bash
   yum clean all && yum makecache  # RHEL 7
   dnf clean all && dnf makecache  # RHEL 8/9
   ```

3. **Backup Before Patching**: Consider creating system snapshots or backups before applying patches in production

4. **Staged Rollout**: Test patches on non-production systems first, then gradually roll out to production

5. **Reboot Coordination**: Plan for system reboots after patching, especially for kernel updates

6. **Monitoring**: Monitor system logs after patching to ensure no issues:
   ```bash
   journalctl -xe
   tail -f /var/log/messages
   ```

7. **Documentation**: Keep records of patched CVEs and reboot requirements

8. **Regular Updates**: Schedule regular security patch reviews and installations

## Playbook Structure

The playbook consists of a single play with the following task groups:

1. **OS Compatibility Validation**: Validates RHEL/CentOS 7/8/9
2. **Input Parameter Validation**: Validates CVE ID format and apply_patch parameter
3. **Repository Validation**: Checks repository configuration and accessibility
4. **Security Plugin Installation**: Installs required plugins (RHEL 7)
5. **CVE Information Display**: Shows CVE ID and system information
6. **CVE Query**: Queries security advisories for the CVE
7. **Patch Application**: Applies security patches (if requested)
8. **Reboot Detection**: Checks if reboot is required
9. **Final Report**: Displays comprehensive summary

## Customization

### Change Default Apply Patch Behavior

Edit the playbook variables or override at runtime:

```bash
# Default to query-only mode
ansible-playbook rhel_patch_install_site.yml -i inventory \
  -e "cve_id=CVE-XXXX-XXXX" \
  -e "apply_patch=no"
```

### Use Variables File

Create a `vars.yml` file:

```yaml
cve_id: "CVE-2023-42753"
apply_patch: "yes"
```

Then use it:

```bash
ansible-playbook rhel_patch_install_site.yml -i inventory -e @vars.yml
```

## Limitations

- Requires root/sudo privileges on target hosts
- Requires valid Red Hat subscription for RHEL systems
- Requires YUM/DNF repositories to be properly configured (see Prerequisites)
- Some CVEs may not have patches available for all RHEL versions
- Patch installation may require system reboot
- Network connectivity to Red Hat CDN or Satellite Server is required
- CVE query depends on current repository metadata

## Security Considerations

1. **Credential Management**: Use Ansible Vault for sensitive credentials if needed
2. **Audit Trail**: Keep logs of all patch installations for compliance
3. **Testing**: Always test patches in non-production environments first
4. **Backup**: Create system backups before applying patches
5. **Rollback Plan**: Have a rollback plan in case patches cause issues

## Support

For issues or questions:
- Check Ansible logs: `ansible-playbook -v` for verbose output
- Review system logs: `journalctl -xe` on target hosts
- Verify repository configuration: `yum repolist` or `dnf repolist`
- Check Red Hat Security Advisories: https://access.redhat.com/security/security-updates

## License

📜 License Type: End User License Agreement (EULA)
🔒 Authorization: Subscription-based License

## Author

GCG AAP SSA Team + v3.01 20260217

