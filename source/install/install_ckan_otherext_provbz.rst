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

5. python setup.py install

6. Add ``provbz_theme``  and ``provbz_harvester`` to the ``ckan.plugins`` setting in your CKAN
   config file (by default the config file is located at
   ``/etc/ckan/default/production.ini``).

7. The ckanext-provbz extension provides some updates for the i18n files for 'it' and 'de' languages. 
   Locale files in CKAN (.mo and .po) for these languages must be replaced with files located in this 
   extension at the ckanext-provbz/ckanext/provbz/i18n/ path.

8. Update the production.ini configuration finding the default property ``licenses_group_url`` and change the value:

licenses_group_url = file:///usr/lib/ckan/default/src/ckanext-provbz/ckanext/provbz/licenses/ckan.json

9. Restart CKAN.

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
   python setup.py install

Add plugin in ``/etc/ckan/default/production.ini``::

   ckan.plugins = [...] geonetwork_harvester

Restart supervisord::

   systemctl stop supervisord
   systemctl start supervisord

===================
Multilang Extension
===================

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

5. python setup.py install

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


====================
Shibboleth Extension
====================

The Shibboleth plugin will allow users to log in into CKAN using an existing Shibboleh environment.  

.. hint:: The CKAN shibboleth plugin repository is at http://github.com/geosolutions-it/ckanext-shibboleth


------------
Installation
------------

Activate your CKAN virtual environment::

   . /usr/lib/ckan/default/bin/activate

Go into your CKAN path for extension::

   cd /usr/lib/ckan/default/src

Import the project from the github repository and install it::

   git clone https://github.com/geosolutions-it/ckanext-shibboleth.git
   cd ckanext-shibboleth
   python setup.py install

        
--------------------	
Plugin configuration
--------------------

You have to configure the shibboleth plugin.
There are a couple of configuration files to edit:

``/etc/ckan/default/production.ini``

   - Tells CKAN to load the shibboleth plugin
    
``/etc/ckan/default/who.ini``

   - Tells the auth framework to use the shibboleth plugin for authentication.
   - Tells the shibboleh plugin how to retrieve the info about the authenticated user.  


``production.ini`` configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Edit the file ``/etc/ckan/default/production.ini`` and append ``shibboleth`` to the ``ckan.plugins`` line::

     ckan.plugins = [...] shibboleth
    

``who.ini`` configuration
^^^^^^^^^^^^^^^^^^^^^^^^^

Inside the directory ``/etc/ckan/default/`` we created the symbolic link ``who.ini`` 
linking the file ``/usr/lib/ckan/default/src/ckan/who.ini``.
We need to edit this file to configure some info for the shibboleth integration.
We don't want to modifiy the original file so we'll have to:

- Rename the symbolic link so we still have a reference to the original file::

   mv /etc/ckan/default/who.ini /etc/ckan/default/orig.who.ini
   
- Create a new file copy to edit::   

   cp /usr/lib/ckan/default/src/ckan/who.ini /etc/ckan/default/shibboleth.who.ini
    
- Create a symlink, so you may easily switch back to the original configuration should you need to::

   ln -s /etc/ckan/default/shibboleth.who.ini /etc/ckan/default/who.ini
 
Now let's edit the ``/etc/ckan/default/shibboleth.who.ini`` file.

Add the ``plugin:shibboleth`` section, customizing the env var names::

   [plugin:shibboleth]
   use = ckanext.shibboleth.repoze.ident:make_identification_plugin
   
   session = HTTP_SHIB_SESSION_ID
   eppn = HTTP_UID
   mail = NO_MAIL_HEADER
   fullname = HTTP_SN
   
   check_auth_key=HTTP_SHIB_AUTHENTICATION_METHOD
   check_auth_value=urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport
 
- ``session`` is used to identify the session id read by the shibboleth integration;
- ``eppn`` is the identifier used to uniquely identify the user;
- ``mail`` is the user mail address. You may set it to a name that will not be resolved; the user's mail address will be left blank, 
  and the user will be reminded about this at every login;
