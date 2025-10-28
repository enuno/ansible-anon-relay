# Docker Setup Role

This role installs and configures Docker CE (Community Edition) and Docker Compose on target systems.

## Requirements

- Ansible 2.15+
- Target systems: Ubuntu 20.04+, Debian 10+, Fedora 35+
- Python 3.8+ on target systems

## Role Variables

### Default Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `docker_edition` | `'ce'` | Docker edition to install (Community Edition) |
| `docker_package` | `"docker-{{ docker_edition }}"` | Docker package name |
| `docker_compose_version` | `"latest"` | Docker Compose version |
| `docker_daemon_config_dir` | `"/etc/docker"` | Docker daemon configuration directory |
| `docker_daemon_config_file` | `"{{ docker_daemon_config_dir }}/daemon.json"` | Docker daemon configuration file |
| `docker_group` | `"docker"` | Docker group name |
| `docker_group_gid` | `999` | Docker group GID |
| `docker_users` | `[]` | List of users to add to docker group |
| `docker_service_enabled` | `true` | Enable Docker service at boot |
| `docker_service_state` | `"started"` | Docker service state |
| `docker_cleanup_old_versions` | `false` | Remove old Docker versions |
| `docker_remove_old_packages` | `false` | Remove conflicting Docker packages |

### Docker Daemon Configuration

The role configures Docker daemon with the following default settings:

```yaml
docker_daemon_config:
  log-driver: "json-file"
  log-opts:
    max-size: "10m"
    max-file: "3"
  storage-driver: "overlay2"
  live-restore: true
```

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Install Docker on relay hosts
  hosts: relays
  become: yes
  roles:
    - docker_setup
  vars:
    docker_users:
      - "relay-operator"
      - "{{ ansible_user }}"
```

## Example with Custom Configuration

```yaml
---
- name: Install Docker with custom settings
  hosts: relays
  become: yes
  roles:
    - docker_setup
  vars:
    docker_daemon_config:
      log-driver: "json-file"
      log-opts:
        max-size: "50m"
        max-file: "5"
      storage-driver: "overlay2"
      live-restore: true
      experimental: false
    docker_users:
      - "relay-operator"
    docker_remove_old_packages: true
```

## Tags

- `docker` - All Docker-related tasks
- `packages` - Package installation tasks
- `repository` - Repository configuration tasks
- `config` - Configuration tasks
- `users` - User management tasks
- `service` - Service management tasks
- `verify` - Verification tasks
- `cleanup` - Cleanup tasks
- `debian` - Debian/Ubuntu specific tasks
- `redhat` - RedHat/Fedora specific tasks

## License

MIT

## Author Information

This role was created for the Anon Relay Deployment project.