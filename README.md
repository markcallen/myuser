# myuser

Ansible role to:
- Create a user
- Add them to the appropriate admin group (`sudo` on Ubuntu/Debian, `wheel` on RHEL/CentOS) and optionally to the `docker` group
- Enable passwordless sudo for the admin group
- Install SSH public keys from a GitHub URL (appends safely to existing keys)
- Fail early if `sudo` is not installed
- Optionally fail if `docker` is not installed (configurable)

## Features

- **Multi-distribution support**: Automatically detects and uses the correct admin group (`sudo` or `wheel`)
- **Automatic Docker installation**: Optionally installs Docker using `geerlingguy.docker` role when `require_docker: true`
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
require_docker: true          # Whether to install Docker using geerlingguy.docker v7.9.0 (default: true)
user_shell: /bin/bash         # User's default shell (default: /bin/bash)
user_groups: []              # Additional groups to add user to (default: [])
create_home: true            # Whether to create home directory (default: true)
admin_group: ""              # Override auto-detected admin group (default: auto-detect)
```

## Example Playbooks

### Basic Usage

**Prerequisites:**

Ensure you have a compatible version of Ansible installed:

```bash
# Check your Ansible version
ansible --version

# Should be >= 2.9, recommended >= 2.12
# If needed, upgrade Ansible:
pip install --upgrade ansible
```

Install this role:

```bash
ansible-galaxy install git+https://github.com/markcallen/myuser.git
```

Create a playbook from `playbook-example.yml`

```yaml
- name: Provision user
  hosts: all
  become: yes
  vars:
    user_name: marka
    ssh_key_url: "https://github.com/markcallen.keys"
  roles:
    - myuser
```

Run the playbook:

```bash
ansible-playbook -i <hostname>, -u root playbook.yml
```

### Without Docker

If your server doesn't need Docker, set `require_docker: false` to skip Docker installation:

```yaml
- name: Provision user without Docker
  hosts: all
  become: yes
  vars:
    user_name: marka
    ssh_key_url: "https://github.com/markcallen.keys"
    require_docker: false  # Skip Docker installation
  roles:
    - myuser
```

**Note:** When `require_docker: true` (default), this role automatically installs Docker using the `geerlingguy.docker` role (v7.9.0) as a dependency.

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
    - myuser
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
    - myuser
```

## Supported Platforms

- Ubuntu (all versions)
- Debian (all versions)
- RHEL/CentOS (all versions)
- Fedora (all versions)

The role automatically detects the OS family and uses the appropriate admin group.

## Requirements

### Controller Requirements (your machine)

- **Ansible >= 2.9** (recommended: >= 2.12 for best compatibility)
- Python 3.8 or higher
- **ansible-lint** (recommended for development)

To check and upgrade:
```bash
# Check versions
ansible --version
python3 --version

# Upgrade Ansible if needed
pip install --upgrade ansible

# Install ansible-lint (for development)
pip install ansible-lint
```

### Target Host Requirements

- Python 3.6 or higher
- `sudo` must be installed on target hosts
- Internet connectivity (for downloading SSH keys and installing Docker if `require_docker: true`)

**Note:** The included `playbook-example.yml` handles Python setup automatically via a bootstrap play.

### Dependencies

When `require_docker: true` (default), this role depends on:
- **geerlingguy.docker** (v7.9.0) - Handles Docker installation and configuration

**Installing dependencies:**

If you installed this role via `ansible-galaxy` from GitHub, you need to manually install the dependencies:

```bash
# Install the required geerlingguy.docker role
ansible-galaxy install geerlingguy.docker,7.9.0

# Or install all dependencies from the meta/main.yml file
ansible-galaxy install -r meta/main.yml
```

If you're using a `requirements.yml` file for your playbook, include both roles:

```yaml
# requirements.yml
- src: git+https://github.com/markcallen/myuser.git
  name: myuser

- src: geerlingguy.docker
  version: 7.9.0
```

Then install all requirements:

```bash
ansible-galaxy install -r requirements.yml
```

## Troubleshooting

### Role Not Found Error

If you get an error like:
```
ERROR! the role 'geerlingguy.docker' was not found in ansible.legacy:/home/...
```

This means the required `geerlingguy.docker` dependency is not installed. Fix this by installing the dependency:

```bash
# Install the specific version required
ansible-galaxy install geerlingguy.docker,7.9.0

# Or install all dependencies from the role's meta/main.yml
ansible-galaxy install -r meta/main.yml
```

**Note:** If you don't need Docker, you can skip this by setting `require_docker: false` in your playbook variables:

```yaml
vars:
  user_name: marka
  ssh_key_url: "https://github.com/markcallen.keys"
  require_docker: false  # Skip Docker installation
```

