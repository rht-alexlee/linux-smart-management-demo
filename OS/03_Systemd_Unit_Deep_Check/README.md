# Systemd Unit Deep Check and Analysis

## Overview

This playbook automates deep investigation of failed systemd units on RHEL/CentOS systems. It identifies all failed systemd units, analyzes each one with detailed status information and recent journal logs, and generates comprehensive investigation reports. The playbook is designed for production environments and focuses on investigation and reporting only—it does NOT restart services or modify system state, allowing administrators to make informed decisions about remediation.

The playbook generates comprehensive JSON format reports that include failed unit details, status information, recent logs, and investigation timestamps. Reports are automatically collected back to the control node for centralized analysis and monitoring.
### screenshots
<img width="2031" height="1325" alt="image" src="https://github.com/user-attachments/assets/b083d583-78e4-437d-9992-07580f7daf96" />

<img width="2536" height="391" alt="image" src="https://github.com/user-attachments/assets/5cb3adac-8dfe-415f-becb-5f4a8f7ca66d" />


## Features

- **Complete Automation**: End-to-end systemd unit investigation from detection to detailed reporting
- **OS Compatibility**: Supports RHEL 7/8/9 and CentOS 7/8/9 with automatic validation
- **Production-Safe**: Read-only investigation that does not modify system state or restart services
- **Comprehensive Reporting**: Generates detailed JSON reports with failed unit details, status, and logs
- **Centralized Collection**: Automatically collects reports back to control node for centralized analysis
- **Robust Error Handling**: Comprehensive error handling with block-rescue structures ensures execution continues even if individual tasks fail
- **Idempotent Design**: Safe to run multiple times without side effects
- **Comprehensive Validation**: OS compatibility checks and prerequisite validation
- **Detailed Unit Analysis**: For each failed unit, collects detailed status and recent journal logs

## Prerequisites

### General Prerequisites

- Ansible 2.9 or later
- Ansible control node with network access to target hosts
- Target hosts: RHEL 7/8/9 or CentOS 7/8/9
- SSH access to target hosts (password or key-based authentication)
- Sudo/root privileges on target hosts

### Required Tools on Target Hosts

The playbook requires the following tools to be available on target hosts:

- **systemctl**: For systemd unit management and status (usually pre-installed on systemd-based systems)
- **journalctl**: For journal log retrieval (usually pre-installed on systemd-based systems)

These tools are typically pre-installed on RHEL/CentOS systems with systemd and do not require additional package installation.

## Files

- `systemd_unit_deep_check_site.yml` - Main systemd unit investigation playbook
- `inventory` - Host inventory file
- `ansible.cfg` - Ansible configuration file
- `README.md` - This file

## Usage

### 1. Prepare Inventory File

Edit the `inventory` file to define your target hosts. The playbook uses the `all` hosts group by default, but you can use specific groups:

```ini
[systemd_investigation]
# Define target hosts for systemd unit investigation here
# Example:
# server-01 ansible_host=192.168.1.100
# server-02 ansible_host=192.168.1.101

# Using the standard inventory format:
[rhel7]
Test-RHEL-7.9-1
Test-RHEL-7.9-2
Test-RHEL-7.9-3

[rhel8]
TEST-RHEL-8.9-1

[rhel9]
ansible25.example.com

# Group all RHEL hosts as systemd_investigation
[systemd_investigation:children]
rhel7
rhel8
rhel9
```

### 2. Verify Prerequisites

Before running the playbook, verify prerequisites on target hosts:

```bash
# Test connectivity
ansible all -i inventory -m ping

# Check if systemctl command is available
ansible all -i inventory -m shell -a "which systemctl"

# Check if journalctl command is available
ansible all -i inventory -m shell -a "which journalctl"

# Verify systemd units
ansible all -i inventory -m shell -a "systemctl list-units --state=failed --no-legend"
```

### 3. Run the Playbook

#### Basic Execution

```bash
# Run investigation on all hosts
ansible-playbook systemd_unit_deep_check_site.yml -i inventory
```

#### Check Specific Host or Group

```bash
# Investigate only RHEL 9 hosts
ansible-playbook systemd_unit_deep_check_site.yml -i inventory --limit rhel9

# Investigate a specific host
ansible-playbook systemd_unit_deep_check_site.yml -i inventory --limit Test-RHEL-7.9-1
```

#### Dry Run (Check Mode)

```bash
# Preview what would be done (check mode)
ansible-playbook systemd_unit_deep_check_site.yml -i inventory --check
```

#### Advanced Usage

```bash
# Run with increased parallelism for bulk execution
ansible-playbook systemd_unit_deep_check_site.yml -i inventory --forks 10

# Run with verbose output for troubleshooting
ansible-playbook systemd_unit_deep_check_site.yml -i inventory -v

# Run with extra verbose output
ansible-playbook systemd_unit_deep_check_site.yml -i inventory -vvv
```

### 4. Review Reports

After successful execution, review the generated reports:

