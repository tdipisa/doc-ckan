.. _install_gn:

##########################
Installing GeoNetwork 2.10
##########################

============
Introduction
============

In this document you'll only find specific information for installing GeoNetwork.

It is expected that the base system has already been properly installed and configured as described in :ref:`setup_centos`.

In such document there are information about how to install some required base components, such as PostgreSQL, 
Apache HTTPD, Oracle Java, Apache Tomcat.

=====================
Installing GeoNetwork
=====================

.. hint::
   GeoNetwork project page at http://geonetwork-opensource.org/
      

Download packages
-----------------

Download the `.war` files needed for a full GeoNetwork installation::

   cd /root/download
   wget http://garr.dl.sourceforge.net/project/geonetwork/GeoNetwork_opensource/v2.10.3/geonetwork.war
   wget http://84.33.2.27/download/iso19139.rndt.zip 

Setup tomcat base
-----------------

Create catalina base directory for MapStore::

   cp -a /var/lib/tomcat/base/       /var/lib/tomcat/geonetwork
   cp /root/download/geonetwork.war  /var/lib/tomcat/geonetwork/webapps/


Create user and DB for GeoNetwork
---------------------------------

Create a PostgreSQL DB for GeoNetwork::

   su - postgres -c "createuser -S -D -R -P -l geonetwork"

Annotate the user password.   
   
Create the DB::
   
   su - postgres -c "createdb -O geonetwork geonetwork -E utf-8"

Add the spatial extension to the ``geonetwork`` DB::

   # su - postgres -c "psql geonetwork"
   geonetwork=# CREATE EXTENSION postgis;
   geonetwork=# GRANT ALL PRIVILEGES ON DATABASE geonetwork TO geonetwork;
   geonetwork=# GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO geonetwork;

Create GN data dir
------------------

Some GN dirs can be externalized.

We'll put such dirs in ``/var/lib/tomcat/geonetwork/gn``, in this structure::

    gn
    ├── data
    │   ├── metadata_data
    │   ├── metadata_subversion
    │   └── resources
    │       └── images
    │           └── logos
    ├── index
    │   └── nonspatial
    └── upload


Create the directory hierarchy::

   cd /var/lib/tomcat/geonetwork/
   mkdir -p gn/data/{metadata_subversion,metadata_data}
   mkdir -p gn/data/resources/images/logos/
   mkdir -p gn/data/upload
   mkdir -p gn/index/nonspatial/

Create the override file:: 

   vim /var/lib/tomcat/geonetwork/gn/config-overrides.xml

and insert :download:`this content <../resources/gn-config-overrides.xml>`.

You will have to customize at least:

* the ``site.host`` element, setting the IP address or the server host name;
* the password for the geonetwork DB

You may also want to customize:

* the site name
* the bounding box and the layers for the search map.
  Please note that there are 2 sets of map definition:

  * ``<mapSearch>`` is about the search map 
  * ``<mapViewer>`` is about the preview map

setenv.sh
---------

Create the file ``setenv.sh``. 
We'll set here some system vars used by tomcat, by the JVM, and by the webapp itself::

   vim /var/lib/tomcat/geonetwork/bin/setenv.sh

Insert this content::

   # Set tomcat vars
   export CATALINA_BASE=/var/lib/tomcat/geonetwork
   export CATALINA_HOME=/opt/tomcat/  
   export CATALINA_PID=$CATALINA_BASE/work/pidfile.pid
  
   # Configure memory and system stuff   
   export JAVA_OPTS="$JAVA_OPTS -Xms1024m -Xmx2048m -XX:MaxPermSize=512m"
   export JAVA_OPTS="$JAVA_OPTS -Dorg.apache.lucene.commitLockTimeout=60000"

   # Configure GeoNetwork  
   export GN_EXT_DIR=$CATALINA_BASE/gn

   # Configure override file  
   export GN_OVR_PROPNAME=geonetwork.jeeves.configuration.overrides.file
   export GN_OVR_FILE=$GN_EXT_DIR/config-overrides.xml 
   export JAVA_OPTS="$JAVA_OPTS -D$GN_OVR_PROPNAME=$GN_OVR_FILE"
  
   #export JAVA_OPTS="$JAVA_OPTS -Dgeonetwork.dir=$GN_DATA_DIR"
  
   # Configure data dirs
   export GN_CTX=geonetwork.  
   export JAVA_OPTS="$JAVA_OPTS -D${GN_CTX}data.dir=$GN_EXT_DIR/data/metadata_data"
   export JAVA_OPTS="$JAVA_OPTS -D${GN_CTX}resources.dir=$GN_EXT_DIR/data/resources"
   export JAVA_OPTS="$JAVA_OPTS -D${GN_CTX}svn.dir=$GN_EXT_DIR/data/metadata_subversion"
   export JAVA_OPTS="$JAVA_OPTS -D${GN_CTX}lucene.dir=$GN_EXT_DIR/index"
   
