.. _install_mapstore:

####################
Installing GeoServer
####################


============
Introduction
============

In this document you'll only find specific information for installing GeoServer and some required ancillary 
applications (GeoStore, http-proxy). 

It is expected that the base system has already been properly installed and configured as described in :ref:`setup_centos`.

In such document there are information about how to install some required base components, such as Apache HTTPD, Apache Tomcat.

==================
Installing OpenJDK
==================

GeoServer can now be used over OpenJDK which may provide better scalability performance so we can proceed installing and using it:

   apt-get install openjdk-7-jre

Now you may have the openjdk installed:

   /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java -version

java version "1.7.0_51"
OpenJDK Runtime Environment (IcedTea 2.4.4) (7u51-2.4.4-0ubuntu0.12.04.2)

====================
Installing GeoServer
====================

.. hint::
   GeoServer info page at http://geoserver.org
   
.. hint::
   GeoStore project page at https://github.com/geoserver/geoserver
   

Download packages
-----------------

Download the `.war` files needed for a full GeoServer installation::

   cd /root/download
   mkdir geoserver
   cd geoserver
   wget http://ares.boundlessgeo.com/geoserver/2.4.x/geoserver-2.4.x-latest-war.zip

Donwload the extensions:
------------------------

   wget http://ares.boundlessgeo.com/geoserver/2.4.x/community-latest/geoserver-2.4-SNAPSHOT-png-plugin.zip
   wget http://ares.boundlessgeo.com/geoserver/2.4.x/community-latest/geoserver-2.4-SNAPSHOT-libjpeg-turbo-plugin.zip
   wget http://ares.boundlessgeo.com/geoserver/2.4.x/ext-latest/geoserver-2.4-SNAPSHOT-oracle-plugin.zip
   wget http://ares.boundlessgeo.com/geoserver/2.4.x/ext-latest/geoserver-2.4-SNAPSHOT-css-plugin.zip

Printing plugin:
----------------

Here you'll find the latest printing plugin which is our patched version with various fixes and improvements (more info here https://github.com/geosolutions-it/mapfish-print/wiki/Clustering-Support):

   wget http://demo.geo-solutions.it/share/mapfish-print-2.0/geoserver-2.4-SNAPSHOT-printing-plugin.zip


Setup tomcat base
-----------------

