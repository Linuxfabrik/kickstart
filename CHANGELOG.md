# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).


## [Unreleased]

tbd


## [1.2.1] - 2026-04-15

### Added

* Add a running `LF_KICKSTART_VERSION` build stamp (format `YYYYMMDDNN`) shared across `lf-rhel.cfg`, `lf-debian.cfg` and `lf-ubuntu.cfg`. The stamp is logged at install time (to `/tmp/kickstart.install.pre.log` and `/var/log/anaconda/ks-script-*.log` on RHEL, to `/var/log/installer/syslog` on Debian, and to `/var/log/installer/installer-journal.txt` on Ubuntu) and written to `/root/lf-install-version` on every installed host — a single distro-independent fleet marker that lets admins trace any installed host back to the exact installer build that produced it. On RHEL the stamp is additionally embedded as a comment at the top of the rendered `dynamic.ks` (archived as `/root/dynamic.ks`). CONTRIBUTING documents when and how to bump the stamp; the README documents where to read it on an installed system

### Fixed

* `lf-rhel.cfg`: Fix `grub2-mkconfig` being silently skipped on every target version. The `%pre` script was built via an unquoted heredoc (`cat << EOT`), which let bash eat every `"$var"` and `$(cmd)` reference embedded in the generated shell code before the Python helper was even written. As a result the grub finder always ran with empty variables and printed "Could not find a grub.cfg in /boot" — hiding the fact that `/etc/default/grub` edits had not been regenerated since the first release. The heredoc is now quoted (`cat << 'EOT'`), the finder targets `/boot/grub2/grub.cfg` directly (which is the right target on RHEL/Rocky 8/9/10 for both BIOS and UEFI — on 9+ UEFI the `/boot/efi/EFI/<vendor>/grub.cfg` is only a thin wrapper that `configfile`s `/boot/grub2/grub.cfg`), and the console-ordering change in `/etc/default/grub` is actually reflected in the generated `grub.cfg`
* `lf-rhel.cfg`: Fix post-install failure on RHEL 10 / Rocky 10 cloud installs. The previous `%post --nochroot` block copied `dynamic.ks` and `70-install-ssh-keys.ks` into `/root` of the installed system for auditing and kept breaking across Anaconda versions (first `/proc/mounts` parsing, then mount-point layout). The archival is now performed by a chrooted `%post` hook generated in `%pre` under `/usr/share/anaconda/post-scripts/`, with the content embedded via base64-decoded heredoc. The same kickstart now works on RHEL/Rocky 8, 9 and 10
* `lf-rhel.cfg`: Replace `fixfiles -R -a restore` with `restorecon -RF -e /boot/efi /` for the SELinux relabel step. The EFI system partition is vfat and cannot carry SELinux contexts, so previously every install produced a stream of harmless but alarming-looking `Could not set context for /boot/efi/EFI/...: Operation not supported` messages in the `%post` log. On RHEL/Rocky 10 the `fixfiles` tool does not accept an exclude flag (`illegal option -- e`), but `restorecon` has had `-e <path>` for years, so switching to `restorecon` both silences the noise and keeps the coverage for a fresh install. The functional `|| true` safety net is still in place
* `lf-rhel.cfg`: Fix `sed: can't read /etc/systemd/logind.conf` on RHEL/Rocky 10 cloud installs. Recent systemd versions no longer ship `logind.conf` under `/etc/`; the `NAutoVTs=0` override is now applied via a drop-in at `/etc/systemd/logind.conf.d/10-lf-no-auto-vts.conf`, which works on all supported systemd versions
* `lf-rhel.cfg`: The "Only touch" comment in the archived `dynamic.ks` now renders the actual target disk path (e.g. `# Only touch vda.`) instead of a literal `$lfdisk`. The Python code was using a shell-style variable reference inside a plain string where a Python format substitution was needed

### Changed

* README: Add a "Log Files" section to each of the RHEL, Debian and Ubuntu chapters that lists the log files most relevant for diagnosing install-time and post-install problems, grouped by "during install" and "after install", so an operator can go straight to the right log for a given symptom
* README: Correct and sharpen the "Modifying this Kickstart" and "Tests" sections. The `lfkeys`, `users_<lftype>` and `post_<lftype>` descriptions now match the actual Python data structures and interpreter behavior, and the `ll /root` test now reflects that Anaconda's own kickstart copies (`anaconda-ks.cfg`, `original-ks.cfg`) are present on every install because they are written after all `%post` scripts and cannot be removed from within the kickstart

### Removed

* `lf-rhel.cfg`: Drop the dead `yum erase authconfig` call from the cloud post-install script. `authconfig` has not existed since RHEL 8 (replaced by `authselect`), so the call was a no-op on every supported target
* `lf-rhel.cfg`: Drop the dead `rm -f /root/anaconda-ks.cfg` and `rm -f /root/original-ks.cfg` calls from the cloud post-install script. Both files are written by Anaconda's boss module (`_writeKS_via_boss` and `CopyLogsTask`) **after** all `%post` scripts have run, so the removals ran against non-existent files and had no effect. A comment at the same spot now documents the ordering so future maintainers do not reintroduce this


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


[Unreleased]: https://github.com/Linuxfabrik/kickstart/compare/v1.2.1...HEAD
[1.2.1]: https://github.com/Linuxfabrik/kickstart/compare/v1.2.0...v1.2.1
[1.2.0]: https://github.com/Linuxfabrik/kickstart/compare/v1.1.1...v1.2.0
[1.1.1]: https://github.com/Linuxfabrik/kickstart/compare/v1.1.0...v1.1.1
[1.1.0]: https://github.com/Linuxfabrik/kickstart/compare/v1.0.0...v1.1.0
