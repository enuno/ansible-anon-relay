# AGENTS.md - Universal AI Coding Agent Instructions

## 📋 Project Overview

**Project:** Ansible Playbook for Anyone Protocol Anon Relay Deployment  
**Type:** Infrastructure as Code (IaC) / DevOps Automation  
**Primary Language:** YAML (Ansible Playbooks)  
**Secondary Languages:** Jinja2 (Templates), Shell (Scripts)  
**Target Platforms:** Ubuntu, Debian, Fedora (amd64/arm64)  

This document serves as the **source of truth** for all AI coding agents working on this project. Follow these guidelines strictly to ensure consistency, maintainability, and quality.

---

## 🚨 CRITICAL RULES

### 1. Documentation First
- **ALWAYS** update documentation before, during, and after code changes
- Every role, task, and variable must be documented inline and in README files
- All configuration changes must be logged in CHANGELOG.md
- Use clear, descriptive commit messages following Conventional Commits format

### 2. Idempotency is Mandatory
- All Ansible tasks MUST be idempotent (safe to run multiple times)
- Use `changed_when`, `failed_when`, and `check_mode` appropriately
- Never use shell commands that modify state without proper conditionals
- Test every task multiple times to verify idempotency

### 3. Security First
- **NEVER** commit secrets, passwords, or API keys to version control
- Use Ansible Vault for ALL sensitive data
- Implement principle of least privilege in all tasks
- Validate all external inputs and user-provided variables
- Set restrictive file permissions (600 for secrets, 644 for configs, 755 for executables)

### 4. Cross-Platform Compatibility
- Support Ubuntu, Debian, and Fedora distributions
- Use distribution-agnostic modules when possible (package vs apt/yum/dnf)
- Test on both amd64 and arm64 architectures
- Abstract distribution-specific logic into separate task files

### 5. Error Handling
- Every task that can fail must have proper error handling
- Use `block`/`rescue`/`always` for complex error scenarios
- Provide meaningful error messages that guide users to solutions
- Implement pre-flight checks before destructive operations

---

## 🎯 PROJECT STRUCTURE

### Standard Ansible Directory Layout
```
anon-relay-ansible-deployment/
├── .github/workflows/          # CI/CD pipelines
├── .gitignore                  # Git ignore patterns
├── AGENTS.md                   # This file
├── CLAUDE.md                   # Claude-specific rules
├── COPILOT.md                  # Copilot-specific rules
├── DEVELOPMENT_PLAN.md         # Project development plan
├── README.md                   # User-facing documentation
├── CONTRIBUTING.md             # Contribution guidelines
├── SECURITY.md                 # Security policy
├── CHANGELOG.md                # Version history
├── LICENSE                     # Project license
├── ansible.cfg                 # Ansible configuration
├── requirements.yml            # Galaxy dependencies
├── inventory.ini               # Sample inventory
├── site.yml                    # Main playbook
├── deploy.yml                  # Deployment playbook
├── update.yml                  # Update playbook
├── remove.yml                  # Cleanup playbook
├── group_vars/                 # Group variables
│   ├── all.yml
│   └── relays.yml
├── host_vars/                  # Host-specific variables
├── roles/                      # Ansible roles
│   ├── docker_setup/
│   ├── anon_relay_deploy/
│   └── anon_relay_monitor/
├── examples/                   # Example configurations
├── docs/                       # Additional documentation
└── tests/                      # Test files
```

### Role Structure (Standard)
```
roles/<role_name>/
├── tasks/
│   └── main.yml               # Main task entry point
├── handlers/
│   └── main.yml               # Event handlers
├── templates/
│   └── *.j2                   # Jinja2 templates
├── files/                     # Static files
├── vars/
│   └── main.yml               # Role variables
├── defaults/
│   └── main.yml               # Default variables
├── meta/
│   └── main.yml               # Role metadata
└── README.md                  # Role documentation
```

---

## 🔧 CODING STANDARDS

### YAML Formatting
```yaml
# ✅ CORRECT: Use 2-space indentation
---
- name: Deploy Anon Relay
  hosts: relays
  become: yes
  
  tasks:
    - name: Install Docker
      package:
        name: docker
        state: present

# ❌ INCORRECT: Inconsistent indentation
---
- name: Deploy Anon Relay
    hosts: relays
      become: yes
  tasks:
  - name: Install Docker
    package:
      name: docker
      state: present
```