Create catalina base directory for GeoServer::

   cp -a /var/lib/tomcat/base/  /var/lib/tomcat/geoserver
   cd /root/download
   unzip geoserver-2.4.x-latest-war.zip
   unzip geoserver.war -d /var/lib/tomcat/geoserver/webapps/geoserver
   for i in `ls *plugin.zip`; do unzip $i -d plugins; done
   cp -a plugins/*.jar /var/lib/tomcat/geoserver/webapps/geoserver/WEB-INF/lib


Create datadir for GeoServer
----------------------------

   mkdir -p /var/lib/geoserver/config
   mkdir -p /var/lib/geoserver/gwc
   mkdir -p /var/lib/geoserver/data
   mkdir -p /var/lib/geoserver/logs
   mkdir -p /var/lib/geoserver/pdf
   cp -Ra /var/lib/tomcat/geoserver/webapps/geoserver/data/* /var/lib/geoserver/config/
   chown tomcat: /var/lib/geoserver/ -R
   

setenv.sh
---------

Create the file `setenv.sh`. 
We'll set here some system vars used by tomcat, by the JVM, and by the webapp itself::

   vim /var/lib/tomcat/geoserver/bin/setenv.sh

Insert this content::
  
   # Set tomcat vars
   export CATALINA_BASE=/var/lib/tomcat/geoserver
   export CATALINA_HOME=/opt/tomcat/
   export CATALINA_PID=$CATALINA_BASE/work/pidfile.pid
   
   # Configure memory and system stuff
   export JAVA_HOME="/usr/lib/jvm/java-7-openjdk-amd64/jre/"
   export JAVA_PATH="${JAVA_HOME}/bin/"
   export JAVA="${JAVA_PATH}/java"
   export JAVA_OPTS="$JAVA_OPTS -Xms512m -Xmx512m -XX:MaxPermSize=128m"

   # Configure GeoServer
   export JAVA_OPTS="$JAVA_OPTS -DGEOSERVER_DATA_DIR=/var/lib/geoserver/config/"
   export JAVA_OPTS="$JAVA_OPTS -DGEOWEBCACHE_CACHE_DIR=/var/lib/geoserver/gwc/"
   export JAVA_OPTS="$JAVA_OPTS -DGEOSERVER_LOG_LOCATION=/var/lib/geoserver/logs/geoServer1.log"
   export JAVA_OPTS="$JAVA_OPTS -DMAPFISH_PDF_FOLDER=/var/lib/geoserver/pdf/"
   
and make it executable::

   chmod +x /var/lib/tomcat/geoserver/bin/setenv.sh

Edit server.xml
---------------

We need to assign 3 ports to this catalina instance.
We'll use 

- 8008 for commands to catalina instance
- 8083 for the HTTP connections
- 8012 for the AJP connections

Remember that you may change these ports in the file `/var/lib/tomcat/geoserver/conf/server.xml`.

See also :ref:`application_ports`.

Tomcat dir ownership
--------------------

Set the ownership of the ``geoserver/`` related directories to user tomcat ::

   chown tomcat: -R /var/lib/tomcat/geoserver

Automatic startup
-----------------

Download the scripts to manage geoserver here::

   wget https://github.com/geosolutions-it/scripts/archive/master.zip
   unzip master.zip -d /root/download/geoserver/
   cp /root/download/geoserver/scripts-master/tomcat-mngmt/Ubuntu/tomcatRunner /etc/init.d/
   cp /root/download/geoserver/scripts-master/tomcat-mngmt/Ubuntu/geoserver /etc/init.d/
   
Make them executable ::
   
   chmod +x /etc/init.d/tomcatRunner 
   chmod +x /etc/init.d/geoserver

Edit the geoserver script as following::

   ##### In this area you can find settings which are likely to change frequently ####
   # unprivileged user running Tomcat server
   tomcatuser=tomcat
   
   # servicename used as pidfile and lockfile name, must correspond to 'processname:' at the top of this file
   # If not linux will not detect the running service during runlevel switch and will not shut it down normally
   servicename=geoserver

   ##### End of frequent settings area #####
   . /var/lib/tomcat/geoserver/bin/setenv.sh

   #script used to startup tomcat
   tomcatRunner=/etc/init.d/tomcatRunner

   pidfile=${CATALINA_BASE}/$servicename

   lockfile=/var/lock/$servicename
   #runsecure=1 #starts tomcat with java security


and set it as autostarting  ::

   chkconfig --add geoserver

Setting watchdog
----------------

Copy the script and configure it::

   cp /root/download/geoserver/scripts-master/watchdog/watchdog.sh.geoserver /var/lib/tomcat/geoserver/bin/watchdog.sh
   chmod +x /var/lib/tomcat/geoserver/bin/watchdog.sh
   
Edit the watchdog script as following::

   LAYER="sf:roads"
   GSURL="http://127.0.0.1:8083/geoserver"
   WFSURL="$GSURL/wfs?request=GetFeature&typeName=${LAYER}&maxfeatures=1"
   
   #set the connection timeout (in seconds)
   TIMEOUT=30
   
   # seconds to wait to recheck the service (this is not cron)
   # tomcat restart time
   TOMCAT_TIMEOUT=50
   
   FILTER="geoserver"
   
   # the service to restart
   # should be a script placed here:
   # /etc/init.d/
   # 
   SERVICE="geoserver"
   CMD_START="service geoserver stop" 
   CMD_STOP="service geoserver start"
   
   # the output file to use as log
   # must exists otherwise the stdout will be used
   # NOTE: remember to logrotate this file!!!
   LOGFILE="/var/lib/geoserver/logs/watchdog.log.geoserver"
   
Configure httpd
---------------
   
Create the file ``/etc/httpd/conf.d/80-geoserver.conf`` and insert these lines::

   ProxyPass        /geoserver   ajp://localhost:8012/geoserver                                                                                                                                                                                                                           
   ProxyPassReverse /geoserver   ajp://localhost:8012/geoserver

NOTE: on ubuntu you may add this file into /etc/apache2/sites-enabled/

Then reload the configuration for apache httpd::

   service httpd reload
   
==================
Document changelog
==================

+---------+------------+--------+------------------+
| Version | Date       | Author | Notes            |
+=========+============+========+==================+
| 1.0     | 2014-02-26 | Carlo C| Initial revision |
+---------+------------+--------+------------------+
