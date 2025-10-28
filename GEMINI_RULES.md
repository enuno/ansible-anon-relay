# GEMINI_RULES.md - Google Gemini AI Agent Configuration

## 🎯 Project Overview

**Project:** Ansible Playbook for Anyone Protocol Anon Relay Deployment  
**Type:** Infrastructure as Code (IaC) - DevOps Automation  
**Primary Language:** YAML (Ansible Playbooks)  
**Secondary Languages:** Jinja2 (Templates), Shell (Scripts), Python (Modules)  
**Target Platforms:** Ubuntu, Debian, Fedora (amd64/arm64)  

This document provides Gemini-specific instructions for working on this Ansible automation project.

---

## 🚨 CRITICAL GEMINI-SPECIFIC RULES

### 1. Multi-Modal Understanding
- Leverage Gemini's ability to understand project structure holistically
- When analyzing code, consider the entire context of related files
- Use Gemini's long context window to review multiple roles simultaneously
- Think about relationships between playbooks, roles, and variables

### 2. Structured Response Format
Gemini works best with structured, organized responses. When generating code:

```yaml
# Gemini Output Format for Ansible
---
# Purpose: [Brief description]
# Requirements: [What this accomplishes]
# Dependencies: [What must exist first]

[actual code here with inline comments]
```

### 3. Idempotency Focus
For every Ansible task you generate, explicitly verify:
- Can this run multiple times safely?
- What state is being ensured?
- Are there any side effects?
- Is `changed_when` needed?

### 4. Error Analysis
When debugging, provide:
1. **Root Cause:** What actually went wrong
2. **Impact:** What this affects
3. **Solution:** Step-by-step fix
4. **Prevention:** How to avoid in future
5. **Related Issues:** Similar problems to check

---

## 📋 PROJECT CONTEXT FOR GEMINI

### Architecture Overview
```
Control Node (Local Machine)
    ↓ [SSH]
Target Nodes (Relay Servers)
    ↓ [Ansible Tasks]
Docker Containers
    ↓ [Anon Relay Software]
Anyone Network
```

### Three Deployment Types

**1. Standard Relay**
- Purpose: Middle/guard relay for network routing
- Complexity: Medium
- Legal Risk: Low
- Maintenance: Low

**2. Exit Relay**
- Purpose: Final hop to public internet
- Complexity: High
- Legal Risk: **HIGH** (requires legal consultation)
- Maintenance: High (abuse complaints)

**3. SOCKS Proxy**
- Purpose: Local network proxy for LAN devices
- Complexity: Low
- Legal Risk: Low
- Maintenance: Low

### Critical Project Constraints
1. **Idempotency:** ALL tasks must be idempotent
2. **Security:** Use Ansible Vault, proper permissions
3. **Cross-Platform:** Ubuntu, Debian, Fedora support
4. **Documentation:** Every change needs documentation
5. **Testing:** Syntax, lint, and idempotency checks required

---

## 🔧 GEMINI CODING PATTERNS

### Pattern 1: Platform-Agnostic Task
```yaml
# ✅ RECOMMENDED PATTERN
- name: Ensure Docker is installed
  package:
    name:
      - docker-ce
      - docker-ce-cli
      - containerd.io
    state: present
  become: yes

# ❌ AVOID PLATFORM-SPECIFIC
- name: Install Docker on Ubuntu
  apt:
    name: docker-ce
    state: present
```

### Pattern 2: Conditional Platform Logic
```yaml
# ✅ RECOMMENDED PATTERN
- name: Include distribution-specific tasks
  include_tasks: "{{ ansible_distribution | lower }}.yml"

# Files: debian.yml, ubuntu.yml, fedora.yml
```

### Pattern 3: Error Handling with Block/Rescue
```yaml
# ✅ RECOMMENDED PATTERN
- name: Deploy relay with error handling
  block:
    - name: Pull Docker image
      docker_image:
        name: svforte/anon:latest
        source: pull
    
    - name: Start container
      docker_container:
        name: anon-relay
        image: svforte/anon:latest
        state: started
  
  rescue:
    - name: Log error
      debug:
        msg: "Failed to deploy relay: {{ ansible_failed_result }}"
    
    - name: Cleanup partial deployment
      docker_container:
        name: anon-relay
        state: absent
  
  always:
    - name: Verify Docker is running
      service:
        name: docker
        state: started
```

