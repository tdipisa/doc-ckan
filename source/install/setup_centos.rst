.. _setup_system:

####################################
Installing basic packages (CentOS 7)
####################################

This documentation page will describe which packages are to be installed on the system.

Please note that most of the steps are the same for both systems. When anything will differ, it will
be clearly pointed out.

============
CentOS setup
============

Here you can find the ISO image used to install the OS:

* http://mi.mirror.garr.it/mirrors/CentOS/7/isos/x86_64/CentOS-7-x86_64-Everything-1503-01.iso

Initial OS installation
-----------------------

When the bootload starts, select "Install CentOS7"
 - Choose language for the installation process and click continue
 - Choose keyboard layout
 - Set Date and Time
 - There will be a warning about the installation destination "Automatic partitioning selected".
   Notice that by default the installer will use an XFS FileSystem for the boot partition and will
   use LVM for the rest of the system.
 - You may now configure the network (button "config network").If you do so you will also be able to setup
   NTP time synchronization in the Time and Date section
 - Choose wich applications will be installed on the system by clicking on Software Selection,
   we will choose "Minimal".
 - Click Begin Installation
 - Create a password for the root user

The install procedure will go on installing a basic set of packages.
At the end you will be requested to reboot the system.


=====================
Network configuration
=====================

If you configured the network interface during the installation process you skip this section

List the network interfaces

service network status

Edit the config file for the intercafe ``/etc/sysconfig/network-scripts/<interface name>`` paying attention to these properties::

   BOOTPROTO="static"
   ONBOOT="yes"
   IPADDR=84.33.2.27
   NETMASK=.......
   GATEWAY=.......

Edit the file ``/etc/resolv.conf`` and add your nameservers.

In the sample VM Google's DNS have been set::

   nameserver 8.8.8.8
   nameserver 8.8.4.4

Start the network service::

   service network start

Check the connection is up by pinging and external server::

   ping google.com

.. attention::
   Please note that in CentOS7 only ssh incoming connections are allowed;
   all other incoming connections are disabled by default.

   In the paragraph related to the httpd service you can find details about
   how to enable incoming traffic.

Note that after configuring the network, you may continue installing the system setup using a ssh connection.

========================
Installing base packages
========================

Internal clock sync
-------------------

CentOS 7 default is to use Chrony::

Install Chrony with the following command::

    yum install -y chrony

Edit /etc/chrony.conf with the desired settings. For example you can sync with
the CentOS servers::

  # Use public servers from the pool.ntp.org project.
  # Please consider joining the pool (http://www.pool.ntp.org/join.html).
  server 0.centos.pool.ntp.org iburst
  server 1.centos.pool.ntp.org iburst
  server 2.centos.pool.ntp.org iburst
  server 3.centos.pool.ntp.org iburst
  ...

Start it::

    systemctl start chronyd

And enable it to autostart at boot::

    systemctl enable chronyd


Other utilities
---------------

Install::

  yum install mc            # mc (along with zip) can be used to navigate inside .war files
  yum install zip unzip
  yum install wget

=================================
Installing PostgreSQL and PostGIS
=================================

Repositories
------------

Download the package for configuring the PGDG repository:

CentOS::

  wget http://yum.postgresql.org/9.4/redhat/rhel-7-x86_64/pgdg-centos94-9.4-1.noarch.rpm

and install it::

  rpm -ivh pgdg-centos94-9.4-1.noarch.rpm

EPEL 7 repository will provide GDAL packages::

  wget http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
  rpm -ivh epel-release-7-5.noarch.rpm

Update the packages list::

    yum clean all
    yum check-update

Install PG::

  yum install postgresql94-server postgis2_94

Verify::

  # rpm -qa | grep postg
  postgresql94-libs-9.4.4-1PGDG.rhel7.x86_64
  postgresql94-9.4.4-1PGDG.rhel7.x86_64
  postgis2_94-2.1.8-1.rhel7.x86_64
  postgresql94-server-9.4.4-1PGDG.rhel7.x86_64

