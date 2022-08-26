# Collection of Kickstart Files

## What are Kickstart Installations?

Kickstart files contain answers to all questions normally asked by the installation program, such as what time zone you want the system to use, how the drives should be partitioned, or which packages should be installed. Providing a prepared Kickstart file when the installation begins therefore allows you to perform the installation automatically, without need for any intervention from the user. This is especially useful when deploying Red Hat Enterprise Linux (RHEL) on a large number of systems at once.

Kickstart files can be kept on a single server system and read by individual computers during the installation. This installation method can support the use of a single Kickstart file to install RHEL and compatible on multiple machines, making it ideal for network and system administrators. ([Source](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/chap-kickstart-installations))


## What these Kickstart Files do

* The kickstart files aim to provide a minimal installation.
* The `*-cis*.cfg` can be used to create VMs that follow the [CIS standards](https://www.cisecurity.org/) for partitioning. These VMs can later be hardened, for example using [linuxfabrik.lfops.stig](https://github.com/Linuxfabrik/lfops/tree/main/roles/stig) role.
* The `*-cloud*.cfg` files are intended for creating base templates for the use with cloud providers (such as OpenStack).


## How to use

These Kickstart files can be used by booting using an ISO file, then pressing ESC on the first screen and using `boot: linux inst.ks=https://raw.githubusercontent.com/Linuxfabrik/kickstart/main/$FILENAME`.

If you need to configure a static IP address to be used while installation, you can do so by pressing TAB on the first screen and add the following to the existing line `inst.ks=https://raw.githubusercontent.com/Linuxfabrik/kickstart/main/$FILENAME ip=[IPADDRESS]::GATEWAY:NETMASK:::none nameserver=NAMESERVER`

(The ip address field is an array, so brackets are mendatory. You could use multiple ip addresses here).

## Known Limitations

* Currently, all kickstart files are designed for the installation of virtual machines using VirtIO, and therefore assume the primary disk to be `vda`.


## Kickstart Syntax References
* [Fedora](https://docs.fedoraproject.org/en-US/fedora/f34/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-bootloader)
* [RHEL 7](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/sect-kickstart-syntax)
* [RHEL 8](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/performing_an_advanced_rhel_installation/kickstart-commands-and-options-reference_installing-rhel-as-an-experienced-user)
