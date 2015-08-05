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



==================
Document changelog
==================

+---------+------------+--------+------------------+
| Version | Date       | Author | Notes            |
+=========+============+========+==================+
| 1.0     |            |        | Initial revision |
+---------+------------+--------+------------------+
