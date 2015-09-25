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
   python setup.py install

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

Shibboleth identification plugin for CKAN 2.4. 

-------------
Installation 
-------------

You can install ckanext-shibboleth either with::

    pip install -e git+git://github.com/geosolutions-it/ckanext-shibboleth.git#egg=ckanext-shibboleth
	
or::

    git clone https://github.com/geosolutions-it/ckanext-shibboleth.git
    python setup.py install
        
--------------------	
Plugin configuration
--------------------

who.ini configuration (general overview)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Add the ``plugin:shibboleth`` section, customizing the env var names::

    [plugin:shibboleth]
    use = ckanext.shibboleth.repoze.ident:make_identification_plugin

    session = YOUR_HEADER_FOR_Shib-Session-ID
    eppn = YOUR_HEADER_FOR_eppn
    mail = YOUR_HEADER_FOR_mail
    fullname = YOUR_HEADER_FOR_cn

    check_auth_key=AUTH_TYPE
    check_auth_value=shibboleth

``check_auth_key`` and ``check_auth_value`` are needed to find out if we are receiving info from the Shibboleth module. Customize both right-side values if needed. For instance, older Shibboleth implementations may need this configuration::

    check_auth_key=HTTP_SHIB_AUTHENTICATION_METHOD 
    check_auth_value=urn:oasis:names:tc:SAML:1.0:am:unspecified

who.ini configuration (Provincia di Bolzano)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Below the configuration to use for the Provincia di Bolzano environment::

	[plugin:shibboleth]
	use = ckanext.shibboleth.repoze.ident:make_identification_plugin
	session = HTTP_SHIB_SESSION_ID
	eppn = HTTP_UID
	mail = NO_MAIL_HEADER
	fullname = HTTP_SN
	check_auth_key=HTTP_SHIB_AUTHENTICATION_METHOD
	check_auth_value=urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport    

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

production.ini configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Add ``shibboleth`` the the ckan.plugins line::

     ckan.plugins = [...] shibboleth

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

**Below the complete ckan.conf configuration file to use when the shibboleth plugin is installed**::

	# https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPApacheConfig

	# RPM installations on platforms with a conf.d directory will
	# result in this file being copied into that directory for you
	# and preserved across upgrades.

	# For non-RPM installs, you should copy the relevant contents of
	# this file to a configuration location you control.

	#
	# Load the Shibboleth module.
	#
	LoadModule mod_shib /usr/lib64/shibboleth/mod_shib_24.so

	#
	# Turn this on to support "require valid-user" rules from other
	# mod_authn_* modules, and use "require shib-session" for anonymous
	# session-based authorization in mod_shib.
	#
	ShibCompatValidUser Off

	#
	# Ensures handler will be accessible.
	#
	<Location /Shibboleth.sso>
	  AuthType None
	  Require all granted
	</Location>

	#
	# Used for example style sheet in error templates.
	#
	<IfModule mod_alias.c>
	  <Location /shibboleth-sp>
		AuthType None
		Require all granted
	  </Location>
	  Alias /shibboleth-sp/main.css /usr/share/shibboleth/main.css
	</IfModule>

	#
	# Configure the module for content.
	#
	# You MUST enable AuthType shibboleth for the module to process
	# any requests, and there MUST be a require command as well. To
	# enable Shibboleth but not specify any session/access requirements
	# use "require shibboleth".
	#
	<Location /secure>
	  AuthType shibboleth
	  ShibRequestSetting requireSession 1
	  require shib-session
	</Location>



	# Basic Apache configuration

	<IfModule mod_alias.c>
	  <Location /shibboleth-ds>
	#    Allow from all
	Require all granted
		<IfModule mod_shib.c>
		  AuthType shibboleth
		  ShibRequestSetting requireSession false
		  require shibboleth
		</IfModule>
	  </Location>
	  Alias /shibboleth-ds/idpselect_config.js /etc/shibboleth-ds/idpselect_config.js
	  Alias /shibboleth-ds/idpselect.js /etc/shibboleth-ds/idpselect.js
	  Alias /shibboleth-ds/idpselect.css /etc/shibboleth-ds/idpselect.css
	  Alias /shibboleth-ds/index.html /etc/shibboleth-ds/index.html
	  Alias /shibboleth-ds/blank.gif /etc/shibboleth-ds/blank.gif
	</IfModule>

	<Location /shibboleth>
	  AuthType shibboleth
	  ShibRequestSetting requireSession 1
	  # old setting?
	  ShibRequireSession On
	  ShibUseHeaders On
	  ShibUseEnvironment On


					 RewriteEngine On
					RewriteBase /

					#IDP PROV
					RewriteCond %{HTTP:Shib-Identity-Provider}  idp.prov.bz
					RewriteRule .* - [E=INFO_HTTP_SHIB_IDENTITY_PROVIDER:prov,NE]
					#IDP EGOV
					RewriteCond %{HTTP:Shib-Identity-Provider}  test-idp.egov.bz.it
					RewriteRule .* - [E=INFO_HTTP_SHIB_IDENTITY_PROVIDER:egov,NE]
					#IDP REG
					RewriteCond %{HTTP:Shib-Identity-Provider}  demo-idp.regione.taa.it
					RewriteRule .* - [E=INFO_HTTP_SHIB_IDENTITY_PROVIDER:reg,NE]


					RequestHeader set INFO_HTTP_REFERER "%{INFO_HTTP_SHIB_IDENTITY_PROVIDER}e"
					#RequestHeader add "SIAG_USERNAME" "%{INFO_HTTP_SHIB_IDENTITY_PROVIDER}e\%{uid}e"
					RequestHeader add "SU" "%{INFO_HTTP_SHIB_IDENTITY_PROVIDER}e:%{uid}e"


	  require valid-user
	</Location>

	ProxyPass  /shibboleth-ds  !
	ProxyPass  /shibboleth-sp  !
	ProxyPass  /Shibboleth.sso !
	ProxyPass  /secure         !

	#<Location ~ "^(?!/[Ss]hibboleth)(.*)$" >
	#  ProxyPass        http://localhost:5000/
	#  ProxyPassReverse http://localhost:5000/
	#</Location>

	ProxyPass        / http://localhost:5000/
	ProxyPassReverse / http://localhost:5000/ 
	
==================
Document changelog
==================

+---------+------------+--------+------------------+
| Version | Date       | Author | Notes            |
+=========+============+========+==================+
| 1.0     |            |        | Initial revision |
+---------+------------+--------+------------------+