### Task Naming Conventions
```yaml
# ✅ CORRECT: Descriptive, actionable names
- name: Ensure Docker service is running and enabled
  systemd:
    name: docker
    state: started
    enabled: yes

# ❌ INCORRECT: Vague or missing names
- systemd:
    name: docker
    state: started
```

### Variable Naming
```yaml
# ✅ CORRECT: Descriptive, namespaced variables
anon_relay_nickname: "MyRelay"
anon_docker_image: "svforte/anon:latest"
anon_base_dir: "/opt/anon"

# ❌ INCORRECT: Generic, non-namespaced variables
nickname: "MyRelay"
image: "svforte/anon:latest"
dir: "/opt/anon"
```

### Template Best Practices
```jinja2
{# ✅ CORRECT: Comments explaining template logic #}
# {{ ansible_managed }}
# Anon Relay Configuration
# Generated from: {{ role_path }}/templates/anonrc.j2

Nickname {{ anon_relay_nickname | default('AnonRelay') }}
ContactInfo {{ anon_relay_contact }}

{% if anon_relay_bandwidth_rate is defined %}
BandwidthRate {{ anon_relay_bandwidth_rate }}
{% endif %}
```

---

## 📦 ANSIBLE BEST PRACTICES

### 1. Use Fully Qualified Collection Names (FQCN)
```yaml
# ✅ CORRECT
- name: Manage Docker containers
  community.docker.docker_container:
    name: anon-relay
    image: svforte/anon:latest

# ❌ INCORRECT
- name: Manage Docker containers
  docker_container:
    name: anon-relay
```

### 2. Always Specify State
```yaml
# ✅ CORRECT
- name: Ensure directory exists
  file:
    path: /opt/anon
    state: directory
    mode: '0755'

# ❌ INCORRECT (state defaults may change)
- name: Ensure directory exists
  file:
    path: /opt/anon
```

### 3. Use Handlers for Service Restarts
```yaml
# ✅ CORRECT: Use handlers
tasks:
  - name: Copy configuration file
    template:
      src: anonrc.j2
      dest: /opt/anon/etc/anon/anonrc
    notify: restart anon relay

handlers:
  - name: restart anon relay
    community.docker.docker_container:
      name: anon-relay
      state: restarted

# ❌ INCORRECT: Restart in task
tasks:
  - name: Copy configuration file
    template:
      src: anonrc.j2
      dest: /opt/anon/etc/anon/anonrc
  
  - name: Restart relay
    community.docker.docker_container:
      name: anon-relay
      state: restarted
```

### 4. Use Tags for Selective Execution
```yaml
- name: Install Docker
  package:
    name: docker
    state: present
  tags:
    - docker
    - setup
    - never-skip
```

### 5. Implement Pre-flight Checks
```yaml
- name: Verify system requirements
  block:
    - name: Check minimum RAM
      assert:
        that:
          - ansible_memtotal_mb >= 512
        fail_msg: "Insufficient RAM: {{ ansible_memtotal_mb }}MB (minimum 512MB required)"
    
    - name: Check available disk space
      assert:
        that:
          - ansible_mounts | selectattr('mount', 'equalto', '/') | map(attribute='size_available') | first > 5368709120
        fail_msg: "Insufficient disk space (minimum 5GB required)"
  tags:
    - preflight
```

---

## 🧪 TESTING REQUIREMENTS

### Local Testing Workflow
```bash
# 1. Syntax check
ansible-playbook site.yml --syntax-check

# 2. Lint check
ansible-lint site.yml

# 3. YAML validation
yamllint .

# 4. Dry run (check mode)
ansible-playbook -i inventory.ini site.yml --check

# 5. Run with increased verbosity
ansible-playbook -i inventory.ini site.yml -vv

# 6. Test idempotency
ansible-playbook -i inventory.ini site.yml
ansible-playbook -i inventory.ini site.yml  # Should show no changes
```

### Required Test Cases
1. **Fresh Installation:** Clean system to fully deployed relay
2. **Update Scenario:** Existing relay to new version
3. **Idempotency:** Run playbook twice, verify no changes on second run
4. **Rollback:** Remove deployment cleanly
5. **Multi-node:** Deploy to multiple hosts simultaneously
6. **Platform Matrix:** Test on Ubuntu, Debian, Fedora

---

## 🛠️ DEVELOPMENT WORKFLOW

### Branch Strategy
```
main (protected)
  ├── develop (integration branch)
  │   ├── feature/docker-setup-role
  │   ├── feature/relay-deployment
  │   └── bugfix/permission-issue
  └── release/v1.0.0
```

