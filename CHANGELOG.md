# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).


## [Unreleased]

### Added

* `/backup` partition
* `nosuid` mount option for `/home` and `/var` in CIS mode
* Sudoers configuration prepared for Ansible use
* One generic `lf-rhel.cfg` kickstart replacing all individual per-distro kickstart files (Fedora 35, Rocky 8, RHEL 8 and their CIS/cloud variants)
* Support for RHEL 7, 8 and 9
* UEFI and BIOS automatic detection for all versions except RHEL 7
* Automatic `lfdisk` detection
* CIS hardening mode
* Cloud variant support with cloud-init integration
* SSH key deployment for users

### Changed

* Do not start systemd-units in chroot environment
* Do not explicitly set network options ([#9](https://github.com/Linuxfabrik/kickstart/issues/9))

### Fixed

* Fix SSH user warning ([#8](https://github.com/Linuxfabrik/kickstart/issues/8))
* Correct ownership of `authorized_keys` for RHEL 7
* Make `grub2-mkconfig` EFI-aware
* Add `restorecon` for `authorized_keys`
* Use `yum` instead of `dnf` for RHEL 7 compatibility
* Detect mount point before copying files
* Fix `sed` error in post-install script
* Fix `.ssh` directory permissions
