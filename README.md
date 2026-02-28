# Ansible Playbooks

My collection of Ansible playbooks for bootstrapping and configuring homelab servers (Ubuntu/Debian). Each feature is an **independent playbook** — pick what you need and run it.

## Playbooks

| Playbook | Description |
|---|---|
| `playbooks/common.yml` | System update, install essential packages, cleanup |
| `playbooks/docker.yml` | Install Docker Engine + Compose plugin |
| `playbooks/caddy.yml` | Install Caddy web server (reverse proxy) |
| `playbooks/tailscale.yml` | Install Tailscale VPN |
| `playbooks/zsh.yml` | Install Oh My Zsh, set Zsh as default shell |
| `playbooks/nvm.yml` | Install nvm + Node.js |
| `playbooks/pyenv.yml` | Install pyenv + Python |
| `playbooks/gvm.yml` | Install gvm + Go |
| `playbooks/phpenv.yml` | Install phpenv + PHP |
| `playbooks/rustup.yml` | Install rustup + Rust |
| **`site.yml`** | **Run all of the above** |

## Prerequisites

- [uv](https://docs.astral.sh/uv/) (recommended) or Ansible installed locally
- Target servers running Ubuntu/Debian with SSH access
- Root or sudo privileges on target servers

## Quick Start

### 1. Configure Inventory

```bash
cp inventory/hosts.ini.example inventory/hosts.ini
# Edit inventory/hosts.ini with your server IP, SSH user, and key path
```

### 2. Run Playbooks

Each playbook is independent — run one, several, or all:

```bash
# Run a single playbook
uvx --from ansible-core ansible-playbook playbooks/docker.yml

# Combine multiple playbooks
uvx --from ansible-core ansible-playbook playbooks/common.yml playbooks/docker.yml playbooks/tailscale.yml

# Run everything
uvx --from ansible-core ansible-playbook site.yml
```

Additional options:

```bash
# Use a custom inventory
uvx --from ansible-core ansible-playbook -i inventory/hosts.ini playbooks/docker.yml

# With password authentication (no SSH key)
uvx --from ansible-core ansible-playbook playbooks/docker.yml -k -K

# With verbose output
uvx --from ansible-core ansible-playbook playbooks/docker.yml -vv
```

If you have Ansible installed globally, replace `uvx --from ansible-core ansible-playbook` with `ansible-playbook`.

## Project Structure

```
.
├── site.yml                    # Run all playbooks
├── ansible.cfg                 # Ansible configuration
├── inventory/
│   └── hosts.ini.example       # Example inventory file
├── playbooks/
│   ├── common.yml              # Base system setup
│   ├── docker.yml              # Docker Engine
│   ├── caddy.yml               # Caddy web server
│   ├── tailscale.yml           # Tailscale VPN
│   ├── zsh.yml                 # Zsh + Oh My Zsh
│   ├── nvm.yml                 # nvm + Node.js
│   ├── pyenv.yml               # pyenv + Python
│   ├── gvm.yml                 # gvm + Go
│   ├── phpenv.yml              # phpenv + PHP
│   └── rustup.yml              # rustup + Rust
└── roles/
    ├── common/                 # System update & essential packages
    ├── docker/                 # Docker Engine
    ├── caddy/                  # Caddy web server
    ├── tailscale/              # Tailscale VPN
    ├── zsh/                    # Oh My Zsh
    ├── nvm/                    # nvm (Node.js)
    ├── pyenv/                  # pyenv (Python)
    ├── gvm/                    # gvm (Go)
    ├── phpenv/                 # phpenv (PHP)
    └── rustup/                 # rustup (Rust)
```

## Roles

### System

| Role | Description |
|---|---|
| **common** | Update & upgrade apt, install essential packages (`git`, `curl`, `wget`, `vim`, `tmux`, etc.), cleanup |
| **docker** | Add Docker official GPG key & repo, install Docker CE/CLI/Compose/Buildx, handle LXC AppArmor, add user to docker group |
| **caddy** | Add Caddy official repo, install and enable Caddy web server |
| **tailscale** | Add Tailscale official apt repo, install packages, configure IP forwarding & ethtool optimisations, enable `tailscaled` service, run `tailscale up` |
| **zsh** | Install Oh My Zsh (unattended), set Zsh as default shell |

### Language Version Managers

All language roles install via the **standard version manager** for that ecosystem, keeping versions isolated per user:

| Role | Manager | What it installs |
|---|---|---|
| **nvm** | [nvm](https://github.com/nvm-sh/nvm) | Node.js (default: latest LTS) |
| **pyenv** | [pyenv](https://github.com/pyenv/pyenv) | Python (default: 3.13) |
| **gvm** | [gvm](https://github.com/moovweb/gvm) | Go (default: go1.23.6) |
| **phpenv** | [phpenv](https://github.com/phpenv/phpenv) | PHP (default: 8.4.4) |
| **rustup** | [rustup](https://rustup.rs/) | Rust stable toolchain |

## Customization

Each role defines sensible defaults in `roles/<role>/defaults/main.yml`. Override any variable via `--extra-vars`:

```bash
# Install a specific Node.js version
uvx --from ansible-core ansible-playbook playbooks/nvm.yml -e 'nodejs_version=20'

# Install a specific Python version
uvx --from ansible-core ansible-playbook playbooks/pyenv.yml -e 'python_version=3.12'

# Install a specific Go version
uvx --from ansible-core ansible-playbook playbooks/gvm.yml -e 'go_version=go1.22.4'

# Override target user
uvx --from ansible-core ansible-playbook playbooks/docker.yml -e 'target_user=myuser'

# Tailscale with custom subnet routes
uvx --from ansible-core ansible-playbook playbooks/tailscale.yml -e 'tailscale_advertise_routes=10.0.0.0/24'

# Tailscale with auth key (unattended login)
uvx --from ansible-core ansible-playbook playbooks/tailscale.yml -e 'tailscale_auth_key=tskey-auth-xxxxx'

# Tailscale without exit node
uvx --from ansible-core ansible-playbook playbooks/tailscale.yml -e 'tailscale_advertise_exit_node=false'
```

## License

[MIT](LICENSE)
