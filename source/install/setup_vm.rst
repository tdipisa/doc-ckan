.. _setup_vm:

################
VM configuration
################


=======================
VM Configuration params
=======================


------------------------------
Suggested system configuration
------------------------------


+-----------+--------------+------------------+----------------------+
|           | Test system  | Prod             | Prod                 |
|           |              | (minimal config) | (recommended config) |
+===========+==============+==================+======================+
| CPU       | 1 CPU / core | 4 CPU / core     | 4-8 CPU / core       |
+-----------+--------------+------------------+----------------------+
| RAM       | 4 GB         | 4 GB             | 8 GB                 |
+-----------+--------------+------------------+----------------------+
| Hard disk | 20 GB        | 20 GB            | 20-40 GB             |
+-----------+--------------+------------------+----------------------+

Provided VM is configured this way:

- 1 CPU with 4 cores
- RAM: 4GB
- Disk space: 20GB


This configuration is enough for a minimal production environment.  


Host info
---------

Here a list of the information you are going to set up in your systems. 
You will need them when configuring the various applications.  

- IP address: ___________________________
- Hostname:   ___________________________

System users
------------

+----------+----------+---------------------------------------------+
| Username | Password | Notes                                       |
+==========+==========+=============================================+
| root     |          | Set during OS installation                  |
+----------+----------+---------------------------------------------+
| tomcat   |          | Set up in section :ref:`create_user_tomcat` |
+----------+----------+---------------------------------------------+
| ckan     | ---      | No login.                                   |
|          |          | `su -s /bin/bash - ckan`                    |
+----------+----------+---------------------------------------------+
|          |          |                                             |
+----------+----------+---------------------------------------------+
|          |          |                                             |
+----------+----------+---------------------------------------------+
|          |          |                                             |
+----------+----------+---------------------------------------------+

   
PostgreSQL users
----------------

+-------------+--------------------------+------------------------------------+
| Username    | Password . . . . . . . . | Notes                              |
+=============+==========================+====================================+
| ckan        |                          | Main DB CKAN user                  |
+-------------+--------------------------+------------------------------------+
| datastore   |                          | R/W User for CKAN datastore plugin |
+-------------+--------------------------+------------------------------------+
| datastorero |                          | RO User for CKAN datastore plugin  |
+-------------+--------------------------+------------------------------------+
| geostore    |                          |                                    |
+-------------+--------------------------+------------------------------------+
| geonetwork  |                          |                                    |
+-------------+--------------------------+------------------------------------+
|             |                          |                                    |
+-------------+--------------------------+------------------------------------+
   
.. _application_ports:   
   
Installed applications
----------------------

+-------------+---------+------+------+---------------+---------------------------+
| Name        | Command | HTTP | AJP  | context       | Note                      |
|             | port    | port | port |               |                           |
+=============+=========+======+======+===============+===========================+
| CKAN        | ---     | 5000 | ---  | `/`           |                           |
+-------------+---------+------+------+---------------+---------------------------+
| Solr        | 8005    | 8009 | 8080 | `/solr`       | Not exposed through httpd |
+-------------+---------+------+------+---------------+---------------------------+
| MapStore    |         |      |      | `/mapstore`   |                           |
+-------------+         +      +      +---------------+---------------------------+
| GeoStore    | 8006    | 8010 | 8081 | `/geostore`   |                           |
+-------------+         |      |      +---------------+                           |
| httpd-proxy |         |      |      | `/http_proxy` |                           |
+-------------+---------+------+------+---------------+---------------------------+
| GeoNetwork  | 8007    | 8011 | 8082 | `/geonetwork` | Optional                  |
+-------------+---------+------+------+---------------+---------------------------+
| GeoServer   | 8008    | 8083 | 8012 | `/geoserver`  | Optional                  |
+-------------+---------+------+------+---------------+---------------------------+
|             |         |      |      |               |                           |
+-------------+---------+------+------+---------------+---------------------------+



VM setup
--------

When creating a VM, you may not want to give VMWare all the information about the system. 
The reason behind this is because VMWare is smart enough to automatically handle some SO installation stages; this stages
will be skipped on the UI, and this will make the deployment procedure different than one performed on a real machine.
   

Setting up VMWare
'''''''''''''''''

Sample settings for creating a new VM:

- VM configuration: Custom
- HW compatibility: workstation 8 
- Install OS from: I will install the the operationg system later
- Guest OS: Linux Centos 64-bit
- VM name: *setup the name*
- Processors: 1 processore, 2 core
- Memory: 4096MB
- Network connection: bridged

Then configure the disk as you need.

This is a sample configuration:

- I/O Controller type: LSI Logic
- Disk: create a new virtual disk
- Virtual disk type: SCSI
- Mode: Independent, persistent
- Max disk size: 20G, store virtual disk as a single file.

Then configure the DVD reader setting the ISO image of the OSinstaller, and start the VM. 


==================
Document changelog
==================

+---------+------------+--------+------------------+
| Version | Date       | Author | Notes            |
+=========+============+========+==================+
| 1.0     | 2014-02-06 | ETj    | Initial revision |
+---------+------------+--------+------------------+
| 1.1     | 2014-02-26 | Carlo C| Added geoserver  |
+---------+------------+--------+------------------+
