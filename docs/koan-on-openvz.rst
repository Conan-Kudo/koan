****************************************
Support for OpenVZ containers in Cobbler
****************************************

THIS FUNCTIONS CONSIDERED AS ALPHA STAGE FOR TESTING AND LIMITED USAGE!
USAGE IN PRODUCTION CAN BE DANGEROUS! YOU WARNED!
 
Cobbler is amazing tool for deploying barebones and virtual machines and I think it is suitable for
deploying OpenVZ containers too.

Current support for OpenVZ is rather basic, but I think this functionality can reach level we have now for KVM.

How to use it?
##############

Because OpenVZ container is in nature chrooted environment we use cobbler+koan to create this on OpenVZ-enabled node.
For cobbler and koan in case of OpenVZ all operations is similar - we should define distros, automated installation
files, profiles, systems and so on with some additions.
Now we do all operations only for RHEL/CentOS6. It may be suitable for recent Fedoras, but we do nothing for other 
distributions.

How it works?
#############

All options keeps on cobbler side as for other VMs.
Besides of common options you can use openvz-specific ones by defining them as ``vz_`` prefixed, low-cased variables
from this list: KMEMSIZE, LOCKEDPAGES, PRIVVMPAGES, SHMPAGES, NUMPROC, VMGUARPAGES, OOMGUARPAGES, NUMTCPSOCK,
NUMFLOCK, NUMPTY, NUMSIGINFO, TCPSNDBUF, TCPRCVBUF, OTHERSOCKBUF, DGRAMRCVBUF, NUMOTHERSOCK, DCACHESIZE, NUMFILE,
AVNUMPROC, NUMIPTENT, DISKINODES, QUOTATIME, VE_ROOT, VE_PRIVATE, SWAPPAGES, ONBOOT (See ctid.conf(5) for meaning 
of this parameters).
Because cobbler does not have a place to keep CTID you MUST use it in ks_meta (as you can see in example below)!
We use it on cobbler-side to be able allocate them from one place.
We turn off pxe-menu creation for openvz containers to not pollute this menu.

For example:

.. code-block:: shell

    # cobbler profile add --name=vz01 --distro=CentOS6-x86_64 --autoinst=/your/autoinst.cfg \
            --ks_meta="lang=ru_RU.UTF-8 keyb=ru vz_ctid=101 vz_swappages=0:2G vz_numproc=120:120" \
            --repos="centos6-x86_64-os centos-x86_64-updates" \
            --virt-type=openvz \
            --virt-ram=1024 \
            --virt-cpus=1

    # cobbler system add --name=vz01 \
            --profile=vz01 \
            --virt-type=openvz \
            --virt-ram=1024 \
            --virt-cpus=1

    # cobbler system edit --name=vz01 \
            --hostname=vz01.example.com \
            --interface=eth0 \
            --mac=YOUR_MAC_HERE \
            --static=1 \
            --ip-address=YOUR_IP \
            --netmask=MASK \
            --gateway=GATEWAY_IP \
            --name-servers=NAME_SERVERS_IPs

On koan side:

.. code-block:: none

    # koan --server=COBBLER_IP --virt --system=vz01

This will start installation process. ovz-install script will install all packages and groups listed in $packages
section.
As root for installation ovz-install will use /vz/private/$VEID (/vz/private/101 for example above), that can be 
overriden with vz_ve_private variable in ks_meta (eg. vz_ve_private=/some/path or vz_ve_private=/other/path/$VEID
or vz_ve_private=/some/path/101 - $VEID will be replaced with CTID).
After installation ovz-install will process "services" option from autoinst like it do anaconda and run
post-installation script, defined in autoinst (only in chroot), so you can tune the container for your needs.
At the end of process ovz-install process installed tree to be truly OpenVZ container - creates dev files, change init 
scripts etc.
Created container started after that, so you should be able to log in to it with root and password you defined for root
in autoinst file.


Options for creating OpenVZ containers
######################################

You should set virt-type to "openvz" in profile or system to create OpenVZ container.

.. code-block:: none

    --virt-file-size 	not used for now. We think we can use it for logical volume creation, or quoting
                        filesystem usage, or for creating containers in ploop-file.
    --virt-ram			as for other VMs
    --virt-cpus			as for other VMs
    --virt-path			not used now. Container will be created in /vz/private/$VEID, where $VEID will be replaced by
                        openvz with CTID (container ID). Can be redefined by vz_ve_private variable you can place in ks_meta.
    --virt_bridge		not used now.
