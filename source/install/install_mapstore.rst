.. _install_mapstore:

###################
Installing MapStore
###################


============
Introduction
============

In this document you'll only find specific information for installing MapStore and some required ancillary 
applications (GeoStore, http-proxy). 

It is expected that the base system has already been properly installed and configured as described in :ref:`setup_system`.

In such document there are information about how to install some required base components, such as PostgreSQL, 
Apache HTTPD, Oracle Java, Apache Tomcat.

===================
Installing MapStore
===================

.. hint::
   MapStore info page at https://github.com/geosolutions-it/mapstore/wiki/MapStore%20Build%20and%20Deployment
   
.. hint::
   GeoStore project page at https://github.com/geosolutions-it/geostore
   

Download packages
-----------------

Download the `.war` files needed for a full MapStore installation::

   cd /root/download
   wget http://demo.geo-solutions.it/share/JRC/deliverables/bin/mapstore.war   
   wget http://maven.geo-solutions.it/it/geosolutions/geostore/geostore-webapp/1.1-SNAPSHOT/geostore-webapp-1.1-SNAPSHOT-postgresql.war
   wget http://maven.geo-solutions.it/proxy/http_proxy/1.0.4/http_proxy-1.0.4.war   

Setup tomcat base
-----------------

Create catalina base directory for MapStore::

   cp -a /var/lib/tomcat/base/  /var/lib/tomcat/mapstore
   cp /root/download/mapstore.war              /var/lib/tomcat/mapstore/webapps/mapstore.war
   cp /root/download/geostore-webapp-1.1-SNAPSHOT-postgresql.war /var/lib/tomcat/mapstore/webapps/geostore.war
   cp /root/download/http_proxy-1.0.4.war      /var/lib/tomcat/mapstore/webapps/http_proxy.war


Create user and DB for GeoStore
-------------------------------

Create a PostgreSQL DB for GeoStore::

   su - postgres -c "createuser -S -D -R -P -l geostore"
   su - postgres -c "createdb -O geostore geostore -E utf-8"
   
Annotate the user password.

Configure GeoStore
------------------

Prepare a property file for setting DB info to geostore::

   vim /var/lib/tomcat/mapstore/conf/geostore.properties
   
and add this content (setting the proper password) ::

   geostoreVendorAdapter.databasePlatform=org.hibernate.dialect.PostgreSQLDialect
   geostoreDataSource.driverClassName=org.postgresql.Driver
   geostoreDataSource.url=jdbc:postgresql://localhost:5432/geostore
   geostoreDataSource.username=geostore
   geostoreDataSource.password=YOUR_PASSWORD_HERE
   #geostoreEntityManagerFactory.jpaPropertyMap[hibernate.hbm2ddl.auto]=validate
   geostoreEntityManagerFactory.jpaPropertyMap[hibernate.hbm2ddl.auto]=create
   geostoreEntityManagerFactory.jpaPropertyMap[hibernate.default_schema]=public
   geostoreVendorAdapter.generateDdl=true
   geostoreVendorAdapter.showSql=false
   

.. warning::
   This is a temporary setup.
   
   After we'll start mapstore the first time, we'll have to reconfigure this file in order not to 
   **destroy** the db content.
   
   See :ref:`init_geostore_db`  

setenv.sh
---------

Create the file `setenv.sh`. 
We'll set here some system vars used by tomcat, by the JVM, and by the webapp itself::

   vim /var/lib/tomcat/mapstore/bin/setenv.sh

Insert this content::
  
   export CATALINA_BASE=/var/lib/tomcat/mapstore
   export CATALINA_HOME=/opt/tomcat/
   export CATALINA_PID=$CATALINA_BASE/work/pidfile.pid

   GEOSTORE_OVR_FILE=file:$CATALINA_BASE/conf/geostore.properties
   
   export JAVA_OPTS="$JAVA_OPTS -Xms512m -Xmx1024m"
   export JAVA_OPTS="$JAVA_OPTS -Dgeostore-ovr=$GEOSTORE_OVR_FILE"     
   
and make it executable::

   chmod +x /var/lib/tomcat/mapstore/bin/setenv.sh


Edit server.xml
---------------

We need to assign 3 ports to this catalina instance.

Edit file ::

   vim /var/lib/tomcat/mapstore/conf/server.xml

and change the connection ports in this way: 

- 8006 for commands to catalina instance
- 8081 for the HTTP connections
- 8010 for the AJP connections


See also :ref:`application_ports`.

Tomcat dir ownership
--------------------

Set the ownership of the ``mapstore/`` related directories to user tomcat ::

   chown tomcat: -R /var/lib/tomcat/mapstore
 

Automatic startup
-----------------

Create the file ``/etc/init.d/mapstore`` and insert :download:`this content <../resources/mapstore>`.

Once downloaded, make it executable ::

   chmod +x /etc/init.d/mapstore

and set it as autostarting  ::

   chkconfig --add mapstore

.. note::    
   If using Ubuntu, you have to use this command instead::
  
      update-rc.d mapstore start 90 2 3 4 5 . stop 10 0 1 6 .
      
   
.. _init_geostore_db:
   
Init DB
-------

Start mapstore to make GeoStore init its db::

   service mapstore start
   
When started, the geostore schema will be created.

Now edit the file ``geostore.properties``::    

   vim /var/lib/tomcat/mapstore/conf/geostore.properties
   
and edit the two lines containing ``hibernate.hbm2ddl.auto`` so that they'll read::

   geostoreEntityManagerFactory.jpaPropertyMap[hibernate.hbm2ddl.auto]=validate
   #geostoreEntityManagerFactory.jpaPropertyMap[hibernate.hbm2ddl.auto]=create

i.e. the ``#`` should be only on the line which ends with  ``create``

Once done, restart mapstore::

   service mapstore restart

   
Configure httpd
---------------
   
Create the file ``/etc/httpd/conf.d/80-mapstore.conf`` and insert these lines::

   ProxyPass        /mapstore   ajp://localhost:8010/mapstore                                                                                                                                                                                                                           
   ProxyPassReverse /mapstore   ajp://localhost:8010/mapstore
   ProxyPass        /geostore   ajp://localhost:8010/geostore                                                                                                                                                                                                                           
   ProxyPassReverse /geostore   ajp://localhost:8010/geostore
   ProxyPass        /http_proxy ajp://localhost:8010/http_proxy                                                                                                                                                                                                                           
   ProxyPassReverse /http_proxy ajp://localhost:8010/http_proxy

.. note::    
   If using Ubuntu, you have to put these lines in file ::
   
      vim /etc/apache2/sites-available/ckan 
      
   just before the ``ProxyPass`` directive redirecting the ``/``.    


Then reload the configuration for apache httpd::

   service httpd reload

  
Configuring MapStore
--------------------

.. warning:: 
   Some configuration steps are mandatory for the CKAN-MapStore integration. 
   Please see :ref:`config_mapstore` for more details.