### Python Compatibility Issues

If you encounter errors like `ModuleNotFoundError: No module named 'ansible.module_utils.six.moves'`, this indicates a compatibility issue between your Ansible version and the Python environment on the target host.

The included `playbook-example.yml` includes a bootstrap play that handles this automatically by installing required Python packages before running the role:

```yaml
- name: Bootstrap Python for Ansible
  hosts: all
  become: yes
  gather_facts: no
  tasks:
    - name: Install Python and dependencies
      ansible.builtin.raw: |
        apt-get update && apt-get install -y python3 python3-apt python3-distutils
      changed_when: false
```

This bootstrap play:
- Runs before the main provisioning play
- Uses the `raw` module to bypass Python requirements
- Installs necessary Python packages for Ansible to function properly
- Is safe to run multiple times (idempotent)

If you prefer not to use the bootstrap play, you can also upgrade your Ansible installation:

```bash
pip install --upgrade ansible
```

## Safety Features

- **Idempotent**: Safe to run multiple times
- **Non-destructive SSH key handling**: Appends keys instead of overwriting
- **Early validation**: Fails fast with clear error messages
- **Sudo validation**: Uses `visudo -cf` to validate sudoers configuration

## Development

### Running the Role Locally

When developing or testing this role from the source directory (without installing via `ansible-galaxy`), you can run it directly using a local playbook.

#### Option 1: Use the included playbook

If you've cloned this role into a subdirectory, you'll need to create a playbook in the parent directory so Ansible can find the role:

```bash
# From the parent directory containing the cloned myuser/ subdirectory
cp myuser/playbook-example.yml playbook.yml

# Edit playbook.yml to set your desired user_name and ssh_key_url values
nano playbook.yml  # or use your preferred editor

# Run the playbook
ansible-playbook -i <hostname>, -u root playbook.yml

# Locally on your machine
ansible-playbook -i localhost, --connection=local playbook.yml

# Dry-run to see what would change
ansible-playbook -i <hostname>, -u root playbook.yml --check
```

**Why copy to parent directory?**

When you reference a role by name (like `myuser` in the playbook), Ansible searches for it in the current directory and standard role paths. By placing `playbook.yml` in the parent directory, Ansible will automatically find the `myuser/` subdirectory as a local role.

#### Option 2: Create a test playbook

Create a file `test-local.yml` in the role directory:

```yaml
- name: Test role locally
  hosts: all
  become: yes
  vars:
    user_name: testuser
    ssh_key_url: "https://github.com/yourusername.keys"
    require_docker: false  # Optional: disable Docker requirement for testing
  roles:
    - role: .  # The dot means "use the role in current directory"
```

Run it:

```bash
ansible-playbook -i <hostname>, -u root test-local.yml
```

#### Option 3: Run tasks directly

For quick testing of the tasks themselves:

```bash
# Syntax check
ansible-playbook tasks/main.yml --syntax-check

# Run with inline variables (requires all variables)
ansible-playbook tasks/main.yml -i <hostname>, -u root \
  -e "user_name=testuser" \
  -e "ssh_key_url=https://github.com/yourusername.keys"
```

### Validation Commands

```bash
# Check YAML syntax
ansible-playbook tasks/main.yml --syntax-check

# Lint the role
ansible-lint tasks/main.yml

# Validate YAML formatting
yamllint .

# Validate sudoers file syntax
visudo -cf files/sudoers.j2
```

### Running ansible-lint Before Committing

Before committing changes to this role, it's important to run `ansible-lint` to ensure code quality and compliance with Ansible best practices.

**Why run ansible-lint?**
- Catches common mistakes and anti-patterns
- Enforces Ansible best practices
- Ensures consistent code style
- Validates FQCN (Fully Qualified Collection Names) usage
- Checks for syntax and formatting issues

**How to run:**

```bash
# Lint the entire role
ansible-lint tasks/main.yml

# Or lint all files in the role
ansible-lint .
```

**Expected output:**

A successful lint will show:
```
Passed: 0 failure(s), 0 warning(s) in 1 files processed
```

**Integration with pre-commit:**

This repository includes a `.pre-commit-config.yaml` file that automatically runs ansible-lint before each commit. To enable it:

```bash
# Install pre-commit (if not already installed)
pip install pre-commit

# Install the git hooks
pre-commit install

# Now ansible-lint will run automatically on every commit
```

**Manual pre-commit run:**

```bash
# Run all pre-commit hooks manually
pre-commit run --all-files
```

With pre-commit hooks enabled, any linting failures will prevent the commit, ensuring that only validated code is committed to the repository.

## License

MIT
