# Changelog

## [0.1.3] - 2026-07-04

### Fixed

- Add fail-fast assert that `bastion_breakglass_ssh_key` is non-empty before the
  `authorized_key` task. The task uses `exclusive: true`, so an empty key silently
  wipes `authorized_keys` and locks out SSH. The assert now aborts the play with a
  clear message explaining how to set `STRATUM_ADMIN_SSH_PUBKEY` when running the
  playbook outside the harness.

## [0.1.2] - 2026-07-03

### Fixed

- Handler "Restart sshd" now uses `{{ bastion_sshd_service }}` (default `ssh`) instead
  of hardcoding `sshd`. On Ubuntu 24.04 the SSH daemon systemd unit is `ssh.service`;
  `sshd` does not exist and caused a fatal "Could not find the requested service sshd"
  when the sshd-hardening task notified the handler. New default `bastion_sshd_service: ssh`
  is overridable (e.g. to `sshd` for RHEL/Fedora targets).

## [0.1.0] - 2026-07-02

### Added

- Initial release: break-glass SSH user, PAM/VPN users, optional sshd hardening
