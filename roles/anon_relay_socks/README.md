# Anon Relay SOCKS Role

This role configures a SOCKS proxy that allows local network devices to use the Anyone Protocol network. **EXPERIMENTAL**: This feature may not work as expected in all scenarios.

## Requirements

- Ansible 2.15+
- `anon_relay_base` role (must be run first)
- Target systems: Ubuntu 20.04+, Debian 10+, Fedora 35+
- Python 3.8+ on target systems
- Local network access

## Role Variables

### Default Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_relay_type` | `"socks"` | Relay type identifier |
| `anon_socks_port` | `9050` | SOCKS proxy port |
| `anon_socks_bind_address` | `"192.168.1.10"` | LAN IP to bind to |
| `anon_socks_policy_accept` | `"192.168.1.0/24"` | Allowed network range |
| `anon_relay_nickname` | `"MySOCKSProxy"` | Relay nickname |
| `anon_relay_contact` | `"admin@example.com"` | Contact email |
| `anon_relay_bandwidth_rate` | `"50 MBytes"` | Bandwidth rate limit |
| `anon_relay_bandwidth_burst` | `"100 MBytes"` | Bandwidth burst limit |

### Network Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_port_forward_required` | `false` | No port forwarding needed |
| `anon_ipv6_enabled` | `false` | IPv6 disabled for simplicity |
| `anon_socks_client_config_enabled` | `true` | Generate client config files |
| `anon_socks_client_config_path` | `"/opt/anon/socks-client-config"` | Client config directory |

### Security Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_ssh_port` | `22` | SSH port |
| `anon_ssh_root_login` | `false` | Disable root SSH login |
| `anon_ssh_password_auth` | `false` | Disable password authentication |
| `anon_fail2ban_enabled` | `true` | Enable Fail2Ban protection |

### Experimental Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_socks_experimental` | `true` | Mark as experimental |
| `anon_socks_warning` | `"SOCKS proxy configuration is experimental..."` | Warning message |

## Dependencies

- `anon_relay_base` role

## Example Playbook

```yaml
---
- name: Deploy SOCKS proxy
  hosts: socks_proxies
  become: yes
  roles:
    - docker_setup
    - anon_relay_base
    - anon_relay_socks
  vars:
    anon_socks_bind_address: "192.168.1.100"
    anon_socks_policy_accept: "192.168.1.0/24"
    anon_socks_port: 9050
    anon_relay_nickname: "MySOCKSProxy"
    anon_relay_contact: "admin@example.com"
```

## Client Configuration

The role automatically generates client configuration files:

### Generated Files

- `socks-config.txt` - Basic configuration instructions
- `browser-config.html` - Browser setup guide
- `set-proxy.sh` - System proxy script
- `test-proxy.sh` - Connection test script

### Browser Configuration

**Firefox:**
1. Settings → Network Settings → Manual proxy configuration
2. SOCKS Host: `{{ anon_socks_bind_address }}`
3. Port: `{{ anon_socks_port }}`
4. Type: SOCKS v5
5. Check "Proxy DNS when using SOCKS v5"

**Chrome/Chromium:**
1. Install "Proxy SwitchyOmega" extension
2. Create new profile with SOCKS5 settings
3. Server: `{{ anon_socks_bind_address }}:{{ anon_socks_port }}`

**Safari:**
1. System Preferences → Network → Advanced → Proxies
2. Check "SOCKS Proxy"
3. Server: `{{ anon_socks_bind_address }}:{{ anon_socks_port }}`

### System Configuration

**Linux:**
```bash
export http_proxy=socks5://{{ anon_socks_bind_address }}:{{ anon_socks_port }}
export https_proxy=socks5://{{ anon_socks_bind_address }}:{{ anon_socks_port }}
```

**Windows:**
```cmd
netsh winhttp set proxy proxy-server="socks={{ anon_socks_bind_address }}:{{ anon_socks_port }}"
```

**macOS:**
```bash
networksetup -setsocksfirewallproxy "Wi-Fi" {{ anon_socks_bind_address }} {{ anon_socks_port }}
```

## Configuration Files

### anonrc_socks.j2
SOCKS proxy configuration template with:
- Disabled relay functionality (ORPort=0, DirPort=0)
- SOCKS port configuration
- Network access policy
- Bandwidth limits

### Client Configuration Templates
- `socks_client_config.txt.j2` - Basic instructions
- `browser_config.html.j2` - Browser setup guide
- `system_proxy.sh.j2` - System proxy script
- `test_proxy.sh.j2` - Connection test script

## Security Features

### Network Access Control
- LAN-only access by default
- Configurable network policy
- Firewall rules for SOCKS port
- No external port forwarding required

### Standard Security
- SSH hardening
- Fail2Ban protection
- UFW firewall configuration
- Key-based authentication

## Tags

- `anon` - All Anon-related tasks
- `socks` - SOCKS proxy specific tasks
- `configure` - Configuration tasks
- `client` - Client configuration tasks
- `validate` - Validation tasks
- `firewall` - Firewall configuration
- `network` - Network policy tasks
- `warning` - Warning messages

## Handlers

- `restart anon relay` - Restarts the Anon container

## Validation

The role includes comprehensive validation:
- Configuration file syntax check
- SOCKS port connectivity
- Network policy validation
- Client configuration file verification
- Proxy connection testing

## Testing

Use the generated test script:
```bash
/opt/anon/socks-client-config/test-proxy.sh
```

Or test manually:
```bash
curl --socks5 {{ anon_socks_bind_address }}:{{ anon_socks_port }} https://httpbin.org/ip
```

## Warnings

⚠️ **EXPERIMENTAL FEATURE**:

1. **Not Production Ready**: This feature is experimental
2. **Limited Testing**: May not work in all scenarios
3. **Use at Own Risk**: No guarantee of functionality
4. **Limited Support**: Experimental features have limited support

## Troubleshooting

### Common Issues

1. **Connection Refused**
   - Check if SOCKS proxy is running
   - Verify firewall settings
   - Check network policy configuration

2. **Timeout**
   - Check network connectivity
   - Verify bind address is correct
   - Check if client is in allowed network

3. **DNS Issues**
   - Enable "Proxy DNS" in browser settings
   - Check DNS resolution through proxy

4. **Application Not Working**
   - Some applications don't support SOCKS5
   - Use browser-based testing first

### Debug Commands

```bash
# Check if proxy is running
docker logs anon-relay

# Test port connectivity
nc -z {{ anon_socks_bind_address }} {{ anon_socks_port }}

# Test with curl
curl --socks5 {{ anon_socks_bind_address }}:{{ anon_socks_port }} https://httpbin.org/ip
```

## Notes

- This role is designed for local network use only
- No port forwarding required
- Lower resource requirements than relay types
- Useful for testing and development
- Not suitable for production use

## License

MIT

## Author Information

This role was created for the Anon Relay Deployment project.