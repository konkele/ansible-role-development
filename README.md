# Ansible Role: Development

The **`development`** role provides a consistent development workstation experience for infrastructure engineers, platform engineers, SREs, DevOps practitioners, and homelab operators.

The role installs common development and automation tooling using vendor-supported repositories whenever possible, provides an isolated Ansible installation, and supports configurable custom commands to standardize environment-specific workflows.

---

## Requirements

This role performs system-level configuration and requires privilege escalation.

The executing account must be permitted to use sudo on the target host.

Supported platforms:

- Debian 12 (Bookworm)
- Debian 13 (Trixie)
- Ubuntu 22.04 (Jammy)
- Ubuntu 24.04 (Noble)

---

## Features

### Development Packages

- Install common development packages
- Install shell productivity utilities
- Install networking and troubleshooting tools
- Install infrastructure automation utilities
- Fully customizable package list

### Ansible

- Install Ansible in an isolated Python virtual environment
- Install Ansible Lint
- Install configurable Python dependencies for Ansible collections
- Create command wrappers in `/usr/local/bin`
- Avoid system-wide Python conflicts

### OpenTofu

- Configure the official OpenTofu repository
- Install OpenTofu through APT
- Receive updates through standard package management

### Infisical

- Configure the official Infisical repository
- Install the Infisical CLI through APT
- Receive updates through standard package management

### Kubernetes Tooling

- Configure the official Kubernetes repository
- Install kubectl through APT
- Support configurable Kubernetes repository tracks

### Helm

- Configure the official Helm repository
- Install Helm through APT
- Receive updates through standard package management

### Talosctl

- Download Talosctl directly from official releases
- Automatically select the appropriate architecture
- Install to `/usr/local/bin`

### Development Commands

- Create custom commands in `/usr/local/bin`
- Support simple command aliases
- Support fully customized shell wrappers
- Configure commands through inventory variables
- Standardize workstation workflows

---

## Variables

### Feature Management

Each component can be enabled or disabled independently.

```yaml
development_ansible_manage: true
development_opentofu_manage: true
development_infisical_manage: true
development_kubectl_manage: true
development_helm_manage: true
development_talosctl_manage: true
development_packages_manage: true
development_commands_manage: false
```

---

### Ansible Configuration

Configure the Ansible installation path:

```yaml
development_ansible_path: /opt/ansible
```

Default installation layout:

```text
/opt/ansible/.venv
```

Installed wrapper commands:

```text
/usr/local/bin/ansible
/usr/local/bin/ansible-playbook
/usr/local/bin/ansible-galaxy
/usr/local/bin/ansible-lint
/usr/local/bin/ansible-vault
```

---

### Ansible Python Packages

The Ansible virtual environment can be extended with additional Python
dependencies required by Ansible collections and automation tooling.

Default:

```yaml
development_ansible_python_packages:
  - ansible
  - ansible-lint
  - proxmoxer
  - requests
  - netaddr
```

Example override:

```yaml
development_ansible_python_packages:
  - ansible
  - ansible-lint
  - proxmoxer
  - requests
  - netaddr
  - kubernetes
  - jmespath
```

---

### Kubernetes Repository Track

Configure the Kubernetes repository track:

```yaml
development_kubectl_repository_version: "v1.36"
```

Example:

```yaml
development_kubectl_repository_version: "v1.37"
```

---

### Development Packages

Default package set:

```yaml
development_packages:
  # Development
  - git
  - make
  - build-essential

  # Shell productivity
  - bash-completion
  - less
  - tree

  # Modern CLI tools
  - bat
  - fd-find
  - jq
  - ripgrep
  - yq

  # Infrastructure and automation
  - direnv
  - openssh-client
  - pre-commit
  - rsync

  # Networking
  - curl
  - dnsutils
  - traceroute
```

Override the list completely:

```yaml
development_packages:
  - git
  - jq
  - yq
  - tmux
  - htop
```

---

### Development Commands

Custom commands can be deployed into `/usr/local/bin`.

Enable command management:

```yaml
development_commands_manage: true
```

#### Simple Commands

Create lightweight aliases:

```yaml
development_commands:
  - name: k
    command: kubectl

  - name: t
    command: talosctl
```

This produces:

```text
/usr/local/bin/k
/usr/local/bin/t
```

Equivalent to:

```bash
#!/bin/sh
exec kubectl "$@"
```

and

```bash
#!/bin/sh
exec talosctl "$@"
```

#### Custom Commands

Create fully customized wrappers:

```yaml
development_commands:
  - name: tf
    content: |
      #!/bin/sh

      set -a
      . /etc/infisical/dev1.env
      set +a

      exec infisical run -- tofu "$@"

  - name: ap
    content: |
      #!/bin/sh

      set -a
      . /etc/infisical/dev1.env
      set +a

      exec infisical run -- ansible-playbook "$@"
```

Result:

```text
/usr/local/bin/tf
/usr/local/bin/ap
```

Usage:

```bash
tf plan
tf apply

ap site.yml
```

Each command entry must define:

- `name`
- exactly one of:
  - `command`
  - `content`

---

## Repository Management

The role manages vendor-supported repositories for:

- OpenTofu
- Infisical
- Kubernetes
- Helm

APT signing keys are managed automatically.

APT keyrings are stored in:

```text
/etc/apt/keyrings
```

Infisical uses the vendor-recommended location:

```text
/usr/share/keyrings/infisical-archive-keyring.gpg
```

---

## Example Playbook

```yaml
---
- name: Development Workstation
  hosts: development
  become: true

  roles:
    - role: konkele.development
```

---

## Example Inventory

### Install All Default Tooling

```yaml
development_ansible_manage: true
development_opentofu_manage: true
development_infisical_manage: true
development_kubectl_manage: true
development_helm_manage: true
development_talosctl_manage: true
```

### Disable Kubernetes Tooling

```yaml
development_kubectl_manage: false
development_helm_manage: false
development_talosctl_manage: false
```

### Track a Newer Kubernetes Version

```yaml
development_kubectl_repository_version: "v1.37"
```

### Add Additional Python Dependencies

```yaml
development_ansible_python_packages:
  - ansible
  - ansible-lint
  - proxmoxer
  - netaddr
  - kubernetes
  - jmespath
```

### Configure Custom Commands

```yaml
development_commands_manage: true

development_commands:
  - name: tf
    content: |
      #!/bin/sh

      set -a
      . /etc/infisical/dev1.env
      set +a

      exec infisical run -- tofu "$@"

  - name: ap
    content: |
      #!/bin/sh

      set -a
      . /etc/infisical/dev1.env
      set +a

      exec infisical run -- ansible-playbook "$@"

  - name: k
    command: kubectl

  - name: t
    command: talosctl
```

---

## Example Result

After execution, the host may contain:

```text
ansible
ansible-playbook
ansible-galaxy
ansible-lint
ansible-vault

tofu
infisical

kubectl
helm
talosctl

git
jq
yq
ripgrep
bat
fd-find
direnv

k
t
tf
ap
```

Repository-managed tooling receives updates through standard APT workflows.

The Ansible virtual environment also includes the configured Python
dependencies such as:

```text
proxmoxer
netaddr
```

to support infrastructure automation collections and workflows.

---

## Tags

| Tag | Description |
|------|-------------|
| `packages` | Development package installation |
| `ansible` | Ansible installation and configuration |
| `opentofu` | OpenTofu repository and installation |
| `infisical` | Infisical repository and installation |
| `kubectl` | Kubernetes repository and kubectl installation |
| `helm` | Helm installation |
| `talosctl` | Talosctl installation |
| `commands` | Custom command creation |

---
