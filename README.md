# Ansible Playbook for Anyone Protocol Anon Relay Deployment

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Ansible](https://img.shields.io/badge/ansible-8.5%2B-green.svg)
![Ansible Core](https://img.shields.io/badge/ansible--core-2.15%2B-green.svg)
![Platform](https://img.shields.io/badge/platform-Ubuntu%20%7C%20Debian%20%7C%20Fedora-orange.svg)
![Architecture](https://img.shields.io/badge/architecture-amd64%20%7C%20arm64-lightgrey.svg)

Automated deployment of **Anyone Protocol Anon Relay** nodes via Docker using Ansible. This playbook simplifies the setup and management of privacy-focused relay infrastructure that contributes to the decentralized Anyone network.

## 🌐 What is Anyone Protocol?

[Anyone Protocol](https://anyone.io) (formerly ATOR) is a decentralized, privacy-first relay network that provides censorship-resistant, anonymous internet routing through onion routing. Relay operators contribute bandwidth and computing power to the network and earn **ANYONE tokens** as rewards.

### Key Features
- 🔒 **Privacy-First:** Anonymous routing through encrypted relay nodes
- 💰 **Token Incentivized:** Earn ANYONE tokens for running relays
- 🌍 **Decentralized:** No central authority controls the network
- 🐳 **Docker-Based:** Containerized for easy deployment and management
- 🔧 **Automated:** This playbook handles the entire setup process

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Documentation](#documentation)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## ⚡ Prerequisites

### Control Node (Your Local Machine)
- **Ansible:** 8.5.0 or higher (includes ansible-core 2.15+)
- **Python:** 3.8 or higher
- **SSH Client:** OpenSSH

### Target Nodes (Relay Servers)
- **Operating System:** 
  - Ubuntu 20.04 LTS or higher
  - Debian 10 or higher
  - Fedora 35 or higher
- **Architecture:** amd64 or arm64 (including Raspberry Pi)
- **RAM:** Minimum 512MB (1GB+ recommended)
- **Disk Space:** Minimum 5GB free
- **Network:** 
  - Public IP address
  - Open ports: 9001 (OR port), 9030 (Dir port)
- **Access:** SSH access with sudo privileges

### Software Requirements (Installed by Playbook)
- Docker CE 20.10+
- Docker Compose V2
- Nyx monitoring tool (optional)

## 🚀 Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com/yourusername/anon-relay-ansible-deployment.git
cd anon-relay-ansible-deployment
```

### 2. Install Ansible Collections
```bash
ansible-galaxy collection install -r requirements.yml
```

### 3. Configure Inventory
```bash
# Edit inventory file with your server details
nano inventory.ini
```

Add your relay server(s):
```ini
[relays]
relay1.example.com ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa

[relays:vars]
ansible_python_interpreter=/usr/bin/python3
```

### 4. Configure Variables
```bash
# Edit relay configuration
nano group_vars/relays.yml
```

Minimum required configuration:
```yaml
anon_relay_nickname: "MyAnonRelay"
anon_relay_contact: "operator@example.com"
anon_accept_terms: true  # Required for v0.4.9.7-live+
```

### 5. Deploy
```bash
# Dry run (check mode)
ansible-playbook -i inventory.ini site.yml --check

# Full deployment
ansible-playbook -i inventory.ini site.yml
```

### 6. Verify Deployment
```bash
# Check relay status
ansible relays -i inventory.ini -m shell -a "docker ps | grep anon-relay"

# View logs
ansible relays -i inventory.ini -m shell -a "docker logs anon-relay"
```

## 📦 Installation

### Method 1: From Source (Recommended)
```bash
# Clone repository
git clone https://github.com/yourusername/anon-relay-ansible-deployment.git
cd anon-relay-ansible-deployment

# Install Ansible (if not already installed)
# On Ubuntu/Debian:
sudo apt update
sudo apt install ansible

# On Fedora:
sudo dnf install ansible

# On macOS:
brew install ansible

# Install required Ansible collections
ansible-galaxy collection install -r requirements.yml
```

### Method 2: Ansible Galaxy (Future)
```bash
# Will be available after initial release
ansible-galaxy role install anon_relay_deploy
```

## ⚙️ Configuration

### Inventory Configuration

#### Single Relay
```ini
# inventory.ini
[relays]
relay1.example.com ansible_user=ubuntu
```

#### Multiple Relays
```ini
# inventory.ini
[relays]
relay1.example.com ansible_user=ubuntu
relay2.example.com ansible_user=ubuntu
relay3.example.com ansible_user=debian

[relays:vars]
ansible_python_interpreter=/usr/bin/python3
```

### Variable Configuration

Variables can be set at multiple levels:

#### Global Variables (`group_vars/all.yml`)
```yaml
---
# Docker Configuration
docker_edition: 'ce'
docker_compose_version: "latest"

# Base Directories
anon_base_dir: "/opt/anon"
compose_dir: "/opt/compose-files"

# User Configuration
anon_user: "anond"
anon_uid: 100
anon_gid: 101
```

#### Relay-Specific Variables (`group_vars/relays.yml`)
```yaml
---
# === REQUIRED CONFIGURATION ===

# Relay nickname (max 19 characters, alphanumeric)
anon_relay_nickname: "MyAnonRelay"

# Contact email (publicly visible)
anon_relay_contact: "operator@example.com"

# Terms acceptance (required for v0.4.9.7-live+)
anon_accept_terms: true


# === OPTIONAL CONFIGURATION ===

# Bandwidth limits
anon_relay_bandwidth_rate: "100 MBytes"
anon_relay_bandwidth_burst: "200 MBytes"

# Port configuration
anon_relay_or_port: 9001
anon_relay_dir_port: 9030

# Docker image version
anon_docker_image: "svforte/anon:latest"
```

#### Host-Specific Variables (`host_vars/relay1.example.com.yml`)
```yaml
---
# Override for specific host
anon_relay_nickname: "Relay1Custom"
anon_relay_bandwidth_rate: "200 MBytes"
```

### Sensitive Data with Ansible Vault

Store sensitive data securely using Ansible Vault:

```bash
# Create encrypted vault file
ansible-vault create group_vars/all/vault.yml

# Add encrypted variables
vault_relay_contact: "secret@example.com"
vault_api_key: "your-secret-key"

# Reference in group_vars/relays.yml
anon_relay_contact: "{{ vault_relay_contact }}"
```

Run playbooks with vault:
```bash
ansible-playbook -i inventory.ini site.yml --ask-vault-pass
```

## 🎯 Usage

### Deploy Complete Infrastructure
```bash
ansible-playbook -i inventory.ini site.yml
```

### Deploy to Specific Host
```bash
ansible-playbook -i inventory.ini site.yml --limit relay1.example.com
```

### Deploy Only Docker (Skip Relay)
```bash
ansible-playbook -i inventory.ini site.yml --tags docker
```

### Update Existing Relays
```bash
ansible-playbook -i inventory.ini update.yml
```

### Check Configuration (Dry Run)
```bash
ansible-playbook -i inventory.ini site.yml --check
```

### Verbose Output (Debugging)
```bash
ansible-playbook -i inventory.ini site.yml -vvv
```

### Remove Relay Deployment
```bash
ansible-playbook -i inventory.ini remove.yml
```

### Monitor Relays
```bash
# Install Nyx monitoring tool
ansible-playbook -i inventory.ini monitor.yml

# Access Nyx on target server
ssh relay1.example.com
sudo nyx -s /opt/anon/run/anon/control
```

## 📁 Project Structure

```
anon-relay-ansible-deployment/
├── .github/
│   └── workflows/
│       └── ci.yml                    # CI/CD pipeline
├── .gitignore                        # Git ignore patterns
├── AGENTS.md                         # AI agent instructions
├── CLAUDE.md                         # Claude-specific rules
├── COPILOT.md                        # Copilot instructions
├── DEVELOPMENT_PLAN.md               # Project roadmap
├── README.md                         # This file
├── CONTRIBUTING.md                   # Contribution guide
├── SECURITY.md                       # Security policy
├── CHANGELOG.md                      # Version history
├── LICENSE                           # MIT License
├── ansible.cfg                       # Ansible configuration
├── requirements.yml                  # Galaxy dependencies
├── inventory.ini                     # Inventory example
├── site.yml                          # Main playbook
├── deploy.yml                        # Deployment playbook
├── update.yml                        # Update playbook
├── remove.yml                        # Cleanup playbook
├── monitor.yml                       # Monitoring setup
├── group_vars/
│   ├── all.yml                       # Global variables
│   └── relays.yml                    # Relay variables
├── host_vars/                        # Host-specific vars
├── roles/
│   ├── docker_setup/                 # Docker installation
│   ├── anon_relay_deploy/            # Relay deployment
│   └── anon_relay_monitor/           # Monitoring setup
├── examples/                         # Example configs
├── docs/                             # Documentation
└── tests/                            # Test files
```

## 📚 Documentation

### Core Documentation
- **[AGENTS.md](AGENTS.md)** - Universal AI coding agent instructions
- **[CLAUDE.md](CLAUDE.md)** - Claude-specific configuration
- **[DEVELOPMENT_PLAN.md](DEVELOPMENT_PLAN.md)** - Project development roadmap
- **[CONTRIBUTING.md](CONTRIBUTING.md)** - Contribution guidelines
- **[SECURITY.md](SECURITY.md)** - Security policies

### Additional Guides
- **[docs/architecture.md](docs/architecture.md)** - System architecture overview
- **[docs/configuration.md](docs/configuration.md)** - Detailed configuration guide
- **[docs/troubleshooting.md](docs/troubleshooting.md)** - Common issues and solutions
- **[docs/faq.md](docs/faq.md)** - Frequently asked questions

### Role Documentation
- **[roles/docker_setup/README.md](roles/docker_setup/README.md)** - Docker installation role
- **[roles/anon_relay_deploy/README.md](roles/anon_relay_deploy/README.md)** - Relay deployment role
- **[roles/anon_relay_monitor/README.md](roles/anon_relay_monitor/README.md)** - Monitoring role

### External Resources
- [Anyone Protocol Documentation](https://docs.anyone.io)
- [Docker Installation Guide](https://docs.docker.com/engine/install/)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)

## 🐛 Troubleshooting

### Common Issues

#### Issue: "Permission denied" errors
**Solution:** Ensure the user has sudo privileges
```bash
# Add user to sudoers
sudo usermod -aG sudo ubuntu

# Or configure passwordless sudo
echo "ubuntu ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ubuntu
```

#### Issue: "Module not found: community.docker"
**Solution:** Install required Ansible collections
```bash
ansible-galaxy collection install community.docker
# Or install all requirements
ansible-galaxy collection install -r requirements.yml
```

#### Issue: Relay container not starting
**Solution:** Check Docker logs and configuration
```bash
# View container logs
docker logs anon-relay

# Check container status
docker ps -a | grep anon

# Verify configuration
cat /opt/anon/etc/anon/anonrc
```

#### Issue: Ports 9001/9030 not accessible
**Solution:** Configure firewall
```bash
# Ubuntu/Debian (UFW)
sudo ufw allow 9001/tcp
sudo ufw allow 9030/tcp
sudo ufw reload

# Fedora (firewalld)
sudo firewall-cmd --permanent --add-port=9001/tcp
sudo firewall-cmd --permanent --add-port=9030/tcp
sudo firewall-cmd --reload
```

#### Issue: "Terms and conditions not accepted"
**Solution:** Set `anon_accept_terms: true` in variables
```yaml
# group_vars/relays.yml
anon_accept_terms: true
```

### Debug Mode
```bash
# Run with maximum verbosity
ansible-playbook -i inventory.ini site.yml -vvvv

# Check Ansible facts
ansible all -i inventory.ini -m setup

# Test connectivity
ansible all -i inventory.ini -m ping
```

### Getting Help
1. Check [docs/troubleshooting.md](docs/troubleshooting.md) for detailed solutions
2. Review [docs/faq.md](docs/faq.md) for common questions
3. Open an issue on [GitHub Issues](https://github.com/yourusername/anon-relay-ansible-deployment/issues)
4. Join the [Anyone Protocol Discord](https://discord.gg/anyone)

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details on:
- Code of Conduct
- Development workflow
- Coding standards
- Testing requirements
- Pull request process

### Quick Contribution Guide
```bash
# Fork and clone
git clone https://github.com/yourusername/anon-relay-ansible-deployment.git

# Create feature branch
git checkout -b feature/your-feature-name

# Make changes and test
ansible-playbook site.yml --syntax-check
ansible-lint site.yml

# Commit with conventional commits format
git commit -m "feat(role): add new feature"

# Push and create pull request
git push origin feature/your-feature-name
```

## 📋 Roadmap

- [x] Phase 1: Project structure and documentation
- [x] Phase 2: Docker installation role
- [ ] Phase 3: Anon relay deployment role
- [ ] Phase 4: Monitoring and management role
- [ ] Phase 5: Main playbooks and inventory
- [ ] Phase 6: Comprehensive testing
- [ ] Phase 7: Complete documentation
- [ ] Phase 8: Security hardening

See [DEVELOPMENT_PLAN.md](DEVELOPMENT_PLAN.md) for detailed roadmap.

## 🧪 Testing

### Run Tests Locally
```bash
# Syntax check
ansible-playbook site.yml --syntax-check

# Linting
ansible-lint site.yml
yamllint .

# Molecule tests (if available)
molecule test

# Integration tests
cd tests
ansible-playbook -i inventory/test_hosts.ini test.yml
```

### CI/CD Pipeline
This project uses GitHub Actions for continuous integration. Tests run automatically on:
- Every push to main/develop branches
- Every pull request
- Nightly builds

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👥 Authors and Acknowledgments

### Authors
- **Elvis Nuno** - *Initial work* - [@enuno](https://github.com/enuno)

### Contributors
See the list of [contributors](https://github.com/yourusername/anon-relay-ansible-deployment/contributors) who participated in this project.

### Acknowledgments
- Anyone Protocol team for creating the decentralized relay network
- Ansible community for excellent automation tools
- All contributors and relay operators

## 🔗 Links

- **Anyone Protocol:** https://anyone.io
- **Documentation:** https://docs.anyone.io
- **GitHub:** https://github.com/anyone-protocol
- **Discord:** https://discord.gg/anyone
- **Twitter:** [@anyone_protocol](https://twitter.com/anyone_protocol)

## 📞 Support

- **Documentation:** Start with this README and explore [docs/](docs/)
- **Issues:** [GitHub Issues](https://github.com/yourusername/anon-relay-ansible-deployment/issues)
- **Discussions:** [GitHub Discussions](https://github.com/yourusername/anon-relay-ansible-deployment/discussions)
- **Email:** support@example.com
- **Discord:** [Anyone Protocol Discord](https://discord.gg/anyone)

## 🌟 Show Your Support

If this project helped you, please consider:
- ⭐ Starring the repository
- 🐛 Reporting bugs or suggesting features
- 🔀 Contributing code or documentation
- 📢 Sharing with others who might benefit

## 📊 Project Status

![GitHub last commit](https://img.shields.io/github/last-commit/yourusername/anon-relay-ansible-deployment)
![GitHub issues](https://img.shields.io/github/issues/yourusername/anon-relay-ansible-deployment)
![GitHub pull requests](https://img.shields.io/github/issues-pr/yourusername/anon-relay-ansible-deployment)

**Current Version:** 1.0.0 (In Development)  
**Status:** Active Development  
**Stability:** Alpha

---

**Made with ❤️ for the Anyone Protocol community**

*Help us build a more private, decentralized internet.*
