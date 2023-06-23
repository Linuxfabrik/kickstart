Generic Kickstart File
======================

This is a generic kickstart file for a minimal install of RHEL 7+ and compatible, on both BIOS as well as UEFI machines.


What are Kickstart Installations?
---------------------------------

Kickstart files contain answers to all questions normally asked by the installation program, such as what time zone you want the system to use, how the drives should be partitioned, or which packages should be installed. Providing a prepared Kickstart file when the installation begins therefore allows you to perform the installation automatically, without need for any intervention from the user. This is especially useful when deploying Red Hat Enterprise Linux (RHEL) on a large number of systems at once.

Kickstart files can be kept on a single server system and read by individual computers during the installation. This installation method can support the use of a single Kickstart file to install RHEL and compatible on multiple machines, making it ideal for network and system administrators. (`Source <https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/chap-kickstart-installations>`_)


What this Kickstart File do
---------------------------

* Supports RHEL 7+ and compatible.
* Works on legacy BIOS as well as UEFI.
* | Can be installed on a user-defined disk by specifying the kernel cmdline argument
  | ``lfdisk=$DISK`` (``$DISK`` defaults to ``vda`` if ommitted)
* The kickstart file is intended to provide a minimal installation.

The kickstart file can be used to install different types of minimal installs by setting the kernel cmdline argument ``lftype=``

.. csv-table::
    :header-rows: 1

    ``lftype=``, Install Type, Partitioning Scheme, User ``linuxfabrik``, User ``root``
    ``cis``, Minimal, CIS-recommended,          "Group: ``wheel``, Password: ``password``, SSH Keys: Linuxfabrik, Locked: no",  "Password: unset, SSH Keys: none, Locked: yes"
    ``cloud``, Minimal, One partition,          "Group: ``wheel``, Password: unset, SSH Keys: none, Locked: yes",               "Password: unset, SSH Keys: none, Locked: yes"
    ``cloud-cis``, Minimal, CIS-recommended,    "Group: ``wheel``, Password: unset, SSH Keys: none, Locked: yes",               "Password: unset, SSH Keys: none, Locked: yes"
    ``minimal``, Minimal, One partition,        "Group: ``wheel``, Password: ``password``, SSH Keys: Linuxfabrik, Locked: no",  "Password: unset, SSH Keys: Linuxfabrik, Locked: yes"


How to use
----------

This Kickstart file can be used by booting from an ISO file, then pressing ``ESC`` on the first screen and providing these cmdline arguments:

.. code-block:: text

    # Line breaks are only for better readability

    boot: linux inst.ks=https://raw.githubusercontent.com/Linuxfabrik/kickstart/main/lf-rhel.cfg \
        lftype=[minimal|cis|cloud|cloud-cis] \
        [lfdisk=$DISK] \
        [ip=[IPADDRESS]::GATEWAY:NETMASK:::none nameserver=NAMESERVER]

Note that the ip address field is an array, so the brackets are mandatory, and you can list multiple ip addresses there.


Modifying this Kickstart
------------------------

This kickstart uses an additional kickstart ``/tmp/dynamic.ks`` in a kickstart pre-script. At the beginning of the pre-script it determines which version of Python is available and then writes and executes a Python script ``/tmp/pre-script.py``. This Python script can be modified as follows:

* | ``lfkeys``
  | An array of SSH keys that will be installed for either the ``root`` or ``linuxfabrik`` user depending on ``lftype`` as documented above.
* | ``packages_<lftype>``
  | An array with package names for a ``<lftype>`` install.
* | ``part_<lftype>``
  | An array of kickstart ``logvol`` lines that define logical volumes for ``<lftype>`` as documented in Fedora Kickstart Syntax Reference: `logvol <https://docs.fedoraproject.org/en-US/fedora/f36/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-logvol>`_
* | ``users_<lftype>``
  | A string containing a ":" separated list in the form ``name="<username>":cmd="<kickstart command used for user creation>":keys="<array of ssh-keys to add>"`` (Fedora Kickstart Syntax Reference: `user <https://docs.fedoraproject.org/en-US/fedora/f36/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-user>`_, `rootpw <https://docs.fedoraproject.org/en-US/fedora/f36/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-rootpw>`_)
* | ``post_<lftype>``
  | A multiline string containing the postscript for ``<lftype>``. Will be executed by ``/bin/sh``.


Useful Kernel Cmdline Arguments
-------------------------------

RHEL:

* ``inst.loglevel=[debug|info]``
* ``inst.ks=[hd:<device>:<path>|[http,https,ftp]://<host>/<path>|nfs:[<options>:]<server>:/<path>`` (MANDATORY)
* ``inst.noverifyssl``
* ``inst.nosave=[<option1>,]<option2>`` (options: ``input_ks,output_ks,all_ks,logs,all``)
* ``inst.rescue``

Linuxfabrik:

* ``lfdisk=$DISK``: For example, use ``sda`` for installation (defaults to ``vda`` if omitted).


Known Limitations
-----------------

This Kickstart file does not work for RHEL 6- (and compatible).


Tested
------

Tested using these ISO-images:

* Fedora 37+
* RHEL 8+
* CentOS 7 BIOS


Kickstart Syntax References
---------------------------

* `Fedora <https://docs.fedoraproject.org/en-US/fedora/f34/install-guide/appendixes/Kickstart_Syntax_Reference/#sect-kickstart-commands-bootloader>`_
* `RHEL 7 <https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/sect-kickstart-syntax>`_
* `RHEL 8 <https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/performing_an_advanced_rhel_installation/kickstart-commands-and-options-reference_installing-rhel-as-an-experienced-user>`_
