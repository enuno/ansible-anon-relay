# Preflight Checks Role

This role performs system requirement validation before deploying Anon relays to ensure the target system meets all necessary requirements.

## Requirements

- Ansible 2.15+
- Target systems: Ubuntu 20.04+, Debian 10+, Fedora 35+
- Python 3.8+ on target systems

## Role Variables

### Default Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `preflight_min_ram_mb` | `512` | Minimum RAM required (MB) |
| `preflight_min_disk_gb` | `5` | Minimum disk space required (GB) |
| `preflight_min_cpu_cores` | `1` | Minimum CPU cores required |
| `preflight_check_network` | `true` | Enable network connectivity checks |
| `preflight_check_dns` | `true` | Enable DNS resolution checks |
| `preflight_check_internet` | `true` | Enable internet connectivity checks |
| `preflight_check_docker` | `true` | Enable Docker installation checks |
| `preflight_docker_version_min` | `"20.10.0"` | Minimum Docker version required |
| `preflight_check_ports` | `true` | Enable port availability checks |
| `preflight_required_ports` | `[22, 9001]` | List of required ports |
| `preflight_exit_legal_required` | `false` | Require legal acknowledgment for exit relays |
| `preflight_fail_on_warnings` | `false` | Fail playbook on warnings |
| `preflight_continue_on_errors` | `false` | Continue playbook on errors |

## Checks Performed

### System Requirements
- **RAM**: Verifies minimum RAM available
- **Disk Space**: Checks available disk space on root partition
- **CPU**: Validates minimum CPU cores
- **OS Compatibility**: Ensures supported operating system

### Network Checks
- **DNS Resolution**: Tests DNS functionality
- **Internet Connectivity**: Verifies internet access
- **IPv6 Support**: Checks IPv6 availability (if enabled)
- **CGNAT Detection**: Warns about Carrier-Grade NAT

### Docker Checks
- **Installation**: Verifies Docker is installed
- **Version**: Checks minimum Docker version
- **Daemon**: Ensures Docker daemon is running
- **Compose**: Validates Docker Compose availability

### Port Checks
- **Availability**: Tests if required ports are available
- **ORPort Reachability**: Checks ORPort connectivity
- **Port Forwarding**: Validates port forwarding setup

### Relay-Specific Checks
- **Legal Acknowledgment**: Requires legal acknowledgment for exit relays
- **Contact Information**: Validates contact email for exit relays
- **Reverse DNS**: Checks reverse DNS for exit relays
- **Network Configuration**: Validates SOCKS proxy network settings
- **Nickname Format**: Ensures relay nickname meets requirements
- **Bandwidth Configuration**: Verifies bandwidth settings

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Run preflight checks
  hosts: relays
  become: yes
  roles:
    - preflight_checks
  vars:
    preflight_min_ram_mb: 1024
    preflight_min_disk_gb: 10
    preflight_fail_on_warnings: true
```

## Example with Custom Requirements

```yaml
---
- name: Run preflight checks for exit relay
  hosts: exit_relays
  become: yes
  roles:
    - preflight_checks
  vars:
    preflight_exit_legal_required: true
    preflight_required_ports:
      - 22    # SSH
      - 9001  # ORPort
      - 80    # DirPort
    preflight_fail_on_warnings: true
```

## Tags

- `preflight` - All preflight check tasks
- `system` - System requirement checks
- `network` - Network connectivity checks
- `docker` - Docker installation checks
- `ports` - Port availability checks
- `relay` - Relay-specific checks
- `summary` - Summary and reporting

## License

MIT

## Author Information

This role was created for the Anon Relay Deployment project.