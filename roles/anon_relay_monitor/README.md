# Anon Relay Monitor Role

This role provides comprehensive monitoring and management capabilities for Anon relays, including Nyx installation, health checks, performance monitoring, log monitoring, and reporting.

## Requirements

- Ansible 2.15+
- `anon_relay_base` role (must be run first)
- Target systems: Ubuntu 20.04+, Debian 10+, Fedora 35+
- Python 3.8+ on target systems
- Docker container running Anon relay

## Role Variables

### Default Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_monitoring_enabled` | `true` | Enable monitoring system |
| `anon_nyx_enabled` | `true` | Enable Nyx monitoring |
| `anon_nyx_install_method` | `"pip"` | Installation method (pip, apt, manual) |
| `anon_nyx_version` | `"latest"` | Nyx version to install |
| `anon_nyx_config_dir` | `"/root/.nyx"` | Nyx configuration directory |
| `anon_nyx_control_socket` | `"/opt/anon/run/anon/control"` | Control socket path |

### Monitoring Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_monitor_scripts_dir` | `"/opt/anon/monitoring"` | Monitoring scripts directory |
| `anon_monitor_log_dir` | `"/var/log/anon/monitoring"` | Monitoring logs directory |
| `anon_monitor_retention_days` | `30` | Log retention period |
| `anon_health_check_interval` | `300` | Health check interval (seconds) |
| `anon_health_check_retries` | `3` | Health check retries |
| `anon_health_check_timeout` | `30` | Health check timeout |

### Performance Monitoring

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_performance_monitoring` | `true` | Enable performance monitoring |
| `anon_cpu_threshold` | `80` | CPU usage threshold (%) |
| `anon_memory_threshold` | `80` | Memory usage threshold (%) |
| `anon_disk_threshold` | `90` | Disk usage threshold (%) |
| `anon_bandwidth_threshold` | `90` | Bandwidth usage threshold (%) |

### Log Monitoring

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_log_monitoring` | `true` | Enable log monitoring |
| `anon_log_error_threshold` | `10` | Error count threshold |
| `anon_log_warning_threshold` | `50` | Warning count threshold |
| `anon_log_rotation` | `true` | Enable log rotation |
| `anon_log_max_size` | `"100M"` | Maximum log file size |
| `anon_log_max_files` | `5` | Maximum log files to keep |

### Alert Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_alerts_enabled` | `false` | Enable alerting |
| `anon_alert_email` | `"admin@example.com"` | Alert email address |
| `anon_alert_slack_webhook` | `""` | Slack webhook URL |
| `anon_alert_discord_webhook` | `""` | Discord webhook URL |

### Backup Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_backup_enabled` | `false` | Enable backup system |
| `anon_backup_dir` | `"/opt/anon/backups"` | Backup directory |
| `anon_backup_retention_days` | `7` | Backup retention period |
| `anon_backup_schedule` | `"0 2 * * *"` | Backup schedule (cron) |

### Reporting Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_reporting_enabled` | `true` | Enable reporting |
| `anon_report_schedule` | `"0 0 * * 0"` | Report schedule (cron) |
| `anon_report_email` | `"admin@example.com"` | Report email address |

## Dependencies

- `anon_relay_base` role

## Example Playbook

```yaml
---
- name: Deploy Anon relay with monitoring
  hosts: relays
  become: yes
  roles:
    - docker_setup
    - anon_relay_base
    - anon_relay_standard
    - anon_relay_monitor
  vars:
    anon_monitoring_enabled: true
    anon_nyx_enabled: true
    anon_performance_monitoring: true
    anon_log_monitoring: true
    anon_alerts_enabled: true
    anon_alert_email: "admin@example.com"
```

## Monitoring Features

### Nyx Integration
- Terminal-based monitoring interface
- Real-time relay statistics
- Bandwidth and connection monitoring
- Circuit and stream visualization

