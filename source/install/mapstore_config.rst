.. _config_mapstore:

####################
Configuring MapStore
####################

.. hint::
   Ref info page at https://github.com/geosolutions-it/mapstore/wiki/mapStoreConfig-File

============
Introduction
============

In this document you'll find specific information for configuring Map and background layers in MapStore. 

It is expected that CKAN and MapStore have already been properly installed and configured as described 
in :ref:`install_ckan` and in :ref:`install_mapstore`.

MapStore uses a different template (and a related configuration) for each WebGIS mode:

- ``composer.html`` is the template used for the advanced viewer 
  (its configuration is inside the ``mapStoreConfig.js`` file)::

		$ vim /var/lib/tomcat/webapps/mapstore/WEB-INF/app/templates/composer.html
		$ vim /var/lib/tomcat/webapps/mapstore/WEB-INF/app/static/config/mapStoreConfig.js

- ``viewer.html`` is the template used for the simple viewer 
  (its configuration is inside the ``viewer.js`` file)::

		$ vim /var/lib/tomcat/webapps/mapstore/WEB-INF/app/templates/viewer.html
		$ vim /var/lib/tomcat/webapps/mapstore/WEB-INF/app/static/config/viewer.js
		
- ``embedded.html`` is the template used for the embedded preview in CKAN 
  (its configuration is inside the ``preview.js`` file)::

		$ vim /var/lib/tomcat/webapps/mapstore/WEB-INF/app/templates/embedded.html
		$ vim /var/lib/tomcat/webapps/mapstore/WEB-INF/app/static/config/preview.js
		
.. note:: The templates used for the CKAN integration are: ``composer.html`` and ``embedded.html``

==============================
Setting the Base configuration
==============================

.. warning:: Configurations in this section are mandatory.

Before running MapStore with the 'ckanext-mapstore' extension some mandatory configuration refinement 
must be provided  to the standard one:

1) Open the ``mapStoreConfig.js`` file::

	$ vim /var/lib/tomcat/webapps/mapstore/WEB-INF/app/static/config/mapStoreConfig.js
	
2) Find the ``gxp_addlayer`` plugin configuration and and changes properties using the values below::

		...
		{
			"ptype": "gxp_addlayer",
			"showCapabilitiesGrid": false,
			"useEvents": true,
			"showReport": "never",
			"directAddLayer": false,
			"id": "addlayer"
		}
		...
		
* ``ptype``: The type name of the plugin.
* ``showCapabilitiesGrid``: Default false. Set to true if you want to automatically show the source dialog after loading the WMS service if any layer (or wrong layer name) has been specified in the original URL.
* ``useEvents``: Default to false. Use this to true for import operations.
* ``showReport``: Default to 'never'. possible values are: 'errors' (shows a repot dialog of imported resorces only if errors occurs), 'never' (never visualize the report), 'always' (always visualize the report).
* ``directAddLayer``: Default to false. Set to true if you want to dyrectly add the layers on the map instead loading the WMS source before.
* ``id``: The plugin identifier.


3) Append the following plugin configuration to the ``customTools`` property::

		...
		{
			"ptype": "gxp_resourcestatus",
			"id": "resourcetree_plugin",
			"outputConfig": {
				"id": "resourcetree"
			},
			"outputTarget": "west",
			"expandServices": true,
			"showAllLayers": false	
		}
		...
		
		
.. note:: This is the plugin that allows the visualization of the imported resources from CKAN in a tree tool. 
	
* ``ptype``: The type name of the plugin.
* ``id``: The plugin's identifier.
* ``outputConfig``: The output configuration options for the plugin's GUI.
* ``outputTarget``: The target element in which render the plugin.
* ``expandServices``: Default false. Use it to true in order to expand WSM services showing his layers.
* ``showAllLayers``: Default false. Set this to true if you want to visualize all imported layers (also for imported services) in the map. Is suggested to maintain this to false if there is a large ammount of layers to show. 
	
4) Then open the ``preview.js`` file::

	$ vim /var/lib/tomcat/webapps/mapstore/WEB-INF/app/static/config/preview.js
	
5) Find the ``gxp_addlayer`` plugin configuration and changes properties using the values below::
			
		...
		{
			"ptype": "gxp_addlayer",
			"showCapabilitiesGrid": true,
			"useEvents": true,
			"showReport": "errors",
			"directAddLayer": false,
			"id": "addlayer"
		}
		...



