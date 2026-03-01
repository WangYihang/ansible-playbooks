# Ansible Playbooks

Ansible playbooks for bootstrapping homelab servers (Ubuntu/Debian). Each playbook is independent — pick what you need.

## Prerequisites

- [uv](https://docs.astral.sh/uv/) (recommended) or Ansible installed locally
- Target servers running Ubuntu/Debian with SSH access and sudo privileges

## Quick Start

```bash
# 1. Configure inventory
cp inventory/hosts.ini.example inventory/hosts.ini

# 2. Run playbooks (pick one, several, or all)
uvx --from ansible-core ansible-playbook playbooks/docker.yml -K
uvx --from ansible-core ansible-playbook site.yml -K
```

> Replace `uvx --from ansible-core ansible-playbook` with `ansible-playbook` if Ansible is installed globally.

## Playbooks

### System

| Playbook | Description |
|---|---|
| `common.yml` | Apt update/upgrade, essential packages (`git`, `curl`, `vim`, `neovim`, `tmux`, `ufw`, …), cleanup |
| `docker.yml` | Docker CE + CLI + Compose + Buildx (handles LXC AppArmor) |
| `caddy.yml` | Caddy web server / reverse proxy |
| `tailscale.yml` | Tailscale VPN with IP forwarding & exit node support |
| `zsh.yml` | Zsh + Oh My Zsh, configurable theme |

### Language Version Managers

Each role installs the standard version manager for its ecosystem, per-user:

| Playbook | Manager | Default version |
|---|---|---|
| `nvm.yml` | [nvm](https://github.com/nvm-sh/nvm) | Node.js LTS |
| `pyenv.yml` | [pyenv](https://github.com/pyenv/pyenv) | Python 3.13 |
| `gvm.yml` | [gvm](https://github.com/moovweb/gvm) | Go 1.23.6 |
| `phpenv.yml` | [phpenv](https://github.com/phpenv/phpenv) | PHP 8.4.4 |
| `rustup.yml` | [rustup](https://rustup.rs/) | Rust stable |
| `sdkman.yml` | [sdkman](https://sdkman.io/) | Java 21.0.6-tem |
| `rbenv.yml` | [rbenv](https://github.com/rbenv/rbenv) | Ruby 3.3.7 |

Run `site.yml` to execute all playbooks in sequence.

## Customization

Override defaults via `-e`:

```bash
uvx --from ansible-core ansible-playbook playbooks/nvm.yml -e 'nodejs_version=20'
uvx --from ansible-core ansible-playbook playbooks/pyenv.yml -e 'python_version=3.12'
uvx --from ansible-core ansible-playbook playbooks/gvm.yml -e 'go_version=go1.22.4'
uvx --from ansible-core ansible-playbook playbooks/zsh.yml -e 'zsh_theme=agnoster'
uvx --from ansible-core ansible-playbook playbooks/docker.yml -e 'docker_add_user_to_group=true'
uvx --from ansible-core ansible-playbook playbooks/tailscale.yml -e 'tailscale_auth_key=tskey-auth-xxxxx'
```

All variables are defined in `roles/<role>/defaults/main.yml`.

## License

[MIT](LICENSE)
