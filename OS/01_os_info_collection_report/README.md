# OS Information Collection and Report Generation

## Overview

This playbook automates the collection of comprehensive system information from RHEL/CentOS hosts and generates professional HTML reports for each target host. Reports are centrally stored on the Ansible control node for easy access and sharing.

## Features

- **OS Compatibility**: Supports RHEL 7/8/9 and CentOS 7/8/9
- **Zero Agent**: Uses Ansible built-in facts, no additional software required on target hosts
- **Centralized Reports**: All reports generated on the control node
- **Robust Error Handling**: Comprehensive validation and error recovery
- **Customizable**: Easy to modify report templates and output locations

## Files

- `os_info_collection_report.yml` - Main playbook file
- `os_info_report_template.j2` - Jinja2 template for HTML report generation
- `README.md` - This file

## Prerequisites

- Ansible 2.9 or later
- Ansible control node with network access to target hosts
- Target hosts: RHEL 7/8/9 or CentOS 7/8/9
- SSH access to target hosts (password or key-based authentication)

## Usage

### 1. Prepare Inventory File

Create an inventory file (e.g., `inventory`) with your target hosts:

```ini
[rhel]
rhel7.example.com
rhel8.example.com
rhel9.example.com

[centos]
centos7.example.com
centos8.example.com
```

### 2. Run the Playbook

Execute the playbook from the control node:

```bash
ansible-playbook os_info_collection_report.yml -i inventory
```

### 3. Access Reports

Reports are generated in `/tmp/` directory on the control node with the naming format:
`<hostname>_report.html`

Example: `/tmp/rhel7.example.com_report.html`

## Customization

### Change Report Output Directory

Modify the `report_output_dir` variable in the playbook:

```yaml
vars:
  report_output_dir: "/var/www/html/reports"
```

### Customize Report Title

Modify the `report_title` variable:

```yaml
vars:
  report_title: "My Custom Report Title"
```

### Modify Report Template

Edit `os_info_report_template.j2` to customize the HTML report layout, styling, or add additional information sections.

## Playbook Structure

1. **OS Compatibility Validation**: Ensures only supported operating systems are processed
2. **Input Parameter Validation**: Validates all required variables and paths
3. **Report Generation**: Creates HTML reports using Jinja2 template
4. **Error Handling**: Comprehensive block-rescue-always structure for robust execution
5. **Debug Output**: Multiple debug tasks for troubleshooting

## Best Practices

- Run in check mode first: `ansible-playbook os_info_collection_report.yml -i inventory --check`
- Use tags for selective execution: `ansible-playbook os_info_collection_report.yml -i inventory --tags validation`
- Review generated reports before sharing
- Archive old reports periodically to manage disk space

## Troubleshooting

### Playbook fails with "Unsupported operating system"

- Verify target host OS is RHEL 7/8/9 or CentOS 7/8/9
- Check `ansible_distribution` and `ansible_distribution_major_version` facts

### Reports not generated

- Verify template file exists in the same directory as playbook
- Check write permissions on output directory
- Review playbook output for error messages

### Template rendering errors

- Ensure all Jinja2 syntax is correct
- Verify required Ansible facts are available
- Check for special characters in hostnames that might break file paths

## Author

GCG AAP SSA Team + v3.01 20260217

## License

📜 License Type: End User License Agreement (EULA)
🔒 Authorization: Subscription-based License
