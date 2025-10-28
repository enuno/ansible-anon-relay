# Security Policy

## Supported Versions

This section outlines which versions of the Ansible Anon Relay Deployment project are currently supported with security updates.

| Version | Supported          | Status     |
| ------- | ------------------ | ---------- |
| 1.0.x   | :white_check_mark: | Current    |
| < 1.0   | :x:                | Development|

**Note:** As this project is currently in development (pre-1.0), security updates will be applied to the `develop` and `main` branches.

---

## Security Considerations

### Overview

This project deploys Anyone Protocol relay nodes that handle network traffic. Security is paramount. This document outlines security considerations, best practices, and our vulnerability disclosure policy.

### What This Project Does

- Automates deployment of Anyone Protocol relay infrastructure
- Configures Docker containers for relay operations
- Manages firewall rules and system security hardening
- Handles sensitive configuration (relay keys, credentials)

### Security Features Implemented

1. **SSH Hardening**
   - Configurable SSH port
   - Disabled root login (optional)
   - Key-based authentication enforced
   - Fail2Ban protection against brute force

2. **Firewall Configuration**
   - UFW-based firewall rules
   - Port-specific access control
   - Default deny policy
   - IPv4 and IPv6 support

3. **File System Security**
   - Restrictive file permissions (600/700 for sensitive files)
   - Dedicated anond user (UID 100, GID 101)
   - Isolated Docker containers
   - No hardcoded credentials

4. **Secrets Management**
   - Ansible Vault support for sensitive variables
   - Secrets excluded from version control (`.gitignore`)
   - `no_log: true` for sensitive tasks
   - Environment variable support

5. **Docker Security**
   - Containers run as non-root user
   - Read-only root filesystem where possible
   - Resource limits (CPU, memory)
   - Network isolation

---

## Reporting a Vulnerability

### Please DO NOT Report Security Vulnerabilities Publicly

**We take security seriously.** If you discover a security vulnerability, please report it responsibly.

### How to Report

**Preferred Method: Private Security Advisory**

