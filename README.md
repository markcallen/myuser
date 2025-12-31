# my-user-ansible

Ansible role to:
- Create a user
- Add them to the appropriate admin group (`sudo` on Ubuntu/Debian, `wheel` on RHEL/CentOS) and optionally to the `docker` group
- Enable passwordless sudo for the admin group
- Install SSH public keys from a GitHub URL (appends safely to existing keys)
- Fail early if `sudo` is not installed
- Optionally fail if `docker` is not installed (configurable)

## Features

- **Multi-distribution support**: Automatically detects and uses the correct admin group (`sudo` or `wheel`)
- **Docker optional**: Control whether Docker is required with `require_docker` variable
- **Safe SSH key management**: Uses `authorized_key` module to safely append keys without overwriting existing ones
- **Flexible configuration**: Sensible defaults with full customization options
- **Early validation**: Checks required variables and prerequisites before making changes

## Variables

### Required Variables

These must be set when using the role:

```yaml
user_name: marka                              # Username to create
ssh_key_url: "https://github.com/markcallen.keys"  # URL to SSH public keys
```

### Optional Variables

These have sensible defaults but can be overridden:

```yaml
require_docker: true          # Whether to require Docker (default: true)
user_shell: /bin/bash         # User's default shell (default: /bin/bash)
user_groups: []              # Additional groups to add user to (default: [])
create_home: true            # Whether to create home directory (default: true)
admin_group: ""              # Override auto-detected admin group (default: auto-detect)
```

## Example Playbooks

### Basic Usage

Install this role:

```bash
ansible-galaxy install git+https://github.com/markcallen/my-user-ansible.git
```

Create a playbook:

```yaml
- name: Provision user
  hosts: all
  become: yes
  vars:
    user_name: marka
    ssh_key_url: "https://github.com/markcallen.keys"
  roles:
    - my-user-ansible
```

Run the playbook:

```bash
ansible-playbook -i <hostname>, -u root playbook.yml
```

### Without Docker Requirement

If your server doesn't need Docker:

```yaml
- name: Provision user without Docker
  hosts: all
  become: yes
  vars:
    user_name: marka
    ssh_key_url: "https://github.com/markcallen.keys"
    require_docker: false
  roles:
    - my-user-ansible
```

### With Additional Groups

Add user to additional groups:

```yaml
- name: Provision user with extra groups
  hosts: all
  become: yes
  vars:
    user_name: marka
    ssh_key_url: "https://github.com/markcallen.keys"
    user_groups:
      - developers
      - www-data
  roles:
    - my-user-ansible
```

### Custom Shell

Use a different shell:

```yaml
- name: Provision user with zsh
  hosts: all
  become: yes
  vars:
    user_name: marka
    ssh_key_url: "https://github.com/markcallen.keys"
    user_shell: /bin/zsh
  roles:
    - my-user-ansible
```

## Supported Platforms

- Ubuntu (all versions)
- Debian (all versions)
- RHEL/CentOS (all versions)
- Fedora (all versions)

The role automatically detects the OS family and uses the appropriate admin group.

## Requirements

- Ansible >= 2.9
- `sudo` must be installed on target hosts
- `docker` must be installed if `require_docker: true` (default)

## Safety Features

- **Idempotent**: Safe to run multiple times
- **Non-destructive SSH key handling**: Appends keys instead of overwriting
- **Early validation**: Fails fast with clear error messages
- **Sudo validation**: Uses `visudo -cf` to validate sudoers configuration

## License

MIT
