# Anon Relay Standard Role

This role configures a standard (middle/guard) Anon relay that routes traffic through the Anyone network without exposing your IP address to destination sites.

## Requirements

- Ansible 2.15+
- `anon_relay_base` role (must be run first)
- Target systems: Ubuntu 20.04+, Debian 10+, Fedora 35+
- Python 3.8+ on target systems

## Role Variables

### Default Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_relay_type` | `"standard"` | Relay type identifier |
| `anon_relay_nickname` | `"MyStandardRelay"` | Relay nickname (max 19 chars) |
| `anon_relay_contact` | `"operator@example.com"` | Contact email for relay operator |
| `anon_relay_or_port` | `9001` | ORPort for incoming connections |
| `anon_relay_socks_port` | `0` | SOCKS port (disabled for relay) |
| `anon_relay_dir_port` | `0` | Directory port (not used for standard) |
| `anon_relay_bandwidth_rate` | `"100 MBytes"` | Bandwidth rate limit |
| `anon_relay_bandwidth_burst` | `"200 MBytes"` | Bandwidth burst limit |
| `anon_ipv6_enabled` | `true` | Enable IPv6 support |
| `anon_ipv6_only` | `false` | IPv6-only mode |
| `anon_relay_myfamily` | `[]` | List of related relay fingerprints |
| `anon_port_forward_required` | `true` | Require port forwarding |
| `anon_firewall_enabled` | `true` | Enable firewall configuration |
| `anon_ssh_port` | `22` | SSH port number |
| `anon_ssh_root_login` | `false` | Allow root SSH login |
| `anon_ssh_password_auth` | `false` | Allow password authentication |
| `anon_fail2ban_enabled` | `true` | Enable Fail2Ban protection |

### Security Configuration

The role includes comprehensive security hardening:

- **Fail2Ban**: Protects against brute force attacks on SSH
- **SSH Hardening**: Configures secure SSH settings
- **Firewall**: UFW rules for ORPort and SSH access
- **Login Banner**: Warning message for unauthorized users

### Firewall Rules

The role configures UFW with the following rules:
- Allow SSH access on configured port
- Allow ORPort (9001) for relay traffic
- Allow IPv6 ORPort if IPv6 is enabled
- Deny all other incoming connections

## Dependencies

- `anon_relay_base` role

## Example Playbook

```yaml
---
- name: Deploy standard Anon relay
  hosts: standard_relays
  become: yes
  roles:
    - docker_setup
    - anon_relay_base
    - anon_relay_standard
  vars:
    anon_relay_nickname: "MyRelay"
    anon_relay_contact: "operator@example.com"
    anon_relay_bandwidth_rate: "150 MBytes"
    anon_relay_bandwidth_burst: "300 MBytes"
```

## Example with Custom Settings

```yaml
---
- name: Deploy standard relay with custom settings
  hosts: standard_relays
  become: yes
  roles:
    - docker_setup
    - anon_relay_base
    - anon_relay_standard
  vars:
    anon_relay_nickname: "CustomRelay"
    anon_relay_contact: "relay@example.com"
    anon_relay_bandwidth_rate: "500 MBytes"
    anon_relay_bandwidth_burst: "1000 MBytes"
    anon_ipv6_enabled: true
    anon_relay_myfamily:
      - "ABCD1234567890ABCD1234567890ABCD1234567890"
      - "EFGH1234567890EFGH1234567890EFGH1234567890"
    anon_ssh_port: 2222
    anon_fail2ban_enabled: true
```

## Configuration Files

### anonrc_standard.j2
Standard relay configuration template with:
- Basic relay settings (nickname, contact, ports)
- Bandwidth configuration
- IPv6 support
- MyFamily configuration
- No exit policy (not an exit relay)

## Security Features

### SSH Hardening
- Custom SSH port (configurable)
- Disabled root login (configurable)
- Disabled password authentication (configurable)
- Key-based authentication only
- Connection timeouts and limits
- User access restrictions
- Warning banner

### Fail2Ban Protection
- SSH brute force protection
- Configurable ban time and retry limits
- Automatic IP blocking
- Log monitoring

### Firewall Configuration
- UFW-based firewall rules
- Port-specific access control
- IPv4 and IPv6 support
- Default deny policy

## Tags

- `anon` - All Anon-related tasks
- `standard` - Standard relay specific tasks
- `configure` - Configuration tasks
- `validate` - Validation tasks
- `security` - Security hardening tasks
- `firewall` - Firewall configuration
- `ipv6` - IPv6 configuration
- `myfamily` - MyFamily configuration

## Handlers

- `restart anon relay` - Restarts the Anon container
- `restart fail2ban` - Restarts Fail2Ban service
- `restart ssh` - Restarts SSH service

## Validation

The role includes comprehensive validation:
- Configuration file syntax check
- ORPort connectivity verification
- Relay descriptor publication check
- Bandwidth configuration validation
- Security settings verification

## Notes

- This role is designed for standard (middle/guard) relays
- Your IP address will NOT be visible to destination sites
- Suitable for home or datacenter hosting
- Lower maintenance requirements compared to exit relays
- Earns ANYONE tokens for bandwidth contribution

## License

MIT

## Author Information

This role was created for the Anon Relay Deployment project.