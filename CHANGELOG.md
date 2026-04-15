# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).


## [Unreleased]

tbd


## [1.2.0] - 2026-04-15

### Added

* Add CONTRIBUTING
* Add pre-commit hooks

### Removed

* `lf-rhel.cfg`: Drop the `dhcp-client` package and the `/etc/dhcp/dhclient.conf` retry/timeout tweak from the cloud variant. Since RHEL 8, NetworkManager uses its internal DHCP client by default and does not consult `dhclient.conf`, so the workaround (originally from CentOS Bug Tracker #6866, 2013) had been a no-op. RHEL 10 removed the `dhcp-client` package upstream entirely, so this also unblocks RHEL 10 cloud installs.

### Security

* Harden the CI supply chain: the `pre-commit` install in the pre-commit-autoupdate workflow is now hash-pinned via `.github/pre-commit/requirements.txt` (generated with `pip-compile --generate-hashes --strip-extras`), and `dependabot/fetch-metadata` is pinned to a commit SHA so all GitHub Actions used in `.github/workflows/` are now pinned by hash. The policy is documented in CONTRIBUTING.md under "CI Supply Chain"


## [1.1.1] - 2026-03-20

### Fixed

* Fix `cp` error preventing installs with Rocky 10 images


## [1.1.0] - 2026-03-10

### Added

* `lf-debian.cfg`: Debian preseed configuration (Debian 11+) with LVM partitioning, matching the RHEL kickstart's minimal type
* `lf-ubuntu.cfg`: Ubuntu autoinstall configuration (Ubuntu 20.04+) with LVM partitioning, matching the RHEL kickstart's minimal type


## 1.0.0 - 2026-03-10

### Added

* `/backup` partition
* `nosuid` mount option for `/home` and `/var` in CIS mode
* Sudoers configuration prepared for Ansible use
* One generic `lf-rhel.cfg` kickstart replacing all individual per-distro kickstart files (Fedora 35, Rocky 8, RHEL 8 and their CIS/cloud variants)
* Support for RHEL 8+, Fedora 38+ and compatible
* UEFI and BIOS automatic detection
* Automatic `lfdisk` detection
* CIS hardening mode
* Cloud variant support with cloud-init integration
* SSH key deployment for users
* Error handling for unknown `lftype` values

### Changed

* Do not start systemd-units in chroot environment
* Do not explicitly set network options ([#9](https://github.com/Linuxfabrik/kickstart/issues/9))
* Drop RHEL 7 / CentOS 7 support (EOL); minimum is now RHEL 8+
* Root account no longer has a password (previously set to "password" with account locked)
* Sudoers entry uses user `linuxfabrik` instead of group `%linuxfabrik`
* `rm -rf` replaced with `rm -f` for single file removals in cloud post-script
* Use `truncate` consistently instead of `cat /dev/null >` for emptying files

### Fixed

* Fix SSH user warning ([#8](https://github.com/Linuxfabrik/kickstart/issues/8))
* Make `grub2-mkconfig` EFI-aware
* Add `restorecon` for `authorized_keys`
* Detect mount point before copying files
* Fix `sed` error in post-install script
* `mkdir` without `-p` for `.ssh` directories caused failures when multiple SSH keys were deployed for the same user
* Sudoers file permissions now set to `0440` (previously used default umask)
* Removed `--asprimary` flag from `/boot` partition (meaningless on GPT disk label)
* Added `--erroronfail` to `%post --nochroot` for consistent error handling
* Fixed README inaccuracies: partition counts, root password description, firewalld status per type


[Unreleased]: https://github.com/Linuxfabrik/kickstart/compare/v1.2.0...HEAD
[1.2.0]: https://github.com/Linuxfabrik/kickstart/compare/v1.1.1...v1.2.0
[1.1.1]: https://github.com/Linuxfabrik/kickstart/compare/v1.1.0...v1.1.1
[1.1.0]: https://github.com/Linuxfabrik/kickstart/compare/v1.0.0...v1.1.0