```bash
# List reports on control node
ls -lh reports/

# View a specific report (JSON format)
cat reports/systemd_unit_investigation_<hostname>.json | jq .

# View reports for all hosts
for report in reports/*.json; do
  echo "=== $report ==="
  cat "$report" | jq .
  echo ""
done

# Count failed units across all hosts
for report in reports/*.json; do
  hostname=$(basename "$report" .json | sed 's/systemd_unit_investigation_//')
  failed_count=$(jq -r '.failed_units.count' "$report")
  echo "$hostname: $failed_count failed unit(s)"
done
```

## What Gets Configured

The playbook performs the following tasks:

1. **OS Compatibility Validation**: Verifies RHEL/CentOS 7/8/9 compatibility
2. **Prerequisites Validation**: Checks for required tools availability (systemctl, journalctl)
3. **Failed Unit Detection**: Scans all systemd units to identify failed units
4. **Failed Unit Counting**: Counts and lists all failed units
5. **Summary Reporting**: Displays investigation summary with failed unit count
6. **Detailed Unit Analysis**: For each failed unit, collects detailed status and recent journal logs
7. **Report Generation**: Compiles all collected data into comprehensive JSON format report
8. **Report Collection**: Fetches reports back to control node for centralized analysis
9. **Summary Display**: Displays execution summary with key statistics

## Configuration Details

### Report File Location

- **On Target Host**: `/tmp/systemd_unit_investigation_<hostname>.json`
- **On Control Node**: `./reports/systemd_unit_investigation_<hostname>.json`

### Customizing Report Path

You can customize the report path by overriding variables:

```bash
ansible-playbook systemd_unit_deep_check_site.yml -i inventory \
  -e "report_path=/var/log/systemd_investigation_{{ ansible_hostname }}.json" \
  -e "report_dest_path=./custom_reports/"
```

### Customizing Log Lines

You can customize the number of log lines retrieved for each failed unit:

```bash
ansible-playbook systemd_unit_deep_check_site.yml -i inventory \
  -e "log_lines=50"
```

### Report Format

The generated report is in JSON format and includes:

- `hostname`: Target host hostname
- `os_distribution`: Operating system distribution
- `os_version`: Operating system version
- `investigation_timestamp`: ISO8601 timestamp of investigation
- `systemctl_available`: Whether systemctl command is available
- `journalctl_available`: Whether journalctl command is available
- `failed_units`: Object containing:
  - `count`: Number of failed units found
  - `list`: Array of failed unit lines from systemctl
  - `unit_names`: Array of extracted unit names
- `detailed_analysis`: Array of detailed analysis output for each failed unit (status and logs)
- `analysis_summary`: Object containing:
  - `all_healthy`: Whether all units are healthy
  - `recommendation`: Remediation recommendation

## Troubleshooting

### Playbook fails with "Unsupported operating system"

- Verify target host OS is RHEL 7/8/9 or CentOS 7/8/9
- Check `ansible_distribution` and `ansible_distribution_major_version` facts
- Remove any unsupported hosts from the inventory

### "Failed to detect failed systemd units"

- Verify `systemctl` command is available: `which systemctl`
- Check if systemd is running: `systemctl --version`
- Ensure sufficient permissions (playbook requires root/sudo access)
- Check if systemd is the init system: `ps -p 1 -o comm=`

### "No failed units found"

This is normal and indicates a healthy system. The playbook will still generate a report with `failed_units.count: 0`.

### Report file was not created successfully

- **Check disk space on target host:**
  ```bash
  df -h /tmp
  ```
- **Check file permissions:**
  ```bash
  ls -la /tmp/systemd_unit_investigation_*.json
  ```
- **Check Ansible logs for errors:**
  ```bash
  ansible-playbook systemd_unit_deep_check_site.yml -i inventory -v
  ```

### Reports not collected to control node

- **Check fetch module permissions:**
  ```bash
  ls -la reports/  # On control node
  ```
- **Verify source path exists on remote host:**
  ```bash
  ansible all -i inventory -m shell -a "ls -la /tmp/systemd_unit_investigation_*.json"
  ```
- **Check network connectivity:**
  ```bash
  ansible all -i inventory -m ping
  ```
- **Ensure reports directory exists:**
  ```bash
  mkdir -p reports/
  ```

### Unit status or logs not available

- This may occur if the unit has been removed or systemd is not properly configured
- The report will indicate "Unable to retrieve status" or "Unable to retrieve logs" for such cases
- Check systemd journal availability: `journalctl --list-boots`

### JSON parsing errors when viewing reports

- Install `jq` for better JSON viewing: `yum install -y jq` or `dnf install -y jq`
- Use `python -m json.tool` as an alternative: `cat report.json | python -m json.tool`

## Best Practices

1. **Regular Monitoring**: Schedule regular systemd unit checks (e.g., daily or weekly):
   ```bash
   # Add to crontab
   0 2 * * * ansible-playbook /path/to/systemd_unit_deep_check_site.yml -i /path/to/inventory
   ```

