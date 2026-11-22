# Ansible Role: Development

Installs common development tooling for infrastructure, platform engineering, Kubernetes, and GitOps workflows.

Supported platforms:

- Debian 12 (Bookworm)
- Debian 13 (Trixie)
- Ubuntu 22.04 (Jammy)
- Ubuntu 24.04 (Noble)

---

## Managed Tooling

The role can manage:

- Ansible
- OpenTofu
- Infisical CLI
- kubectl
- Helm
- Talosctl
- ArgoCD CLI
- Development packages
- Custom command wrappers

---

## Example

```yaml
- hosts: development
  become: true

  roles:
    - role: konkele.development
```

---

## Feature Management

Enable or disable components independently.

```yaml
development_ansible_manage: true
development_argocd_cli_manage: true
development_opentofu_manage: true
development_infisical_manage: true
development_kubectl_manage: true
development_helm_manage: true
development_talosctl_manage: true
development_packages_manage: true
development_commands_manage: false
```

---

## Common Configuration

### ArgoCD CLI

Pin the ArgoCD CLI version:

```yaml
development_argocd_cli_version: "v3.6.2"
```

Disable installation:

```yaml
development_argocd_cli_manage: false
```

---

### Kubernetes Repository Track

```yaml
development_kubectl_repository_version: "v1.36"
```

Example:

```yaml
development_kubectl_repository_version: "v1.37"
```

---

### Ansible Virtual Environment

Default installation path:

```yaml
development_ansible_path: /opt/ansible
```

Additional Python packages:

```yaml
development_ansible_python_packages:
  - ansible
  - ansible-lint
  - proxmoxer
  - requests
  - netaddr
```

Example:

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

### Development Packages

Override the default package list:

```yaml
development_packages:
  - git
  - jq
  - yq
  - tmux
  - htop
```

---

## Custom Commands

Enable command management:

```yaml
development_commands_manage: true
```

### Simple Aliases

```yaml
development_commands:
  - name: k
    command: kubectl

  - name: t
    command: talosctl

  - name: a
    command: argocd
```

Creates:

```text
/usr/local/bin/k
/usr/local/bin/t
/usr/local/bin/a
```

---

### Custom Wrappers

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

Each command must define:

- `name`
- one of:
  - `command`
  - `content`

---

## Example Inventory

Install all tooling:

```yaml
development_ansible_manage: true
development_argocd_cli_manage: true
development_opentofu_manage: true
development_infisical_manage: true
development_kubectl_manage: true
development_helm_manage: true
development_talosctl_manage: true
```

Pin ArgoCD:

```yaml
development_argocd_cli_version: "v3.6.2"
```

Upgrade Kubernetes tooling track:

```yaml
development_kubectl_repository_version: "v1.37"
```

---

## Tags

```text
ansible
argocd-cli
opentofu
infisical
kubectl
helm
talosctl
packages
commands
```
