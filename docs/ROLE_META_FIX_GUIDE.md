# Role Metadata Fix Guide

## Overview

All role `meta/main.yml` files currently use default template values and need to be customized. This guide provides the correct metadata for each role.

## Why This Matters

- **Ansible Galaxy:** Roles cannot be published without proper metadata
- **Documentation:** Metadata provides role description and requirements
- **Dependencies:** Specifies minimum Ansible version and role dependencies
- **Discoverability:** Helps users find and understand roles

---

## Quick Fix Script

Run this script from the project root to update all role metadata at once:

```bash
#!/bin/bash
# update_role_metadata.sh

ROLES=(
    "docker_setup"
    "preflight_checks"
    "anon_relay_base"
    "anon_relay_standard"
    "anon_relay_exit"
    "anon_relay_socks"
    "anon_relay_monitor"
    "health_checks"
)

for role in "${ROLES[@]}"; do
    echo "Updating roles/$role/meta/main.yml..."
    # Backup original
    cp "roles/$role/meta/main.yml" "roles/$role/meta/main.yml.bak"

    # TODO: Copy appropriate template from below
done
```

---

## Role-Specific Metadata

### 1. docker_setup

**File:** `roles/docker_setup/meta/main.yml`

```yaml
---
galaxy_info:
  author: Your Name
  description: Install and configure Docker CE and Docker Compose
  company: Your Company (optional)

  license: MIT

  min_ansible_version: "2.15"

  platforms:
    - name: Ubuntu
      versions:
        - focal      # 20.04
        - jammy      # 22.04
        - mantic     # 23.10
    - name: Debian
      versions:
        - buster     # 10
        - bullseye   # 11
        - bookworm   # 12
    - name: Fedora
      versions:
        - 37
        - 38
        - 39

  galaxy_tags:
    - docker
    - container
    - containerization
    - deployment
    - infrastructure

dependencies: []

collections:
  - name: community.docker
    version: ">=3.0.0"
  - name: ansible.posix
    version: ">=1.4.0"
```

---

### 2. preflight_checks

**File:** `roles/preflight_checks/meta/main.yml`

```yaml
---
galaxy_info:
  author: Your Name
  description: Pre-deployment validation checks for Anon relay infrastructure
  company: Your Company (optional)

  license: MIT

  min_ansible_version: "2.15"

  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
        - mantic
    - name: Debian
      versions:
        - buster
        - bullseye
        - bookworm
    - name: Fedora
      versions:
        - 37
        - 38
        - 39

  galaxy_tags:
    - validation
    - checks
    - preflight
    - infrastructure
    - deployment
    - testing

dependencies: []

collections:
  - name: ansible.posix
    version: ">=1.4.0"
  - name: community.general
    version: ">=5.0.0"
```

---

### 3. anon_relay_base

**File:** `roles/anon_relay_base/meta/main.yml`

```yaml
---
galaxy_info:
  author: Your Name
  description: Base configuration for Anyone Protocol Anon relay deployments
  company: Your Company (optional)

  license: MIT

  min_ansible_version: "2.15"

  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
        - mantic
    - name: Debian
      versions:
        - buster
        - bullseye
        - bookworm
    - name: Fedora
      versions:
        - 37
        - 38
        - 39

  galaxy_tags:
    - anyone
    - anon
    - relay
    - privacy
    - networking
    - docker
    - container

dependencies:
  - role: docker_setup

collections:
  - name: community.docker
    version: ">=3.0.0"
  - name: ansible.posix
    version: ">=1.4.0"
```

---

### 4. anon_relay_standard

**File:** `roles/anon_relay_standard/meta/main.yml`

