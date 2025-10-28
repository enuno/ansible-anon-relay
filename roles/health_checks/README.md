# Health Checks Role

This role performs post-deployment validation and health monitoring for Anon relay deployments to ensure services are running correctly.

## Requirements

- Ansible 2.15+
- Target systems with Docker and Anon relay deployed
- Python 3.8+ on target systems

## Role Variables

### Default Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `health_check_timeout` | `30` | Health check timeout in seconds |
| `health_check_retries` | `3` | Number of retry attempts |
| `health_check_delay` | `10` | Delay between retries |
| `health_check_container` | `true` | Enable container health checks |
| `health_check_container_name` | `"anon-relay"` | Name of the Anon container |
| `health_check_container_healthy` | `true` | Require container to be healthy |
| `health_check_docker_service` | `true` | Enable Docker service checks |
| `health_check_anon_service` | `true` | Enable Anon service checks |
| `health_check_orport` | `true` | Enable ORPort connectivity checks |
| `health_check_orport_timeout` | `10` | ORPort check timeout |
| `health_check_dns` | `true` | Enable DNS resolution checks |
| `health_check_logs` | `true` | Enable log health checks |
| `health_check_log_errors` | `true` | Check logs for errors |
| `health_check_log_warnings` | `true` | Check logs for warnings |
| `health_check_performance` | `false` | Enable performance checks |
| `health_check_memory_usage` | `80` | Maximum memory usage percentage |
| `health_check_cpu_usage` | `80` | Maximum CPU usage percentage |

## Health Checks Performed

### Docker Service Health
- **Service Status**: Verifies Docker service is running
- **Daemon Connectivity**: Tests Docker daemon accessibility
- **Service State**: Ensures Docker service is active

### Container Health
- **Container Existence**: Verifies Anon container exists
- **Container Status**: Checks container is running
- **Health Status**: Validates container health (if available)
- **Restart Count**: Monitors container restart frequency

### Service Health
- **Process Status**: Verifies Anon process is running
- **Control Socket**: Checks Anon control socket exists
- **Configuration**: Validates configuration file presence

### Network Health
- **ORPort Connectivity**: Tests ORPort reachability
- **DirPort Connectivity**: Tests DirPort for exit relays
- **SOCKS Port Connectivity**: Tests SOCKS port for proxies
- **DNS Resolution**: Verifies DNS functionality

### Log Health
- **Error Detection**: Scans logs for error messages
- **Warning Detection**: Identifies warning messages
- **Startup Messages**: Verifies successful startup
- **Log Analysis**: Provides log health summary

### Performance Health (Optional)
- **CPU Usage**: Monitors container CPU usage
- **Memory Usage**: Tracks memory consumption
- **Resource Limits**: Validates resource constraints
- **Performance Metrics**: Provides performance summary

## Dependencies

- `community.docker` collection
- Anon relay container deployed

## Example Playbook

```yaml
---
- name: Run health checks
  hosts: relays
  become: yes
  roles:
    - health_checks
  vars:
    health_check_performance: true
    health_check_memory_usage: 70
    health_check_cpu_usage: 70
```

## Example with Custom Settings

```yaml
---
- name: Run comprehensive health checks
  hosts: relays
  become: yes
  roles:
    - health_checks
  vars:
    health_check_container_name: "my-anon-relay"
    health_check_timeout: 60
    health_check_retries: 5
    health_check_performance: true
    health_check_log_errors: true
    health_check_log_warnings: true
```

## Tags

- `health` - All health check tasks
- `docker` - Docker service health checks
- `container` - Container health checks
- `service` - Anon service health checks
- `network` - Network connectivity checks
- `logs` - Log health checks
- `performance` - Performance health checks
- `summary` - Health check summary

## Return Values

The role sets the following facts:
- `health_check_passed`: Boolean indicating overall health status
- `container_restart_count`: Number of container restarts
- `cpu_usage`: Container CPU usage percentage
- `memory_usage_percent`: Container memory usage percentage

## License

MIT

## Author Information

This role was created for the Anon Relay Deployment project.