- ``fullname`` is the string used as the username in CKAN, displayed on the UI;
- ``check_auth_key`` and ``check_auth_value`` are needed to find out if we are properly receiving info from the Shibboleth module.


Add ``shibboleth`` to the list of the identifier plugins::

    [identifiers]
    plugins =
        shibboleth
        friendlyform;browser
        auth_tkt

Add ``ckanext.shibboleth.repoze.auth:ShibbolethAuthenticator`` to the list of the authenticator plugins::

    [authenticators]
    plugins =
        auth_tkt
        ckan.lib.authenticator:UsernamePasswordAuthenticator
        ckanext.shibboleth.repoze.auth:ShibbolethAuthenticator

Add ``shibboleth`` to the list of the challengers plugins::

    [challengers]
    plugins =
        shibboleth
    #    friendlyform;browser
    #   basicauth


Apache HTTPD configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^

The ckanext-shibboleth extension requires that the ``/shibboleth`` path to be externally filtered by the shibboleth
client module.

Using ``mod_shib`` on your apache httpd installation, you need these lines in your configuration file::

    <Location ~ /shibboleth >
        AuthType shibboleth
        ShibRequireSession On
        require valid-user
    </Location>


:download:`This is the complete ckan.conf configuration file <resources/92_ckan.conf>` you can use as a reference.

CKAN locales configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^

The ckanext-shibboleth extension defines some own locale strings defined into the internal .mo and .po files at ``ckanext-shibboleth/ckanext/shibboleth/i18n/``.
As reported above, for the ckanext-provbz installation steps, at this point you have already updated the default CKAN's locale files. So the locales information of the 
ckanext-shibboleth extension should be just appended to the existing ones ('it' and 'de') in CKAN as described below:

1 - Open the file::

	ckanext-shibboleth/ckanext/shibboleth/i18n/it/LC_MESSAGES/ckanext-shibboleth.po

2 - Copy the content reported below::

	#: ckanext/repoze/who/shibboleth/controller.py:25
	msgid "No user info received for login"
	msgstr "Non sono state ricevute informazioni sull'utente"

	#: ckanext/repoze/who/shibboleth/templates/user/snippets/login_form.html:25
	msgid "Shibboleth"
	msgstr "Shibboleth"

	#: ckanext/repoze/who/shibboleth/templates/user/snippets/login_form.html:26
	msgid "Login through Shibboleth."
	msgstr "Accedi attraverso Shibboleth"

	#: ckanext/repoze/who/shibboleth/templates/user/snippets/login_form.html:33
	msgid "Login via Shibboleth"
	msgstr "Accedi attraverso Shibboleth"

	#: ckanext/repoze/who/shibboleth/templates/user/snippets/login_form.html:45
	msgid "Authentication by using local account"
	msgstr "Autenticazione con account locale"

	#: ckanext/repoze/who/shibboleth/templates/user/snippets/login_form.html:49
	msgid "Username"
	msgstr "Nome utente"

	#: ckanext/repoze/who/shibboleth/templates/user/snippets/login_form.html:50
	msgid "Password"
	msgstr "Password"

	#: ckanext/repoze/who/shibboleth/templates/user/snippets/login_form.html:59
	msgid "Log in"
	msgstr "Accedi"
	
3 - Append it at the end of the CKAN's related file for 'it'::

	ckan/ckan/i18n/it/LC_MESSAGES/ckan.po

4 - Rebuild the ckan.mo file with the updated content using the following command::

	cd /usr/lib/ckan/default/src/ckan
	. /usr/lib/ckan/default/bin/activate
	
	python setup.py compile_catalog --locale it
	
5 - Repete the steps above for the 'de' locales and finally restart CKAN.
	
==================
Document changelog
==================

+---------+------+--------+---------------------------------------+
| Version | Date | Author | Notes                                 |
+=========+======+========+=======================================+
| 1.0     |      |        | Initial revision                      |
+---------+------+--------+---------------------------------------+
| 1.1     |      |        | Improve doc for installing shibboleth |
+---------+------+--------+---------------------------------------+