```yaml
---
galaxy_info:
  author: Your Name
  description: Standard (middle/guard) Anyone Protocol relay configuration
  company: Your Company (optional)

  license: MIT

  min_ansible_version: "2.15"

  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
        - mantic
    - name: Debian
      versions:
        - buster
        - bullseye
        - bookworm
    - name: Fedora
      versions:
        - 37
        - 38
        - 39

  galaxy_tags:
    - anyone
    - anon
    - relay
    - standard
    - privacy
    - networking
    - security

dependencies:
  - role: anon_relay_base

collections:
  - name: community.docker
    version: ">=3.0.0"
  - name: ansible.posix
    version: ">=1.4.0"
  - name: community.general
    version: ">=5.0.0"
```

---

### 5. anon_relay_exit

**File:** `roles/anon_relay_exit/meta/main.yml`

```yaml
---
galaxy_info:
  author: Your Name
  description: Exit relay configuration for Anyone Protocol (use with caution)
  company: Your Company (optional)

  license: MIT

  min_ansible_version: "2.15"

  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
        - mantic
    - name: Debian
      versions:
        - buster
        - bullseye
        - bookworm
    - name: Fedora
      versions:
        - 37
        - 38
        - 39

  galaxy_tags:
    - anyone
    - anon
    - relay
    - exit
    - privacy
    - networking
    - security

dependencies:
  - role: anon_relay_base

collections:
  - name: community.docker
    version: ">=3.0.0"
  - name: ansible.posix
    version: ">=1.4.0"
  - name: community.general
    version: ">=5.0.0"
```

---

### 6. anon_relay_socks

**File:** `roles/anon_relay_socks/meta/main.yml`

```yaml
---
galaxy_info:
  author: Your Name
  description: SOCKS proxy configuration for Anyone Protocol
  company: Your Company (optional)

  license: MIT

  min_ansible_version: "2.15"

  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
        - mantic
    - name: Debian
      versions:
        - buster
        - bullseye
        - bookworm
    - name: Fedora
      versions:
        - 37
        - 38
        - 39

  galaxy_tags:
    - anyone
    - anon
    - socks
    - proxy
    - privacy
    - networking

dependencies:
  - role: anon_relay_base

collections:
  - name: community.docker
    version: ">=3.0.0"
  - name: ansible.posix
    version: ">=1.4.0"
```

---

### 7. anon_relay_monitor

**File:** `roles/anon_relay_monitor/meta/main.yml`

```yaml
---
galaxy_info:
  author: Your Name
  description: Monitoring and health checks for Anyone Protocol relays
  company: Your Company (optional)

  license: MIT

  min_ansible_version: "2.15"

  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
        - mantic
    - name: Debian
      versions:
        - buster
        - bullseye
        - bookworm
    - name: Fedora
      versions:
        - 37
        - 38
        - 39

  galaxy_tags:
    - monitoring
    - health
    - checks
    - nyx
    - relay
    - anyone

dependencies:
  - role: anon_relay_base

collections:
  - name: community.docker
    version: ">=3.0.0"
  - name: ansible.posix
    version: ">=1.4.0"
  - name: community.general
    version: ">=5.0.0"
```

---

### 8. health_checks

**File:** `roles/health_checks/meta/main.yml`

```yaml
---
galaxy_info:
  author: Your Name
  description: Post-deployment health validation for Anon relay infrastructure
  company: Your Company (optional)

  license: MIT

  min_ansible_version: "2.15"

  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
        - mantic
    - name: Debian
      versions:
        - buster
        - bullseye
        - bookworm
    - name: Fedora
      versions:
        - 37
        - 38
        - 39

  galaxy_tags:
    - health
    - validation
    - checks
    - testing
    - deployment
    - relay

dependencies: []

collections:
  - name: community.docker
    version: ">=3.0.0"
  - name: ansible.posix
    version: ">=1.4.0"
```

---

## Customization Guidelines

### Author Information

Replace `Your Name` and `Your Company` with:
```yaml
author: John Doe
company: Acme Corporation
```

Or use GitHub handle:
```yaml
author: github_username
company: Your Project Name
```

### License

This project uses MIT license. Keep as:
```yaml
license: MIT
```

### Platforms

Only include platforms you've tested on. Remove untested versions:
```yaml
platforms:
  - name: Ubuntu
    versions:
      - jammy      # Keep only if tested
```

