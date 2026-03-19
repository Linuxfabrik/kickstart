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

* `lfkeys`: An array of SSH keys that will be installed for either the `root` or `linuxfabrik` user depending on `lftype` as documented above.
* `packages_<lftype>`: An array with package names for a `<lftype>` install.
* `part_<lftype>`: An array of kickstart `logvol` lines that define logical volumes for `<lftype>` as documented in Fedora Kickstart Syntax Reference: [logvol](https://docs.fedoraproject.org/en-US/fedora/f36/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-logvol)
* `users_<lftype>`: A string containing a ":" separated list in the form `name="<username>":cmd="<kickstart command used for user creation>":keys="<array of ssh-keys to add>"` (Fedora Kickstart Syntax Reference: [user](https://docs.fedoraproject.org/en-US/fedora/f36/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-user), [rootpw](https://docs.fedoraproject.org/en-US/fedora/f36/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-rootpw))
* `post_<lftype>`: A multiline string containing the postscript for `<lftype>`. Will be executed by `/bin/sh`.


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
* `ll /root`: Should list at least two Anaconda files.


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


### References

* [Ubuntu Autoinstall Reference](https://canonical-subiquity.readthedocs-hosted.com/en/latest/reference/autoinstall-reference.html)
* [Ubuntu Autoinstall Quick Start](https://canonical-subiquity.readthedocs-hosted.com/en/latest/howto/autoinstall-quickstart.html)
