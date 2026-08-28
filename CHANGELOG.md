# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).


## [Unreleased]


## [v1.3.2] - 2026-08-05

**Highlights:** Maintenance release. The kickstart files themselves are unchanged, so the `LF_KICKSTART_VERSION` build stamps stay untouched and installed hosts are unaffected. Nothing to do.


## [v1.3.1] - 2026-06-29

### Changed

* `lf-rhel.cfg`: A host installed from this release reports `2026062901` in `/root/lf-install-version`


## [v1.3.0] - 2026-04-27

### Added

* `lf-rhel.cfg`: The `nftables` package is installed on every `lftype` variant (`cis`, `cloud`, `cloud-cis`, `minimal`)

### Changed

* `lf-rhel.cfg`: A host installed from this release reports `2026042701` in `/root/lf-install-version`


## [v1.2.2] - 2026-04-24

### Changed

* All three cfg files carry the same stamp again, `2026042401`


## [v1.2.1] - 2026-04-15

**Highlights:** Changes to the grub configuration reach installed hosts again, cloud installs work on RHEL and Rocky 10, and every installed host records the installer build it came from in `/root/lf-install-version`.

### Added

* A `LF_KICKSTART_VERSION` build stamp (format `YYYYMMDDNN`) is written to `/root/lf-install-version` on every installed host and logged during the install, so a host can be traced back to the installer build that produced it

### Fixed

* `lf-rhel.cfg`: A cloud install on RHEL and Rocky 10 completes, and archives `dynamic.ks` and `70-install-ssh-keys.ks` in `/root` the way it does on 8 and 9
* `lf-rhel.cfg`: A cloud install on RHEL and Rocky 10 no longer fails with `sed: can't read /etc/systemd/logind.conf`
* `lf-rhel.cfg`: The archived `dynamic.ks` names the target disk in its "Only touch" comment instead of a literal `$lfdisk`
* `lf-rhel.cfg`: The console ordering set in `/etc/default/grub` reaches the generated `grub.cfg`; `grub2-mkconfig` was silently skipped on every install so far
* `lf-rhel.cfg`: The SELinux relabel runs on RHEL and Rocky 10 and no longer fills the `%post` log with `Operation not supported` for the EFI partition


## [v1.2.0] - 2026-04-15

### Removed

* `lf-rhel.cfg`: The `dhcp-client` package and its `dhclient.conf` tweak are gone from the cloud variant, which unblocks RHEL 10 cloud installs. NetworkManager has used its internal DHCP client since RHEL 8, so the tweak had no effect anyway


## [v1.1.1] - 2026-03-20

### Fixed

* An install from a Rocky 10 image no longer fails with a `cp` error


## [v1.1.0] - 2026-03-10

### Added

* `lf-debian.cfg`: Debian preseed configuration (Debian 11+) with LVM partitioning, matching the RHEL kickstart's minimal type
* `lf-ubuntu.cfg`: Ubuntu autoinstall configuration (Ubuntu 20.04+) with LVM partitioning, matching the RHEL kickstart's minimal type


## [v1.0.0] - 2026-03-10

**Highlights:** First release number after nearly four years of unversioned kickstart files, so an installation can be tied to a version of this repository. One generic `lf-rhel.cfg` replaces the per-distro kickstart files and installs RHEL 8+, Fedora 38+ and compatible, with BIOS and UEFI detection, a CIS hardening mode and a cloud variant. The root account has no password any more.

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

* Network options are no longer set explicitly ([#9](https://github.com/Linuxfabrik/kickstart/issues/9))
* RHEL 7 and CentOS 7 are no longer supported, the minimum is RHEL 8
* Root account no longer has a password (previously set to "password" with account locked)
* Sudoers entry uses user `linuxfabrik` instead of group `%linuxfabrik`
* systemd units are no longer started in the chroot environment

### Fixed

* `authorized_keys` gets its SELinux context
* `grub2-mkconfig` is EFI-aware
* `mkdir` without `-p` for `.ssh` directories caused failures when multiple SSH keys were deployed for the same user
* A failing `%post --nochroot` script aborts the install instead of running on
* The `--asprimary` flag is gone from the `/boot` partition, it is meaningless on a GPT disk label
* The mount point is detected before files are copied
* Sudoers file permissions are set to `0440`, instead of taking the default umask
* The post-install script no longer fails with a `sed` error
* The SSH user warning is gone ([#8](https://github.com/Linuxfabrik/kickstart/issues/8))


[Unreleased]: https://github.com/Linuxfabrik/kickstart/compare/v1.3.2...HEAD
[v1.3.2]: https://github.com/Linuxfabrik/kickstart/compare/v1.3.1...v1.3.2
[v1.3.1]: https://github.com/Linuxfabrik/kickstart/compare/v1.3.0...v1.3.1
[v1.3.0]: https://github.com/Linuxfabrik/kickstart/compare/v1.2.2...v1.3.0
[v1.2.2]: https://github.com/Linuxfabrik/kickstart/compare/v1.2.1...v1.2.2
[v1.2.1]: https://github.com/Linuxfabrik/kickstart/compare/v1.2.0...v1.2.1
[v1.2.0]: https://github.com/Linuxfabrik/kickstart/compare/v1.1.1...v1.2.0
[v1.1.1]: https://github.com/Linuxfabrik/kickstart/compare/v1.1.0...v1.1.1
[v1.1.0]: https://github.com/Linuxfabrik/kickstart/compare/v1.0.0...v1.1.0
[v1.0.0]: https://github.com/Linuxfabrik/kickstart/releases/tag/v1.0.0