### Galaxy Tags

Choose 3-5 most relevant tags from:
- anyone, anon, relay, exit, socks
- privacy, security, networking
- docker, container, deployment
- monitoring, health, checks
- validation, testing

### Dependencies

Specify only direct role dependencies:
```yaml
# anon_relay_standard depends on anon_relay_base
dependencies:
  - role: anon_relay_base

# preflight_checks has no dependencies
dependencies: []
```

---

## Validation

After updating metadata, validate with:

```bash
# Ansible Galaxy validation
ansible-galaxy role info roles/role_name

# Ansible Lint
ansible-lint roles/role_name

# Syntax check
ansible-playbook site.yml --syntax-check
```

---

## Automated Update Script

Save this script as `scripts/update_all_role_meta.sh`:

```bash
#!/bin/bash
# update_all_role_meta.sh
# Updates all role metadata with correct values

set -e

PROJECT_ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"
AUTHOR="${1:-Your Name}"
COMPANY="${2:-Your Company}"

echo "Updating role metadata..."
echo "Author: $AUTHOR"
echo "Company: $COMPANY"

# Function to update meta file
update_meta() {
    local role=$1
    local meta_file="$PROJECT_ROOT/roles/$role/meta/main.yml"

    if [ -f "$meta_file" ]; then
        echo "Updating $role..."

        # Backup
        cp "$meta_file" "$meta_file.bak"

        # Update author and company
        sed -i.tmp "s/author: your name/author: $AUTHOR/g" "$meta_file"
        sed -i.tmp "s/author: Your Name/author: $AUTHOR/g" "$meta_file"
        sed -i.tmp "s/company: your company (optional)/company: $COMPANY/g" "$meta_file"
        sed -i.tmp "s/company: Your Company (optional)/company: $COMPANY/g" "$meta_file"

        # Update license
        sed -i.tmp "s/license: license (GPL-2.0-or-later, MIT, etc)/license: MIT/g" "$meta_file"

        # Update min_ansible_version
        sed -i.tmp "s/min_ansible_version: 2.1/min_ansible_version: \"2.15\"/g" "$meta_file"

        # Clean up temp files
        rm -f "$meta_file.tmp"

        echo "✓ Updated $role"
    else
        echo "✗ Not found: $meta_file"
    fi
}

# Update all roles
ROLES=(
    "docker_setup"
    "preflight_checks"
    "anon_relay_base"
    "anon_relay_standard"
    "anon_relay_exit"
    "anon_relay_socks"
    "anon_relay_monitor"
    "health_checks"
)

for role in "${ROLES[@]}"; do
    update_meta "$role"
done

echo ""
echo "✓ All role metadata updated!"
echo ""
echo "Next steps:"
echo "1. Review changes: git diff roles/*/meta/main.yml"
echo "2. Validate: ansible-lint roles/"
echo "3. Commit: git commit -am 'fix: update role metadata'"
```

**Usage:**
```bash
chmod +x scripts/update_all_role_meta.sh
./scripts/update_all_role_meta.sh "Your Name" "Your Company"
```

---

## Verification Checklist

After updating all metadata files:

- [ ] `author` field is customized (not "your name")
- [ ] `description` is accurate and descriptive
- [ ] `license` is set to MIT
- [ ] `min_ansible_version` is "2.15"
- [ ] `platforms` list includes only tested OSes
- [ ] `galaxy_tags` are relevant (3-5 tags)
- [ ] `dependencies` are correct
- [ ] `collections` versions match requirements.yml
- [ ] No template comments remain
- [ ] ansible-lint passes
- [ ] Syntax check passes

---

## References

- [Ansible Galaxy Role Metadata](https://galaxy.ansible.com/docs/contributing/creating_role.html)
- [Role Dependencies](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#role-dependencies)
- [Galaxy Requirements](https://galaxy.ansible.com/docs/contributing/importing.html)

---

**Last Updated:** 2025-10-28
**Version:** 1.0
