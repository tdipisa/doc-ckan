.. _install_mapstore_ext:

######################################
Installing MapStore extension for CKAN
######################################

.. hint::
   Ref info page at https://github.com/geosolutions-it/ckanext-mapstore/wiki#wiki-installation

============
Introduction
============

In this document you'll only find specific information for installing the MapStore plugin 
for CKAN. 

It is expected that CKAN and MapStore have already been properly installed and configured as described 
in :ref:`install_ckan` and in :ref:`install_mapstore`.


==================
MapStore extension
==================

In order to install the ckanext-mapstore, copy (or clone using git) the ``ckanext-mapstore/`` directory inside 
the CKAN src directory (i.e. ``ckan/default/src``).

Before using the plugin, the extension must installed into the CKAN virtual environment.

As user ``ckan``::

   $ . /usr/lib/ckan/default/bin/activate
   (default)$ cd /usr/lib/ckan/default/src
   (default)$ git clone https://github.com/geosolutions-it/ckanext-mapstore.git
   (default)$ cd ckanext-mapstore
   (default)$ python setup.py develop

Once done, you can add the plugins that the MapStore extension provides.

There are 2 provided plugins:

* ``mapstore_preview``: enables preview of WMS layers or MapStore context inside CKAN resource pages
* ``geostore_harvest``: enables the harvesting of map contextes from MapStore (GeoStore is the backend used by MapStore) 

To enable both of them, edit file ``/etc/ckan/default/production.ini`` and add the plugins::  

   ckan.plugins = [...] mapstore_preview geostore_harvester
   
In next sections there are some more info on the needed configuration for both plugins.


==================
WMS preview plugin
==================

Enable the MapStore preview plugin by adding ``mapstore_preview`` to the plugin list 
in  ``/etc/ckan/default/production.ini``::  

   ckan.plugins = [...] mapstore_preview

Then you have to enable the custom MapStore CSS in ``/etc/ckan/default/production.ini``::

   ckan.template_head_end = <link rel="stylesheet" href="/css/mapstore.css" type="text/css">

In general, the preview through the MapStore WebGIS is managed in CKAN with an HTML iFrame; 
So the connections parameters to the MapStore instance must be specified. 

Please check/modify these values inside the ``preview_config.js`` file::

   $ vim /usr/lib/ckan/default/src/ckanext-mapstore/ckanext/mapstore/theme/public/preview_config.js

Below the configurations parameters to use::

   var preview_config = {
      viewerConfigName: "preview",
      viewerPath: "/viewer",
      composerPath: "/composer",
      mapStoreBaseURL: "http://84.33.2.79/mapstore"
   }

* ``viewerConfigName``: the MapStore WebGIS configuration to use (see 
  the `MapStore WIKI <https://github.com/geosolutions-it/mapstore/wiki/mapStoreConfig-File>`_ for more details).
* ``viewerPath``: the relative URL of the MapStore viewer (used for the basic preview inside the CKAN resource page).
* ``composerPath``: the relative URL of the MapStore advanced viewer (used inside the CKAN preview page in order to open the advanced MapStore viewer in a separate browser page). 
* ``mapStoreBaseURL``: the MapStore base URL.


=======================
MapStore harvest plugin
=======================




==================
Document changelog
==================

+---------+------------+--------+------------------+
| Version | Date       | Author | Notes            |
+=========+============+========+==================+
| 1.0     | 2014-02-11 | ETj    | Initial revision |
+---------+------------+--------+------------------+