### Commit Message Format (Conventional Commits)
```
<type>(<scope>): <subject>

<body>

<footer>

Examples:
feat(docker): add multi-architecture support for ARM64
fix(relay): correct permission on /opt/anon/run/anon directory
docs(readme): update installation instructions
test(docker): add molecule scenario for Fedora
chore(deps): update Ansible requirements to 2.15
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

### Pull Request Checklist
- [ ] Code follows project conventions (this file)
- [ ] All tests pass locally
- [ ] Documentation updated (README, CHANGELOG, inline comments)
- [ ] Idempotency verified
- [ ] Cross-platform compatibility tested
- [ ] No secrets committed
- [ ] ansible-lint passes with no errors
- [ ] Commit messages follow Conventional Commits format

---

## 📚 DOCUMENTATION REQUIREMENTS

### Inline Comments
```yaml
# Document WHY, not WHAT (code shows what)
# ✅ CORRECT
- name: Set restrictive permissions on anon data directory
  file:
    path: /opt/anon/run/anon
    mode: '0700'
    owner: "{{ anon_uid }}"
    group: "{{ anon_gid }}"
  # Anon requires 700 permissions to protect relay keys
  # See: https://docs.anyone.io/relay/security

# ❌ INCORRECT
- name: Set permissions
  file:
    path: /opt/anon/run/anon
    mode: '0700'
  # This sets permissions to 700
```

### Variable Documentation
```yaml
# defaults/main.yml
---
# Docker Configuration
# Docker edition to install (ce = Community Edition)
docker_edition: 'ce'

# Anon Relay Configuration
# Nickname displayed on the Anyone network (max 19 characters)
anon_relay_nickname: "MyAnonRelay"

# Contact email for relay operator (required)
anon_relay_contact: "admin@example.com"

# Bandwidth limits
# Maximum bandwidth rate for relay traffic
anon_relay_bandwidth_rate: "100 MBytes"

# Burst bandwidth limit
anon_relay_bandwidth_burst: "200 MBytes"
```

### Role README Structure
Each role must include:
1. **Description:** What the role does
2. **Requirements:** Dependencies, system requirements
3. **Role Variables:** All variables with descriptions and defaults
4. **Dependencies:** Other roles required
5. **Example Playbook:** Usage example
6. **License:** Role license
7. **Author Information:** Maintainer contact

---

## 🔐 SECURITY PRACTICES

### Using Ansible Vault
```bash
# Create encrypted file
ansible-vault create group_vars/all/vault.yml

# Edit encrypted file
ansible-vault edit group_vars/all/vault.yml

# Encrypt existing file
ansible-vault encrypt secrets.yml

# Run playbook with vault
ansible-playbook site.yml --ask-vault-pass
# OR
ansible-playbook site.yml --vault-password-file ~/.vault_pass
```

### Sensitive Variable Pattern
```yaml
# group_vars/all/vars.yml (unencrypted)
anon_relay_nickname: "MyRelay"
anon_relay_contact: "{{ vault_relay_contact }}"

# group_vars/all/vault.yml (encrypted)
vault_relay_contact: "admin@example.com"
vault_api_key: "secret_key_here"
```

### File Permissions
```yaml
- name: Deploy sensitive configuration
  template:
    src: secret.j2
    dest: /etc/secret.conf
    mode: '0600'  # Read/write for owner only
    owner: root
    group: root
  no_log: true  # Prevent logging sensitive data
```

---

## 🐛 DEBUGGING GUIDE

### Enable Verbose Output
```bash
# Level 1: Basic info
ansible-playbook site.yml -v

# Level 2: Task results
ansible-playbook site.yml -vv

# Level 3: Connection debugging
ansible-playbook site.yml -vvv

# Level 4: Connection plugin debugging
ansible-playbook site.yml -vvvv
```

### Debug Tasks
```yaml
- name: Debug variable values
  debug:
    msg: "Relay nickname is {{ anon_relay_nickname }}"
    verbosity: 1  # Only shown with -v flag

- name: Debug all variables
  debug:
    var: hostvars[inventory_hostname]
```

### Common Issues and Solutions
1. **"Permission denied" errors**
   - Check `become: yes` is set
   - Verify SSH user has sudo privileges
   - Check file permissions

2. **"Module not found" errors**
   - Install required collections: `ansible-galaxy collection install -r requirements.yml`
   - Use FQCN for modules

3. **Idempotency failures**
   - Check for shell commands without `changed_when`
   - Verify state parameters are set correctly

---

## 📞 CLI COMMANDS REFERENCE

### Common Ansible Commands
```bash
# Ping all hosts
ansible all -i inventory.ini -m ping

