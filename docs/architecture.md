# Architecture Documentation

## System Architecture Overview

This document describes the technical architecture of the Ansible Anon Relay Deployment system.

## Table of Contents

- [High-Level Architecture](#high-level-architecture)
- [Component Overview](#component-overview)
- [Deployment Flow](#deployment-flow)
- [Role Architecture](#role-architecture)
- [Security Architecture](#security-architecture)
- [Network Architecture](#network-architecture)
- [Data Flow](#data-flow)

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Control Node                             │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Ansible Engine                                        │ │
│  │  - Playbooks (site.yml, deploy_*.yml)                 │ │
│  │  - Inventory (inventory.ini)                          │ │
│  │  - Variables (group_vars/, host_vars/)               │ │
│  │  - Roles (8 specialized roles)                        │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                           │
                           │ SSH/Ansible
                           │
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │ Relay 1  │    │ Relay 2  │    │ Relay N  │
    │  Ubuntu  │    │  Debian  │    │  Fedora  │
    └──────────┘    └──────────┘    └──────────┘
         │               │               │
         │               │               │
    ┌────▼───────┐  ┌────▼───────┐  ┌────▼───────┐
    │   Docker   │  │   Docker   │  │   Docker   │
    │ ┌────────┐ │  │ ┌────────┐ │  │ ┌────────┐ │
    │ │  Anon  │ │  │ │  Anon  │ │  │ │  Anon  │ │
    │ │ Relay  │ │  │ │ Relay  │ │  │ │ Relay  │ │
    │ │Container│ │  │ │Container│ │  │ │Container│ │
    │ └────────┘ │  │ └────────┘ │  │ └────────┘ │
    └────────────┘  └────────────┘  └────────────┘
         │               │               │
         └───────────────┼───────────────┘
                         │
                         ▼
              ┌────────────────────┐
              │ Anyone Protocol    │
              │ Network            │
              └────────────────────┘
```

---

## Component Overview

### Control Node Components

#### 1. Ansible Engine
- **Purpose:** Orchestrates deployment across target hosts
- **Version:** 2.15+
- **Key Features:**
  - Idempotent operations
  - Parallel execution (configurable forks)
  - Fact gathering and caching

#### 2. Playbooks
| Playbook | Purpose | Primary Roles |
|----------|---------|---------------|
| `site.yml` | Main orchestrator | All roles |
| `deploy_standard.yml` | Standard relay deployment | docker_setup, anon_relay_base, anon_relay_standard |
| `deploy_exit.yml` | Exit relay deployment | docker_setup, anon_relay_base, anon_relay_exit |
| `deploy_socks.yml` | SOCKS proxy deployment | docker_setup, anon_relay_base, anon_relay_socks |
| `update.yml` | Update existing relays | anon_relay_monitor |
| `remove.yml` | Remove relay deployment | All (cleanup) |

#### 3. Inventory Management
```
inventory.ini              # Main inventory (user-customized)
├── [standard_relays]     # Standard relay hosts
├── [exit_relays]         # Exit relay hosts
└── [socks_proxies]       # SOCKS proxy hosts

group_vars/
├── all.yml               # Global variables
├── standard_relays.yml   # Standard relay specific
├── exit_relays.yml       # Exit relay specific
└── socks_proxies.yml     # SOCKS proxy specific

host_vars/
└── hostname.yml          # Per-host overrides
```

#### 4. Role System
```
roles/
├── preflight_checks/     # Pre-deployment validation
├── docker_setup/         # Docker installation
├── anon_relay_base/      # Common relay setup
├── anon_relay_standard/  # Standard relay config
├── anon_relay_exit/      # Exit relay config
├── anon_relay_socks/     # SOCKS proxy config
├── anon_relay_monitor/   # Monitoring setup
└── health_checks/        # Post-deployment validation
```

### Target Node Components

#### 1. Operating System Layer
- **Supported:** Ubuntu 20.04+, Debian 10+, Fedora 35+
- **Components:**
  - Base OS with security updates
  - Python 3.8+ for Ansible
  - SSH server for remote management

#### 2. Docker Layer
- **Version:** Docker CE 20.10+
- **Components:**
  - Docker Engine
  - Docker Compose V2
  - Docker daemon with custom configuration

#### 3. Security Layer
- **UFW Firewall:** Port-based access control
- **Fail2Ban:** Brute-force attack mitigation
- **SSH Hardening:** Secure remote access
- **File Permissions:** Restrictive access controls

#### 4. Application Layer
- **Anon Relay Container:**
  - Image: `svforte/anon:latest`
  - User: `anond` (UID 100, GID 101)
  - Network: Bridge mode with port publishing
  - Volumes: Persistent configuration and data

#### 5. Monitoring Layer (Optional)
- **Nyx:** Terminal-based relay monitor
- **Docker Logs:** Centralized logging
- **Health Checks:** Automated validation scripts

---

## Deployment Flow

### Standard Deployment Sequence

```
1. Pre-Deployment Phase
   ├── User runs: ansible-playbook -i inventory.ini site.yml
   ├── Ansible parses inventory and variables
   ├── Fact gathering from target hosts
   └── Variable precedence resolution

2. Preflight Checks Phase (preflight_checks role)
   ├── System requirements check
   │   ├── RAM >= 512MB
   │   ├── Disk space >= 5GB
   │   └── CPU cores >= 1
   ├── Network connectivity check
   │   ├── Internet access verification
   │   ├── DNS resolution test
   │   └── Port availability (9001, 9030)
   ├── Docker availability check
   └── Relay-specific requirements
       └── Generate summary report

3. Docker Setup Phase (docker_setup role)
   ├── Detect OS distribution
   ├── Add Docker repository
   │   ├── Import GPG keys
   │   └── Configure package source
   ├── Install Docker packages
   │   ├── docker-ce
   │   ├── docker-ce-cli
   │   ├── containerd.io
   │   └── docker-compose-plugin
   ├── Configure Docker daemon
   │   └── /etc/docker/daemon.json
   ├── Add users to docker group
   ├── Start and enable Docker service
   └── Verify Docker installation

4. Base Relay Setup Phase (anon_relay_base role)
   ├── Create directory structure
   │   ├── /opt/anon/etc/anon (config)
   │   └── /opt/anon/run/anon (data)
   ├── Create anond user/group
   │   ├── UID: 100
   │   └── GID: 101
   ├── Set file permissions
   │   ├── Config: 755 (readable)
   │   └── Data: 700 (restricted)
   └── Deploy Docker Compose file

5. Relay Type Configuration Phase
   ├── Standard Relay (anon_relay_standard role)
   │   ├── Generate anonrc configuration
   │   ├── Configure ORPort (9001)
   │   ├── Set bandwidth limits
   │   └── Apply security hardening
   ├── Exit Relay (anon_relay_exit role)
   │   ├── Generate anonrc with exit policy
   │   ├── Configure ORPort and DirPort
   │   ├── Legal terms acceptance check
   │   └── Enhanced security hardening
   └── SOCKS Proxy (anon_relay_socks role)
       ├── Generate anonrc for SOCKS
       ├── Configure SOCKS port (9050)
       └── Local-only access configuration

6. Container Deployment Phase
   ├── Pull Docker image
   │   └── svforte/anon:latest
   ├── Start container
   │   ├── Name: anon-relay
   │   ├── Restart policy: unless-stopped
   │   └── Volume mounts
   └── Wait for container health

7. Monitoring Setup Phase (anon_relay_monitor role)
   ├── Install Nyx tool
   ├── Configure monitoring scripts
   ├── Set up log rotation
   └── Deploy health check cron jobs

8. Health Checks Phase (health_checks role)
   ├── Container status verification
   ├── Port accessibility test
   ├── Configuration validation
   ├── Relay descriptor publication check
   └── Generate deployment report

9. Post-Deployment
   ├── Display deployment summary
   └── Provide next steps instructions
```

### Error Handling

Each phase includes:
- **Rescue blocks** for graceful error handling
- **Rollback capabilities** for failed deployments
- **Detailed logging** for troubleshooting
- **User notifications** for required actions

---

## Role Architecture

### Role Structure Standard

Each role follows this structure:

```
role_name/
├── defaults/
│   └── main.yml          # Default variables (lowest precedence)
├── vars/
│   └── main.yml          # Role variables (higher precedence)
├── tasks/
│   ├── main.yml          # Main task entrypoint
│   ├── subtask1.yml      # Modular task files
│   └── subtask2.yml
├── templates/
│   ├── config.j2         # Jinja2 templates
│   └── script.sh.j2
├── files/
│   └── static_file.conf  # Static files
├── handlers/
│   └── main.yml          # Event-driven tasks
├── meta/
│   └── main.yml          # Role metadata
├── tests/
│   ├── inventory         # Test inventory
│   └── test.yml          # Test playbook
└── README.md             # Role documentation
```

### Role Dependencies

```
site.yml
├── preflight_checks (no dependencies)
├── docker_setup (no dependencies)
├── anon_relay_base
│   └── requires: docker_setup
├── anon_relay_standard
│   └── requires: anon_relay_base
├── anon_relay_exit
│   └── requires: anon_relay_base
├── anon_relay_socks
│   └── requires: anon_relay_base
├── anon_relay_monitor
│   └── requires: anon_relay_base
└── health_checks
    └── requires: anon_relay_*
```

---

## Security Architecture

### Defense in Depth Strategy

```
┌────────────────────────────────────────────┐
│ Layer 1: Network Security                 │
│ - UFW Firewall                             │
│ - Port filtering (9001, 9030, SSH)        │
│ - IPv6 support                             │
└────────────────────────────────────────────┘
              ▼
┌────────────────────────────────────────────┐
│ Layer 2: Host Security                    │
│ - SSH hardening (key-only auth)           │
│ - Fail2Ban (brute-force protection)       │
│ - Non-standard SSH port (optional)        │
│ - Regular security updates                │
└────────────────────────────────────────────┘
              ▼
┌────────────────────────────────────────────┐
│ Layer 3: Container Security               │
│ - Non-root user (anond, UID 100)          │
│ - Read-only root filesystem (where viable)│
│ - Resource limits (CPU, memory)           │
│ - Network isolation                        │
└────────────────────────────────────────────┘
              ▼
┌────────────────────────────────────────────┐
│ Layer 4: Application Security             │
│ - Restricted file permissions (600/700)   │
│ - Secrets management (Ansible Vault)      │
│ - Secure defaults                          │
│ - Minimal attack surface                  │
└────────────────────────────────────────────┘
```

### Security Controls Matrix

| Control | Type | Implemented | Configurable |
|---------|------|-------------|--------------|
| SSH Key Authentication | Access Control | ✅ | ✅ |
| UFW Firewall | Network Security | ✅ | ✅ |
| Fail2Ban | Intrusion Prevention | ✅ | ✅ |
| Container Isolation | Containerization | ✅ | ❌ |
| File Permissions | Access Control | ✅ | ❌ |
| Ansible Vault | Secrets Management | ✅ | ✅ |
| Non-Root User | Privilege Reduction | ✅ | ❌ |
| Resource Limits | DoS Prevention | ✅ | ✅ |

---

## Network Architecture

### Port Configuration

| Port | Protocol | Purpose | Firewall | Container |
|------|----------|---------|----------|-----------|
| 9001 | TCP | ORPort (relay traffic) | Open | Published |
| 9030 | TCP | DirPort (directory) | Open | Published |
| 9050 | TCP | SOCKS proxy | Closed | Published (SOCKS only) |
| 22/custom | TCP | SSH management | Restricted | N/A |

### Network Flow Diagram

```
Internet
   │
   ├─► Port 9001 (ORPort)
   │     │
   │     └─► Docker Bridge
   │            │
   │            └─► anon-relay container
   │                   │
   │                   └─► Anyone Protocol Network
   │
   ├─► Port 9030 (DirPort)
   │     │
   │     └─► Docker Bridge
   │            │
   │            └─► anon-relay container
   │                   │
   │                   └─► Directory Services
   │
   └─► Port 22 (SSH)
         │
         └─► Host OS
               │
               └─► System Management
```

### IPv6 Support

- **Dual-stack** by default (IPv4 + IPv6)
- **IPv6-only mode** available
- Automatic address detection
- Firewall rules for both protocols

---

## Data Flow

### Configuration Data Flow

```
1. Variable Definition
   ├── defaults/main.yml (lowest priority)
   ├── group_vars/all.yml
   ├── group_vars/relays.yml
   ├── host_vars/hostname.yml
   └── extra-vars (highest priority)

2. Template Rendering
   ├── Jinja2 template processing
   ├── Variable substitution
   └── Conditional logic

3. File Deployment
   ├── Template → Target host
   └── File permissions applied

4. Service Configuration
   ├── Docker Compose file generated
   ├── Container configuration applied
   └── Service restart (if needed)
```

### Runtime Data Flow

```
Relay Container
   ├── Configuration: /etc/anon/anonrc (read-only)
   ├── State data: /var/lib/anon (read-write)
   │   ├── Keys (persisted)
   │   ├── State files (persisted)
   │   └── Cache (ephemeral)
   └── Logs: Docker logging driver
       └── /var/log/anon/notices.log
```

---

## Scalability Considerations

### Horizontal Scaling

- **Multiple relays** across different hosts
- **Parallel deployment** using Ansible forks
- **Independent operation** of each relay
- **No shared state** between relays

### Vertical Scaling

- **Bandwidth limits** configurable per relay
- **Resource limits** (CPU, memory) adjustable
- **Multiple relays** per host (different ports)

### Performance Optimization

- **Fact caching** reduces overhead
- **SSH pipelining** improves transfer speed
- **Parallel execution** via Ansible forks setting
- **Smart gathering** only when needed

---

## Backup and Recovery

### Backup Strategy

```
Critical Data:
├── Relay Keys: /opt/anon/run/anon/keys/
├── Configuration: /opt/anon/etc/anon/anonrc
└── State: /opt/anon/run/anon/state

Backup Methods:
├── Manual: Copy directories to secure location
├── Automated: Cron job + rsync
└── Ansible role: anon_relay_monitor/tasks/backup.yml
```

### Recovery Process

```
1. Stop container: docker stop anon-relay
2. Restore data: Copy backup to /opt/anon/
3. Verify permissions: chown -R 100:101 /opt/anon/run/anon
4. Start container: docker start anon-relay
5. Verify: Check logs and connectivity
```

---

## Monitoring and Observability

### Monitoring Stack

```
┌─────────────────────────────────────────┐
│ Monitoring Layer                        │
│ ┌─────────────┐  ┌──────────────────┐ │
│ │     Nyx     │  │  Docker Stats    │ │
│ │  Terminal   │  │  (CPU, Memory)   │ │
│ │   Monitor   │  │                  │ │
│ └─────────────┘  └──────────────────┘ │
└─────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────┐
│ Logging Layer                           │
│ ┌─────────────────────────────────────┐ │
│ │  Docker Logs (JSON driver)          │ │
│ │  - Container stdout/stderr          │ │
│ │  - Rotated (10MB × 3 files)         │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────┐
│ Health Check Layer                      │
│ ┌─────────────────────────────────────┐ │
│ │  - Container status                 │ │
│ │  - Port connectivity                │ │
│ │  - Descriptor publication           │ │
│ │  - Bandwidth usage                  │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### Health Check Types

1. **Container Health:** Docker container status
2. **Network Health:** Port accessibility from internet
3. **Application Health:** Relay descriptor published
4. **Performance Health:** Bandwidth and resource usage

---

## Future Architecture Enhancements

### Planned Improvements

1. **Monitoring Integration**
   - Prometheus metrics exporter
   - Grafana dashboards
   - Alerting via multiple channels

2. **High Availability**
   - Automatic failover
   - Load balancing (for SOCKS)
   - Health-based routing

3. **Automation**
   - CI/CD pipeline integration
   - Automated testing framework
   - Infrastructure as Code validation

4. **Security Enhancements**
   - SELinux/AppArmor profiles
   - Automated vulnerability scanning
   - Certificate management

---

## References

- [Ansible Architecture](https://docs.ansible.com/ansible/latest/dev_guide/overview_architecture.html)
- [Docker Architecture](https://docs.docker.com/get-started/overview/)
- [Anyone Protocol Documentation](https://docs.anyone.io)
- [Security Best Practices](../SECURITY.md)

---

**Document Version:** 1.0
**Last Updated:** 2025-10-28
**Maintainer:** Development Team