### Pattern 4: Jinja2 Template Best Practices
```jinja2
{# ✅ RECOMMENDED PATTERN #}
# {{ ansible_managed }}
# Anon Relay Configuration
# Generated: {{ template_run_date }}
# Host: {{ inventory_hostname }}

Nickname {{ anon_relay_nickname | default('AnonRelay') }}
ContactInfo {{ anon_relay_contact }}

{% if anon_relay_bandwidth_rate is defined %}
BandwidthRate {{ anon_relay_bandwidth_rate }}
{% endif %}

{% if anon_relay_bandwidth_burst is defined %}
BandwidthBurst {{ anon_relay_bandwidth_burst }}
{% endif %}
```

---

## 🎨 GEMINI RESPONSE STRUCTURE

### For Code Generation Requests
```markdown
## [Task Name]

### Purpose
[What this code accomplishes and why it's needed]

### Prerequisites
- [Requirement 1]
- [Requirement 2]

### Implementation
[Code with inline comments]

### Verification
[How to test that it works]

### Idempotency Check
[Explanation of how this is idempotent]

### Error Scenarios
- Scenario 1: [What happens and how it's handled]
- Scenario 2: [What happens and how it's handled]
```

### For Debugging Assistance
```markdown
## Issue Analysis

### Problem Summary
[Brief description of the issue]

### Root Cause
[Technical explanation of what went wrong]

### Affected Components
- Component 1: [How it's affected]
- Component 2: [How it's affected]

### Solution Steps
1. [Step 1 with command]
2. [Step 2 with command]
3. [Verification step]

### Prevention
[How to avoid this in future]

### Related Documentation
- [Link to relevant docs]
```

### For Architecture Decisions
```markdown
## Architecture Decision: [Topic]

### Context
[Current situation and constraints]

### Options Considered
1. **Option A:** [Description]
   - Pros: [List]
   - Cons: [List]
   
2. **Option B:** [Description]
   - Pros: [List]
   - Cons: [List]

### Recommendation
[Chosen option with rationale]

### Implementation Plan
1. [Step 1]
2. [Step 2]
3. [Verification]

### Risks & Mitigation
- Risk 1: [Description] → [Mitigation]
```

---

## 🧠 GEMINI CONTEXT UTILIZATION

### File Relationships to Remember
```
site.yml (main playbook)
  ├── Uses: group_vars/all.yml (global settings)
  ├── Calls: roles/docker_setup (Docker installation)
  ├── Calls: roles/anon_relay_base (base setup)
  ├── Calls: roles/anon_relay_standard (standard config)
  ├── Calls: roles/anon_relay_exit (exit config)
  ├── Calls: roles/anon_relay_socks (SOCKS config)
  ├── Calls: roles/anon_relay_monitor (monitoring)
  └── Calls: roles/security_hardening (security)

Variable Precedence (lowest → highest):
1. defaults/main.yml (in roles)
2. group_vars/all.yml
3. group_vars/relays.yml
4. host_vars/hostname.yml
5. extra vars (command line -e)
```

### Key Configuration Patterns
```yaml
# Standard Relay Configuration
anon_relay_type: "standard"
anon_relay_or_port: 9001
anon_relay_socks_port: 0  # Disabled
anon_relay_bandwidth_rate: "100 MBytes"

# Exit Relay Configuration
anon_relay_type: "exit"
anon_relay_exit_relay: 1
anon_relay_exit_policy:
  - "reject *:25"      # Block SMTP
  - "reject *:587"     # Block SMTP Submission
  - "accept *:*"       # Allow others

# SOCKS Proxy Configuration
anon_relay_type: "socks"
anon_socks_port: 9050
anon_socks_bind_address: "192.168.1.10"
anon_relay_or_port: 0  # Disabled (no relay function)
```

---

## 🔍 GEMINI ANALYSIS CAPABILITIES

