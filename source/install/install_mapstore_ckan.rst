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

.. _extension_mapstore:

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

   $ vim /usr/lib/ckan/default/src/ckanext-mapstore/ckanext/mapstore/preview/preview_config.js
   
Below the configurations parameters to use::

   var preview_config = {
		viewerConfigName: "preview",
		viewerPath: "/viewer",
		composerPath: "/composer",
		mapStoreBaseURL: "http://84.33.2.79/mapstore"
		basketStatus: true,
		storageMethod: "sessionstorage"
   }

* ``viewerConfigName``: the MapStore WebGIS configuration to use (see 
  the `MapStore WIKI <https://github.com/geosolutions-it/mapstore/wiki/mapStoreConfig-File>`_ for more details).
* ``viewerPath``: the relative URL of the MapStore viewer (used for the basic preview inside the CKAN resource page).
* ``composerPath``: the relative URL of the MapStore advanced viewer (used inside the CKAN preview page in order to open the advanced MapStore viewer in a separate browser page). 
* ``mapStoreBaseURL``: the MapStore base URL.
* ``basketStatus``: Set to true if you want to see the icon related to the WMS status inside the Shopping Cart items (the icon show the status of teh WMS resource after the harves procedure).
* ``storageMethod``: the storage method to use in order to store usage information about the Shopping Cart status. Valid values are: 'localstorage', 'sessionstorage' or 'cookies'.

.. note::

		 **localstorage**: The localStorage object stores the data with no expiration date. The data will not be deleted when the browser is closed, and will be available the next day, week, or year.
		 
		 **sessionstorage**: The sessionStorage object is equal to the localStorage object, except that it stores the data for only one session. The data is deleted when the user closes the browser window.
		 
		 **cookies**: Use the standar cookie behavior.  

==========================================
Enable the basket for multiple WMS preview
==========================================

With the MapStore CKAN extension you have also the possibility to enable a basket component that allows 
to select multiple WMS resource for a preview. This extension provides also a separate button inside the dataset list items 
in order to visualize, directly from the dataset list page, a preview on Map of the dataset WMS resource. 

   .. figure:: img/basket_overview.jpeg
      :width: 600
 		  
      The basket component

The basket control uses a template snippets that you have to enable in order to use it.
In order to enable this component you need to follow the steps below:

* Add the basket snippet to the relevant template package inside the 'block secondary_content' element::

	$ vim /usr/lib/ckan/default/src/ckanext-mapstore/ckanext/mapstore/templates/package/search.html

  for example::
  
		{% block secondary_content %}
		  {% snippet 'snippets/organization.html', organization=c.group_dict, show_nums=true %}

		  {% snippet "snippets/mapstore_basket.html" %}
		  
		  {% block organization_facets %}{% endblock %}
		{% endblock %}
  
  and:: 	
	
	$ vim /usr/lib/ckan/default/src/ckanext-mapstore/ckanext/mapstore/templates/organization/read_base.html
	
  for example::
  
		{% block secondary_content %}
		  {% snippet 'snippets/organization.html', organization=c.group_dict, show_nums=true %}

		  {% snippet "snippets/mapstore_basket.html" %}
		  
		  {% block organization_facets %}{% endblock %}
		{% endblock %}

* Then you have to edit the 'package_item' CKAN template::

	$ vim /usr/lib/ckan/default/src/ckanext-mapstore/ckanext/mapstore/template/snippets/package_item.html

  adding the fragment below at the end of the container block::

	...
	
	<!-- -------------------------------------------------------------------------- -->
	<!-- New elements for the MapStore extension: The control of the basket. -->
	<!-- -------------------------------------------------------------------------- -->

	{% if package.resources and not hide_resources %}
	  <ul class="dataset-resources unstyled" style="float: right; display: inline-block;">
		{% set index = 0 %}
		{% for id in h.dict_list_reduce(package.resources, 'id') %}	
		   
			{% set format = package.resources[index].format %}	
						
			{% if format == 'wms' or format == 'mapstore' %}
				
				{% set url = package.resources[index].url %}
				{% set name = package.resources[index].name %}				
				
				{% if format == 'wms'%}
					<li>
						<a id="cart-{{ id }}" onClick="javascript:basket_utils.prepareKeyForBasket(this.id, &#34;{{url}}&#34;, &#34;{{name}}&#34;, &#34;{{format}}&#34;);" class="label basket-label-cart"><i class="icon-shopping-cart"></i><spam> Add to Cart</spam></a>
					</li>
				{% endif %}
				
				<li>
					<a id="{{ id }}" onClick="javascript:basket_utils.preparePreviewURL(&#34;{{ id }}&#34;, &#34;{{url}}&#34;, &#34;{{name}}&#34;, &#34;{{format}}&#34;);" class="label basket-label-preview"><i class="icon-map-marker"></i><spam> Preview on Map</spam></a>
				</li>

			{% endif %}

			{% set index = index + 1 %}
			
		{% endfor %}
	  </ul>
	{% endif %}
	
	<!-- -------------------------------------------------------------------------- -->

	{% endblock %}
	
=======================
MapStore harvest plugin
=======================


Should you use the mapstore harvester, you need to add the ``harvester`` sysadmin
in order to comply with some CKAN internal handling::
   
   paster --plugin=ckan sysadmin add harvest -c /etc/ckan/default/production.ini

