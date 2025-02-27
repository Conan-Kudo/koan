Koan
****

koan - kickstart over a network, client side helper for cobbler

SYNOPSIS
########

.. code-block:: shell

    koan --server=hostname [--list=type] [--virt|--replace-self|--display] [--profile=name] [--system=name] [--image=name] [--add-reinstall-entry] [--virt-name=name] [--virt-path=path] [--virt-type=type] [--nogfx] [--static-interface=name] [--kexec]

DESCRIPTION
###########

Koan stands for "kickstart-over-a-network" and is a client-side helper program for use with Cobbler.  koan allows for
both network provisioning of new virtualized guests (Xen, QEMU/KVM, VMware) and re-installation of an existing system.

When invoked, koan requests install information from a remote cobbler boot server, it then kicks off installations based
on what is retrieved from cobbler and fed in on the koan command line. The examples below show the various use cases.

LISTING REMOTE COBBLER OBJECTS
##########

To browse remote objects on a cobbler server and see what you can install using koan, run one of the following commands:

.. code-block:: shell

    koan --server=cobbler.example.org --list=profiles
    koan --server=cobbler.example.org --list=systems
    koan --server=cobbler.example.org --list=images

LEARNING MORE ABOUT REMOTE COBBLER OBJECTS
##########################################

To learn more about what you are about to install, run one of the following commands:

.. code-block:: shell

    koan --server=cobbler.example.org --display --profile=name
    koan --server=cobbler.example.org --display --system=name
    koan --server=cobbler.example.org --display --image=name

REINSTALLING EXISTING SYSTEMS
#############################

Using --replace-self will reinstall the existing system the next time you reboot.

koan --server=cobbler.example.org --replace-self --profile=name

koan --server=cobbler.example.org --replace-self --system=name

Additionally, adding the flag --add-reinstall-entry will make it add the entry to grub for reinstallation
but will not make it automatically pick that option on the next boot.

Also the flag --kexec can be appended, which will launch the installer without needing to reboot.  Not
all kernels support this option.

INSTALLING VIRTUALIZED SYSTEMS
##############################

Using ``--virt`` will install virtual machines as defined by Cobbler. There are various overrides you can use if not
everything in cobbler is defined as you like it.

.. code-block:: shell

    koan --server=cobbler.example.org --virt --profile=name
    koan --server=cobbler.example.org --virt --system=name
    koan --server=cobbler.example.org --virt --image=name

Some of the overrides that can be used with --virt are:

+-------------------+---------------------------------------+---------------------------+
| Flag              | Explanation                           | Example                   |
+===================+=======================================+===========================+
| ``--virt-name``   | name of virtual machine to create     | testmachine               |
+-------------------+---------------------------------------+---------------------------+
| ``--virt-type``   | forces usage of qemu/xen/vmware       | qemu                      |
+-------------------+---------------------------------------+---------------------------+
| ``--virt-bridge`` | name of bridge device                 | virbr0                    |
+-------------------+---------------------------------------+---------------------------+
| ``--virt-path``   | overwrite this disk partition         | `/dev/sda4`               |
+-------------------+---------------------------------------+---------------------------+
| ``--virt-path``   | use this directory                    | `/opt/myimages`           |
+-------------------+---------------------------------------+---------------------------+
| ``--virt-path``   | use this existing LVM volume          | `VolGroup00`              |
+-------------------+---------------------------------------+---------------------------+
| ``--nogfx``       | do not use VNC graphics (Xen only)    | (does not take options)   |
+-------------------+---------------------------------------+---------------------------+


Nearly all of these variables can also be defined and centrally managed by the Cobbler server.

If installing virtual machines in environments without DHCP, use of ``--system`` instead of ``--profile`` is required.
Additionally use ``--static-interface=eth0`` to supply which interface to use to supply network information. The
installer will boot from this virtual interface. Leaving off ``--static-interface`` will result in an unsuccessful
network installation.

CONFIGURATION MANAGEMENT
########################

Using ``--update-config`` will update a system configuration as defined by Cobbler.

.. code-block:: shell

    koan --server=cobbler.example.org --update-config

Additionally, adding the flag ``--summary`` will print configuration run stats.

Koan passes in the system's FQDN in the background during the configuration request. Cobbler will match this FQDN to a
configured system defined by Cobbler.

The FQDN (Fully Qualified Domain Name) maps to the system's hostname field.

ENVIRONMENT VARIABLES
#####################

Koan respects the COBBLER_SERVER variable to specify the cobbler server to use. This is a convenient way to avoid using
the ``--server`` option for each command. This variable is set automatically on systems installed via cobbler, assuming
standard kickstart templates are used. If you need to change this on an installed system, edit
``/etc/profile.d/cobbler.{csh,sh}``.

ADDITIONAL
##########

Reading the koan manpage, www.cobbler.github.io or this readthedocs project is highly recommended.

AUTHOR
######

Michael DeHaan <michael.dehaan AT gmail>

Revised by: Enno Gotthold <matrixfueller@gmail.com>
