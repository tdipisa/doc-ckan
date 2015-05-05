.. _install_ckan_other:

#####################
Other CKAN extensions
#####################

============
Introduction
============

In this document you'll only find specific information for installing some CKAN unofficial extension
provided for LaMMA.

.. _extension_tracker:

   
===================
LaMMA UI  Extension
===================

This is a Ckan's theme extension for LaMMA. The plugin inlcudes all customizzations provided for teh look & feel 
and search funzionalities based on 'metadata-date' dataset field.

.. hint::
   Repo page at https://github.com/geosolutions-it/ckanext-lamma.git

Check out the extension::

   . /usr/lib/ckan/default/bin/activate
   cd default/src/
   pip install -e git+https://github.com/geosolutions-it/ckanext-lamma.git#egg=ckanext-lamma

==================
Document changelog
==================

+---------+------------+--------+------------------+
| Version | Date       | Author | Notes            |
+=========+============+========+==================+
| 1.0     | 2014-02-14 | ETj    | Initial revision |
+---------+------------+--------+------------------+
| 1.0     | 2015-04-05 | TDP    | LaMMA revision   |
+---------+------------+--------+------------------+