and make it executable::

   chmod +x /var/lib/tomcat/geonetwork/bin/setenv.sh


Edit server.xml
---------------

We need to assign 3 ports to this catalina instance.

Edit file ::

   vim /var/lib/tomcat/geonetwork/conf/server.xml

and change the connection ports in this way: 

- 8007 for commands to catalina instance
- 8082 for the HTTP connections
- 8011 for the AJP connections

See also :ref:`application_ports`.

Tomcat dir ownership
--------------------

Set the ownership of the ``geonetwork/`` related directories to user tomcat ::

   chown tomcat: -R /var/lib/tomcat/geonetwork
 

Automatic startup
-----------------

Create the file ``/etc/init.d/geonetwork`` and insert :download:`this content <../resources/geonetwork>`.

Once downloaded, make it executable ::

   chmod +x /etc/init.d/geonetwork

and set it as autostarting  ::

   chkconfig --add geonetwork

.. note::    
   If using Ubuntu, you have to use this command instead::
  
      update-rc.d geonetwork start 90 2 3 4 5 . stop 10 0 1 6 .
      
   
Configure httpd
---------------
   
Create the file ``/etc/httpd/conf.d/80-geonetwork.conf`` and insert these lines::

   ProxyPass        /geonetwork   ajp://localhost:8011/geonetwork                                                                                                                                                                                                                           
   ProxyPassReverse /geonetwork   ajp://localhost:8011/geonetwork

.. note::    
   If using Ubuntu, you have to put these lines in file ::
   
      vim /etc/apache2/sites-available/ckan 
      
   just before the ``ProxyPass`` directive redirecting the ``/``.    


Then reload the configuration for apache httpd::

   service httpd reload


=============
Further setup
=============

Once GeoNetwork is up and running, you have to perform some other steps using the web interface.

Login as ``admin`` / ``admin``.

Change the admin pw
-------------------

Go to "Administration" >  "Users and groups" >  "Change password".
Change and annotate the new password.

Check the system configuration
------------------------------

Go to "Administration" >  "Catalogue settings" >  "System configuration".

Check if the right values are set in these fields:

* Site name
* Site organization
* Host
* Port (you may want to put ``80`` here) 

You then may want to:

* Disable Z39.50 server
* Enable search statistics
* Enable INSPIRE
* Enable INSPIRE view
* Setup the CSW server info

Default language
----------------

The only way to change de default UI language is to edit the index.html file::

   vim webapps/geonetwork/index.html
   
The default language is set as a 3 letters ISO code in this line::
   
   window.location="srv/eng/home" + search;
   
so you may for instance change the string to ``srv/ita/home`` to have Italian as default language. 

=========================
Installing schema plugins
=========================

You may want to add additional schemas to GeoNetwork.

For instance let's add the RNDT profile.

You need the zip file containing the definition of the schema, or a URL pointing to such file.

Go to "Administration" >  "Metadata & Template" >  "Add a metadata schema/profile".

Set as schema name ``iso19139.rndt``

Then check the "URL of Schema Zip Archive" option, and set ``http://84.33.2.27/download/iso19139.rndt.zip``.

Then press the "Add" button.

Now in "Administration" you can "Add templates" and "Add sample metadata" for the new schema.

============
KNOWN ISSUES
============

* site name and site URL set in the override file are not put in the DB during the initialization, 
  so a manual setup in the configuration page is required. 


