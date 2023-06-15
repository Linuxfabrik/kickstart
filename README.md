# Generic kickstart file for a minimal install of RHEL version 7,8,9 on both BIOS as well as UEFI machines

## What are Kickstart Installations?

Kickstart files contain answers to all questions normally asked by the installation program, such as what time zone you want the system to use, how the drives should be partitioned, or which packages should be installed. Providing a prepared Kickstart file when the installation begins therefore allows you to perform the installation automatically, without need for any intervention from the user. This is especially useful when deploying Red Hat Enterprise Linux (RHEL) on a large number of systems at once.

Kickstart files can be kept on a single server system and read by individual computers during the installation. This installation method can support the use of a single Kickstart file to install RHEL and compatible on multiple machines, making it ideal for network and system administrators. ([Source](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/chap-kickstart-installations))


## What these Kickstart File does

* The kickstart file aims to provide a minimal installation.
* Supports RHEL 7,8,9 and derives
* Works on legacy BIOS as well as UEFI
* Can install to a user defined disk by specifying the kernel cmdline argument `lfdisk=<disk>` (`<disk>` defaults to "vda" when unset)
* The kickstart file can be used to install different types of minimal installs by setting the kernel cmdline argument `lftype=` to:
    * `minimal`
        * Minimal install 
        * `root` user: locked with linuxfabrik ssh-keys installed. password=`password`
        * `linuxfabrik` user: has password-less sudo rights no ssh-keys installed. password=`password`
    * `cis`
        * Minmal install with cis recommended partitioning scheme (https://www.cisecurity.org)
        * `root` user: locked. password=`password`
        * `linuxfabrik` user: with linuxfabrik ssh-keys installed. password=`password`
    * `cloud`
        * Minimal install for installs at cloud providers
        * `root` user: locked.
        * `linuxfabrik` user: locked
    * `cloud-cis`
        * Minimal install for installs at cloud providers with cis recommended partitioning scheme (https://cisecurity.org)
        * `root` user: locked.
        * `linuxfabrik` user: locked


## How to use

These Kickstart files can be used by booting using an ISO file, then pressing ESC on the first screen and using `boot: linux inst.ks=https://raw.githubusercontent.com/Linuxfabrik/kickstart/main/lf-rhel.cfg lftype=[minimal|cis|cloud|cloud-cis] [lfdisk=<disk>]`.

If you need to configure a static IP address to be used while installation, you can do so by pressing TAB on the first screen and add the following to the existing line `inst.ks=https://raw.githubusercontent.com/Linuxfabrik/kickstart/main/lf-rhel.cfg lftype=[minimal|cis|cloud|cloud-cis] ip=[IPADDRESS]::GATEWAY:NETMASK:::none nameserver=NAMESERVER`

Note that the ip address field is an array, so the brackets are mandatory, and you could list multiple ip addresses there.

## Modifying the kickstart

The kickstart works by generating an additional kickstart (`/tmp/dynamic.ks`) in a kickstart pre-script. At the beginning of the pre-script it determines which python is available for use and then writes a python script (`/tmp/pre-script.py`) that it then executes. The definitions in this python script can be modified as you see fit. Below a description of the definitions:
* `lfkeys`
    * an array of ssh-keys that get installed for either the `root` or `linuxfabrik` user depending on `lftype` as documented above.
* `packages_<type>`
    * An array with package names for a `<type>` install.
* `part_<type>`
    * An array of kickstart `logvol` lines that define logical volumes for `<type>` as documented at: [Fedora Kickstart Syntax Reference:logvol](https://docs.fedoraproject.org/en-US/fedora/f36/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-logvol)
* `users_<type>`
    * A string containing a ":" seperated list in the form `name="<username>":cmd="<kickstart command used for user creation>":keys="<array of ssh-keys to add>"` (Fedora Kickstart Syntax Reference: [user](https://docs.fedoraproject.org/en-US/fedora/f36/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-user) , [rootpw](https://docs.fedoraproject.org/en-US/fedora/f36/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-rootpw))
* `post_<type>`
    * A multi-line string containing the post script for `<type>`. Gets executed using `/bin/sh`

### Additional usefull kernel cmdline arguments

* RHEL: `inst.loglevel=[debug|info]`
* RHEL: `inst.ks=[hd:<device>:<path>|[http,https,ftp]://<host>/<path>|nfs:[<options>:]<server>:/<path>` (MANDATORY)
* RHEL: `inst.noverifyssl`
* RHEL: `inst.nosave=[<option1>,]<option2>` (options: `input_ks,output_ks,all_ks,logs,all`)
* RHEL: `inst.rescue`
* LF: `lfdisk=<disk>`: use only sda for installation. (defaults to `vda` when unset.)

## Known Limitations

* The kickstart file does not work for RHEL 2,3,4,5,6 or derived distributions.

## Tested

* fedora38
* fedora37
* rhel9.2
* rhel9.0
* rhel8.8
* rhel8.7
* rocky8.7
* centos7 2009(centos7 only works on legacy)

## Kickstart Syntax References
* [Fedora](https://docs.fedoraproject.org/en-US/fedora/f34/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-bootloader)
* [RHEL 7](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/sect-kickstart-syntax)
* [RHEL 8](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/performing_an_advanced_rhel_installation/kickstart-commands-and-options-reference_installing-rhel-as-an-experienced-user)