### Code Review Checklist
When reviewing Ansible code, verify:

**Idempotency:**
- [ ] Tasks can run multiple times safely
- [ ] `state` parameter specified explicitly
- [ ] `changed_when` used for shell commands
- [ ] No destructive operations without checks

**Security:**
- [ ] No secrets in plaintext
- [ ] Proper file permissions (600/644/755)
- [ ] `no_log: true` for sensitive operations
- [ ] Ansible Vault used for credentials

**Cross-Platform:**
- [ ] Distribution-agnostic modules used
- [ ] Platform-specific logic in separate files
- [ ] Variables account for platform differences

**Documentation:**
- [ ] Task names are descriptive
- [ ] Complex logic has inline comments
- [ ] Variables documented in defaults/main.yml
- [ ] Role README.md exists and complete

**Testing:**
- [ ] Can pass ansible-lint
- [ ] Can pass yamllint
- [ ] Syntax check passes
- [ ] Check mode compatible

---

## 📊 GEMINI WORKFLOW INTEGRATION

### Workflow 1: Role Development
```
1. Review AGENTS.md for standards
   ↓
2. Generate role structure with ansible-galaxy init
   ↓
3. Define variables in defaults/main.yml with documentation
   ↓
4. Write tasks/main.yml with idempotency
   ↓
5. Create handlers/main.yml for service management
   ↓
6. Write templates/*.j2 with proper headers
   ↓
7. Document in README.md
   ↓
8. Test locally: syntax → lint → check mode → deploy
```

### Workflow 2: Debugging Ansible Errors
```
1. Collect full error output with -vvv
   ↓
2. Identify error type (syntax, runtime, idempotency)
   ↓
3. Check variable precedence and values
   ↓
4. Verify task ordering and dependencies
   ↓
5. Test in isolation (single task/role)
   ↓
6. Apply fix and re-test idempotency
```

### Workflow 3: Adding New Feature
```
1. Read DEVELOPMENT_PLAN.md for context
   ↓
2. Identify affected roles and playbooks
   ↓
3. Design changes (which files, what tasks)
   ↓
4. Update variables (group_vars, defaults)
   ↓
5. Implement with error handling
   ↓
6. Update documentation (README, CHANGELOG)
   ↓
7. Test thoroughly (all relay types)
   ↓
8. Commit with conventional message
```

---

## 🎯 GEMINI-SPECIFIC OPTIMIZATION

### Leverage Multi-Turn Conversations
Gemini excels at maintaining context across turns. Use this for:
1. **Iterative Refinement:** Start with basic implementation, refine based on feedback
2. **Comprehensive Planning:** Break large tasks into multiple steps
3. **Contextual Debugging:** Reference previous errors and solutions
4. **Knowledge Building:** Build up understanding of the codebase gradually

### Use Structured Prompts
```
Task: [What you need]

Context:
- Working on: [role/playbook name]
- Relay type: [standard/exit/socks]
- Platform: [Ubuntu/Debian/Fedora/all]

Requirements:
- Must be idempotent
- Must support [specific requirement]
- Must handle [error scenario]

Constraints:
- Use native Ansible modules
- Follow project conventions in AGENTS.md
- Include comprehensive error handling

Output Format:
- Ansible code with inline comments
- Explanation of approach
- Testing instructions
```

### Provide Full Context Files
When asking for help with code, provide:
1. The full file content (not just snippet)
2. Related variable files (group_vars, defaults)
3. Error messages (complete output with -vvv)
4. Expected vs actual behavior
5. Platform and environment details

---

## 🛡️ SECURITY AWARENESS FOR GEMINI

### Always Check For:
```yaml
# ❌ INSECURE PATTERN
- name: Deploy API key
  lineinfile:
    path: /etc/config
    line: "api_key=my_secret_key_123"

# ✅ SECURE PATTERN
- name: Deploy API key from vault
  lineinfile:
    path: /etc/config
    line: "api_key={{ vault_api_key }}"
    mode: '0600'
    owner: root
    group: root
  no_log: true
```

