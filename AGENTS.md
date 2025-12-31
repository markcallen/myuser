# Agent Guidelines for my-user-ansible Ansible Role

This document provides coding guidelines and commands for AI agents working on this Ansible role.

## Project Overview

This is an Ansible role that creates a user, adds them to sudo and docker groups, enables passwordless sudo, and installs SSH public keys from GitHub.

**Key Files:**
- `tasks/main.yml` - Main task definitions
- `handlers/main.yml` - Handler definitions (currently empty)
- `meta/main.yml` - Role metadata and dependencies
- `files/sudoers.j2` - Sudoers template for passwordless sudo
- `README.md` - User-facing documentation

## Testing and Validation Commands

### Syntax Checking
```bash
# Check YAML syntax for all playbooks
ansible-playbook tasks/main.yml --syntax-check

# Lint the entire role
ansible-lint tasks/main.yml

# Validate all YAML files
yamllint .
```

### Testing the Role
```bash
# Test with molecule (if configured)
molecule test

# Dry-run the role with a test playbook
ansible-playbook test.yml --check -i localhost, --connection=local

# Run against a specific host
ansible-playbook -i <hostname>, -u root playbook.yml
```

### Manual Validation
```bash
# Validate sudoers file syntax
visudo -cf files/sudoers.j2

# Check role structure
ansible-galaxy role info .
```

## Code Style Guidelines

### YAML Formatting

**Indentation:**
- Use 2 spaces for indentation (no tabs)
- Maintain consistent indentation throughout files
- Lists should be indented with 2 spaces

**Structure:**
```yaml
---
- name: Descriptive task name in sentence case
  ansible.builtin.module_name:
    parameter: value
    another_parameter: "{{ variable }}"
  register: result_variable
  when: condition
```

### Naming Conventions

**Tasks:**
- Use descriptive, action-oriented names
- Start with a verb (Ensure, Check, Create, Download, etc.)
- Use sentence case
- Example: `Ensure user "{{ user_name }}" exists`

**Variables:**
- Use snake_case: `user_name`, `ssh_key_url`
- Be descriptive and clear
- Avoid abbreviations unless widely understood

**Registered Variables:**
- Use `_result` or descriptive suffix: `sudo_bin`, `docker_bin`
- Clearly indicate what data they contain

### Module Usage

**Always use Fully Qualified Collection Names (FQCN):**
```yaml
# Correct
- name: Create user
  ansible.builtin.user:
    name: "{{ user_name }}"

# Incorrect
- name: Create user
  user:
    name: "{{ user_name }}"
```

**Common modules to use:**
- `ansible.builtin.user` - User management
- `ansible.builtin.file` - File/directory operations
- `ansible.builtin.copy` - Copy files from role
- `ansible.builtin.template` - Template files with Jinja2
- `ansible.builtin.get_url` - Download files
- `ansible.builtin.stat` - Check file existence
- `ansible.builtin.fail` - Fail with custom message

### Error Handling

**Early Failure:**
- Check prerequisites at the start of tasks
- Use `ansible.builtin.stat` to verify binaries exist
- Fail early with descriptive messages

**Example:**
```yaml
- name: Check if 'sudo' is installed
  ansible.builtin.stat:
    path: /usr/bin/sudo
  register: sudo_bin

- name: Fail if 'sudo' is not installed
  ansible.builtin.fail:
    msg: "'sudo' is not installed on the target host. Please install it before running this role."
  when: not sudo_bin.stat.exists
```

### File Permissions

**Always specify security-critical permissions:**
- SSH directories: `0700`
- SSH keys: `0600`
- Sudoers files: `0440`
- Always specify owner and group for user files

**Example:**
```yaml
- name: Ensure .ssh directory exists
  ansible.builtin.file:
    path: "/home/{{ user_name }}/.ssh"
    state: directory
    owner: "{{ user_name }}"
    group: "{{ user_name }}"
    mode: '0700'
```

### Variables and Templating

**Quoting:**
- Quote variables in module parameters: `"{{ user_name }}"`
- Quote file modes: `'0700'` (prevent octal interpretation)
- Quote URLs and paths with special characters

**Best Practices:**
- Use variables for all user-configurable values
- Document required variables in README.md
- Use defaults/main.yml for default values (if needed)

### Documentation

**Task Documentation:**
- Every task must have a descriptive `name` field
- Names should clearly indicate what the task does
- Include variable context in names when relevant

**Role Documentation:**
- Update README.md when adding new features
- Document all required variables
- Provide complete example playbooks
- Explain role dependencies and prerequisites

### File Organization

**Role Structure:**
```
my-user-ansible/
├── tasks/main.yml          # Main task list
├── handlers/main.yml       # Handlers (if needed)
├── templates/              # Jinja2 templates (if needed)
├── files/                  # Static files
├── defaults/main.yml       # Default variables (if needed)
├── vars/main.yml          # Role variables (if needed)
├── meta/main.yml          # Role metadata
└── README.md              # Documentation
```

**Guidelines:**
- Keep tasks/main.yml focused and readable
- Split complex logic into separate task files if needed
- Use `files/` for static content (like sudoers.j2)
- Use `templates/` for dynamic Jinja2 templates

### Security Best Practices

1. **Always validate sudoers files:**
   ```yaml
   validate: '/usr/sbin/visudo -cf %s'
   ```

2. **Set restrictive permissions on sensitive files**

3. **Use HTTPS for downloading content:**
   - GitHub SSH keys: `https://github.com/username.keys`

4. **Avoid hardcoded secrets** - use Ansible Vault if needed

## Commit Guidelines

- Use conventional commit format: `feat:`, `fix:`, `docs:`, `refactor:`
- Keep commits focused and atomic
- Update README.md when changing role behavior
- Test changes before committing

## Common Tasks for Agents

**Adding a new task:**
1. Add to `tasks/main.yml` in logical order
2. Use FQCN for modules
3. Include descriptive name
4. Update README.md if it affects usage

**Modifying variables:**
1. Update variable references in tasks
2. Document in README.md
3. Update example playbooks

**Adding prerequisites:**
1. Add stat check at beginning of tasks
2. Add corresponding fail task
3. Document in README.md
