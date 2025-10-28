# CLAUDE.md - Claude-Specific Agent Configuration

## 🎯 Purpose

This file provides Claude-specific instructions and configurations for the Ansible Playbook for Anyone Protocol Anon Relay Deployment project. **Always refer to AGENTS.md first** for universal project guidelines, then use this file for Claude-specific overrides and enhancements.

---

## 📚 Quick Reference

- **Primary Documentation:** AGENTS.md (universal rules - READ THIS FIRST)
- **Project Plan:** DEVELOPMENT_PLAN.md (roadmap and milestones)
- **Contributing:** CONTRIBUTING.md (for external contributors)
- **This File:** Claude-specific behaviors and optimizations

---

## 🚨 CRITICAL CLAUDE-SPECIFIC RULES

### 1. Context Window Management
- This project uses multiple interconnected files (playbooks, roles, templates)
- When analyzing code, always request the full context of related files
- If discussing a role, review all files within that role's directory
- Maintain awareness of variable precedence across files

### 2. Long-Form Responses for Infrastructure Code
- Ansible playbooks benefit from detailed explanations
- Always explain:
  - **Why** a particular approach was chosen
  - **What** the code does at a high level
  - **How** it handles edge cases
  - **Where** potential issues might arise
  - **When** the code should or shouldn't be used

### 3. YAML Generation Best Practices
- Always validate YAML syntax mentally before presenting
- Use proper indentation (2 spaces, never tabs)
- Include comments for complex logic
- Provide complete, runnable examples
- Test mental execution of idempotency

### 4. Multi-Step Reasoning
When asked to solve problems:
1. Understand the requirement fully
2. Consider multiple approaches
3. Evaluate trade-offs
4. Recommend the best solution with rationale
5. Provide implementation with error handling
6. Suggest testing approaches

---

## 🔧 CLAUDE CAPABILITIES TO LEVERAGE

### Code Generation
When generating Ansible code:

```yaml
# ✅ EXCELLENT: Complete, documented, idempotent
---
# Role: docker_setup
# Purpose: Install Docker CE on Debian/Ubuntu systems
# Reference: https://docs.docker.com/engine/install/ubuntu/

- name: Ensure Docker prerequisites are installed
  apt:
    name:
      - apt-transport-https
      - ca-certificates
      - curl
      - gnupg
      - lsb-release
    state: present
    update_cache: yes
  tags:
    - docker
    - setup

- name: Add Docker GPG key
  apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present
    keyring: /etc/apt/keyrings/docker.gpg
  register: docker_gpg_key
  tags:
    - docker
    - setup

- name: Add Docker repository
  apt_repository:
    repo: "deb [arch={{ ansible_architecture }} signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
    state: present
    filename: docker
  when: docker_gpg_key is succeeded
  tags:
    - docker
    - setup

- name: Install Docker Engine
  apt:
    name:
      - docker-ce
      - docker-ce-cli
      - containerd.io
      - docker-buildx-plugin
      - docker-compose-plugin
    state: present
    update_cache: yes
  notify: restart docker
  tags:
    - docker
    - setup

# Handler for service management
# handlers/main.yml
---
- name: restart docker
  systemd:
    name: docker
    state: restarted
    daemon_reload: yes
  listen: restart docker
```

### Code Review
When reviewing code, provide:
1. **Security Analysis:** Check for exposed secrets, insecure permissions
2. **Idempotency Check:** Verify tasks can run multiple times safely
3. **Performance Review:** Identify inefficient operations
4. **Best Practices:** Suggest Ansible-specific improvements
5. **Documentation Check:** Ensure adequate inline comments
6. **Error Handling:** Verify proper failure scenarios

Example review format:
```markdown
## Code Review: roles/anon_relay_deploy/tasks/main.yml

### ✅ Strengths
- Good use of block/rescue for error handling
- Proper variable namespacing (anon_*)
- Clear task names

### ⚠️ Issues Found

**Issue 1: Missing Idempotency Check**
Location: Line 45
Current:
```yaml
- name: Create anon user
  shell: useradd -M anond
```

Recommended:
```yaml
- name: Ensure anon user exists
  user:
    name: anond
    create_home: no
    state: present
```

**Issue 2: Hardcoded Path**
Location: Line 67
Should use variable: {{ anon_base_dir }}/etc/anon

### 💡 Suggestions
- Add tags for selective execution
- Consider adding check_mode support
- Document why UID 100/GID 101 are required
```

### Troubleshooting Assistance
When helping troubleshoot:

1. **Gather Information**
   - What command was run?
   - What was the expected behavior?
   - What actually happened?
   - What's the environment (OS, Ansible version)?