Init the DB::

  /usr/pgsql-9.4/bin/postgresql94-setup initdb

Setting PostgreSQL access
-------------------------

Edit the file ``/var/lib/pgsql/9.4/data/pg_hba.conf`` so that the local connection entries
will change to::

  # "local" is for Unix domain socket connections only

  local   all             postgres                                peer
  local   all             all                                     md5

  # IPv4 local connections:

  host    all             postgres        127.0.0.1/32            ident
  host    all             all             127.0.0.1/32            md5

  # IPv6 local connections:
  host    all             postgres        ::1/128                 ident
  host    all             all             ::1/128                 md5


Setup automatic start
---------------------

Configure automatic service startup at boot time ::

  systemctl enable postgresql-9.4

Start the service right now ::

  systemctl start postgresql-9.4


=====================
Creating system users
=====================

.. _create_user_tomcat:

Create tomcat user
------------------
::

  [root@cerco ~]# adduser -m -s /bin/bash tomcat
  [root@cerco ~]# passwd tomcat


========================
Installing  apache httpd
========================

Apache httpd is used as entry point for web accesses.
It will be configured as a reverse proxy for the requests to the running web applications.

Install httpd::

    yum install httpd

Create the file ``/etc/httpd/conf.d/00_servername.conf`` and configure the ``ServerName``.

If no name is assigned to the IP address assigned to this machine, we'll set the IP address here::

  ServerName 84.33.2.27:80

Configure the automatic start at boot ::

  systemctl enable httpd

Start the service right away ::

  systemctl start httpd

Check if the machine is reachable from outside, pointing your browser to::

  http://84.33.2.27

If you cannot reach the machine, proceed with next section.

Configure incoming requests
---------------------------

If the machine is not reachable from the outside, allow the incoming connections by issuing this command::

  firewall-cmd --zone=public --add-port=80/tcp --permanent
  firewall-cmd --reload

Configuring httpd
-----------------

Enable gz compression
'''''''''''''''''''''

Create file ``/etc/httpd/conf.d/05_deflate.conf`` with the following content::

  SetOutputFilter DEFLATE
  AddOutputFilterByType DEFLATE text/html text/plain text/xml text/javascript text/css

===============
Installing java
===============

CentOS
------

::

For CentOS systems, you can download the JDK RPM from this page:

  http://www.oracle.com/technetwork/java/javase/downloads/index.html

Oracle does not expose a URL to automatically dowload the JDK because an interactive licence acceptance is requested.
You may start downloading the JDK RPM from a browser, and then either:

* stop the download from the browser and use on the server the dynamic download URL your browser has been assigned, or
* finish the download and transfer the JDK RPM to the server using ``scp``.
* install the RPM using the following command line

::

  rpm -ivh jdk-7u51-linux-x64.rpm

Verify the proper installation on the JDK::

  # java -version
  java version "1.7.0_79"
  Java(TM) SE Runtime Environment (build 1.7.0_79-b13)
  Java HotSpot(TM) 64-Bit Server VM (build 24.79-b03, mixed mode)
  # javac -version
  javac 1.7.0_79


You may want anyway to use the Oracle JDK.

.. _deploy_tomcat:

========================
Installing apache tomcat
========================

CentOS
------

::

    wget http://it.apache.contactlab.it/tomcat/tomcat-7/v7.0.63/bin/apache-tomcat-7.0.63.tar.gz
    tar xvf apache-tomcat-7.0.63.tar.gz

    mv apache-tomcat-7.0.63 /opt/
    ln -s /opt/apache-tomcat-7.0.63 /opt/tomcat


.. _create_catalina_base:

==============================
Creating apache tomcat context
==============================

Creating `base/` template directory
-----------------------------------

::

  mkdir -p /opt/tomcat/base/{bin,conf,logs,temp,webapps,work}
  cp -r /opt/tomcat/conf/* /opt/tomcat/base/conf/
