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
			"showCapabilitiesGrid": true,
			"useEvents": true,
			"showReport": false,
			"directAddLayer": false,
			"id": "addlayer"
		}
		...

3) Append the following plugin configuration to the ``customTools`` property::

		...
		{
			"ptype": "gxp_resourcestatus",
			"id": "resourcetree_plugin",
			"outputConfig": {
				"id": "resourcetree"
			},
			"outputTarget": "west"
		}
		...
		
	.. note:: This is the plugin that allows the visualization of the imported resources from CKAN in a tree tool. 
		
4) Then open the ``preview.js`` file::

	$ vim /var/lib/tomcat/webapps/mapstore/WEB-INF/app/static/config/preview.js
	
5) Find the ``gxp_addlayer`` plugin configuration and changes properties using the values below::
			
		...
		{
			"ptype": "gxp_addlayer",
			"showCapabilitiesGrid": true,
			"useEvents": true,
			"showReport": true,
			"directAddLayer": false,
			"id": "addlayer"
		}
		...

===================================
Setting the MapStore Map Projection
===================================

.. note:: Optional configuration.

In order to change or configure a Projection in MapStore one or more of the configuration files described 
above should be modified.

The Projection information is a part of the Map configuration, as an instance::

	"map": {
		"projection": "EPSG:900913",
		"units": "m",
		"center": [1250000.000000, 5370000.000000],
		"zoom":5,
		"maxExtent": [
			-20037508.34, -20037508.34,
			20037508.34, 20037508.34
		],
		...
		
Changes to do in this case depends of which SRS you want to use. For example, in order to switch to the EPSG:4326,
main configuration properties to change are::

	...
	"projection": "EPSG:900913",
	"units": "m",
	"center": [1250000.000000, 5370000.000000],
	"maxExtent": [
		-20037508.34, -20037508.34,
		20037508.34, 20037508.34
	],
	...
	
So we will have for example::

	...
	"projection": "EPSG:4326",
	"units": "degrees",
	"center": [10.37201, 42.59248],
	"maxExtent": [
		-180, -90,
		180, 90
	],
	...

.. note:: If your projection is EPSG:4326 or EPSG:900913, the ``maxExtent`` property can be omitted because these SRS are 
		  supported by default by OpenLayers. If the intent is to use a different SRS from EPSG:4326 and EPSG:900913,
		  the ``maxExtent`` is a mandatory configuration.
		  
======================================
Setting the MapStore Background Layers
======================================

.. note:: Optional configuration.

In order to change or configure the backgrounds layers in MapStore one or more of the configuration files described 
above should be modified. 
In order to manage backgrounds you have to consider that:

1) A background is a layer so he necessarily need a WMS source like the other overlay.
2) A background configuration must have the ``group`` property set to ``background``.

In order to configure a new background you have to follow the steps below (it is just an example).

- Add the related WMS source of the background layer::

		...
		"geosolutions": {
			"ptype": "gxp_wmssource",
			"url": "http://demo1.geo-solutions.it/geoserver-enterprise/ows",
			"title": "GeoSolutions GeoServer",
			"version":"1.1.1",
			"layerBaseParams":{
				"FORMAT": "image/png8",
				"TILED": true
			}
		},
		...

- Add the background layers configuration to the ``layers`` property::

		...
		{
			"source": "geosolutions",
			"title": "GeoSulutions Shaded",
			"name": "GeoSolutions:ne_shaded",
			"group": "background"
		}
		...

Now youe background will be added to the background layers list inside MapStore.

.. note:: You can add a background with a native SRS different from the MapStore Map Projection. In this case the WMS server 
          will reproject the background.
		  
MapStore allow the possibility to add an empty background to the map. In this case you have to add the configuration below
to the 'layers' property::

		...
		{
			"source": "ol",
			"title": "Vuoto",
			"group": "background",
			"fixed": true,
			"type": "OpenLayers.Layer",
			"visibility": false,
			"args": [
				"None", {"visibility": false}
			]
		}
		...

Below a complete example with the complete Map's configuration section as described in steps above::

		{			   
		   "advancedScaleOverlay": false,
		   "gsSources":{ 
				"geosolutions": {
					"ptype": "gxp_wmssource",
					"url": "http://demo1.geo-solutions.it/geoserver-enterprise/ows",
					"title": "GeoSolutions GeoServer",
					"version":"1.1.1",
					"layerBaseParams":{
						"FORMAT": "image/png8",
						"TILED": true
					}
				},
				"mapquest": {
					"ptype": "gxp_mapquestsource"
				}, 
				"osm": { 
					"ptype": "gxp_osmsource"
				},
				"google": {
					"ptype": "gxp_googlesource" 
				},
				"bing": {
					"ptype": "gxp_bingsource" 
				}, 
				"ol": { 
					"ptype": "gxp_olsource" 
				}
			},
			"map": {
				"projection": "EPSG:900913",
				"units": "m",
				"center": [1250000.000000, 5370000.000000],
				"zoom":5,
				"maxExtent": [
					-20037508.34, -20037508.34,
					20037508.34, 20037508.34
				],
				"layers": [
					{
						"source": "google",
						"title": "Google Roadmap",
						"name": "ROADMAP",
						"group": "background"
					},{
						"source": "google",
						"title": "Google Terrain",
						"name": "TERRAIN",
						"group": "background"
					},{
						"source": "google",
						"title": "Google Hybrid",
						"name": "HYBRID",
						"group": "background"
					},{
						"source": "mapquest",
						"title": "MapQuest OpenStreetMap",
						"name": "osm",
						"group": "background"
					},{
						"source": "osm",
						"title": "Open Street Map",
						"name": "mapnik",
						"group": "background"
					},{
						"source": "bing",
						"title": "Bing Aerial",
						"name": "Aerial",
						"group": "background"
					},{
						"source": "bing",
						"title": "Bing Aerial With Labels",
						"name": "AerialWithLabels",
						"group": "background"
					},{
						"source": "geosolutions",
						"title": "Shaded",
						"name": "GeoSolutions:ne_shaded",
						"group": "background"
					},{
						"source": "ol",
						"title": "Vuoto",
						"group": "background",
						"fixed": true,
						"type": "OpenLayers.Layer",
						"visibility": false,
						"args": [
							"None", {"visibility": false}
						]
					}
				]
			}
		}			
		...
			