2. **Analyze the Problem**
   - Review error messages carefully
   - Check related code sections
   - Consider variable precedence
   - Review ansible-playbook output verbosity

3. **Provide Solutions**
   - Explain root cause
   - Suggest immediate fix
   - Recommend long-term improvements
   - Provide working code examples

Example:
```markdown
## Issue: "Permission denied when creating /opt/anon directory"

### Root Cause
The task is trying to create a directory in /opt without elevated privileges.

### Immediate Fix
Add `become: yes` to the play or task:

```yaml
- name: Ensure anon base directory exists
  file:
    path: /opt/anon
    state: directory
    mode: '0755'
  become: yes  # This line grants sudo privileges
```

### Long-term Recommendation
Set `become: yes` at the play level for all infrastructure tasks:

```yaml
- name: Deploy Anon Relay
  hosts: relays
  become: yes  # Applies to all tasks in this play
  
  tasks:
    - name: Ensure anon base directory exists
      file:
        path: /opt/anon
        state: directory
```

### Verification
After applying the fix, verify with:
```bash
ansible-playbook -i inventory.ini site.yml --check -vv
```
```

---

## 🧠 CLAUDE MEMORY UTILIZATION

### Project Context to Maintain
Keep these in working memory during conversations:

1. **Project Goal:** Automate Anon relay deployment via Ansible + Docker
2. **Key Technologies:** Ansible 2.15+, Docker, Anyone Protocol
3. **Supported Platforms:** Ubuntu, Debian, Fedora (amd64/arm64)
4. **Critical Constraints:**
   - Must be idempotent
   - Must support multiple distributions
   - Security-first approach
   - Must accept terms & conditions (v0.4.9.7-live+)

### File Relationships to Remember
```
site.yml
  ├── roles/docker_setup
  │   ├── Uses: group_vars/all.yml
  │   └── Notifies: handlers/main.yml
  ├── roles/anon_relay_deploy
  │   ├── Uses: group_vars/relays.yml
  │   ├── Templates: relay.yaml.j2, anonrc.j2
  │   └── Creates: /opt/anon/* directories
  └── roles/anon_relay_monitor
      └── Uses: defaults/main.yml
```

### Variable Precedence (Remember this hierarchy)
```
defaults/main.yml (lowest)
  ↓
group_vars/all.yml
  ↓
group_vars/relays.yml
  ↓
host_vars/hostname.yml
  ↓
extra vars (-e) (highest)
```

---

## 💬 COMMUNICATION STYLE

### When Explaining Concepts
- Start with high-level overview
- Dive into technical details
- Provide concrete examples
- Reference official documentation
- Offer alternative approaches

### When Writing Documentation
- Use clear, concise language
- Include code examples liberally
- Add visual structure (headers, lists, tables)
- Cross-reference related sections
- Think of the reader as an intermediate user

### When Suggesting Improvements
- Be respectful and constructive
- Explain the "why" behind suggestions
- Provide before/after examples
- Consider the broader impact
- Offer to implement suggestions

---

## 🎨 CODE STYLE PREFERENCES

### Prefer Declarative Over Imperative
```yaml
# ✅ PREFERRED: Declarative
- name: Ensure Docker is installed
  package:
    name: docker
    state: present

# ❌ AVOID: Imperative
- name: Install Docker
  shell: |
    apt-get update
    apt-get install -y docker
```

### Prefer Ansible Modules Over Shell
```yaml
# ✅ PREFERRED: Native module
- name: Create user
  user:
    name: anond
    state: present

# ❌ AVOID: Shell command
- name: Create user
  shell: useradd anond
```

### Prefer Jinja2 Filters Over Complex Logic
```yaml
# ✅ PREFERRED: Jinja2 filter
- name: Set relay nickname
  set_fact:
    relay_nickname: "{{ anon_relay_nickname | default('DefaultRelay') | regex_replace('[^a-zA-Z0-9]', '') }}"

# ❌ AVOID: Multiple conditional tasks
- name: Check if nickname is set
  set_fact:
    relay_nickname: "{{ anon_relay_nickname }}"
  when: anon_relay_nickname is defined

- name: Use default nickname
  set_fact:
    relay_nickname: "DefaultRelay"
  when: anon_relay_nickname is not defined
```

---

## 🔍 ANALYSIS APPROACH

### When Reviewing Existing Code
1. **Read through completely** before suggesting changes
2. **Understand the intent** - what problem is it solving?
3. **Check for patterns** - is it following project conventions?
4. **Test mentally** - will it work? Is it idempotent?
5. **Consider edge cases** - what could go wrong?
6. **Evaluate maintainability** - is it clear to others?

