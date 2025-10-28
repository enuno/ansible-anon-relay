# Contributing to Ansible Anon Relay Deployment

First off, thank you for considering contributing to this project! It's people like you that make this a great tool for the Anyone Protocol community.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Process](#development-process)
- [How Can I Contribute?](#how-can-i-contribute)
- [Style Guidelines](#style-guidelines)
- [Testing](#testing)
- [Pull Request Process](#pull-request-process)
- [Community](#community)

---

## Code of Conduct

### Our Pledge

We are committed to providing a welcoming and inspiring community for all. Please be respectful and constructive in your interactions.

### Our Standards

**Positive behavior includes:**
- Using welcoming and inclusive language
- Being respectful of differing viewpoints
- Gracefully accepting constructive criticism
- Focusing on what is best for the community
- Showing empathy towards other community members

**Unacceptable behavior includes:**
- Trolling, insulting/derogatory comments, and personal attacks
- Public or private harassment
- Publishing others' private information without permission
- Other conduct which could reasonably be considered inappropriate

---

## Getting Started

### Prerequisites

Before contributing, ensure you have:
- **Ansible** 2.15 or higher
- **Python** 3.8 or higher
- **Git** for version control
- Basic understanding of Ansible playbooks and roles
- Familiarity with Docker and container orchestration

### Setting Up Development Environment

1. **Fork the repository**
   ```bash
   # Visit https://github.com/yourusername/ansible-anon-relay and click Fork
   ```

2. **Clone your fork**
   ```bash
   git clone https://github.com/YOUR_USERNAME/ansible-anon-relay.git
   cd ansible-anon-relay
   ```

3. **Add upstream remote**
   ```bash
   git remote add upstream https://github.com/original/ansible-anon-relay.git
   ```

4. **Install Ansible collections**
   ```bash
   ansible-galaxy collection install -r requirements.yml
   ```

5. **Install development tools**
   ```bash
   # Ansible Lint
   pip install ansible-lint

   # YAML Lint
   pip install yamllint

   # Pre-commit hooks (optional)
   pip install pre-commit
   pre-commit install
   ```

---

## Development Process

### Branching Strategy

We follow **Git Flow**:

- `main` - Production-ready code
- `develop` - Integration branch for features
- `feature/*` - New features
- `bugfix/*` - Bug fixes
- `hotfix/*` - Critical production fixes
- `release/*` - Release preparation

### Creating a Feature Branch

```bash
# Ensure you're on develop
git checkout develop
git pull upstream develop

# Create feature branch
git checkout -b feature/your-feature-name
```

### Keeping Your Branch Updated

```bash
# Fetch upstream changes
git fetch upstream

# Rebase your branch on develop
git checkout feature/your-feature-name
git rebase upstream/develop
```

---

## How Can I Contribute?

### Reporting Bugs

Before creating bug reports, please check existing issues. When creating a bug report, include:

**Required Information:**
- Ansible version (`ansible --version`)
- Target OS and version
- Python version
- Steps to reproduce
- Expected behavior
- Actual behavior
- Error messages and logs
- Playbook run with `-vvv` verbosity

**Bug Report Template:**
```markdown
## Description
Brief description of the bug

## Environment
- Ansible version: 2.15.0
- Target OS: Ubuntu 22.04
- Python version: 3.10.0

## Steps to Reproduce
1. Run `ansible-playbook site.yml`
2. Observe error...

## Expected Behavior
What should happen

## Actual Behavior
What actually happens

## Error Output
```
[Paste error with -vvv verbosity]
```

## Additional Context
Any other relevant information
```

### Suggesting Enhancements

Enhancement suggestions are tracked as GitHub issues. Include:

- **Clear title** - Descriptive summary
- **Use case** - Why is this needed?
- **Proposed solution** - How would it work?
- **Alternatives considered** - Other approaches?
- **Impact** - Who benefits? Breaking changes?

### Your First Code Contribution

**Good First Issues** are labeled `good-first-issue`:
- Simple bug fixes
- Documentation improvements
- Adding tests
- Improving error messages

**Areas needing help:**
- Testing on different distributions
- Documentation improvements
- Performance optimizations
- Additional relay configurations

---

## Style Guidelines

### Ansible Style Guide

Follow the official [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html) and these project-specific rules:

#### YAML Formatting
```yaml
# ✅ CORRECT: Use 2 spaces for indentation
---
- name: Install package
  apt:
    name: docker-ce
    state: present
  tags:
    - docker

# ❌ WRONG: Don't use tabs or 4 spaces
- name: Install package
    apt:
        name: docker-ce
```

#### Task Naming
```yaml
# ✅ CORRECT: Clear, descriptive, action-oriented
- name: Ensure Docker daemon configuration directory exists

# ❌ WRONG: Vague or missing
- name: Create directory
- name: do stuff
```

#### Variable Naming
```yaml
# ✅ CORRECT: Use role prefix, snake_case
anon_relay_nickname: "MyRelay"
docker_daemon_config_dir: "/etc/docker"

# ❌ WRONG: No prefix, camelCase
nickname: "MyRelay"
daemonConfigDir: "/etc/docker"
```

#### Idempotency
```yaml
# ✅ CORRECT: Idempotent using Ansible modules
- name: Ensure Docker user exists
  user:
    name: docker
    state: present

# ❌ WRONG: Not idempotent with shell commands
- name: Create Docker user
  shell: useradd docker
```

#### Module Selection
```yaml
# ✅ CORRECT: Use native Ansible modules
- name: Copy configuration file
  copy:
    src: config.yaml
    dest: /etc/app/config.yaml

# ❌ WRONG: Using shell when module exists
- name: Copy file
  shell: cp config.yaml /etc/app/config.yaml
```

### Documentation Style

#### Task Comments
```yaml
# ✅ CORRECT: Explain WHY, not WHAT
# Create directory with restrictive permissions to protect relay keys
- name: Ensure Anon configuration directory exists
  file:
    path: /etc/anon
    mode: '0700'

# ❌ WRONG: States the obvious
# This creates a directory
- name: Create directory
  file:
    path: /etc/anon
```

#### Role Documentation
Each role MUST have a README.md with:
- Purpose and description
- Requirements
- Role variables (table format)
- Dependencies
- Example playbook
- Tags
- License

### Git Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation only
- `style:` Code style (formatting, no logic change)
- `refactor:` Code refactoring
- `test:` Adding/updating tests
- `chore:` Maintenance tasks

**Examples:**
```bash
# Good commits
git commit -m "feat(docker): add support for Docker daemon configuration"
git commit -m "fix(relay): resolve port binding issue on IPv6"
git commit -m "docs(readme): update installation instructions"

# Bad commits
git commit -m "fixed stuff"
git commit -m "update"
git commit -m "WIP"
```

---

## Testing

### Pre-Submission Checklist

Before submitting a PR, ensure:

- [ ] Code follows style guidelines
- [ ] All tests pass
- [ ] Documentation is updated
- [ ] Commit messages follow conventions
- [ ] No sensitive data in commits
- [ ] `ansible-lint` passes
- [ ] `yamllint` passes
- [ ] Syntax check passes
- [ ] Tested on target platforms

### Running Tests

```bash
# Syntax check
ansible-playbook site.yml --syntax-check

# Ansible Lint
ansible-lint site.yml
ansible-lint roles/*/

# YAML Lint
yamllint .

# Dry run (requires inventory)
ansible-playbook -i inventory.ini site.yml --check

# Full integration test (requires test environment)
cd tests
ansible-playbook -i inventory/test_hosts.ini test.yml
```

### Testing Environments

Test on these platforms before submitting:
- Ubuntu 20.04 LTS
- Ubuntu 22.04 LTS
- Debian 11
- Fedora 37+

---

## Pull Request Process

### Before Submitting

1. **Update documentation**
   - Role README if variables changed
   - Main README if usage changed
   - CHANGELOG.md with your changes

2. **Test thoroughly**
   - Run all linters
   - Test on supported platforms
   - Verify idempotency (run twice)

3. **Clean commit history**
   ```bash
   # Squash fixup commits
   git rebase -i upstream/develop

   # Force push to your branch
   git push origin feature/your-feature --force-with-lease
   ```

### Submitting the PR

1. **Create pull request** from your fork to upstream `develop`

2. **Use PR template** (fill out all sections):
   ```markdown
   ## Description
   Brief description of changes

   ## Type of Change
   - [ ] Bug fix
   - [ ] New feature
   - [ ] Breaking change
   - [ ] Documentation update

   ## Testing
   - [ ] Tested on Ubuntu 22.04
   - [ ] Tested on Debian 11
   - [ ] Ansible-lint passes
   - [ ] Syntax check passes

   ## Checklist
   - [ ] Code follows style guidelines
   - [ ] Documentation updated
   - [ ] CHANGELOG.md updated
   - [ ] No breaking changes (or documented)
   ```

3. **Link related issues**: Use `Fixes #123` or `Relates to #456`

### PR Review Process

1. **Automated checks** must pass:
   - Syntax validation
   - Linting
   - Basic tests

2. **Code review** by maintainers:
   - At least one approval required
   - Address review comments
   - Update PR as needed

3. **Final approval and merge**:
   - Maintainer merges to `develop`
   - Feature appears in next release

### After Your PR is Merged

1. Delete your feature branch:
   ```bash
   git branch -d feature/your-feature
   git push origin --delete feature/your-feature
   ```

2. Update your fork:
   ```bash
   git checkout develop
   git pull upstream develop
   git push origin develop
   ```

---

## Community

### Getting Help

- **Documentation:** Start with README.md and docs/
- **Issues:** Search existing issues first
- **Discussions:** Use GitHub Discussions for questions
- **Discord:** Join Anyone Protocol Discord

### Communication Channels

- **GitHub Issues:** Bug reports and feature requests
- **GitHub Discussions:** General questions and ideas
- **Pull Requests:** Code contributions
- **Discord:** Real-time community chat

### Recognition

Contributors are recognized in:
- GitHub contributors page
- Release notes (for significant contributions)
- Project README (for major features)

---

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

## Questions?

Don't hesitate to ask! We're here to help:
- Open a GitHub Discussion
- Comment on an existing issue
- Reach out on Discord

**Thank you for contributing!** 🎉
