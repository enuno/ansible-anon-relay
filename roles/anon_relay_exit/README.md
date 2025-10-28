# Anon Relay Exit Role

This role configures an exit Anon relay that allows traffic to exit the Anyone Protocol network to the regular internet. **WARNING: Your IP address will be visible to destination sites!**

## Requirements

- Ansible 2.15+
- `anon_relay_base` role (must be run first)
- Target systems: Ubuntu 20.04+, Debian 10+, Fedora 35+
- Python 3.8+ on target systems
- **Legal acknowledgment required** (`anon_exit_legal_acknowledged: true`)

## Role Variables

### Default Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_relay_type` | `"exit"` | Relay type identifier |
| `anon_exit_legal_acknowledged` | `false` | **REQUIRED**: Legal acknowledgment |
| `anon_relay_nickname` | `"MyExitRelay"` | Relay nickname (max 19 chars) |
| `anon_relay_contact` | `"abuse@example.com"` | Contact email (non-personal) |
| `anon_relay_or_port` | `9001` | ORPort for incoming connections |
| `anon_relay_dir_port` | `80` | Directory port for exit relays |
| `anon_relay_socks_port` | `0` | SOCKS port (disabled for exit) |
| `anon_relay_exit_relay` | `1` | Enable exit relay functionality |
| `anon_relay_ipv6_exit` | `0` | Disable IPv6 exit (recommended) |
| `anon_relay_bandwidth_rate` | `"200 MBytes"` | Bandwidth rate limit |
| `anon_relay_bandwidth_burst` | `"400 MBytes"` | Bandwidth burst limit |

### Exit Policy Configuration

The role configures a restrictive exit policy that blocks high-risk ports:

```yaml
anon_relay_exit_policy:
  - "reject *:25"      # SMTP
  - "reject *:587"     # SMTP Submission
  - "reject *:465"     # SMTPS
  - "reject *:2525"    # SMTP Alternative
  - "reject *:3389"    # RDP
  - "reject *:23"      # Telnet
  - "reject *:3128"    # HTTP Proxy
  - "reject *:5900"    # VNC
  - "reject *:9999"    # Custom high-risk
  - "accept *:*"       # Accept everything else
```

### DoS Mitigation (Required)

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_dos_circuit_creation_enabled` | `true` | Enable circuit creation limits |
| `anon_dos_circuit_creation_burst` | `30` | Circuit creation burst limit |
| `anon_dos_circuit_creation_rate` | `3` | Circuit creation rate limit |
| `anon_dos_connection_enabled` | `true` | Enable connection limits |
| `anon_dos_stream_creation_enabled` | `true` | Enable stream creation limits |

### Exit Notice Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_exit_notice_enabled` | `true` | Enable exit notice website |
| `anon_exit_notice_email` | `"abuse@example.com"` | Abuse contact email |
| `anon_exit_notice_title` | `"Anyone Protocol Exit Relay"` | Website title |
| `anon_reverse_dns_hostname` | `"anon-exit-1.example.com"` | Reverse DNS hostname |

### Security Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_ssh_port` | `52231` | Non-standard SSH port |
| `anon_ssh_root_login` | `false` | Disable root SSH login |
| `anon_ssh_password_auth` | `false` | Disable password authentication |
| `anon_fail2ban_enabled` | `true` | Enable Fail2Ban protection |
| `anon_disk_encryption_required` | `true` | Document encryption requirement |

## Dependencies

- `anon_relay_base` role
- `nginx` (for exit notice website)

## Example Playbook

```yaml
---
- name: Deploy exit Anon relay
  hosts: exit_relays
  become: yes
  roles:
    - docker_setup
    - anon_relay_base
    - anon_relay_exit
  vars:
    anon_exit_legal_acknowledged: true
    anon_relay_nickname: "MyExitRelay"
    anon_relay_contact: "abuse@example.com"
    anon_relay_bandwidth_rate: "500 MBytes"
    anon_relay_bandwidth_burst: "1000 MBytes"
    anon_reverse_dns_hostname: "exit.example.com"
```

## Legal Requirements

**CRITICAL**: You must set `anon_exit_legal_acknowledged: true` before deployment.

By running an exit relay, you are responsible for:
- All traffic that exits through your relay
- Legal compliance in your jurisdiction
- Handling abuse complaints
- Potential legal liability

## Security Features

### Enhanced Security
- Non-standard SSH port (52231)
- Fail2Ban protection
- Restrictive exit policy
- DoS mitigation enabled
- Disk encryption recommended

### Exit Notice Website
- Professional abuse complaint handling
- Clear explanation of relay purpose
- Contact information for issues
- Legal disclaimers

### Firewall Configuration
- UFW-based firewall rules
- ORPort and DirPort access
- IPv4 and IPv6 support
- Default deny policy

## Configuration Files

### anonrc_exit.j2
Exit relay configuration template with:
- Exit relay settings
- Restrictive exit policy
- DoS mitigation configuration
- Bandwidth limits
- IPv6 configuration

### exit_notice.html.j2
Professional exit notice website with:
- Relay information
- Legal disclaimers
- Contact information
- Exit policy explanation
- Abuse reporting instructions

### abuse_template.txt.j2
Abuse complaint handling template with:
- Instructions for reporting abuse
- Technical information
- Legal disclaimers
- Contact details

## Tags

- `anon` - All Anon-related tasks
- `exit` - Exit relay specific tasks
- `legal` - Legal validation tasks
- `configure` - Configuration tasks
- `notice` - Exit notice tasks
- `dos` - DoS mitigation tasks
- `validate` - Validation tasks
- `firewall` - Firewall configuration
- `ipv6` - IPv6 configuration
- `myfamily` - MyFamily configuration
- `webserver` - Web server tasks

## Handlers

- `restart anon relay` - Restarts the Anon container
- `restart nginx` - Restarts the web server

## Validation

The role includes comprehensive validation:
- Legal acknowledgment verification
- Configuration file syntax check
- ORPort and DirPort connectivity
- Exit policy validation
- DoS mitigation verification
- Reverse DNS check
- Website accessibility test

## Warnings

⚠️ **IMPORTANT WARNINGS**:

1. **IP Visibility**: Your IP address will be visible to destination sites
2. **Legal Liability**: You are responsible for all traffic exiting your relay
3. **Abuse Complaints**: You must handle abuse complaints professionally
4. **Resource Usage**: Exit relays require more resources than standard relays
5. **Security Risk**: Higher security risk due to traffic visibility

## Notes

- This role is designed for experienced operators
- Requires understanding of legal implications
- Higher maintenance requirements
- Earns more ANYONE tokens than standard relays
- Essential for Anyone Protocol network functionality

## License

MIT

## Author Information

This role was created for the Anon Relay Deployment project.