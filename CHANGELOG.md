# Changelog

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
