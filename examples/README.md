# Examples Directory

This directory contains example configurations and use cases for the Ansible Anon Relay Deployment project.

## Overview

These examples demonstrate common deployment scenarios and configuration patterns. Copy and customize them for your specific needs.

## Table of Contents

- [Basic Deployments](#basic-deployments)
- [Advanced Configurations](#advanced-configurations)
- [Security Configurations](#security-configurations)
- [Multi-Region Deployments](#multi-region-deployments)
- [Testing Configurations](#testing-configurations)

---

## Basic Deployments

### Single Standard Relay

**File:** `single-standard-relay.ini`

```ini
[standard_relays]
relay.example.com ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/relay_key

[standard_relays:vars]
anon_relay_nickname="MyHomeRelay"
anon_relay_contact="operator@example.com"
anon_relay_bandwidth_rate="100 MBytes"
anon_relay_bandwidth_burst="200 MBytes"
```

**Deploy:**
```bash
ansible-playbook -i examples/single-standard-relay.ini deploy_standard.yml
```

---

### Home Lab Setup (Multiple Relays)

**File:** `homelab-setup.ini`

```ini
[standard_relays]
pi4-relay.local ansible_host=192.168.1.100 ansible_user=pi
nas-relay.local ansible_host=192.168.1.50 ansible_user=admin

[standard_relays:vars]
anon_relay_bandwidth_rate="50 MBytes"
anon_ipv6_enabled=false
anon_ssh_port=2222
```

**Variable File:** `homelab-vars.yml`

```yaml
---
# Home lab specific configuration
anon_relay_contact: "homelab@example.com"
anon_firewall_enabled: true
anon_monitoring_enabled: true

# Lower resource usage for home environment
anon_max_mem_in_mb: 256
anon_relay_bandwidth_rate: "50 MBytes"
anon_relay_bandwidth_burst: "100 MBytes"
```

---

### SOCKS Proxy for Local Network

**File:** `local-socks-proxy.ini`

```ini
[socks_proxies]
socks.local ansible_host=192.168.1.10 ansible_user=ubuntu

[socks_proxies:vars]
anon_socks_port=9050
anon_socks_policy="accept 192.168.1.0/24"
anon_relay_nickname="HomeSocksProxy"
```

---

## Advanced Configurations

### Multi-Region Deployment

**File:** `multi-region.ini`

```ini
[us_east_relays]
relay-us-east-1.example.com
relay-us-east-2.example.com

[us_west_relays]
relay-us-west-1.example.com
relay-us-west-2.example.com

[eu_relays]
relay-eu-1.example.com
relay-eu-2.example.com

[standard_relays:children]
us_east_relays
us_west_relays
eu_relays

[us_east_relays:vars]
anon_relay_nickname_suffix="USE"

[us_west_relays:vars]
anon_relay_nickname_suffix="USW"

[eu_relays:vars]
anon_relay_nickname_suffix="EU"
```

---

### Exit Relay with Reduced Policy

**File:** `exit-relay-reduced.yml`

```yaml
---
# WARNING: Operating an exit relay has legal implications
# Consult legal counsel before deploying

# Exit relay configuration
anon_relay_type: exit
anon_relay_nickname: "MyExitRelay"
anon_relay_contact: "abuse@example.com"

# Use reduced exit policy (recommended)
anon_exit_policy_type: "reduced"

# Enhanced security for exit relays
anon_ssh_port: 2222
anon_ssh_root_login: false
anon_ssh_password_auth: false
anon_fail2ban_enabled: true
anon_fail2ban_maxretry: 3

# Higher bandwidth for exit relays
anon_relay_bandwidth_rate: "500 MBytes"
anon_relay_bandwidth_burst: "1000 MBytes"

# IPv6 support
anon_ipv6_enabled: true
```

---

### Family Configuration (Multiple Related Relays)

**File:** `relay-family.yml`

```yaml
---
# Configure multiple relays as a family
# Prevents them from being used in same circuit

anon_relay_myfamily:
  - "ABCD1234567890ABCD1234567890ABCD1234567890"  # Relay 1 fingerprint
  - "EFGH1234567890EFGH1234567890EFGH1234567890"  # Relay 2 fingerprint
  - "IJKL1234567890IJKL1234567890IJKL1234567890"  # Relay 3 fingerprint

# Get fingerprints after initial deployment:
# docker exec anon-relay cat /var/lib/anon/fingerprint
```

---

## Security Configurations

### High Security Configuration

**File:** `high-security.yml`

```yaml
---
# High security configuration for production relays

# SSH Hardening
anon_ssh_port: 2222
anon_ssh_root_login: false
anon_ssh_password_auth: false
anon_ssh_permit_root_login: "no"
anon_ssh_password_authentication: "no"
anon_ssh_pubkey_authentication: "yes"
anon_ssh_max_auth_tries: 3
anon_ssh_max_sessions: 2

# Fail2Ban Configuration
anon_fail2ban_enabled: true
anon_fail2ban_maxretry: 3
anon_fail2ban_findtime: 600
anon_fail2ban_bantime: 3600

# Firewall Strictness
anon_firewall_enabled: true
anon_firewall_default_policy: deny

# Automated Updates
anon_unattended_upgrades: true
anon_auto_reboot: false

# Monitoring
anon_monitoring_enabled: true
anon_log_level: notice
```

---

### Ansible Vault Example

**File:** `vault-secrets.yml` (encrypted)

```yaml
---
# Store sensitive data encrypted
# Create: ansible-vault create examples/vault-secrets.yml
# Edit: ansible-vault edit examples/vault-secrets.yml

vault_anon_relay_contact: "secret-email@example.com"
vault_ssh_port: 2222
vault_api_key: "your-secret-api-key"
```

**Reference in Playbook:**

```yaml
# group_vars/all.yml
anon_relay_contact: "{{ vault_anon_relay_contact }}"
anon_ssh_port: "{{ vault_ssh_port }}"
```

**Deploy:**
```bash
ansible-playbook -i inventory.ini site.yml --ask-vault-pass
```

---

## Multi-Region Deployments

### Geographic Distribution

**File:** `geo-distributed.ini`

```ini
[na_relays]
relay-na-1.example.com location="North America"
relay-na-2.example.com location="North America"

[eu_relays]
relay-eu-1.example.com location="Europe"
relay-eu-2.example.com location="Europe"

[asia_relays]
relay-asia-1.example.com location="Asia"
relay-asia-2.example.com location="Asia"

[standard_relays:children]
na_relays
eu_relays
asia_relays

[na_relays:vars]
timezone="America/New_York"
ntp_servers=['time.nist.gov']

[eu_relays:vars]
timezone="Europe/London"
ntp_servers=['ntp.ubuntu.com']

[asia_relays:vars]
timezone="Asia/Tokyo"
ntp_servers=['ntp.nict.jp']
```

---

## Testing Configurations

### Test Environment

**File:** `test-environment.ini`

```ini
[test_relays]
test-relay-1.local ansible_host=192.168.56.10 ansible_user=vagrant
test-relay-2.local ansible_host=192.168.56.11 ansible_user=vagrant

[test_relays:vars]
# Minimal resources for testing
anon_relay_bandwidth_rate="10 MBytes"
anon_max_mem_in_mb=128

# Disable production features
anon_monitoring_enabled=false
anon_fail2ban_enabled=false

# Fast deployment
preflight_check_network=false
```

**Vagrantfile for Testing:**

```ruby
# Vagrantfile
Vagrant.configure("2") do |config|
  (1..2).each do |i|
    config.vm.define "test-relay-#{i}" do |relay|
      relay.vm.box = "ubuntu/jammy64"
      relay.vm.hostname = "test-relay-#{i}"
      relay.vm.network "private_network", ip: "192.168.56.#{9+i}"

      relay.vm.provider "virtualbox" do |vb|
        vb.memory = "512"
        vb.cpus = 1
      end
    end
  end
end
```

---

## Usage Patterns

### Pattern 1: Development → Staging → Production

```bash
# Development
ansible-playbook -i examples/dev-inventory.ini site.yml --check

# Staging
ansible-playbook -i examples/staging-inventory.ini site.yml --check
ansible-playbook -i examples/staging-inventory.ini site.yml

# Production (after validation)
ansible-playbook -i examples/prod-inventory.ini site.yml --check
ansible-playbook -i examples/prod-inventory.ini site.yml
```

### Pattern 2: Gradual Rollout

```bash
# Deploy to first relay
ansible-playbook -i inventory.ini site.yml --limit relay-1.example.com

# Verify successful deployment
ansible relay-1.example.com -i inventory.ini -m shell -a "docker ps | grep anon"

# Deploy to remaining relays (serial execution)
ansible-playbook -i inventory.ini site.yml --limit "relays:!relay-1.example.com" -f 1
```

### Pattern 3: Update Existing Deployment

```bash
# Update specific variable
ansible-playbook -i inventory.ini site.yml \
  --tags configure \
  --extra-vars "anon_relay_bandwidth_rate='200 MBytes'"

# Update Docker image
ansible-playbook -i inventory.ini update.yml
```

---

## Example Directory Structure

Organize your examples like this:

```
examples/
├── README.md                          # This file
├── inventories/
│   ├── production.ini                 # Production inventory
│   ├── staging.ini                    # Staging inventory
│   └── development.ini                # Dev/test inventory
├── group_vars/
│   ├── production/
│   │   ├── vars.yml                   # Production variables
│   │   └── vault.yml                  # Encrypted secrets
│   ├── staging/
│   │   └── vars.yml
│   └── development/
│       └── vars.yml
├── playbooks/
│   ├── deploy-homelab.yml            # Custom playbooks
│   └── deploy-exit-relay.yml
└── scripts/
    ├── backup-relay.sh                # Helper scripts
    └── monitor-bandwidth.sh
```

---

## Best Practices

1. **Version Control:** Keep examples in git, but exclude actual inventory files
   ```gitignore
   examples/inventories/production.ini
   examples/group_vars/*/vault.yml
   ```

2. **Documentation:** Document why specific values are chosen
   ```yaml
   # Use 100 MBytes because ISP limits to 100Mbps
   anon_relay_bandwidth_rate: "100 MBytes"
   ```

3. **Testing:** Always test in dev/staging before production
   ```bash
   ansible-playbook -i examples/dev.ini site.yml --check
   ```

4. **Security:** Use Ansible Vault for sensitive data
   ```bash
   ansible-vault encrypt examples/secrets.yml
   ```

---

## Contributing Examples

Have a useful configuration? Contribute it!

1. Create example file with descriptive name
2. Add inline comments explaining configuration
3. Test the configuration
4. Document any prerequisites
5. Submit PR with example + documentation

---

## Additional Resources

- [Main README](../README.md) - Project overview
- [Configuration Guide](../docs/configuration.md) - Detailed variable reference
- [Troubleshooting](../docs/troubleshooting.md) - Common issues
- [CONTRIBUTING](../CONTRIBUTING.md) - Contribution guidelines

---

**Need help?** Open an issue or discussion on GitHub.