1. Go to the [Security tab](https://github.com/yourusername/ansible-anon-relay/security)
2. Click "Report a vulnerability"
3. Fill out the form with details

**Alternative Method: Email**

If GitHub Security Advisories are not available, email:
- **Email:** security@example.com
- **Subject:** [SECURITY] Ansible Anon Relay Vulnerability Report

### What to Include

Please provide:

1. **Description** - What is the vulnerability?
2. **Impact** - What can an attacker do?
3. **Affected Components** - Which files/roles/playbooks?
4. **Attack Scenario** - Step-by-step exploit (if possible)
5. **Suggested Fix** - Ideas for remediation (optional)
6. **Your Details** - Name/handle for acknowledgment (optional)

**Example Report:**
```
Subject: [SECURITY] Privilege Escalation in docker_setup Role

Description:
The docker_setup role creates a user with sudo privileges without
password verification, which could allow local privilege escalation.

Impact:
An attacker with access to the ansible_user account could execute
arbitrary commands as root without authentication.

Affected Components:
- roles/docker_setup/tasks/users.yml (line 45-52)

Attack Scenario:
1. SSH into target as ansible_user
2. Run: sudo su -
3. Gain root access without password

Suggested Fix:
Remove NOPASSWD directive or restrict sudo commands to Docker-specific
operations only.

Reporter: YourName (optional acknowledgment)
```

### Response Timeline

- **Initial Response:** Within 48 hours
- **Impact Assessment:** Within 7 days
- **Fix Development:** Varies by severity (see below)
- **Public Disclosure:** 90 days after fix (coordinated disclosure)

### Severity Classification

We use [CVSS v3.1](https://www.first.org/cvss/) for severity scoring:

| Severity | CVSS Score | Response Time | Example |
|----------|-----------|---------------|---------|
| Critical | 9.0-10.0 | 1-7 days | Remote code execution |
| High | 7.0-8.9 | 7-30 days | Privilege escalation |
| Medium | 4.0-6.9 | 30-90 days | Information disclosure |
| Low | 0.1-3.9 | 90+ days | Denial of service |

### What to Expect

1. **Acknowledgment** - We'll confirm receipt of your report
2. **Investigation** - We'll analyze and reproduce the issue
3. **Fix Development** - We'll create and test a patch
4. **Disclosure** - We'll coordinate public disclosure with you
5. **Credit** - We'll acknowledge your contribution (if desired)

---

## Security Best Practices for Users

### Deployment Security

#### 1. Use Ansible Vault for Secrets

```bash
# Create encrypted vault file
ansible-vault create group_vars/all/vault.yml

# Add sensitive variables
vault_anon_relay_contact: "your@email.com"
vault_api_key: "secret-key"

# Run with vault password
ansible-playbook site.yml --ask-vault-pass
```

#### 2. Secure SSH Access

```yaml
# group_vars/all.yml
anon_ssh_port: 2222                    # Non-standard port
anon_ssh_root_login: false            # Disable root login
anon_ssh_password_auth: false         # Key-only authentication
anon_fail2ban_enabled: true           # Enable brute-force protection
```

#### 3. Firewall Configuration

```yaml
# Enable firewall on all hosts
anon_firewall_enabled: true

# Restrict access to specific IPs (optional)
anon_ssh_allowed_ips:
  - "192.168.1.0/24"
  - "10.0.0.1/32"
```

#### 4. Regular Updates

```bash
# Update relay software regularly
ansible-playbook -i inventory.ini update.yml

# Update underlying OS
ansible all -i inventory.ini -m apt -a "upgrade=dist" -b
```

#### 5. Monitoring and Auditing

```yaml
# Enable monitoring
anon_monitoring_enabled: true

# Check logs regularly
ansible all -i inventory.ini -m shell -a "tail -100 /var/log/anon/notices.log"
```

#### 6. Principle of Least Privilege

```yaml
# Don't run entire playbook as root
# Use become only where needed
- name: Non-privileged task
  debug:
    msg: "This doesn't need root"

- name: Privileged task
  apt:
    name: docker-ce
    state: present
  become: yes
```

### Exit Relay Specific Security

**⚠️ WARNING:** Operating an exit relay has legal implications.

```yaml
# Only enable exit relay if you understand the risks
anon_relay_type: exit

# Configure reduced exit policy
anon_exit_policy_type: "reduced"  # Not "full"

# Set up abuse contact
anon_relay_contact: "abuse@example.com"

# Consider legal jurisdiction
# Consult legal counsel before operating exit relays
```

### Securing the Control Node

Your Ansible control node (laptop/server) is critical:

1. **Encrypt SSH Keys**
   ```bash
   # Create password-protected SSH key
   ssh-keygen -t ed25519 -C "ansible@control" -f ~/.ssh/ansible_key
   ```

2. **Restrict SSH Key Permissions**
   ```bash
   chmod 600 ~/.ssh/ansible_key
   chmod 644 ~/.ssh/ansible_key.pub
   ```

3. **Use SSH Agent**
   ```bash
   # Add key to agent (enter passphrase once)
   ssh-add ~/.ssh/ansible_key
   ```

4. **Protect Vault Passwords**
   ```bash
   # Store vault password securely (not in repo)
   echo "VaultPassword123" > ~/.ansible_vault_pass
   chmod 600 ~/.ansible_vault_pass

   # Reference in ansible.cfg
   # vault_password_file = ~/.ansible_vault_pass
   ```

---

## Known Security Limitations

### Current Limitations

1. **No Built-in Certificate Management**
   - Manual SSL/TLS certificate setup required
   - Consider integrating Let's Encrypt in future versions

2. **Basic Firewall Rules**
   - UFW provides basic protection
   - Advanced users may want iptables/nftables

3. **Limited Intrusion Detection**
   - Fail2Ban provides basic protection
   - Consider OSSEC/Wazuh for production

4. **Docker Socket Access**
   - Users in docker group have root-equivalent access
   - Limit docker group membership

### Mitigations

These limitations are tracked in our issue tracker. Contributions welcome!

---

## Security Updates

### Notification Methods

Subscribe to security updates:

1. **GitHub Watch** - Watch repository for security advisories
2. **Release Notes** - Check CHANGELOG.md for security fixes
3. **RSS Feed** - Subscribe to release feed
4. **Discord** - Join announcement channel

### Applying Security Updates

```bash
# 1. Backup current configuration
ansible-playbook -i inventory.ini backup.yml

# 2. Pull latest code
git pull origin main

# 3. Review CHANGELOG.md for breaking changes
cat CHANGELOG.md

# 4. Test in dry-run mode
ansible-playbook -i inventory.ini site.yml --check

# 5. Apply updates
ansible-playbook -i inventory.ini site.yml

# 6. Verify deployment
ansible-playbook -i inventory.ini verify.yml
```

---

## Compliance and Legal

### Data Protection

- **Personal Data:** Relay contact emails are publicly visible
- **GDPR:** Consider privacy implications for EU deployments
- **Logs:** Relay logs may contain IP addresses

### Legal Considerations

- **Exit Relays:** Understand local laws regarding exit traffic
- **Liability:** Review [Anyone Protocol legal resources](https://anyone.io/legal)
- **Terms of Service:** Comply with ISP and datacenter ToS

### Recommended Actions

1. **Consult Legal Counsel** - Especially for exit relays
2. **Abuse Contact** - Set up dedicated abuse email
3. **Terms Acceptance** - Set `anon_accept_terms: true`
4. **Documentation** - Keep records of configuration

---

## Security Checklist

Before deploying to production:

- [ ] Ansible Vault configured for secrets
- [ ] SSH keys are password-protected
- [ ] Non-standard SSH port configured
- [ ] Root login disabled
- [ ] Firewall enabled and configured
- [ ] Fail2Ban enabled
- [ ] Regular updates scheduled
- [ ] Monitoring enabled
- [ ] Backups configured
- [ ] Incident response plan documented
- [ ] Legal review completed (for exit relays)
- [ ] Abuse contact configured

---

## Resources

### External Security Resources

- [Ansible Security Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#security)
- [Docker Security](https://docs.docker.com/engine/security/)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Anyone Protocol Security](https://docs.anyone.io/security)

### Security Tools

- [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
- [Fail2Ban](https://www.fail2ban.org/)
- [UFW](https://help.ubuntu.com/community/UFW)
- [Docker Bench Security](https://github.com/docker/docker-bench-security)

---

## Acknowledgments

We thank the security researchers who have responsibly disclosed vulnerabilities:

- *(No vulnerabilities reported yet)*

---

## Questions?

For general security questions (not vulnerabilities):
- **GitHub Discussions:** [Security Category](https://github.com/yourusername/ansible-anon-relay/discussions/categories/security)
- **Documentation:** See docs/security.md
- **Discord:** #security channel

**Thank you for helping keep this project secure!** 🔒
