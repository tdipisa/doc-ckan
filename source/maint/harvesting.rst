.. _ckan_harvesting:

########################
Harvesting configuration
########################

Adding an harvesting source
===========================

In order to add an harvesting source, you need to log into CKAN web interface as an administrator.
You have to type the address for the harvesting page, since there is no direct link for it ::

   http://YOUR_SITE/harvest


.. figure:: /maint/img/harvest1.png
   :width: 600
   :align: center

Press the "Add Harvest Source" button and this page will open:

.. figure:: /maint/img/harvest2.png
   :width: 600
   :align: center

In order to add properly a CSW source you'll have to set:

* The CSW URL endpoint
* A title/name for this source, for your reference
* The harvest type, i.e. "PBZ CSW Server" in "Source type"
* The update frequency
* The configuration in JSON format.
* The owner Organization: all the dataset harvested from this source will be assigned to that Organization.

The JSON configuration allows these parameters:

* ``cql``: a CQL filter for harvesting only a subset of the records in the remote node
* ``default_tags``: all dataset harvested from this source will have these tags attached.
* ``default_extras``: all dataset harvested from this source will have these key/value pairs attached.
  The value may have tokens in the form ``{token}``, where the allowed tokens are:

  * ``harvest_source_url``
  * ``harvest_source_title``
  * ``harvest_job_id``
  * ``harvest_object_id``

* ``version``: geonetwork version. Currently only 2.6, and 2.10 are accepted. This is needed to find out which services are available on the GeoNetowrk instance. For instance, in order to extract the categories associated to a metadata in version 2.6, the only way to find it is to get a MEF of the metadata and to parse the info.xml file packed into it.
  
* ``private_datasets``:  If True sets as private the dataset. A dataset can be private if belongs to an organization Furthermore these other parameters will be read:

* ``group_mapping``:  it's a dict that associates category names to group names. If this parameter is defined, GN will be queried about the categories of the metadata, a matching group for each category will be searched in the provided mapping and will then be associated to the metadata if one or more are found

* ``harvest_iso_categories``: requires 'group_mapping' defined. If True allow to harvest metadata using ISO categories instead GeoNetwork internal categories.

* ``ckan_locales_mapping``: provides a mapping between CKAN locales and localed in CSW (used for the gmd:LocalisedCharacterString element).

* ``default_license``: with this property you can specify the default license to use for the CKAN's dataset if none useLimitation has been found into the metadata. Below an example:
  
.. note::
   *cql filtering* has been added with `this commit <https://github.com/ckan/ckanext-spatial/commit/55497f037e5add55f5890315e9c7c4f396cc49ac>`_.

.. note::
   ``default_tags`` and ``default_extras`` will be available only if, when installing ckanext-spatial, these commits
   have been included manually::

      https://github.com/ckan/ckanext-spatial/pull/58

Below the configuration used for the Provincia di Bolzano Harvester::

		{
			"private_datasets": "False", 
			"version": "2.6", 
			"harvest_iso_categories": "True",
			"group_mapping": {
				"farming": "farming", 
				"utilitiesCommunication": "boundaries", 
				"transportation": "boundaries", 
				"inlandWaters": "environment", 
				"geoscientificInformation": "geoscientificinformation", 
				"environment": "environment", 
				"climatologyMeteorologyAtmosphere": "climatologymeteorologyatmosphere", 
				"planningCadastre": "boundaries", 
				"imageryBaseMapsEarthCover": "boundaries", 
				"elevation": "boundaries", 
				"boundaries": "boundaries",
				"structure": "boundaries", 
				"location": "boundaries", 
				"economy": "economy",
				"society": "economy",
				"biota": "environment",
				"intelligenceMilitary": "boundaries",
				"oceans": "environment",
				"health": "health"
			},
			"ckan_locales_mapping":{
				"ita": "it",
				"ger": "de"
			},
			"default_license": "cc-zero"
		}

