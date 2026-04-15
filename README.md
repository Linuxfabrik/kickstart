<h1 align="center">
  Linuxfabrik Kickstart
</h1>
<p align="center">
  Kickstart (RHEL/Fedora), Preseed (Debian), Autoinstall (Ubuntu). BIOS/UEFI, auto disk detection, four install types (minimal, CIS, cloud, cloud-CIS), locked root, LVM, SELinux enforcing.
  <span>&#8226;</span>
  <b>made by <a href="https://linuxfabrik.ch/">Linuxfabrik</a></b>
</p>
<div align="center">

![GitHub Stars](https://img.shields.io/github/stars/linuxfabrik/kickstart)
![License](https://img.shields.io/github/license/linuxfabrik/kickstart)
![Version](https://img.shields.io/github/v/release/linuxfabrik/kickstart?sort=semver)
![Platforms](https://img.shields.io/badge/Platforms-RHEL%20%7C%20Fedora%20%7C%20Debian%20%7C%20Ubuntu-informational)
![GitHub Issues](https://img.shields.io/github/issues/linuxfabrik/kickstart)
[![GitHubSponsors](https://img.shields.io/github/sponsors/Linuxfabrik?label=GitHub%20Sponsors)](https://github.com/sponsors/Linuxfabrik)
[![PayPal](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=7AW3VVX62TR4A&source=url)

</div>

<br />

# Generic Kickstart / Autoinstall Files

This repository provides automated installation configurations for:

* **RHEL 8+, Fedora 38+ and compatible** (`lf-rhel.cfg`, Kickstart format)
* **Debian 11+** (`lf-debian.cfg`, Preseed format)
* **Ubuntu 20.04+** (`lf-ubuntu.cfg`, Autoinstall format)


## RHEL, Fedora and compatible


### What are Kickstart Installations?

Kickstart files contain answers to all questions normally asked by the installation program, such as what time zone you want the system to use, how the drives should be partitioned, or which packages should be installed. Providing a prepared kickstart file when the installation begins therefore allows you to perform the installation automatically, without need for any intervention from the user. This is especially useful when deploying Red Hat Enterprise Linux (RHEL) on a large number of systems at once.

Kickstart files can be kept on a single server system and read by individual computers during the installation. This installation method can support the use of a single kickstart file to install RHEL and compatible on multiple machines, making it ideal for network and system administrators. ([Source](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/automatically_installing_rhel/kickstart-script-file-format-reference_rhel-installer))


### How to use

This kickstart file can be used by booting from an ISO file, then either

* BIOS: pressing `ESC` on the first screen and providing these cmdline arguments (line breaks are only for better readability):

```text
boot: linux inst.ks=https://linuxfabrik.ch/ks
    [lftype=cis|cloud|cloud-cis|minimal]
    [lfdisk=$DISK]
    [ip=[IPADDRESS]::GATEWAY:NETMASK::INTERFACE:none]
    [nameserver=NAMESERVER]
    [...]
```

* UEFI: pressing `e` on the "Install ..." entry and appending the following to the `linuxefi` cmdline (line breaks are only for better readability), then booting by pressing `Ctrl-X`.

```text
linuxefi ... inst.ks=https://linuxfabrik.ch/ks
    [lftype=cis|cloud|cloud-cis|minimal]
    [lfdisk=$DISK]
    [ip=[IPADDRESS]::GATEWAY:NETMASK::INTERFACE:none]
    [nameserver=NAMESERVER]
    [...]
```

Note that `ip=` is an array (for providing multiple IP addresses), so the inner brackets are mandatory.

For convenience, we provide https://linuxfabrik.ch/ks as a shortened URL to https://raw.githubusercontent.com/Linuxfabrik/kickstart/main/lf-rhel.cfg.


### What this Kickstart File does

* Supports RHEL 8+, Fedora 38+ and compatible.
* Works on legacy BIOS as well as UEFI.
* The kickstart file is intended to provide a minimal installation, with `firewalld` disabled and SELinux in "Enforcing" mode.
* Can be installed on a user-defined disk by specifying the kernel cmdline argument `lfdisk=$DISK`. If unset, it tries to find the first block device, in the order `vda` > `sda` > `nvme0n1`, and fails otherwise.
* There are two users: `linuxfabrik` and `root`. The root account is always locked and has no password or SSH keys. Login with the `linuxfabrik` user, which is also part of the `wheel` group. `sudo` is configured to gain root.

The kickstart file can be used to install different types of minimal installs by setting the kernel cmdline argument `lftype=`:

| `lftype=` | Install Type | Partitioning Scheme | Password of User `linuxfabrik` | SSH Keys of User `linuxfabrik` |
|---|---|---|---|---|
| `cis` | Minimal | CIS, LVM | `password` | Those of Linuxfabrik |
| `cloud` | Minimal | Minimal, LVM | unset (inject via `cloud-init`) | none (inject via `cloud-init`) |
| `cloud-cis` | Minimal | CIS, LVM | unset (inject via `cloud-init`) | none (inject via `cloud-init`) |
| `minimal` (default) | Minimal | Minimal, LVM | `password` | Those of Linuxfabrik |


### Useful Kernel Cmdline Arguments

RHEL:

* `inst.loglevel=[debug|info]`: Note: Option is removed in RHEL9 and is always set to debug
* `inst.ks=[hd:<device>:<path>|[http,https,ftp]://<host>/<path>|nfs:[<options>:]<server>:/<path>` (MANDATORY)
* `inst.noverifyssl`: Prevents Anaconda from verifying the ssl certificate for all HTTPS connections ("insecure")
* `inst.nosave=[<option1>,]<option2>` (options: `input_ks,output_ks,all_ks,logs,all`)
* `inst.rescue`
* [more Kernel Cmdline Arguments](https://anaconda-installer.readthedocs.io/en/latest/boot-options.html)

Specific to this kickstart file:

* `lfdisk=$DISK`: User-defined disk for installing the OS. Default: unset (so tries to find the first block device, in the order `vda` > `sda` > `nvme0n1`, and fails otherwise).
* `lftype`: See table above. Defaults to `minimal`.


### Modifying this Kickstart

This kickstart includes an additional kickstart `/tmp/dynamic.ks`. This `/tmp/dynamic.ks` file is generated in a kickstart pre-script.
At the beginning of the pre-script the available Python version is determined, the Python script is written to `/tmp/pre-script.py` and executed.
The Python script then generates the `/tmp/dynamic.ks`.
This Python script can be modified as follows:

* `lfkeys`: An array of SSH public keys that will be authorized for the `linuxfabrik` user on `lftype=cis` and `lftype=minimal`. On `lftype=cloud` and `lftype=cloud-cis` no keys are deployed at install time, because `cloud-init` handles SSH key injection there. The `root` account never receives SSH keys regardless of `lftype`.
* `packages_<lftype>`: An array with package names for a `<lftype>` install.
* `part_<lftype>`: An array of kickstart `logvol` lines that define logical volumes for `<lftype>` as documented in Fedora Kickstart Syntax Reference: [logvol](https://docs.fedoraproject.org/en-US/fedora/f36/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-logvol)
* `users_<lftype>`: A Python list of dicts, one entry per user, with the keys `name` (the username), `cmd` (the kickstart command used to create the user, e.g. `user --name=linuxfabrik --groups=wheel --password=password --plaintext` or `rootpw --lock`) and `keys` (a list of SSH public keys to authorize for that user). See Fedora Kickstart Syntax Reference: [user](https://docs.fedoraproject.org/en-US/fedora/f36/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-user), [rootpw](https://docs.fedoraproject.org/en-US/fedora/f36/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-rootpw).
* `post_<lftype>`: A multiline string containing the chrooted post-install script for `<lftype>`. Anaconda executes it with `/bin/sh`, which on every supported target (RHEL/Rocky 8/9/10, Fedora 38+) is bash invoked in sh-compat mode, so bash features like globs inside `[ -f "$var" ]` loops work as expected.


### Known Limitations

This kickstart file does not work for RHEL 7- (and compatible).


### Tests

Test combinations:

* OS: rocky8, rocky9, rocky10
* Firmware: BIOS, UEFI
* Disk: vda, sda
* `lftype`: `cis`, `cloud`, `cloud-cis`, `minimal`

What to test within the VM:

* Console login using "root": Should not work (account is locked, no password).
* Console login using "linuxfabrik" + "password": Should work on non-cloud. On cloud, password depends on cloud-init.
* `ip a`: Should get an IP.
* `ssh root@vm`: Should not work.
* `ssh linuxfabrik@vm`: Should work on non-cloud. On cloud, it depends on cloud-init.
* `sudo su -`: Should work.
* `cat /etc/shadow`: Should show that root's password is locked.
* `df -hT`: Three partitions (`/`, `/backup`, `/boot`) on non-cis, eight partitions on cis.
* `lvs`: Should work.
* `sudo dnf -y install nano`: Should work.
* `systemctl status cloud-init`: Not found on non-cloud, should work on cloud.
* `systemctl status firewalld`: Should be inactive on non-cloud. On cloud, firewalld is removed.
* `ll /root`: Should list `dynamic.ks` (the rendered kickstart fragment Linuxfabrik applied, always present). Anaconda additionally writes `original-ks.cfg` on every install and, on most targets, `anaconda-ks.cfg` — both are written after all `%post` scripts, so they are not cleaned up by this kickstart. On `lftype=cis` and `lftype=minimal`, `70-install-ssh-keys.ks` is also archived (the SSH key deployment fragment); on `lftype=cloud` and `lftype=cloud-cis`, no SSH key fragment is generated because `cloud-init` handles keys.
* `grep 'Linuxfabrik Kickstart version' /root/dynamic.ks`: Should show the `YYYYMMDDNN` build stamp of the `lf-rhel.cfg` variant that was applied during installation. The same stamp is also written to `/var/log/anaconda/anaconda.log` and to the `%post` `ks-script-*.log` under `/var/log/anaconda/`, so the origin of any installed host is traceable to a specific `lf-rhel.cfg` build.


### Log Files

During and after an Anaconda install, the following log files are the fastest path to diagnosing problems.

**During the install (installer environment, switch to a shell with `Ctrl+Alt+F2` or `Ctrl+Alt+F3`):**

* `/tmp/dynamic.ks`: The dynamically generated kickstart fragment that `%pre` produced. If Anaconda reports a kickstart syntax error in an included file, `cat` this file to see what was actually rendered.
* `/tmp/kickstart.install.pre.log`: Output of the `%pre` script, including the `LF_KICKSTART_VERSION` stamp, the `lftype`/`lfdisk` detection, and the Python helper's progress messages. First place to look if the install aborts before the package selection.
* `/tmp/ks-script-*.log`: Per-script output of each `%post` block and each post-script under `/usr/share/anaconda/post-scripts/`. Same files that will later be copied to `/var/log/anaconda/`.
* `/tmp/pre-script.py`: The Python helper itself. Useful if `%pre` crashes on a specific Python line.

**After a successful install (on the installed system, under `/var/log/anaconda/`):**

* `anaconda.log`: Main Anaconda narrative for the whole install. Start here when a post-install symptom shows up after first boot.
* `dbus.log`: Anaconda DBus module communication. Only needed for deeper Anaconda-internal issues (for example when the `boss` or `storage` module misbehaves, as with the `original-ks.cfg` write ordering).
* `journal.log`: Snapshot of the installer's journal at the end of install. Covers kernel, dracut and systemd messages; useful when the installer itself had boot problems.
* `ks-script-*.log`: Output of each `%post` block. `grep -l 'Linuxfabrik Kickstart version' /var/log/anaconda/ks-script-*.log` identifies the script that ran our `post_cloud`/`post_cis` block; that file contains the yum/dnf transaction output, the systemd-in-chroot warnings, the `grub2-mkconfig` result, and any messages from our cleanup/tweaks.
* `packaging.log`: DNF/yum transaction detail during the install. Goes deeper than `ks-script-*.log` when a package install or removal looks wrong.
* `program.log`: External programs Anaconda invoked (e.g. `grub2-mkconfig`, `mkfs.xfs`). Complements the other logs when the failing step was an external binary.
* `storage.log`: Partitioning, LVM creation, filesystem formatting. First stop for "disk not found", `ignoredisk` or LVM-related failures.

**On the installed system, outside `/var/log/anaconda/`:**

* `/root/anaconda-ks.cfg`: Anaconda's generated copy of the effective kickstart. Not present on targets where `can_save_output_kickstart=False` (including Rocky 10 cloud).
* `/root/dynamic.ks`: The kickstart fragment Linuxfabrik archived on the installed system. The first comment line reads `# Linuxfabrik Kickstart version: YYYYMMDDNN`, which attributes the host to a specific `lf-rhel.cfg` build.
* `/root/original-ks.cfg`: Anaconda's verbatim copy of the `lf-rhel.cfg` served from the `inst.ks=` URL.

**For `cloud-init` problems on `lftype=cloud` and `lftype=cloud-cis` (after first boot):**

* `/var/log/cloud-init-output.log`: Stdout/stderr of any user-data scripts that cloud-init executed.
* `/var/log/cloud-init.log`: Full cloud-init log with DEBUG-level entries. First stop for "my SSH keys aren't there" or "hostname not set".


### Troubleshooting

* `page_poison=1` kernel cmdline option installed by bootloader cmd can leave the system unbootable due to a buggy UEFI firmware. This was observed with TianoCore firmware on qemu. Remove this option to boot. See https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/8.7_release_notes/known-issues.
* Fedora 38: We observed problems booting into the installer. Try `inst.neednet=1 rd.debug` to get to the installer.
* Last resort: If this Kickstart file doesn't work, copy it to a webserver and modify it to suit your needs.


### References

* [Fedora Kickstart Syntax](https://docs.fedoraproject.org/en-US/fedora/f34/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-bootloader)
* [RHEL 8 Kickstart Syntax](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/automatically_installing_rhel/kickstart-commands-and-options-reference_rhel-installer)
* [RHEL 9 Kickstart Syntax](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/automatically_installing_rhel/kickstart-commands-and-options-reference_rhel-installer)
* [Rocky 8 Generic Cloud LVM Kickstart](https://git.resf.org/sig_core/kickstarts/src/branch/r8/Rocky-8-GenericCloud-LVM.ks)
* [OpenStack Image Requirements](https://docs.openstack.org/image-guide/openstack-images.html)


## Debian and compatible


### What is Preseed?

Preseed provides a way to set answers to questions asked during the Debian installation process, without having to manually enter the answers while the installation is running. This makes it possible to fully automate Debian installations. ([Source](https://wiki.debian.org/DebianInstaller/Preseed))

Note: Preseed only works with the Debian Installer (d-i). It does **not** work with Ubuntu 20.04+ (which uses autoinstall/subiquity).


### How to use

Boot from a Debian netinst or DVD ISO, press `ESC` at the boot menu, then:

```text
boot: auto url=http://<server>/<path>/lf-debian.cfg
```


### What this Preseed File does

* Supports Debian 11+ (Bullseye and newer).
* Works on legacy BIOS as well as UEFI (GPT partitioning).
* Provides a minimal server installation with LVM partitioning (`/`, `/backup`, `/boot`, swap).
* Installs on the first available disk (`/dev/vda`, `/dev/sda` or `/dev/nvme0n1`).
* There are two users: `linuxfabrik` and `root`. The root account is not created (disabled). Login with the `linuxfabrik` user, which has password-less sudo. The default password is `password` (change after first login).
* SSH server is installed and Linuxfabrik SSH keys are deployed.
* The system shuts down after installation.


### Tests

Test combinations:

* OS: Debian 11, Debian 12
* Firmware: BIOS, UEFI
* Disk: vda, sda

What to test within the VM:

* Console login using "root": Should not work (no root account).
* Console login using "linuxfabrik" + "password": Should work.
* `ip a`: Should get an IP.
* `ssh root@vm`: Should not work.
* `ssh linuxfabrik@vm`: Should work.
* `sudo su -`: Should work.
* `df -hT`: Three partitions (`/`, `/backup`, `/boot`).
* `lvs`: Should work.
* `sudo apt install -y nano`: Should work.


### Log Files

During and after a Debian Installer (d-i) preseed run, the following log files are the fastest path to diagnosing problems.

**During the install (switch to a shell with `Ctrl+Alt+F2`; live installer syslog tails on `Ctrl+Alt+F4`):**

* `/var/log/partman`: Partition manager log. First stop for disk, LVM or filesystem errors during install.
* `/var/log/syslog`: Main installer syslog. First stop for any preseed or d-i failure.

**After a successful install (on the installed system, under `/var/log/installer/`):**

* `cdebconf/questions.dat`: Preseed questions and the answers that were given (by our preseed file or interactively). Useful to verify that a preseed directive actually reached debconf.
* `cdebconf/templates.dat`: The debconf template catalogue, in case a preseed key was silently ignored because the template changed.
* `hardware-summary`: Detected hardware and driver decisions.
* `lsb-release`, `media-info`: Base distribution and installation media metadata.
* `partman`: Partition manager log, persisted from the install.
* `status`: `apt`/`dpkg` status of every package installed during the install phase.
* `syslog`: Main installer narrative for the whole install.


### References

* [Debian Preseed Documentation](https://www.debian.org/releases/stable/amd64/apb.en.html)
* [Debian Preseed Example](https://www.debian.org/releases/stable/example-preseed.txt)


## Ubuntu and compatible


### What is Autoinstall?

Ubuntu's autoinstall provides a way to perform unattended installations of Ubuntu Server. It uses a YAML configuration file to answer all installation questions automatically, similar to Kickstart on RHEL. Autoinstall is available on Ubuntu 20.04+ and uses the subiquity installer. ([Source](https://canonical-subiquity.readthedocs-hosted.com/en/latest/intro-to-autoinstall.html))

Note: Autoinstall does **not** work with Debian, which uses a different system called preseed.


### How to use

Serve `lf-ubuntu.cfg` as `user-data` alongside an empty `meta-data` file via HTTP, then boot from an Ubuntu Server ISO and append to the kernel cmdline:

```text
autoinstall ds=nocloud-net;s=http://<server>/<path>/
```


### What this Autoinstall File does

* Supports Ubuntu 20.04+ (subiquity installer).
* Targets UEFI systems (see comments in the file for BIOS adaptation).
* Provides a minimal server installation with LVM partitioning (`/`, `/backup`, `//boot`, swap).
* There are two users: `linuxfabrik` and `root`. The root account is locked with no password. Login with the `linuxfabrik` user, which has password-less sudo. The default password is `password` (change after first login).
* SSH server is installed and Linuxfabrik SSH keys are deployed.
* `ufw` and `kdump` are disabled.
* The system shuts down after installation.


### Tests

Test combinations:

* OS: Ubuntu 22.04, Ubuntu 24.04
* Firmware: UEFI

What to test within the VM:

* Console login using "root": Should not work (account is locked).
* Console login using "linuxfabrik" + "password": Should work.
* `ip a`: Should get an IP.
* `ssh root@vm`: Should not work.
* `ssh linuxfabrik@vm`: Should work.
* `sudo su -`: Should work.
* `cat /etc/shadow`: Should show that root's password is locked.
* `df -hT`: Three partitions (`/`, `/backup`, `/boot`).
* `lvs`: Should work.
* `sudo apt install -y nano`: Should work.


### Log Files

During and after a subiquity autoinstall run, the following log files are the fastest path to diagnosing problems.

**During the install (switch to a shell with `Ctrl+Alt+F2`):**

* `/var/log/installer/autoinstall-user-data`: The autoinstall YAML subiquity actually consumed. Useful to verify that the `user-data` file served over HTTP is what subiquity actually parsed.
* `/var/log/installer/subiquity-server-debug.log`: Subiquity server-side debug log, streams live during install. First stop for any autoinstall/subiquity failure.

**After a successful install (on the installed system, under `/var/log/installer/`):**

* `autoinstall-user-data`: The autoinstall YAML subiquity consumed, persisted.
* `cloud-init-output.log`, `cloud-init.log`: Cloud-init logs from inside the installer — subiquity uses cloud-init internally to fetch `user-data` and apply some steps.
* `curtin-install-cfg.yaml`, `curtin-install.log`: Curtin (the actual installer under subiquity) effective config and run log. First stop for partitioning, LVM or grub issues.
* `installer-journal.txt`: Journal snapshot from the installer environment.
* `subiquity-client-debug.log`, `subiquity-server-debug.log`: Subiquity's full debug logs.


### References

* [Ubuntu Autoinstall Reference](https://canonical-subiquity.readthedocs-hosted.com/en/latest/reference/autoinstall-reference.html)
* [Ubuntu Autoinstall Quick Start](https://canonical-subiquity.readthedocs-hosted.com/en/latest/howto/autoinstall-quickstart.html)
