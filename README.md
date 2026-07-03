# stratum_factory.bastion_users

Ansible role that declaratively manages local system users on the **STRATUM dev bastion** (Ubuntu 24.04).

## What it does

1. **Break-glass SSH user** — a last-resort / first-config local account.
   - SSH `authorized_key` installed (exclusive — no other keys allowed).
   - Added to the `sudo` group.
   - Passwordless sudo via `/etc/sudoers.d/<user>` (validated with `visudo`).
   - No password login (sshd hardening disables `PasswordAuthentication`).

2. **PAM / VPN users** — ocserv uses PAM authentication, so each VPN identity is a Linux account.
   - Passwords are **never stored in this role**; they are supplied pre-hashed by the wrapper from Vault or CI secrets.

3. **Optional sshd hardening** — sets `PasswordAuthentication no` in `/etc/ssh/sshd_config` and restarts sshd.

## Requirements

- Ansible >= 2.15
- Collections: `ansible.builtin`, `ansible.posix`
- Target: Ubuntu 24.04 (systemd, OpenSSH server pre-installed)
- The `sudo` group must exist on the target (default on Ubuntu).

Install collections:

```bash
ansible-galaxy collection install ansible.posix
```

## Role variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `bastion_breakglass_user` | `stratum-admin` | no | Username for the break-glass account |
| `bastion_breakglass_ssh_key` | `""` | **YES** | SSH public key string (from Vault) |
| `bastion_breakglass_shell` | `/bin/bash` | no | Login shell for break-glass user |
| `bastion_breakglass_sudo` | `true` | no | Install passwordless sudoers entry |
| `bastion_vpn_users` | `[]` | no | List of PAM/VPN user dicts (see below) |
| `bastion_sshd_harden` | `true` | no | Apply sshd password-auth hardening |
| `bastion_sshd_password_auth` | `"no"` | no | Value for `PasswordAuthentication` |
| `bastion_sshd_service` | `"ssh"` | no | systemd unit name for the SSH daemon (`ssh` on Debian/Ubuntu; `sshd` on RHEL) |

### `bastion_vpn_users` item schema

| Key | Required | Description |
|---|---|---|
| `name` | yes | Linux username |
| `password` | yes | Pre-hashed password (SHA-512 recommended). **Never commit plaintext.** |
| `shell` | no | Login shell, defaults to `/bin/bash` |

## Example playbook

```yaml
- name: Configure bastion users
  hosts: bastion
  become: true

  vars:
    bastion_breakglass_user: stratum-admin
    bastion_breakglass_ssh_key: "{{ lookup('ansible.builtin.env', 'BASTION_BREAKGLASS_PUBKEY') }}"
    bastion_vpn_users:
      - name: vpn-alice
        password: "{{ lookup('community.hashi_vault.hashi_vault', 'secret=stratum/dev/vpn/alice:password_hash') }}"
      - name: vpn-bob
        password: "{{ lookup('community.hashi_vault.hashi_vault', 'secret=stratum/dev/vpn/bob:password_hash') }}"

  roles:
    - role: stratum_factory.bastion_users
```

## Security notes

- **Passwords must come from Vault or CI secrets — never committed to any repository.**
  The argument_specs enforce `no_log: true` on all password fields.
- The break-glass authorized_keys is set `exclusive: true` — any manually added keys on
  the target will be removed on the next run.
- sshd hardening (`bastion_sshd_harden: true`) disables all password-based SSH logins
  system-wide. Ensure the break-glass SSH key is correct before enabling this on a live host.
- The sudoers file is validated with `visudo -cf` before being installed.

## Testing

Uses [Molecule](https://ansible.readthedocs.io/projects/molecule/) with the Docker driver:

```bash
molecule test
```

## License

MIT

## Author

apellini / STRATUM Factory
