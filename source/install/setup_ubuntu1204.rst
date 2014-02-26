.. _setup_ubuntu:

##################################
Installing basic packages (Ubuntu)
##################################

This documentation page will describe which packages are to be installed on the system.

Instructions for Ubuntu 12.04 only will be provided.


==================
Ubuntu 12.04 setup
==================

Here you can find the ISO image used to install the OS:

* http://www.ubuntu.com/start-download?distro=server&bits=64&release=lts

.. note::
   This is not a permanent link, and will be probably linked to version 14.04 once available.

Initial OS installation
-----------------------

- Choose language
- Choose "Install Ubuntu server"
- if translation in the choosen language is not complete, select "ok"
- Select keyboard layout

Network
~~~~~~~

If DHCP configuration fails, you'll be given the options to retry the DHCP configuration or 
to configure the IP address by hand.
We'll set the given IP address, netmask, gateway and nameserver.
Then you'll have to config the hostname (setting ``ckan-ubuntu`` for the sample system) 
and the domain name (``localdomain``)

Then you'll have to setup a username for a non-privileged user (and related real name).


Encode dir: no

Timezone guessing: ok if it's ok


Disk partitioning
~~~~~~~~~~~~~~~~~

Proceed with disk partitioning.
"Guided partitioning" is s good choice if you don't have particular requirements.


Packages handler
~~~~~~~~~~~~~~~~

Setup http proxy if needed


tasksel setup
~~~~~~~~~~~~~

You will be requested how the system should handle the software updates.
We'll set "no automatic update"


Software selection
------------------

Only select 
 * OpenSSH server

so that we will able to finish the system setup from a remote ssh.


Install GRUB on MBR: select ``yes``.
 


========================
Installing base packages
========================
      

Other utilities
---------------

Install::

  apt-get install mc zip unzip          # mc (along with zip) can be used to navigate inside .war files


===================
Installing iptables
===================

Ubuntu does not have default iptables rules.

Install the package::

   apt-get install iptables-persistent
   
and allow the creation of the rules files for IPv4.

Then edit the rules file::

   vim /etc/iptables/rules.v4
   
replace its content with the following lines::

   *filter
   :INPUT ACCEPT [0:0]
   :FORWARD ACCEPT [0:0]
   :OUTPUT ACCEPT [26:14198]
   -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT 
   -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT 
   -A INPUT -p icmp -j ACCEPT 
   -A INPUT -i lo -j ACCEPT 
   -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT 
   -A INPUT -j REJECT --reject-with icmp-host-prohibited 
   -A FORWARD -j REJECT --reject-with icmp-host-prohibited 
   COMMIT

and reload the rules::

   service iptables-persistent reload
   
=================================
Installing PostgreSQL and PostGIS
=================================

.. hint::
   Ref info page at http://www.postgresql.org/download/linux/ubuntu/

Repositories
------------
  
Configure the PGDG repository:

.. note::
   The ~pitti repo, used in the command ::

      add-apt-repository ppa:pitti/postgresql
      
   is obsolete, as reported here:
   
   * https://launchpad.net/~pitti/+archive/postgresql
   * http://wiki.postgresql.org/wiki/Apt

Create the ``pgdg.list`` file and set the apt source in it::

   vim /etc/apt/sources.list.d/pgdg.list

and add the line::

   deb http://apt.postgresql.org/pub/repos/apt/ precise-pgdg main
            
Add the key for the repo::

   wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
   
And then finally update the package list::
   
   apt-get update
      

Install PostgreSQL and PostGIS::

  apt-get install postgresql-9.2 postgresql-9.2-postgis-2.1

Verify::

  dpkg-query -l "*postgr*" | grep ii
  

Setting PostgreSQL access
-------------------------

Edit the file ``/etc/postgresql/9.2/main/pg_hba.conf`` so that the local connection entries 
will change to::

  # "local" is for Unix domain socket connections only
  local   all             postgres                                peer
  local   all             all                                     md5

  # IPv4 local connections:
  host    all             all             127.0.0.1/32            md5

  # IPv6 local connections:
  host    all             all             ::1/128                 md5
   

Setup automatic start
---------------------

PostgreSQL should already be configured for automatic startup.

Check the current running status using::

   # service postgresql status
   9.2/main (port 5432): online


=====================
Creating system users
=====================

.. _create_user_tomcat:

Create tomcat user
------------------
:: 

  # useradd -m -s /bin/bash tomcat
  # passwd tomcat


========================
Installing  apache httpd
========================

Apache httpd is used as entry point for web accesses. 
It will be configured as a reverse proxy for the requests to the running web applications.

Install httpd::

   apt-get install apache2-mpm-worker

The prev command will install and start apache httpd.

Check if the machine is reachable from outside, pointing your browser to:: 

  http://YOUR_SERVER_IP


Enable gz compression
---------------------

Create file ``/etc/apache2/conf.d/05_deflate.conf`` with the following content::

  SetOutputFilter DEFLATE
  AddOutputFilterByType DEFLATE text/html text/plain text/xml text/javascript text/css

===============
Installing java
===============

You can download the JDK ``tar.gz`` from this page:

  http://www.oracle.com/technetwork/java/javase/downloads/index.html

Oracle does not expose a URL to automatically dowload the JDK because an interactive licence acceptance is requested.  
You may start downloading the JDK TGZ from a browser, and then either:

* stop the download from the browser and use on the server the dynamic download URL your browser has been assigned, or
* finish the download and transfer the JDK TGZ to the server using ``scp``.   

Install the JDK stuff into ``/usr/local/java``::

   mkdir /usr/local/java
   cd /usr/local/java
   tar xzvf /root/download/jdk-7u51-linux-x64.tar.gz

Create a symlink to the directory created from the tgz::

   ln -s jdk1.7.0_51 jdk7
      
Install executables::

   update-alternatives --install /usr/bin/java java /usr/local/java/jdk1.7.0_51/bin/java    70051
   update-alternatives --install /usr/bin/java java /usr/local/java/jdk7/bin/java           79999
   
   update-alternatives --install /usr/bin/javac javac /usr/local/java/jdk1.7.0_51/bin/javac 70051
   update-alternatives --install /usr/bin/javac javac /usr/local/java/jdk7/bin/javac        79999


Verify the proper installation on the bin::

  # java -version
  java version "1.7.0_51"
  Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
  Java HotSpot(TM) 64-Bit Server VM (build 24.51-b03, mixed mode) 
  # javac -version
  javac 1.7.0_51
  
     
.. _deploy_tomcat:

========================
Installing apache tomcat
========================

Download apache tomcat and install it under ``/opt``::

  wget http://mirror.nohup.it/apache/tomcat/tomcat-6/v6.0.39/bin/apache-tomcat-6.0.39.tar.gz
  tar xzvf apache-tomcat-6.0.39.tar.gz -C /opt/

Let's use a symlink to ease future upgrades::

  ln -s /opt/apache-tomcat-6.0.39/ /opt/tomcat


.. _create_catalina_base:

Creating `base/` template directory
-----------------------------------

::

  mkdir -p /var/lib/tomcat/base/{bin,conf,logs,temp,webapps,work}
  cp /opt/tomcat/conf/* /var/lib/tomcat/base/conf/


==================
Document changelog
==================

+---------+------------+--------+------------------+
| Version | Date       | Author | Notes            |
+=========+============+========+==================+
| 1.0     | 2014-02-24 | ETj    | Initial revision |
+---------+------------+--------+------------------+
