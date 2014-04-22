.. _cloning_vm:

##############
Cloning the VM
##############

Once the OS and the various applications have been deployed, you may find useful to clone the VM in order to backup it.  

===================================
Copying the virtual disk and the VM
===================================

`VMWare ref doc at <http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=1027876>`_.

Log into the host machine using ssh.

``CD`` into the datastore dir where the VM to be cloned is ::

   cd /vmfs/volumes/datastore1

Create the directory for the new VM (e.g.) ::

   mkdir your_new_disk_image

Copy the virtual disk ::

   vmkfstools -i src/src.vmdk your_new_disk_image/your_new_disk_image.vmdk

Create a new VM, with the params you need.  
In order to use the new disk image, you need to specify the option "*Use an existing virtual disk*" 
when you are requested for the "_Disk_" definition; 
Then you'll select your file ``your_new_disk_image/your_new_disk_image.vmdk``.

Note that once the VM is created, you'll also get the directory ``your_new_disk_image_2/`` 
which contains the VM configuration info.


=============================
System settings on cloned VMs
=============================

Once you have a copy of the VM set up, run it.

You'll get a system almost identical to the source. 
One of the small changes inside the guest machine will be the network MAC address, while the 
IP address configuration will be the same as the source.

On most Linux systems, since the MAC is changed, it will not match the IP configuration, and the external network interface will
be left unconfigured.  
  

Network setup
-------------

You'll have to log into the system using the VMWare console, since the IP address will be not configured, or it could
be in conflict with the source machine (i.e. the machine the VM was cloned from). 

In the new VM the network MAC address has changed, so the configuration shall be reset.

The file ``/etc/udev/rules.d/70-persistent-net.rules`` contains the rules for assigning the device name (``ethN``) 
based on the MAC address.

You'll probably find the first rule that assign ``eth0``to the MAC address of the source VM, 
and then a second rule assignign ``eth1`` to the current MAC address.

You may either:

* remove the  file ``/etc/udev/rules.d/70-persistent-net.rules`` and reboot the system; a new file will be 
  created at boot time with the new proper configuration;
* edit the file, removing the first rule assigning the ``eth0`` name, and replace ``eth1`` with ``eth0`` on the second line; 
  you have to reboot the system anyway to let the system know the interface name has changed.
 
Before rebooting the system you may want to change the IP address.

You need to know the new MAC address of your network interface. You may find it in the aforementioned ``70-persistent-net.rules``
file, or you can dig into the system log::  

   # dmesg | grep eth0
   e1000 0000:02:00.0: eth0: (PCI:66MHz:32-bit) 00:0c:29:40:53:f3
   e1000 0000:02:00.0: eth0: Intel(R) PRO/1000 Network Connection
   e1000: eth0 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: None
   eth0: no IPv6 routers present   


Now edit the file ``/etc/sysconfig/network-scripts/ifcfg-eth0``.

You have to edit these fields: 

- ``HWADDR``, the MAC address;
- ``IPADDR``, the new IP address for this VM

If the VM is going to be installed in a network different from the source VM, you'll have to edit the 
``netmask`` and ``gateway`` lines as well.

In the same way, if the VM is in a different network, you will likely have to edit the nameservers in ``/etc/resolv.conf``.

Now update the ``HOSTNAME`` in ``/etc/sysconfig/network``::

  NETWORKING=yes
  HOSTNAME=your_new_hostname

Then restart the machine to make sure every configuration has been set properly  ::

   reboot
   
Reconfiguring applications
--------------------------

Reconfig the ``ServerName`` in ``/etc/httpd/conf.d/00_servername.conf`` .

If the current IP is not bound to any name, set the server name to the current IP address; e.g.::

   ServerName aa.bb.cc.dd:80

Refer to the application configuration page for reconfiguring any application covered in this guide.
E.g.:

* :ref:`reconfig_ckan` 

Security reconfig
-----------------

You may want to change the passwords for the system users ``root`` and ``tomcat``. 


