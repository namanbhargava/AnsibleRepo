# k3s + Rancher Ansible Automation

Automates single-node k3s cluster setup and Rancher UI deployment on Linux machines.
Each machine gets its own independent cluster.
Rancher UI is accessible at `https://<machine_ip>.sslip.io`.

## How it works

1. Connects to each machine over SSH
2. Installs **k3s** — a lightweight Kubernetes distribution
3. Installs **Helm** — the Kubernetes package manager
4. Installs **cert-manager** — handles TLS certificates inside the cluster
5. Installs **Rancher** — a Kubernetes management UI, exposed via HTTPS

## Prerequisites

- Ansible >= 2.12 installed on your **local machine** (the control node)
- SSH access to each target Linux machine (key-based auth preferred)
- Python 3 on the target machines (Ansible requires it)
- Internet access from the target machines

## Project Structure

```
.
├── ansible.cfg                              # Ansible settings
├── site.yml                                 # Main playbook (entry point)
├── inventory/
│   ├── hosts.ini                            # List of target machines
│   └── group_vars/k3s_nodes/
│       ├── main.yml                         # Version pins and config
│       └── vault.yml                        # Encrypted secrets (bootstrap password)
└── roles/
    ├── k3s/                                 # Installs and configures k3s
    └── rancher/                             # Installs Helm, cert-manager, Rancher
```

## Quickstart

### 1. Add your machines to the inventory

Edit `inventory/hosts.ini`:

```ini
[k3s_nodes]
node1  ansible_host=<your_machine_ip>
```

### 2. Set the Rancher bootstrap password

Edit `inventory/group_vars/k3s_nodes/vault.yml` and set a strong password (min 12 chars):

```yaml
rancher_bootstrap_password: "YourStrongPassword!"
```

Then encrypt it (recommended before committing to git):

```bash
ansible-vault encrypt inventory/group_vars/k3s_nodes/vault.yml
```

### 3. Run the playbook

```bash
# With vault encryption:
ansible-playbook site.yml --ask-vault-pass

# Without vault encryption (if you didn't encrypt vault.yml):
ansible-playbook site.yml
```

### 4. Target a single machine

```bash
ansible-playbook site.yml --limit node1 --ask-vault-pass
```

## After Deployment

- Open `https://<machine_ip>.sslip.io` in your browser
- Accept the self-signed certificate warning
- Log in: username `admin`, password = your `rancher_bootstrap_password`
- Rancher will prompt you to set a permanent password on first login

## Quick verification (SSH into the machine)

```bash
k3s kubectl get nodes                  # Should show: Ready
k3s kubectl get pods -A                # All pods: Running or Completed
curl -k https://<ip>.sslip.io/ping     # Should return: pong
```

## Adding a new machine

1. Add a new line to `inventory/hosts.ini`
2. Run: `ansible-playbook site.yml --limit new_node --ask-vault-pass`

The playbook is idempotent — safe to re-run on existing machines without side effects.

## Component Versions

| Component    | Version      |
|--------------|--------------|
| k3s          | v1.29.4+k3s1 |
| Helm         | v3.15.1      |
| cert-manager | v1.14.5      |
| Rancher      | 2.8.4        |

Update versions in `inventory/group_vars/k3s_nodes/main.yml`.