==================================
FOLLOWING TEXT IS WORK IN PROGRESS
==================================


Notare che, tra le customizzazione effettuate, c'è anche la posizione dei file di log: 
la prima esecuzione creerà dei file di log in ``/home/tomcat/logs/geonetwork.log`` mentre, 
una volta eseguita la customizzazione, i nuovi file saranno creati in ``/var/lib/tomcat/geonetwork/logs/``.

Se si desidera effettuare il patch di GN prima di lanciare il servizio, per evitare di passare 
per stati intermedi con la creazione di file di log temporanei:

- Espandere il file war manualmente ::

   cd /var/lib/tomcat/geonetwork/webapps/
   mkdir geonetwork
   cd geonetwork
   jar xvf /root/geonetwork-main-2.8.0.war

- quindi espandere il file ``webapp_geonetwork.tgz`` come specificato precedentemente.

Configurazione file di log
--------------------------

È possibile modificare le impostazioni di log nel file ``WEB-INF/log4j.cfg``.

Questo file viene già ridefinito nell'espansione di ``webapp_geonetwork.tgz``.
I valori così impostati dovrebbero essere corretti; controllare in ogni caso la posizione del file di log:
la riga dovrebbe presentarsi così::

   log4j.appender.jeeves.file = ${catalina.base}/logs/geonetwork.log

Fare particolare attenzione a che appaia ``${catalina.base}``. Il file di log dovrebbe in questo modo 
essere creato nella directory ``/var/lib/tomcat/geonetwork/logs/``.


.. _gn_web_config:


Logo
----

È possibile personalizzare il logo del sito dalla schermata di amministrazione.

Nel gruppo di opzioni "Configurazione del catalogo", selezionare "Configurazione del logo".
Caricare l’immagine che si vuole usare come logo. Una volta caricata, selezionarla e cliccare su "Usa per il catalogo".

Nelle versioni precedenti di GN si poteva usare solo la procedura non interattiva:
Si doveva individuare l'UUID del sito (dalla pagina di informazioni -- link "Info" sulla toolbar). 
Dopodiché si doveva copiare l'immagine gif del logo desiderato all'interno della directory ``images/logos``, 
con il nome ``UUID_del_sito.gif``.

==============================
Riconfigurazione su VM clonate
==============================

Per informazioni sulla clonazione di VM, seguire le istruzioni riportate sul documento :ref:`cerco_cloning_vm`.

I paragrafi seguenti mostrano solo come riconfigurare GeoNetwork su una VM clonata e riconfigurata correttamente.

Configurazioni di GeoNetwork da modificare
------------------------------------------

Configurazioni su file
``````````````````````

Layer per la mappa di ricerca
_____________________________

Modificare WMS server, layer e bounding box come descritto nella sezione di configurazione di GeoNetwork.

Configurazioni da webapp / DB
`````````````````````````````

Le configurazioni su DB includono anche la generazione automatica dell’UUID del sito.

Per essere sicuri di avere dati completamente puliti, conviene eliminare il DB esistente e crearne uno nuovo.

Fermare l'istanza di GeoNetwork
_______________________________

Poiché questa è un clone completo della macchina di partenza, GeoNetwork è impostato per partire automaticamente, 
ed andrà quindi fermato ::

   service geonetwork stop

Eliminare il DB
_______________
Da utente postgres ::

   dropdb geonetwork
   
Ricreare il DB
______________
Seguire i passi indicati nella sezione :ref:`gn_create_db`::

   createdb -O geonetwork  geonetwork
   psql -W -U geonetwork -d geonetwork -c "CREATE EXTENSION postgis;"
   psql -W -U geonetwork -d geonetwork -c "CREATE EXTENSION postgis_topology;"
   
Rilanciare GeoNetwork
_____________________
::

    service geonetwork start
    
Impostazioni
____________

Seguire i passi nella sezione :ref:`gn_web_config`.



========
Versioni
========

+----------+------------+--------+-------------------+
| Versione | Data       | Autore | Note              |
+==========+============+========+===================+
| 1.0      | 2014-02-25 | ETj    | Versione iniziale |
+----------+------------+--------+-------------------+