### Health Checks
- Container status monitoring
- Port availability checks
- Process health verification
- Control socket validation
- Log error detection

### Performance Monitoring
- CPU usage tracking
- Memory usage monitoring
- Disk space monitoring
- Bandwidth utilization
- Threshold-based alerts

### Log Monitoring
- Error and warning detection
- Log rotation management
- Pattern-based alerting
- Retention policy enforcement

### Network Monitoring
- Port connectivity checks
- Service availability verification
- Network performance metrics
- Connection quality assessment

### Backup System
- Automated configuration backups
- Data retention management
- Scheduled backup execution
- Recovery procedures

### Reporting
- Weekly performance reports
- HTML email reports
- Trend analysis
- Historical data

## Generated Scripts

The role creates several monitoring scripts:

### Core Scripts
- `health_check.sh` - Comprehensive health monitoring
- `performance_monitor.sh` - Performance metrics collection
- `log_monitor.sh` - Log analysis and alerting
- `network_monitor.sh` - Network connectivity checks
- `backup.sh` - Automated backup system
- `report.sh` - Report generation

### Utility Scripts
- `monitoring_utils.sh` - Common monitoring functions
- `alert.sh` - Alert notification system
- `status.sh` - Quick status overview
- `maintenance.sh` - Maintenance tasks

## Cron Jobs

The role sets up automated monitoring:

- **Health Check**: Every 5 minutes
- **Performance Monitoring**: Every minute
- **Network Monitoring**: Every minute
- **Backup**: Daily at 2 AM (if enabled)
- **Reporting**: Weekly on Sunday

## Systemd Services

- `nyx.service` - Nyx monitoring interface
- `anon-health-check.service` - Health check service
- `anon-health-check.timer` - Health check timer

## Configuration Files

### Nyx Configuration
- `nyx_config.j2` - Nyx settings and display options
- Control socket configuration
- Log file settings
- Display preferences

### Log Rotation
- `logrotate_anon.j2` - Log rotation configuration
- Size-based rotation
- Retention policies
- Compression settings

## Tags

- `anon` - All Anon-related tasks
- `monitor` - Monitoring tasks
- `install` - Installation tasks
- `nyx` - Nyx-specific tasks
- `health` - Health monitoring
- `performance` - Performance monitoring
- `logs` - Log monitoring
- `network` - Network monitoring
- `backup` - Backup tasks
- `reporting` - Reporting tasks
- `scripts` - Script generation
- `alert` - Alerting tasks

## Handlers

- `reload systemd` - Reloads systemd daemon
- `restart nyx` - Restarts Nyx service
- `restart health check` - Restarts health check service

## Usage Examples

### Manual Health Check
```bash
/opt/anon/monitoring/health_check.sh
```

### Performance Monitoring
```bash
/opt/anon/monitoring/performance_monitor.sh
```

### Status Overview
```bash
/opt/anon/monitoring/status.sh
```

### Generate Report
```bash
/opt/anon/monitoring/report.sh
```

### Run Backup
```bash
/opt/anon/monitoring/backup.sh
```

## Alerting

When enabled, the monitoring system can send alerts via:
- Email notifications
- Slack webhooks
- Discord webhooks
- Custom alert scripts

## Log Files

Monitoring logs are stored in:
- `/var/log/anon/monitoring/health_check.log`
- `/var/log/anon/monitoring/performance.log`
- `/var/log/anon/monitoring/network.log`
- `/var/log/anon/monitoring/backup.log`
- `/var/log/anon/monitoring/report.log`

## Maintenance

The role includes maintenance capabilities:
- Log cleanup and rotation
- Backup management
- Performance optimization
- Alert management
- Report generation

## Notes

- Monitoring is designed to be lightweight and non-intrusive
- All scripts include error handling and logging
- Thresholds are configurable per environment
- Alerts can be customized for different severity levels
- Reports include historical trends and analysis

## License

MIT

## Author Information

This role was created for the Anon Relay Deployment project.