# Check host facts
ansible all -i inventory.ini -m setup

# Run ad-hoc command
ansible relays -i inventory.ini -m shell -a "docker ps"

# List hosts
ansible-inventory -i inventory.ini --list

# Validate syntax
ansible-playbook site.yml --syntax-check

# Check mode (dry run)
ansible-playbook site.yml --check

# Run with tags
ansible-playbook site.yml --tags "docker,setup"

# Skip tags
ansible-playbook site.yml --skip-tags "testing"

# Limit to specific hosts
ansible-playbook site.yml --limit relay1.example.com
```

---

## 🏗️ ARCHITECTURE NOTES

### Deployment Flow
```
site.yml (main playbook)
  ↓
Pre-tasks (system verification)
  ↓
roles/docker_setup
  ├── Install Docker repository
  ├── Install Docker packages
  ├── Configure Docker daemon
  └── Start Docker service
  ↓
roles/anon_relay_deploy
  ├── Create directories
  ├── Set permissions
  ├── Deploy configuration files
  ├── Pull Docker image
  └── Start container
  ↓
roles/anon_relay_monitor
  ├── Install Nyx
  └── Configure monitoring
  ↓
Post-tasks (health verification)
```

### Variable Precedence (lowest to highest)
1. role defaults (defaults/main.yml)
2. inventory file group_vars
3. inventory group_vars
4. playbook group_vars
5. inventory host_vars
6. playbook host_vars
7. host facts
8. registered vars
9. set_facts
10. play vars
11. play vars_prompt
12. play vars_files
13. role and include vars
14. block vars
15. task vars
16. extra vars (-e flag)

---

## 🔄 UPDATE PROCEDURES

### When to Update Versions
- **Patch (1.0.X):** Bug fixes, documentation updates
- **Minor (1.X.0):** New features, backward compatible
- **Major (X.0.0):** Breaking changes, major refactor

### Update Checklist
1. Update CHANGELOG.md with all changes
2. Update version in all relevant files
3. Test on all supported platforms
4. Tag release in git: `git tag -a v1.0.0 -m "Release 1.0.0"`
5. Push tags: `git push --tags`
6. Update Ansible Galaxy metadata (if applicable)

---

## 🤖 AI AGENT SPECIFIC NOTES

### For Code Generation
- Always generate complete, working code with comments
- Include error handling in all generated tasks
- Provide examples of usage
- Test generated code mentally for idempotency

### For Code Review
- Check for hardcoded values that should be variables
- Verify error handling is present
- Ensure documentation is updated
- Check for security issues (exposed secrets, insecure permissions)

### For Refactoring
- Maintain backward compatibility unless major version bump
- Update all dependent code
- Add deprecation warnings for removed features
- Update documentation thoroughly

### For Troubleshooting
- Provide step-by-step debugging instructions
- Include relevant log excerpts
- Suggest multiple potential solutions
- Reference official documentation

---

## 📋 QUICK REFERENCE CHECKLIST

Before committing code, verify:
- [ ] Code is idempotent (tested by running twice)
- [ ] All tasks have descriptive names
- [ ] Variables are properly namespaced
- [ ] Documentation is updated
- [ ] No secrets in code
- [ ] ansible-lint passes
- [ ] Syntax check passes
- [ ] Tested on at least one supported platform
- [ ] CHANGELOG.md updated
- [ ] Commit message follows Conventional Commits

---

## 📖 ADDITIONAL RESOURCES

### Official Documentation
- [Ansible Documentation](https://docs.ansible.com)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [Anyone Protocol Docs](https://docs.anyone.io)
- [Docker Documentation](https://docs.docker.com)

### Community Resources
- [Ansible Galaxy](https://galaxy.ansible.com)
- [Ansible Lint Rules](https://ansible-lint.readthedocs.io/en/latest/default_rules.html)
- [Jinja2 Template Designer](https://jinja.palletsprojects.com)

### Project-Specific
- See DEVELOPMENT_PLAN.md for project roadmap
- See CLAUDE.md for Claude-specific instructions
- See COPILOT.md for GitHub Copilot instructions
- See CONTRIBUTING.md for contribution guidelines

---

**Version:** 1.0.0  
**Last Updated:** 2025-10-27  
**Maintained By:** Development Team  
**Status:** Active

---

*This document is the authoritative source for all AI coding agents. When in doubt, refer to this file first.*