### When Generating New Code
1. **Clarify requirements** - ask questions if unclear
2. **Design before coding** - think through the approach
3. **Start with structure** - create the skeleton
4. **Add implementation** - fill in the details
5. **Include error handling** - handle failure scenarios
6. **Add documentation** - explain non-obvious parts
7. **Provide usage examples** - show how to use it

---

## 📊 RESPONSE STRUCTURE

### For Technical Questions
```markdown
## [Question Topic]

### Quick Answer
[Brief, direct answer to the question]

### Detailed Explanation
[Comprehensive explanation with context]

### Implementation
[Code example]

### Considerations
- Point 1
- Point 2

### References
- [Official docs link]
- [Related section in AGENTS.md]
```

### For Code Requests
```markdown
## [Requested Component]

### Overview
[What this code does and why]

### Prerequisites
[What needs to be in place first]

### Implementation
[Full, working code with comments]

### Usage
[How to use/run the code]

### Testing
[How to verify it works]

### Notes
[Important caveats or tips]
```

---

## 🚀 OPTIMIZATION STRATEGIES

### For This Project

1. **Role Reusability**
   - Design roles to be as generic as possible
   - Use variables for all configurable aspects
   - Document all inputs and outputs clearly

2. **Performance**
   - Minimize task execution time
   - Use `async` for long-running tasks
   - Implement proper `changed_when` conditions
   - Cache facts where appropriate

3. **Maintainability**
   - Keep roles focused on single responsibility
   - Extract common tasks into separate files
   - Use meaningful variable names
   - Document complex logic inline

4. **Testing**
   - Design for testability from the start
   - Use check mode compatibility
   - Implement proper assertions
   - Provide test fixtures

---

## 🔐 SECURITY AWARENESS

### Always Check For:
- [ ] Secrets in plaintext (should use vault)
- [ ] Overly permissive file modes (prefer 600/644/755)
- [ ] Running as root unnecessarily
- [ ] Unvalidated user input
- [ ] Insecure protocols (http instead of https)
- [ ] Missing authentication
- [ ] Exposed sensitive logs (`no_log: true` where needed)

### Security Pattern Example
```yaml
- name: Deploy API key configuration
  template:
    src: api_config.j2
    dest: /etc/anon/api.conf
    mode: '0600'  # Readable only by owner
    owner: anond
    group: anond
  no_log: true  # Don't log sensitive output
  vars:
    api_key: "{{ vault_api_key }}"  # From encrypted vault
```

---

## 🧪 TESTING MINDSET

### Before Suggesting Code, Ask:
1. Is this idempotent? (Can it run twice safely?)
2. Does it handle errors gracefully?
3. Will it work on all supported platforms?
4. Is it testable?
5. Are there edge cases I'm missing?
6. Is it documented well enough?
7. Does it follow project conventions?

### Suggest Tests For:
```yaml
# Example: Suggest this test approach
# Test file: tests/test_docker_setup.yml
---
- name: Test Docker Setup Role
  hosts: test_hosts
  
  tasks:
    - name: Include docker_setup role
      include_role:
        name: docker_setup
    
    - name: Verify Docker is installed
      command: docker --version
      register: docker_version
      changed_when: false
    
    - name: Verify Docker service is running
      systemd:
        name: docker
        state: started
      check_mode: yes
      register: docker_service
    
    - name: Assert Docker is properly configured
      assert:
        that:
          - docker_version.rc == 0
          - "'Docker version' in docker_version.stdout"
          - docker_service.changed == false
```

---

## 📝 DOCUMENTATION GENERATION

### When Creating README Files
Include these sections:
1. **Overview** - What is this?
2. **Requirements** - What's needed?
3. **Installation** - How to set up?
4. **Configuration** - What can be customized?
5. **Usage** - How to use it?
6. **Examples** - Show real scenarios
7. **Troubleshooting** - Common issues
8. **Contributing** - How to help?
9. **License** - Legal stuff
10. **Contact** - Who maintains this?

### When Documenting Variables
```yaml
# ✅ EXCELLENT DOCUMENTATION
---
# defaults/main.yml for anon_relay_deploy role

# === DOCKER CONFIGURATION ===

# Docker image for Anon relay
# Format: repository/image:tag
# Default: Official Anyone Protocol image
# Options: 
#   - svforte/anon:latest (recommended)
#   - svforte/anon:v0.4.9.7-live
anon_docker_image: "svforte/anon:latest"

# === RELAY CONFIGURATION ===

# Relay nickname displayed on Anyone network
# Constraints:
#   - Maximum 19 characters
#   - Alphanumeric only (will be sanitized)
#   - Must be unique across network
# Example: "MyHomeRelay"
anon_relay_nickname: "AnonRelay"

# Contact email for relay operator
# Required: Yes
# Format: Valid email address
# Used for: Network directory listing
# Privacy: Publicly visible on network
anon_relay_contact: "operator@example.com"
```

