.. _install_ckan_other:

#####################
Other CKAN extensions
#####################

============
Introduction
============

In this document you'll only find specific information for installing some CKAN official and
unofficial extensions.

.. _extension_tracker:

==============================
Provincia Di Bolzano Extension
==============================

The ckanext-provbz CKAN's extension provide some customizations for the CKAN Look and Feel.
In addition this extension provides an harvester that merge functionalities between two other 
harvesters built on the ckanext-spatial extension like:

- https://github.com/geosolutions-it/ckanext-multilang
- https://github.com/geosolutions-it/ckanext-geonetwork

------------
Requirements
------------

The ckanext-multilang extension has been developed for CKAN 2.4 or later.

------------------------
Development Installation
------------------------

To install ckanext-provbz:

1. Activate your CKAN virtual environment, for example::

     . /usr/lib/ckan/default/bin/activate

2. Go into your CKAN path for extension (like /usr/lib/ckan/default/src)

3. git clone https://github.com/geosolutions-it/ckanext-provbz

4. cd ckanext-provbz

5. python setup.py develop

6. Add ``provbz_theme``  and ``provbz_harvester`` to the ``ckan.plugins`` setting in your CKAN
   config file (by default the config file is located at
   ``/etc/ckan/default/production.ini``).

7. The ckanext-provbz extension provides some updates for the i18n files for 'it' and 'de' languages. 
   Locale files in CKAN (.mo and .po) for these languages must be replaced with files located in this 
   extension at the ckanext-provbz/ckanext/provbz/i18n/ path.

8. Restart CKAN.

====================
GeoNetwork harvester
====================

The GeoNetwork harvester extends the base CSW harvester type, adding some features
as explained in :ref:`geonetwork_harvester_params`, such as:

* handling the ``default_tags`` and ``default_extras`` parameters;
* adding a couple of ``extras`` entries which contain URLs to GeoNetwork info.


In order to install the extension, log in as user ``ckan``, activate the virtual env and check out the extension::

   . /usr/lib/ckan/default/bin/activate
   cd default/src/
   git clone https://github.com/geosolutions-it/ckanext-geonetwork.git
   cd ckanext-geonetwork
   python setup.py develop

Add plugin in ``/etc/ckan/default/production.ini``::

   ckan.plugins = [...] geonetwork_harvester

Restart supervisord::

   systemctl stop supervisord
   systemctl start supervisord

====================
Multilang Extension
====================

The ckanext-multilang CKAN's extension provides a way to localize your CKAN's title and description contents for: 
Dataset, Resources, Organizations and Groups. This extension creates some new DB tables for this purpose containing 
localized contents in base of the configured CKAN's locales in configuration (the production.ini file). So, accessing 
the CKAN's GUI in 'en', for example, the User can create a new Dataset and automatically new localized records for that 
language will be created in the multilang tables. In the same way, changing the GUI's language, from the CKAN's language 
dropdown, the User will be able to edit again the same Dataset in order to specify 'title' and 'description' of the Dataset 
for the new selected language. In this way Dataset's title and description will automatically changed simply switching the 
language from the CKAN's dropdonw.

.. warning:: The ckanext-multilang provides also an harvester built on the ckanext-spatial extension, and inherits all of its functionalities. Currently a forked branch of the stable ckanext-spatial extension is used in order to allow an after import stage functionality (used for the ckanext-multilang persistence):

			 https://github.com/geosolutions-it/ckanext-spatial/tree/stable_official_after_imp_st
			 
			 Installing the ckanext-multilang extension make sure to use this fork and branch of the ckanext-spatial. The update will be ported on the official branch as soon as possible.

In order to install the extension, log in as user ``ckan``, activate the virtual env and check out the extension::

1. Activate your CKAN virtual environment, for example::

     . /usr/lib/ckan/default/bin/activate

2. Go into your CKAN path for extension (like /usr/lib/ckan/default/src)

3. git clone https://github.com/geosolutions-it/ckanext-multilang.git

4. cd ckanext-multilang

5. python setup.py develop

6. Initilize the multilang tables::

	paster --plugin=ckanext-multilang multilangdb initdb --config=/etc/ckan/default/production.ini

7. Add ``multilang`` and ``multilang_harvester`` to the ``ckan.plugins`` setting in your CKAN
   config file (by default the config file is located at
   ``/etc/ckan/default/production.ini``).
   
8. Update the Solr schema.xml file used by CKAN (located at /etc/solr/ckan/conf/) introducing the following elements:
   
   Inside the 'fields' Tag::
   
		<dynamicField name="multilang_localized_*" type="text" indexed="true" stored="true" multiValued="false"/>
   
   as first 'dynamicField'
   
   A new 'copyField' to append::
   
		<copyField source="multilang_localized_*" dest="text"/>

9. Restart Solr.

10. Restart CKAN.

.. warning:: Make sure that the final order of the plugins list into the CKAN's configuration (production.ini file) is the folowing::

				ckan.plugins = shibboleth datastore harvest ckan_harvester provbz_theme spatial_metadata spatial_query csw_harvester geonetwork_harvester stats text_view image_view recline_view multilang multilang_harvester provbz_harvester


==================
Document changelog
==================

+---------+------------+--------+------------------+
| Version | Date       | Author | Notes            |
+=========+============+========+==================+
| 1.0     |            |        | Initial revision |
+---------+------------+--------+------------------+
