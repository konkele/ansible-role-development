# Ansible Role: Development

The **`development` role** provides a consistent development and infrastructure engineering workstation experience for Debian and Ubuntu systems.

The role installs and configures a curated set of development tools commonly used by platform engineers, DevOps engineers, SREs, and homelab operators. Tools are installed using vendor-supported package repositories whenever possible, allowing updates to be managed through the operating system package manager while maintaining a simple and maintainable implementation.

In addition to development tooling, the role can install common development packages and configure an isolated Ansible installation.

---

## Requirements

This role performs system-level configuration and executes privileged tasks using Ansible privilege escalation.

The account used to run the role must be permitted to use sudo on the target host.

Supported platforms:

- Debian 12 (Bookworm)
- Debian 13 (Trixie)
- Ubuntu 22.04 (Jammy)
- Ubuntu 24.04 (Noble)

---

## Features

### Development Packages

- Install common development packages
- Install shell productivity tools
- Install infrastructure automation utilities
- Install networking and troubleshooting tools
- Skip package installation when disabled

### Ansible

- Install Ansible in an isolated Python virtual environment
- Install Ansible Lint
- Create command wrappers in `/usr/local/bin`
- Avoid system-wide Python package conflicts

### OpenTofu

- Configure the official OpenTofu package repository
- Install OpenTofu using APT
- Receive updates through standard package management

### Infisical CLI

- Configure the official Infisical package repository
- Install the Infisical CLI using APT
- Receive updates through standard package management

### Kubernetes Tooling

- Configure the official Kubernetes package repository
- Install kubectl using APT
- Support configurable Kubernetes repository tracks

### Helm

- Configure the Helm package repository
- Install Helm using APT
- Receive updates through standard package management

### Talosctl

- Download Talosctl directly from official releases
- Automatically select the correct architecture
- Install to `/usr/local/bin`

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
```

---

### Ansible Configuration

The Ansible virtual environment location:

```yaml
development_ansible_path: /opt/ansible
```

Default installation layout:

```text
/opt/ansible/.venv
```

Wrapper commands are installed into:

```text
/usr/local/bin/ansible
/usr/local/bin/ansible-playbook
/usr/local/bin/ansible-galaxy
/usr/local/bin/ansible-lint
```

---

### Kubernetes Repository Track

The Kubernetes package repository can be configured using:

```yaml
development_kubernetes_minor_version: "v1.36"
```

Example:

```yaml
development_kubernetes_minor_version: "v1.37"
```

This determines which Kubernetes package repository is configured for kubectl installations.

---

### Development Packages

Packages are installed using APT with `state: present`.

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

Additional packages can be added or the list can be replaced entirely:

```yaml
development_packages:
  - git
  - jq
  - yq
  - tmux
  - htop
```

---

## Repository Management

The role manages vendor-supported repositories for:

- OpenTofu
- Infisical
- Kubernetes
- Helm

Repository signing keys are managed automatically.

APT key material is stored within:

```text
/etc/apt/keyrings
```

Infisical uses the vendor-supported keyring location:

```text
/usr/share/keyrings/infisical-archive-keyring.gpg
```

---

## Example Playbook

```yaml
- name: Development Workstation
  hosts: development

  become: true

  roles:
    - role: konkele.development
```

---

## Example Inventory

Install all default tooling:

```yaml
development_ansible_manage: true
development_opentofu_manage: true
development_infisical_manage: true
development_kubectl_manage: true
development_helm_manage: true
development_talosctl_manage: true
```

Disable Kubernetes-related tooling:

```yaml
development_kubectl_manage: false
development_helm_manage: false
development_talosctl_manage: false
```

Track a newer Kubernetes repository:

```yaml
development_kubernetes_minor_version: "v1.37"
```

Override the default package list:

```yaml
development_packages:
  - git
  - jq
  - yq
  - curl
  - tmux
  - htop
```

---

## Example Result

After execution, the host will contain tools such as:

```text
ansible
ansible-playbook
ansible-galaxy
ansible-lint

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
```

Repository-managed tooling receives updates through standard APT workflows.

---

## Tags

| Tag | Description |
|------|-------------|
| `packages` | Development package installation |
| `ansible` | Ansible virtual environment and wrappers |
| `opentofu` | OpenTofu repository and installation |
| `infisical` | Infisical repository and installation |
| `kubectl` | Kubernetes repository and kubectl installation |
| `helm` | Helm repository and installation |
| `talosctl` | Talosctl installation |