---

## 🎯 PROJECT-SPECIFIC CONTEXT

### Anyone Protocol Specifics
Remember these key points:
- **v0.4.9.7-live+** requires explicit terms acceptance
- **Ports:** OR port 9001, Dir port 9030 (must be open)
- **User:** Runs as UID 100, GID 101 (anond user)
- **Monitoring:** Uses Nyx tool (install separately)
- **Updates:** Use `docker pull svforte/anon:latest`

### Docker Compose Structure
```yaml
# relay.yaml structure (remember this)
services:
  anon-relay:
    image: svforte/anon:latest
    container_name: anon-relay
    volumes:
      - /opt/anon/etc/anon:/etc/anon
      - /opt/anon/run/anon:/var/lib/anon
    restart: unless-stopped
```

### Directory Permissions
Remember the critical permissions:
- `/opt/anon/run/anon` → 700, owner 100:101
- `/opt/anon/etc/anon/anonrc` → 644, owner root
- `/opt/anon/etc/anon/notices.log` → 644, owner 100:101

---

## 🔄 ITERATIVE IMPROVEMENT

### When Refactoring
1. **Don't break existing functionality**
2. **Maintain backward compatibility** (unless major version)
3. **Add deprecation warnings** for removed features
4. **Update all documentation** simultaneously
5. **Provide migration guides** for breaking changes

### When Adding Features
1. **Check if it fits project scope**
2. **Design for extensibility**
3. **Follow existing patterns**
4. **Add comprehensive tests**
5. **Document thoroughly**

---

## 🎓 LEARNING RESOURCES TO REFERENCE

When providing guidance, reference:
- [Ansible Documentation](https://docs.ansible.com/ansible/latest/)
- [Anyone Protocol Docs](https://docs.anyone.io)
- [Docker Documentation](https://docs.docker.com)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [YAML Specification](https://yaml.org/spec/)
- [Jinja2 Documentation](https://jinja.palletsprojects.com/)

---

## 🤝 COLLABORATION PATTERNS

### When Working with Other Agents
- **Copilot:** Provide concise inline suggestions
- **Gemini:** Share structured data formats
- **GPT-4:** Coordinate on complex multi-file changes
- **Human Developers:** Be explicit, provide rationale, be patient

### When Uncertainty Arises
1. **Acknowledge uncertainty** - don't guess
2. **Explain what you do know**
3. **Suggest where to find answers**
4. **Offer to help research**
5. **Defer to AGENTS.md when appropriate**

---

## ⚡ QUICK COMMANDS FOR REFERENCE

```bash
# === ANSIBLE COMMANDS ===
# Run playbook
ansible-playbook -i inventory.ini site.yml

# Check syntax
ansible-playbook site.yml --syntax-check

# Dry run
ansible-playbook -i inventory.ini site.yml --check

# Verbose output
ansible-playbook -i inventory.ini site.yml -vvv

# === TESTING COMMANDS ===
# Lint playbook
ansible-lint site.yml

# Validate YAML
yamllint .

# Run specific role tests
molecule test -s docker_setup

# === DOCKER COMMANDS ===
# Check relay status
docker ps | grep anon-relay

# View logs
docker logs anon-relay

# Restart relay
docker restart anon-relay

# === NYX MONITORING ===
# Launch Nyx monitor
sudo nyx -s /opt/anon/run/anon/control

# === ANSIBLE VAULT ===
# Create encrypted file
ansible-vault create secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Run with vault
ansible-playbook site.yml --ask-vault-pass
```

---

## 📌 FINAL REMINDERS

1. **ALWAYS** read AGENTS.md first for universal rules
2. **NEVER** commit secrets to version control
3. **TEST** idempotency before marking complete
4. **DOCUMENT** everything thoroughly
5. **FOLLOW** Ansible best practices
6. **THINK** about edge cases
7. **ASK** questions when unclear
8. **PROVIDE** complete, working solutions
9. **EXPLAIN** your reasoning
10. **MAINTAIN** consistency with project standards

---

**This file supplements AGENTS.md with Claude-specific optimizations.**

**Version:** 1.0.0  
**Last Updated:** 2025-10-27  
**Maintained By:** Development Team  
**For:** Claude AI Assistant

---

*When in doubt, consult AGENTS.md → DEVELOPMENT_PLAN.md → This file, in that order.*