### Exit Relay Security Checklist
When working on exit relay code, always consider:
- [ ] Legal disclaimer prominent in documentation
- [ ] NOT recommended for home use clearly stated
- [ ] Restrictive exit policy configured (block high-risk ports)
- [ ] DoS mitigation enabled
- [ ] Exit notice HTML deployed
- [ ] Reverse DNS setup documented
- [ ] Abuse complaint procedures included

---

## 💡 GEMINI PROMPT TEMPLATES

### Template 1: Generate Ansible Role
```
Create an Ansible role for [purpose] with these requirements:

1. Role name: [name]
2. Functionality:
   - [Function 1]
   - [Function 2]
   - [Function 3]

3. Variables needed:
   - [var1]: [description]
   - [var2]: [description]

4. Constraints:
   - Must be idempotent
   - Support Ubuntu/Debian/Fedora
   - Include error handling
   - Follow AGENTS.md conventions

5. Testing:
   - How to verify it works
   - Idempotency check approach

Provide complete role structure with all files.
```

### Template 2: Debug Ansible Error
```
I'm getting this error in my Ansible playbook:

[Paste full error with -vvv output]

Context:
- Playbook: [name]
- Role: [name]
- Task: [name]
- Target: [OS] [version]
- Expected: [what should happen]
- Actual: [what happens]

Please provide:
1. Root cause analysis
2. Step-by-step solution
3. How to test the fix
4. How to prevent this in future
```

### Template 3: Refactor Ansible Code
```
Refactor this Ansible code to improve:
- Idempotency
- Error handling
- Cross-platform compatibility
- Documentation

Current code:
[paste code]

Follow these principles:
- Use native Ansible modules (no shell unless necessary)
- Proper variable namespacing (anon_*)
- Include inline documentation
- Add appropriate tags
- Implement block/rescue for error handling

Explain what you changed and why.
```

---

## 📚 KEY REFERENCES FOR GEMINI

### Project Documentation Hierarchy
1. **AGENTS.md** - Universal standards (READ FIRST)
2. **DEVELOPMENT_PLAN.md** - Project roadmap and architecture
3. **README.md** - User-facing documentation
4. **CLAUDE.md** - Claude-specific notes (useful patterns)
5. **Role README.md files** - Role-specific documentation

### External References
- Ansible Docs: https://docs.ansible.com
- Anyone Protocol: https://docs.anyone.io
- Docker Docs: https://docs.docker.com
- Jinja2 Docs: https://jinja.palletsprojects.com

---

## 🎓 GEMINI LEARNING APPROACH

### When Encountering New Concepts
1. **Ask for clarification** - Don't assume
2. **Reference documentation** - Link to relevant docs
3. **Provide examples** - Show don't just tell
4. **Explain trade-offs** - Why this approach vs alternatives
5. **Consider edge cases** - What could go wrong

### When Generating Solutions
1. **Start simple** - Basic working solution first
2. **Add error handling** - Make it robust
3. **Optimize for idempotency** - Ensure safe re-runs
4. **Document thoroughly** - Explain the why
5. **Provide testing steps** - How to verify it works

---

## ⚡ QUICK REFERENCE

### Common Ansible Commands
```bash
# Validation
ansible-playbook site.yml --syntax-check
ansible-lint site.yml
yamllint .

# Deployment
ansible-playbook -i inventory.ini site.yml --check  # Dry run
ansible-playbook -i inventory.ini site.yml          # Execute
ansible-playbook -i inventory.ini site.yml -vvv     # Verbose

# Testing
ansible all -i inventory.ini -m ping
ansible all -i inventory.ini -m setup | grep ansible_distribution
```

### File Permissions Quick Reference
- **600** - Secrets (owner read/write only)
- **644** - Configs (owner read/write, others read)
- **755** - Executables/Directories (owner full, others read/execute)
- **700** - Sensitive directories (owner only)

---

**Version:** 1.0.0  
**Last Updated:** 2025-10-27  
**Optimized For:** Google Gemini AI  
**Project:** anon-relay-ansible-deployment

---

*Leverage Gemini's long context window and multimodal understanding to build robust, secure, and well-documented Ansible automation.*
