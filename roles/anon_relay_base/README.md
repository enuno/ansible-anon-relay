# Anon Relay Base Role

This role provides the base deployment functionality shared by all Anon relay types (standard, exit, and SOCKS proxy). It handles user creation, directory setup, configuration file deployment, and Docker container management.

## Requirements

- Ansible 2.15+
- Docker installed and running (via docker_setup role)
- Target systems: Ubuntu 20.04+, Debian 10+, Fedora 35+
- Python 3.8+ on target systems

## Role Variables

### Default Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_user` | `"anond"` | Anon system user |
| `anon_uid` | `100` | Anon user UID |
| `anon_gid` | `101` | Anon group GID |
| `anon_base_dir` | `"/opt/anon"` | Base directory for Anon files |
| `compose_dir` | `"/opt/compose-files"` | Docker Compose files directory |
| `nyx_dir` | `"/root/.nyx"` | Nyx configuration directory |
| `anon_docker_image` | `"svforte/anon:latest"` | Docker image for Anon |
| `anon_container_name` | `"anon-relay"` | Docker container name |
| `anon_container_restart_policy` | `"unless-stopped"` | Container restart policy |
| `anon_accept_terms` | `true` | Accept Anyone Protocol terms |
| `anon_log_level` | `"notice"` | Log level for Anon |
| `anon_log_file` | `"/var/log/anon/notices.log"` | Log file path |
| `anon_ipv6_enabled` | `true` | Enable IPv6 support |
| `anon_nyx_enabled` | `true` | Enable Nyx monitoring |
| `anon_log_rotation_enabled` | `true` | Enable log rotation |

### Container Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_container_ports` | `[]` | Port mappings for container |
| `anon_container_volumes` | `[]` | Volume mounts for container |
| `anon_container_environment` | `{}` | Environment variables for container |

### File Permissions

| Variable | Default | Description |
|----------|---------|-------------|
| `anon_config_mode` | `'0644'` | Configuration file permissions |
| `anon_config_owner` | `"{{ anon_user }}"` | Configuration file owner |
| `anon_config_group` | `"{{ anon_user }}"` | Configuration file group |
| `anon_data_mode` | `'0700'` | Data directory permissions |
| `anon_data_owner` | `"{{ anon_user }}"` | Data directory owner |
| `anon_data_group` | `"{{ anon_user }}"` | Data directory group |

## Dependencies

- `docker_setup` role (must be run first)

## Example Playbook

```yaml
---
- name: Deploy base Anon relay
  hosts: relays
  become: yes
  roles:
    - docker_setup
    - anon_relay_base
  vars:
    anon_relay_nickname: "MyRelay"
    anon_relay_contact: "operator@example.com"
    anon_relay_or_port: 9001
```

## Example with Custom Settings

```yaml
---
- name: Deploy base Anon relay with custom settings
  hosts: relays
  become: yes
  roles:
    - docker_setup
    - anon_relay_base
  vars:
    anon_base_dir: "/opt/anond"
    anon_docker_image: "svforte/anon:stable"
    anon_container_name: "my-anon-relay"
    anon_max_mem_in_mb: 1024
    anon_max_cpu_percent: 80
    anon_log_rotation_max_files: 5
```

## Directory Structure Created

The role creates the following directory structure:

```
/opt/anon/
├── etc/anon/          # Configuration files
│   └── anonrc         # Main Anon configuration
├── run/anon/          # Runtime data and control socket
├── logs/              # Log files
└── ...

/opt/compose-files/
└── relay.yaml         # Docker Compose configuration

/root/.nyx/
└── config             # Nyx monitoring configuration
```

## Templates

### relay.yaml.j2
Docker Compose configuration file that defines the Anon container with appropriate ports, volumes, and environment variables.

### anonrc_base.j2
Base Anon configuration file with common settings. This template is extended by relay-type-specific roles.

### config.j2
Nyx monitoring tool configuration for relay monitoring and management.

## Handlers

- `restart anon relay` - Restarts the Anon container
- `stop anon relay` - Stops the Anon container
- `start anon relay` - Starts the Anon container

## Tags

- `anon` - All Anon-related tasks
- `users` - User and group management
- `directories` - Directory creation and permissions
- `configuration` - Configuration file deployment
- `container` - Docker container management
- `logs` - Log rotation configuration

## Notes

- This role creates the foundation for all relay types
- The actual relay configuration is handled by type-specific roles
- Container ports and volumes are configured based on relay type
- Log rotation is automatically configured for the Anon log file
- The role ensures proper file permissions for security

## License

MIT

## Author Information

This role was created for the Anon Relay Deployment project.