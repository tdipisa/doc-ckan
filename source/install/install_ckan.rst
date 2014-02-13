.. _install_ckan:

###################
Installing CKAN 2.2
###################

============
Introduction
============

In this document you'll only find specific information for installing CKAN, some required ancillary applications 
and some ufficial CKAN extensions.

It is expected that the base system has already been properly installed and configured as described in :ref:`setup_centos`.

In such document there are information about how to install some required base components, such as PostgreSQL, 
Apache HTTPD, Oracle Java, Apache Tomcat.


===============
Installing Solr
===============

Solr is a java webapp used by CKAN as a backend for dataset indexing.  
Solr shall be installed in a tomcat instance on its own, in order to decouple it from other installed webapps (i.e. GeoNetwork, LifeRay).

We're going to install its catalina base in ``/var/lib/tomcat/solr``; we'll put its configuration files 
in ``/etc/solr``.

Install
-------

Download solr (it's a 127MB *.tgz* file) and untar it::

   cd /root/download
   wget http://archive.apache.org/dist/lucene/solr/4.5.0/solr-4.5.0.tgz
   tar xzvf  solr-4.5.0.tgz
   
Make sure you already:

- created the tomcat user (:ref:`create_user_tomcat`)
- installed tomcat (:ref:`deploy_tomcat`)
- created the base catalina template (:ref:`create_catalina_base`)


Create catalina base directory for solr::

   cp -a /var/lib/tomcat/base/  /var/lib/tomcat/solr

Copy .war file ::

   cp -av /root/download/solr-4.5.0/dist/solr-4.5.0.war /var/lib/tomcat/solr/webapps/solr.war
   
Copy configuration files ::

   mkdir -p /etc/solr/ckan
   cp -r /root/download/solr-4.5.0/example/solr/collection1/conf /etc/solr/ckan
   
Create file ``/etc/solr/solr.xml`` ::

   <solr persistent="true" sharedLib="lib">
      <cores adminPath="/admin/cores" defaultCoreName="ckan">
         <core name ="ckan-schema-2.0" instanceDir="ckan"> 
            <!-- <property name="dataDir" value="/var/lib/solr/data/ckan" /> -->
         </core>
      </cores>
   </solr>
   
Copy libs ::
   
   mkdir -p /opt/solr/libs
   cp solr-4.5.0/dist/*.jar                        /opt/solr/libs
   cp solr-4.5.0/contrib/analysis-extras/lib/*     /opt/solr/libs
   cp solr-4.5.0/contrib/clustering/lib/*          /opt/solr/libs
   cp solr-4.5.0/contrib/dataimporthandler/lib/*   /opt/solr/libs
   cp solr-4.5.0/contrib/extraction/lib/*          /opt/solr/libs
   cp solr-4.5.0/contrib/langid/lib/*              /opt/solr/libs
   cp solr-4.5.0/contrib/uima/lib/*                /opt/solr/libs
   cp solr-4.5.0/contrib/velocity/lib/*            /opt/solr/libs  

Backup solr config files ::

   cp /etc/solr/ckan/conf/solrconfig.xml /etc/solr/ckan/conf/solrconfig.xml.orig
   
Edit config file, commenting out all the  ``<lib dir= .....`` entries, and add::

   <lib dir="/opt/solr/libs/" regex=".*\.jar" />


Create data dir::
   
   mkdir /var/lib/tomcat/solr/data
   

Edit file ``/var/lib/tomcat/solr/bin/setenv.sh``. 
We'll set here some system vars used by tomcat, by the JVM, and by the wabapp itself

::

    export CATALINA_BASE=/var/lib/tomcat/solr
    export CATALINA_HOME=/opt/tomcat/
    export CATALINA_PID=$CATALINA_BASE/work/pidfile.pid

    export JAVA_OPTS="$JAVA_OPTS -Xms512m -Xmx800m -XX:MaxPermSize=256m"

    export JAVA_OPTS="$JAVA_OPTS -Dsolr.solr.home=/etc/solr/"
    export JAVA_OPTS="$JAVA_OPTS -Dsolr.data.dir=$CATALINA_BASE/data"

Make ``setenv.sh`` executable::

    chmod +x /var/lib/tomcat/solr/bin/setenv.sh
   
Edit server.xml
---------------

Solr is the first tomcat instance we are installing in this VM, so we can keep the default ports:

- 8005 for commands to catalina instance
- 8080 for the HTTP connection

We won't need the AJP connection, since Solr will be not exposed to the internet via apache httpd.

Remember that you may change these ports in the file `/var/lib/tomcat/solr/conf/server.xml`.

See also :ref:`application_ports`.


Automatic startup
-----------------


Create the file ``/etc/init.d/solr`` and insert :download:`this content <../resources/solr>`.

Once downloaded, make it executable ::

   chmod +x /etc/init.d/solr

and set it as autostarting  ::

   chkconfig --add solr

Final configurations
--------------------

Set the ownership of the ``solr/`` related directories to user tomcat ::

   chown tomcat: -R /var/lib/tomcat/solr
   chown tomcat: -R /etc/solr/
   
In order to make solr work with CKAN, a schema needs to be set.
It will be set in a following section, so we do not want to start solr right away.  

============================
Installing required packages
============================

Install the software packages needed by CKAN::

   yum install gcc gcc-c++ make python-devel libxml2-devel libxslt-devel python-babel python-psycopg2 python-lxml \
               python-pylons python-repoze-who python-repoze-who-plugins-sa python-repoze-who-testutil \
               python-repoze-who-friendlyform python-tempita python-zope-interface \ 
               policycoreutils-python git postgresql92-devel

====================
Creating a CKAN user
====================
 
The ``ckan`` user is created with a shell of ``/sbin/nologin`` and a home directory of ``/usr/lib/ckan``::

   useradd -m -s /sbin/nologin -d /usr/lib/ckan -c "CKAN User" ckan

Should you need to run anything as user ``ckan``, you can switch to the ckan account
by issuing this command as ``root`` ::
   
   su -s /bin/bash - ckan

==============
Setup CKAN dir
==============

Open the ckan home directory up for read access so that the content 
will eventually be able to be served out via httpd ::

   chmod 755 /usr/lib/ckan

Modify the defaults and the current file context of the newly created directory 
such that it is able to be served out via httpd ::

   semanage fcontext --add --ftype -- --type httpd_sys_content_t "/usr/local/ckan(/.*)?"
   semanage fcontext --add --ftype -d --type httpd_sys_content_t "/usr/local/ckan(/.*)?"
   restorecon -vR /usr/lib/ckan
    
========================
PostgreSQL configuration
========================

Create the ``ckan`` user in postgres::

   su - postgres -c "createuser -S -D -R -P ckan"
   
and annotate the password for such user.
As an example, we'll use ``ckan_pw`` to show where this info will be needed.

Create the ckan db::

   su - postgres -c "createdb -O ckan ckan -E utf-8"


============================
Configuring CKAN environment
============================


Installing python dependencies
------------------------------

As user ``root`` run::

   easy_install pip
   pip install virtualenv


As user ``ckan``, go to ckan home dir::

   cd
   
Create a virtualenv called ``default``::

   virtualenv --no-site-packages default
   
Activate the vitualenv::
   
   . default/bin/activate
   
Download and install CKAN::

   pip install -e 'git+https://github.com/ckan/ckan.git@release-v2.2#egg=ckan'
   
Enable pgsql92 path::   
   
   export PATH=$PATH:/usr/pgsql-9.2/bin/
   
Download and install the necessary Python modules to run CKAN into the isolated Python environment::
 
   pip install -r default/src/ckan/requirements.txt --allow-external argparse
   
  
.. _install_ckan_solr_conf:

Solr configuration
------------------

Configure in Solr the CKAN schema::

   service solr stop
   cd /etc/solr/ckan/conf/ 
   mv schema.xml schema.xml.original
   ln -s /usr/lib/ckan/default/src/ckan/ckan/config/solr/schema-2.0.xml /etc/solr/ckan/conf/schema.xml   
   chown tomcat: schema.xml
   service solr start

.. note::
   Should Solr complain about missing libs, copy them from the dist directory::   

      service solr stop
      cp /root/download/solr-4.5.0/dist/solrj-lib/* /var/lib/tomcat/solr/webapps/solr/WEB-INF/lib/
      service solr start

.. important::   
   Note that solr requires the current hostname to be bound to a real IP address.

   This is an example of a hostname not properly bound::   

     [root@ckan conf]# hostname 
     ckan
     [root@ckan conf]# ping ckan
     ping: unknown host ckan
     [root@ckan conf]#
   
   You'll have to edit the ``/etc/hosts`` file and add a line like this::
   
     10.10.100.70 ckan

Start solr and make sure it's working::

   service solr start

   curl -i http://localhost:8080/solr/ | less
   
.. _install_ckan_ckan_conf:
   
CKAN configuration
------------------

Create a default configuration file. 

As ``root`` create the directory ::

   mkdir /etc/ckan
   chown ckan: /etc/ckan/

As user ``ckan``, enter the *virtualenv* ::

   $ . /usr/lib/ckan/default/bin/activate
   (pyenv)$ paster make-config ckan /etc/ckan/default/production.ini 
   

Edit the file ``/etc/ckan/default/production.ini`` 

- DB connection parameters ::

   sqlalchemy.url = postgresql://ckan:PASSWORD@localhost/ckan
   solr_url = http://127.0.0.1:8080/solr/ckan-schema-2.0
    
- Site data ::

    ckan.site_id:
    ckan.site_title:
    ckan.site_url:
    
- Mail notifications (es.) ::

    email_to = info@the.project.org
    smtp_server = server.smtp.for.the.project.org
    error_email_from = notifications@project.org

- Language ::

    ckan.locale_default = en
    ckan.locales_offered = en
    ckan.locale_order = en


The file ``who.ini`` (the *Repoze.who* configuration file) needs to be accessible 
in the same directory as your CKAN config file, so create a symlink to it::

    ln -s /usr/lib/ckan/default/src/ckan/who.ini /etc/ckan/default/who.ini


Directories init
''''''''''''''''

As  ``root``::
  
   mkdir /var/log/ckan
   chown ckan: /var/log/ckan

   
DB init
'''''''

As user ``ckan``::

   . default/bin/activate
   paster --plugin=ckan db init -c /etc/ckan/default/production.ini

.. note::
   The ``db init`` procedure needs solr to be running.


CKAN users
''''''''''

Add a user with sysadmin privileges using this command ::

   (pyenv)$ paster --plugin=ckan sysadmin add USERNAME -c /etc/ckan/default/production.ini
   

Test  CKAN
''''''''''

Run CKAN as user ``ckan``::

   (pyenv)$ paster serve /etc/ckan/default/production.ini &

==========================
Apache httpd configuration
==========================

As ``root``, create the file ``/etc/httpd/conf.d/92-ckan.conf`` and add the following content::

   ProxyPass        / http://localhost:5000/
   ProxyPassReverse / http://localhost:5000/

and reload the configuration ::

   service httpd reload

SElinux
-------

`httpd` is blocked by default by SELinux so that it can't establish internal TCP connections; 
in order to allow http proxying, issue the following command ::

   setsebool -P httpd_can_network_connect 1


================
DataStore plugin
================

.. hint::
   Ref info page at http://docs.ckan.org/en/ckan-2.0/datastore-setup.html

Create database users (``datastore`` with RW privs, and ``datastorero`` with RO), and a DB for the datastore:: 

   su - postgres -c "createuser -S -D -R -P -l datastore"
   su - postgres -c "createuser -S -D -R -P -l datastorero"
   su - postgres -c "createdb -O datastore datastore -E utf-8"

Open the file ``/etc/ckan/default/production.ini`` and edit the lines::

   ckan.datastore.write_url = postgresql://datastore:PASSWORD@localhost/datastore
   ckan.datastore.read_url = postgresql://datastorero:PASSWORD@localhost/datastore

Also, add the ``datastore`` plugin::

   ckan.plugins = datastore [... other plugins...] 

CKAN needs to change some grants on the datastore, but the python script uses the ``sudo`` command, 
which works just fine on Ubuntu but is not configured on CentOS machines.
We're going to run the SQL script by hand, but it requires some setup::

   cd /usr/lib/ckan/default/src/ckan/ckanext/datastore/bin
   cp set_permissions.sql set_permissions_new.sql 
   
Edit ``set_permissions_new.sql`` and set the proper values for the variables in braces::
 
   \set maindb "ckan"
   \set datastoredb "datastore"
   \set ckanuser "ckan"
   \set wuser "datastore"
   \set rouser "datastorero"
   
As ``root`` run::

   su - postgres -c "psql  postgres -f /usr/lib/ckan/default/src/ckan/ckanext/datastore/bin/set_permissions_new.sql"
   
   

(also check this mail http://lists.okfn.org/pipermail/ckan-discuss/2013-March/002593.html).


===================
File storage plugin
===================

.. hint::
   Ref info page at http://docs.ckan.org/en/latest/filestore.html

*FileStore* is used to enable data upload in CKAN. 

Create directory ::

   mkdir -p /var/lib/ckan/upload
   chown ckan: /var/lib/ckan/upload


Set the storage config in ``production.ini``::

   ckan.storage_path = /var/lib/ckan/upload

================
Harvester plugin
================

As root install::

   yum install redis
   chkconfig redis on
   service redis start

Installing ckan harvester
-------------------------

As user ``ckan``::

   . default/bin/activate
   pip install -e git+https://github.com/ckan/ckanext-harvest.git@release-v2.0#egg=ckanext-harvest
   cd default/src/ckanext-harvest/
   pip install -r pip-requirements.txt
   
.. note::
   I (etj, 20131204) found that the redis module is not installed, so it will throw an error
   when running the harvesting commands.
   
   You can install it by running (within the virtualenv) the command::
   
       pip install redis       

Edit file ``/etc/ckan/default/production.ini`` and add the harvest related plugins::  

   ckan.plugins = [...] harvest ckan_harvester
   ckan.harvest.mq.type = redis

Init the db for the harvester services::

   paster --plugin=ckanext-harvest harvester initdb --config=/etc/ckan/default/production.ini

Script harvesting
-----------------

Running harvesting procedure requires issuing a couple of command lines.
It's handy to create a script file that runs them. We'll use the same script to run the cron'ed harvest.

Create the file ``/usr/lib/ckan/run_harvester.sh`` and add the following lines::

   #!/bin/bash

   . /usr/lib/ckan/default/bin/activate

   paster --plugin=ckanext-harvest harvester job-all --config=/etc/ckan/default/production.ini
   paster --plugin=ckanext-harvest harvester run     --config=/etc/ckan/default/production.ini

and make it executable::

   chmod +x /usr/lib/ckan/run_harvester.sh

Periodic harvesting
-------------------

Add a cron job for the harvester::

   crontab -e -u ckan

Add in the crontab the following line to run the harvesting every 15 minutes::

   */15 * * * * /usr/lib/ckan/run_harvester.sh

==============
Spatial plugin
==============

The *spatial* plugin allows CKAN to harvest spatial metadata (ISO 19139) using the CSW protocol.

Upgrade libxml2
---------------

.. important:: 
   As reported on https://github.com/okfn/ckanext-spatial:
   
      NOTE: The ISO19139 XSD Validator requires system library libxml2 v2.9 (released Sept 2012).

   Check the installed libs using
      
      ll /usr/lib64/libxml*
   

Perform the upgrade::
   
   yum install readline-devel xz-devel
   
   cd /root/download
   wget ftp://ftp.pbone.net/mirror/ftp5.gwdg.de/pub/opensuse/repositories/X11:/Enlightenment:/EWebKit/CentOS_CentOS-6/x86_64/libxml2-2-2.9.1-99.1.x86_64.rpm
   wget ftp://ftp.pbone.net/mirror/ftp5.gwdg.de/pub/opensuse/repositories/X11:/Enlightenment:/EWebKit/CentOS_CentOS-6/x86_64/libxml2-devel-2.9.1-99.1.x86_64.rpm
   wget ftp://ftp.pbone.net/mirror/ftp5.gwdg.de/pub/opensuse/repositories/X11:/Enlightenment:/EWebKit/CentOS_CentOS-6/x86_64/libxml2-tools-2.9.1-99.1.x86_64.rpm
   
   rpm -U --force libxml2-2-2.9.1-99.1.x86_64.rpm libxml2-devel-2.9.1-99.1.x86_64.rpm libxml2-tools-2.9.1-99.1.x86_64.rpm


DB configuration
----------------

Add the spatial extension to the ``ckan`` DB::

   # su - postgres -c "psql ckan"
   ckan=# CREATE EXTENSION postgis;
   ckan=# GRANT ALL PRIVILEGES ON DATABASE ckan TO ckan;
   ckan=# GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO ckan;

Installing ckan spatial
-----------------------

As user ``ckan``::

   . default/bin/activate   
   pip install -e git+https://github.com/okfn/ckanext-spatial.git@stable#egg=ckanext-spatial
   cd default/src/ckanext-spatial/
   pip install -r pip-requirements.txt

Init spatial DB
---------------

Init database, where 4326 is the default SRID::

   (pyenv)$ cd /usr/lib/ckan/default/src/ckan
   (pyenv)$ paster --plugin=ckanext-spatial spatial initdb 4326 --config=/etc/ckan/default/production.ini

.. note:: 
   If you get an error saying ::

     ValueError: VDM only works with SQLAlchemy versions 0.4 through 0.7, not: 0.8.3

   just reinstall the proper SQLAlchemy version:: 
      
      pip install -r /usr/lib/ckan/default/src/ckan/requirements.txt
   
Config
------

Edit file ``/etc/ckan/default/production.ini`` and add the spatial related plugins::  

   ckan.plugins = [...] spatial_metadata spatial_query csw_harvester

You may also specify the default SRID::

   ckan.spatial.srid = 4326

Then you may force the validation profiles when harvesting::

   ckan.spatial.validator.profiles = iso19139,gemini2,constraints

Configure Spatial search
''''''''''''''''''''''''

In order to show the widget for the spatial search, you have to 
**TODO**

Configure map extents
'''''''''''''''''''''

In order to display the map that shows the map extents,  
**TODO**


Update Solr clause configuration
--------------------------------

As ``root``, edit the file ``/etc/solr/ckan/conf/solrconfig.xml`` and 
update the value of ``maxBooleanClauses`` to 16384.

Then restart Solr::

   service solr restart


=========================
supervisord configuration
=========================

CKAN does not provide a default script for autostarting; we'll use the *supervisord* deamon to do that.

As root::

   yum install supervisor
   chkconfig supervisord on

Edit the file ``/etc/supervisord.conf`` and add the following lines to handle CKAN::

   [program:ckan]
   command=/usr/lib/ckan/default/bin/paster serve /etc/ckan/default/production.ini
   user=ckan
   autostart=true
   autorestart=true
   numprocs=1
   log_stdout=true
   log_stderr=true
   stdout_logfile=/var/log/ckan/out.log
   stderr_logfile=/var/log/ckan/err.log
   logfile=/var/log/ckan/ckan.log
   startsecs=10
   startretries=3

Add these lines related to the CKAN Harvester::

   [program:ckan_gather_consumer]
   command=/usr/lib/ckan/default/bin/paster --plugin=ckanext-harvest harvester gather_consumer --config=/etc/ckan/default/production.ini
   user=ckan
   autostart=true
   autorestart=true
   numprocs=1
   log_stdout=true
   log_stderr=true
   stdout_logfile=/var/log/ckan/gather_out.log
   stderr_logfile=/var/log/ckan/gather_err.log
   logfile=/var/log/ckan/gather.log
   startsecs=10
   startretries=3

   [program:ckan_fetch_consumer]
   command=/usr/lib/ckan/default/bin/paster --plugin=ckanext-harvest harvester fetch_consumer --config=/etc/ckan/default/production.ini
   user=ckan
   autostart=true
   autorestart=true
   numprocs=1
   log_stdout=true
   log_stderr=true
   stdout_logfile=/var/log/ckan/fetch_out.log
   stderr_logfile=/var/log/ckan/fetch_err.log
   logfile=/var/log/ckan/fetch.log
   startsecs=10
   startretries=3

Run supervisord::

   /etc/init.d/supervisord start


.. _reconfig_ckan:
   
=================================
Reconfiguring CKAN in a cloned VM
=================================

If you are configuring a cloned VM, there is no need to review the whole stuff: only a few data should be reconf. 
 
Usually, in a cloned machine, you only need to reconfigure the references to the IP address. Anyway you may set up 
more stuff as you see fit.


Mandatory reconfig
------------------

There are a few configurations that may prevent the application to work at all.


As reported in ":ref:`install_ckan_solr_conf`", make sure the hostname is resolved somehow.

Also, reconfig the ``ckan.site_url`` property defined in ":ref:`install_ckan_ckan_conf`".


Other reconfig
--------------

If the machine has already run, you may want to clear the CKAN DB, or if security is a concern, you may want to redefine the 
users and/or their related password. Here a list of what you may want to reset (only related to the CKAN installation):

* Password for PostgreSQL user ``ckan``
* Password for PostgreSQL user ``datastore``
* Password for PostgreSQL user ``datastorero``
* Password for CKAN sysadmin ``ckan``
* Clear and reinit db ``ckan`` 
* Clear and reinit db ``datastore`` 
* Clear and reinit Solr index
* Clear redis data
 

System account ``ckan`` was created as a *nologin* account so you don't need to reset any password for it.



==================
Document changelog
==================

+---------+------------+--------+------------------+
| Version | Date       | Author | Notes            |
+=========+============+========+==================+
| 1.0     | 2014-02-06 | ETj    | Initial revision |
+---------+------------+--------+------------------+
