# Ansible Playbooks

Ansible playbooks for bootstrapping homelab servers (Ubuntu/Debian). Each playbook is independent — pick what you need.

## Prerequisites

- [uv](https://docs.astral.sh/uv/) (recommended) or Ansible installed locally
- Target servers running Ubuntu/Debian with SSH access and sudo privileges

## Quick Start

```bash
# 1. Install required collections (community.general — used by the hardening role)
ansible-galaxy collection install -r requirements.yml

# 2. Configure inventory
cp inventory/hosts.ini.example inventory/hosts.ini

# 3. Run playbooks (pick one, several, or all)
uvx --from ansible-core ansible-playbook playbooks/docker.yml -K
uvx --from ansible-core ansible-playbook site.yml -K
```

> Replace `uvx --from ansible-core ansible-playbook` with `ansible-playbook` if Ansible is installed globally.

### Fresh server baseline

`server-baseline.yml` provisions a brand-new server end to end — **system update → security hardening → Docker → zsh**:

```bash
uvx --from ansible-core ansible-playbook playbooks/server-baseline.yml -K
```

Drop `-K` once you've stored the sudo password in Vault (see [Inventory & secrets](#inventory--secrets)).

## Playbooks

### System

| Playbook | Description |
|---|---|
| `common.yml` | Apt update/upgrade, essential packages (`git`, `curl`, `vim`, `neovim`, `tmux`, `ufw`, …), cleanup |
| `hardening.yml` | SSH hardening (key-only, no root), ufw firewall, fail2ban, automatic security updates, sysctl network hardening |
| `docker.yml` | Docker CE + CLI + Compose + Buildx (handles LXC AppArmor) |
| `caddy.yml` | Caddy web server / reverse proxy |
| `tailscale.yml` | Tailscale VPN with IP forwarding & exit node support |
| `zsh.yml` | Zsh + Oh My Zsh, configurable theme |
| `gh.yml` | GitHub CLI (`gh`) from GitHub's official apt repo |
| `proxychains.yml` | proxychains-ng; writes `/etc/proxychains4.conf` from `proxychains_proxies` (only when at least one proxy is set) |

### Language Version Managers

Each role installs the standard version manager for its ecosystem, per-user:

| Playbook | Manager | Default version |
|---|---|---|
| `nvm.yml` | [nvm](https://github.com/nvm-sh/nvm) | Node.js LTS |
| `pyenv.yml` | [pyenv](https://github.com/pyenv/pyenv) | Python 3.13 |
| `uv.yml` | [uv](https://docs.astral.sh/uv/) | Python (uv as manager) |
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
uvx --from ansible-core ansible-playbook playbooks/uv.yml -e 'uv_python_version=3.12'
uvx --from ansible-core ansible-playbook playbooks/gvm.yml -e 'go_version=go1.22.4'
uvx --from ansible-core ansible-playbook playbooks/zsh.yml -e 'zsh_theme=agnoster'
uvx --from ansible-core ansible-playbook playbooks/docker.yml -e 'docker_add_user_to_group=true'
uvx --from ansible-core ansible-playbook playbooks/tailscale.yml -e 'tailscale_auth_key=tskey-auth-xxxxx'
```

All variables are defined in `roles/<role>/defaults/main.yml`.

## Security hardening

`hardening.yml` applies sensible, secure-by-default server hardening. Every area
is an independent toggle (all default to `true`) in `roles/hardening/defaults/main.yml`:

| Area | What it does |
|---|---|
| `harden_ssh` | Drop-in `sshd_config.d/99-hardening.conf`: no password login (key-only), no root login, `MaxAuthTries`, keepalive, X11 off. Validated with `sshd -t` before reload. |
| `harden_ufw` | Enables ufw, default-deny incoming. The **active SSH port is always allowed automatically**, so a run can't lock you out. Add more via `harden_ufw_allowed_tcp_ports`. |
| `harden_fail2ban` | Installs fail2ban with an `sshd` jail (systemd backend). |
| `harden_unattended_upgrades` | Automatic security updates; optional auto-reboot. |
| `harden_sysctl` | Network sysctl hardening. Deliberately leaves `ip_forward` alone so Docker keeps working. |

> **Before disabling password login, confirm key-based SSH already works** — otherwise you'll lock yourself out.

## Inventory & secrets

Real inventory (`inventory/hosts.ini`) and `inventory/group_vars/*` / `host_vars/*`
are **gitignored** — only the `*.example` templates are committed. Never commit
real IPs, keys, or passwords.

Group hosts so they share config, with per-host secrets kept separate:

```ini
[servers:children]      # parent — group_vars/servers.yml applies to all
web1
web2

[web1]                  # leaf — group_vars/web1.yml holds web1's secrets
<IP>:22 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_ed25519 ansible_python_interpreter=/usr/bin/python3
```

### Sudo password via Ansible Vault (drop the `-K` flag)

1. Create a vault password file (kept outside the repo) and point `ansible.cfg` at it
   (`vault_password_file = ~/.ansible_vault_pass`):

   ```bash
   openssl rand -base64 32 > ~/.ansible_vault_pass && chmod 600 ~/.ansible_vault_pass
   ```

2. Encrypt each host's sudo password into its `group_vars/<host>.yml`:

   ```bash
   ansible-vault encrypt_string --stdin-name 'ansible_become_password'
   # type the sudo password, Enter, Ctrl-D, then paste the output into the file
   ```

Runs then need neither `-K` nor `--ask-vault-pass`. If a host has passwordless
sudo, skip this entirely.
