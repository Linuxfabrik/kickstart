# Collection of Kickstart Files

* The kickstart files aim to provide a minimal installation.
* The `*-cis*.cfg` can be used to create VMs that follow the [CIS standards](https://www.cisecurity.org/) for partitioning. These VMs can later be hardened, for example using [linuxfabrik.lfops.stig](https://github.com/Linuxfabrik/lfops/tree/main/roles/stig) role.
* The `*-cloud*.cfg` files are intended for creating base templates for the use with cloud providers (such as OpenStack).
* These files can be used by booting using corresponding ISO file, then pressing ESC on the first screen, and setting `boot: linux inst.ks=https://raw.githubusercontent.com/Linuxfabrik/kickstart/main/$FILENAME`.


## Known Limitations

* Currently, all kickstart files are designed for the installation of virtual machines using VirtIO, and therefore assume the primary disk to be `vda`.


## Kickstart Syntax References
* [Fedora](https://docs.fedoraproject.org/en-US/fedora/f34/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-bootloader)
* [RHEL 7](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/sect-kickstart-syntax)
* [RHEL 8](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/performing_an_advanced_rhel_installation/kickstart-commands-and-options-reference_installing-rhel-as-an-experienced-user)