2. **Baseline Establishment**: Run initial checks to establish baseline failed unit counts

3. **Alerting Integration**: Integrate with monitoring systems to alert on failed unit detection:
   - Parse JSON reports for `failed_units.count > 0`
   - Set up alerts for persistent failed units
   - Monitor unit health trends

4. **Remediation Planning**: Use investigation reports to plan remediation:
   - Review failed unit logs for error patterns
   - Identify common failure causes
   - Plan service restarts or configuration fixes

5. **Report Archival**: Archive reports for historical analysis:
   ```bash
   # Archive reports with timestamp
   tar -czf systemd_reports_$(date +%Y%m%d).tar.gz reports/
   ```

6. **Documentation**: Keep records of investigation results and any remediation actions taken

7. **Testing**: Test playbook on non-production systems first

8. **Root Cause Analysis**: Use detailed logs to identify application bugs or misconfigurations that cause unit failures

## Understanding Systemd Units

### What are Failed Systemd Units?

Failed systemd units are services, timers, mounts, or other units that have entered a failed state. This typically occurs when:
- A service fails to start
- A service crashes during execution
- A dependency cannot be satisfied
- A configuration error prevents the unit from running

### Why Investigate Failed Units?

- **Service Availability**: Failed units indicate services that are not running
- **System Health Indicator**: High failed unit counts may indicate system-wide issues
- **Application Problems**: Failed units often indicate application bugs or misconfigurations
- **System Stability**: Persistent failed units can impact system functionality

### Remediation Strategies

1. **Review Logs**: Use the detailed logs in the report to identify the root cause
2. **Restart Failed Units**: `systemctl restart <unit-name>`
3. **Check Dependencies**: `systemctl list-dependencies <unit-name>`
4. **Review Configuration**: Check unit configuration files in `/etc/systemd/system/` or `/usr/lib/systemd/system/`
5. **Check Resource Constraints**: Verify disk space, memory, and other resources

**Note**: This playbook does NOT perform any of these remediation actions. It only investigates and reports.

## Limitations

- **Read-Only Investigation**: This playbook only investigates and reports; it does not restart or modify units
- **Unit Availability**: Unit information may not be available if the unit has been removed or systemd is not properly configured
- **Log Retention**: Journal logs may be rotated or truncated, limiting historical analysis
- **Network Dependency**: Requires network connectivity between control node and target hosts
- **Storage Requirements**: Reports consume disk space on both target hosts and control node
- **Real-Time Detection**: Failed units may appear or disappear between scans
- **No Automatic Remediation**: This playbook only performs investigation and reporting; remediation must be performed manually

## Security Considerations

1. **Credential Management**: Use Ansible Vault for sensitive credentials if needed
2. **Report Sensitivity**: Investigation reports may contain system information; protect report files appropriately
3. **Access Control**: Limit access to reports directory on control node
4. **Audit Trail**: Keep logs of all investigation executions for compliance
5. **Network Security**: Ensure secure communication channels between control node and target hosts
6. **Privilege Escalation**: Playbook requires root/sudo access; ensure proper access controls

## Support

For issues or questions:

- Check Ansible logs: `ansible-playbook -v` for verbose output
- Review system logs: `journalctl -xe` on target hosts
- Verify systemd tools: `which systemctl` and `systemctl --version`
- Check Red Hat documentation: https://access.redhat.com/documentation/

## Example Report Output

```json
{
  "hostname": "test-server-01",
  "os_distribution": "RedHat",
  "os_version": "8.9",
  "investigation_timestamp": "2026-02-06T12:00:00Z",
  "systemctl_available": "Yes",
  "journalctl_available": "Yes",
  "failed_units": {
    "count": 1,
    "list": [
      "myapp.service failed failed myapp.service"
    ],
    "unit_names": [
      "myapp.service"
    ]
  },
  "detailed_analysis": [
    "==========================================\nAnalysis for Unit: myapp.service\n==========================================\n\n--- Unit Status ---\n● myapp.service - My Application Service\n   Loaded: loaded (/etc/systemd/system/myapp.service; enabled; vendor preset: disabled)\n   Active: failed (Result: exit-code) since Mon 2026-02-06 12:00:00 UTC; 5min ago\n  Process: 12345 ExecStart=/usr/bin/myapp (code=exited, status=1/FAILURE)\n Main PID: 12345 (code=exited, status=1/FAILURE)\n\n--- Recent Logs (last 20 lines) ---\nFeb 06 12:00:00 test-server-01 myapp[12345]: Error: Unable to connect to database\nFeb 06 12:00:00 test-server-01 myapp[12345]: Fatal error occurred\n=========================================="
  ],
  "analysis_summary": {
    "all_healthy": "No",
    "recommendation": "Review failed units and their logs for remediation. Consider restarting failed units or investigating root causes."
  }
}
```

## License

📜 License Type: End User License Agreement (EULA)
🔒 Authorization: Subscription-based License

## Author

GCG AAP SSA Team + v3.01 20260217


