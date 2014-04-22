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

=======
Tracker
=======

Tracks visit to the site and to single datasets.

.. hint::
   Doc page at http://docs.ckan.org/en/tracking-fixes/tracking.html
    
Edit the file ``/etc/ckan/default/production.ini`` and add the line ::

   ckan.tracking_enabled = true
   
Then create a script ``tracker_update.sh`` like this::

    #!/bin/bash

    . /usr/lib/ckan/default/bin/activate

    paster --plugin=ckan tracking update         -c /etc/ckan/default/production.ini 
    paster --plugin=ckan search-index rebuild -r -c /etc/ckan/default/production.ini

You can use this file directly to run the index rebuilding, or use it in cron to make it run periodically.

As user ``root`` use ::

     crontab -e -u ckan
     
or as user ``ckan``::

     crontab -e
     
and add the line::

   0 * * * * /usr/lib/ckan/tracker_update.sh >>/var/log/ckan/tracker.log 2>&1


======================
Multilingual extension
======================

.. hint::
   Info page at http://docs.ckan.org/en/latest/maintaining/multilingual.html

The Multilingual extension is aleady checked out together with the main CKAN repository,
so its sources are already available on the system.

In order to enable the multilingual plugin, you have to edit the file 
``/etc/ckan/default/production.ini`` and add the `multilingual` plugins::

   ckan.plugins = [...] multilingual_dataset multilingual_group multilingual_tag
   
and restart CKAN in order to reload the plugin list::

   service supervisord restart   

Then follow the details in the official page in order to add the translated entries.
 
.. hint::
   Official documentation is not yet complete (as of 2014-02-17).  
   
   See also https://github.com/ckan/ckan/issues/1021
   
The solr schema file ``/etc/solr/ckan/conf/schema.xml`` should be updated with the one with 
multilang capabilities::

   ln -sf /usr/lib/ckan/default/src/ckan/ckanext/multilingual/solr/schema.xml /etc/solr/ckan/conf/schema.xml
   
Then also link all the stopword files::

   for txtfile in /usr/lib/ckan/default/src/ckan/ckanext/multilingual/solr/*.txt ; do ln -sv $txtfile /etc/solr/ckan/conf/  ; done

Restart solr::

   service solr restart   
      
You have to rebuild the index with the new schema.
As ``ckan`` user, ``activate`` the virtual env and run::

   paster --plugin=ckan search-index rebuild --config=/etc/ckan/default/production.ini
   
    
.. warning::
   It causes lots of *CRITICAL* logging due to old code::
    
      CRITI [ckan.logic] Action `term_translation_show` is being called directly all action calls should be accessed via logic.get_action
      
   The fix https://github.com/ckan/ckan/issues/1520 has been applied.      
      
.. warning::
   Multilingual seems not very stable: it breaks on some situations::
   
      Error - <type 'exceptions.AttributeError'>: 'int' object has no attribute 'get'

   and this will make the client display an "Internal error" message. 
         
   *Not enabled* for now.   

.. warning::
   When replacing the schema file, remember to also add the settings for indexing the spatial fields, 
   as reported in :ref:`configure_spatial_search`.

   
==================
ECPortal extension
==================

.. hint::
   Repo page at https://github.com/okfn/ckanext-ecportal
   
* Latest versioned branch: 1.8.1 release
* Latest commit 3 months ago

Check out the extension::

   . /usr/lib/ckan/default/bin/activate
   cd default/src/
   pip install -e  git+https://github.com/okfn/ckanext-ecportal.git#egg=ckanext-ecportal


Install dependencies::

   pip install -r pip-requirements.txt
   
.. warning::
   ECPortal has to be ported to 2.x auth architecture. 
   
   *Not enabled* for now.   

This is the error returned when running an *ECPortal* command:: 

   (default)[ckan@server-84-33-2-79 ckanext-ecportal]$ paster --plugin=ckanext-ecportal ecportal --help
   Traceback (most recent call last):
     File "/usr/lib/ckan/default/bin/paster", line 9, in <module>
       load_entry_point('PasteScript==1.7.5', 'console_scripts', 'paster')()
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/paste/script/command.py", line 103, in run
       command = commands[command_name].load()
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/pkg_resources.py", line 2029, in load
       entry = __import__(self.module_name, globals(),globals(), ['__name__'])
     File "/usr/lib/ckan/default/src/ckanext-ecportal/ckanext/ecportal/commands.py", line 13, in <module>
       import ckanext.ecportal.forms as forms
     File "/usr/lib/ckan/default/src/ckanext-ecportal/ckanext/ecportal/forms.py", line 5, in <module>
       from ckan.authz import Authorizer
     ImportError: No module named authz
     

=====================
ECPortal UI extension
=====================

Since the original ECPortal extension does not work on CKAN 2.2, an extension has been reimplemented in order
to customize the UI according to other EU site. 
 
.. hint::
   Repo page at https://github.com/geosolutions-it/ckanext-ecportal

You need :ref:`extension_tracker` and :ref:`extension_mapstore` in order for this extension
to work properly.
   
Check out the extension. As user ``ckan``::
   
   . /usr/lib/ckan/default/bin/activate
   cd default/src/
   git clone https://github.com/geosolutions-it/ckanext-ecportal.git 
   cd ckanext-ecportal
   python setup.py develop      
      
Add plugin in ``/etc/ckan/default/production.ini``::
      
   ckan.plugins = [...] ecportal
   
Make sure that ``ecportal`` is defined *before* ``mapstore_preview``, or some templates will 
not be overridden properly.
   
Restart supervisord::
      
   service supervisord restart    
     
============
QA extension
============

Checks each of your package resources and give these resources an openness score 
based Tim Berners-Lee's five stars of openness.

.. hint::
   Repo page at https://github.com/ckan/ckanext-qa

* Latest versioned branch: release-v2.0
* Latest commit 5 months ago

.. warning::
   QA needs *archiver* extension to be installed. 
   
   Please install :ref:`install_archiver_extension` before installing *QA*.

Check out the extension::
   
   . /usr/lib/ckan/default/bin/activate
   cd default/src/
   pip install -e  git+https://github.com/ckan/ckanext-qa.git#egg=ckanext-qa
   
Install the plugin::

   cd ckanext-qa
   pip install -e ./
   
Add plugin in ``/etc/ckan/default/production.ini``::
      
   ckan.plugins = [...] qa
   
Restart supervisord::
      
   service supervisord restart   

Now you may want to update the score of all the dataset in your CKAN instance::

   paster --plugin=ckanext-qa qa update  --config=/etc/ckan/default/production.ini 


After you reload the site, the Quality Assurance plugin and openness score interface 
should be available at ``http://your-ckan-instance/qa`` .

.. note::
   When processing the dataset the first time, you will get lots of log lines 
   of this kind::

      Error - <type 'exceptions.KeyError'>: 'resources'
      
   They do not seem to be logged again after first update is run.

.. _install_archiver_extension:

==================
Archiver extension
==================
 
Provides a set of Celery tasks for downloading and saving CKAN resources. 

*Dependency of QA extension.*
  
.. hint::
   Repo page at https://github.com/ckan/ckanext-archiver

* Latest versioned branch: release-2.0
* Latest commit 10 months ago

.. warning::
   Official documentation is quite outdated.

Check out the extension::
   
   . /usr/lib/ckan/default/bin/activate
   cd default/src/   
   pip install -e git+https://github.com/okfn/ckanext-archiver.git#egg=ckanext-archiver

Install dependencies::

   cd ckanext-archiver/
   pip install -r pip-requirements.txt
   
Add plugin in ``/etc/ckan/default/production.ini``::
      
   ckan.plugins = [...] archiver
   
   
Archiver needs *celery* to run. 
Celery is installed when configuring the *archiver*, but needs configuration to be run 
automatically at system startup.
   
Add these lines to file ``/etc/supervisord.conf``::

   [program:celery]
   ; Full Path to executable, should be path to virtural environment,
   ; Full path to config file too.
   command=/usr/lib/ckan/default/bin/paster --plugin=ckan celeryd --config=/etc/ckan/default/production.ini
   user=ckan
   numprocs=1
   stdout_logfile=/var/log/ckan/celeryd.log
   stderr_logfile=/var/log/ckan/celeryd.log
   autostart=true
   autorestart=true
   startsecs=10
   ; Need to wait for currently executing tasks to finish at shutdown.
   ; Increase this if you have very long running tasks (default was 600)
   stopwaitsecs = 10
   priority=998
   
And restart supervisord::
      
   service supervisord restart   
   
You can test if *celery* is running properly by issuing this command from and activated env::   
   
   (default)$ paster --plugin=ckan celeryd view --config=/etc/ckan/default/production.ini
   2014-02-17 17:10:44,214 DEBUG [ckanext.harvest.model] Harvest tables defined in memory
   2014-02-17 17:10:44,217 DEBUG [ckanext.harvest.model] Harvest tables already exist
   2014-02-17 17:10:44,242 DEBUG [ckanext.spatial.model.package_extent] Spatial tables defined in memory
   2014-02-17 17:10:44,251 DEBUG [ckanext.spatial.model.package_extent] Spatial tables already exist
   0 messages (total)
   0 visible messages
   $

In order to add to the archive all exising datasets, you have to run this command::
   
   paster --plugin=ckanext-archiver archiver update --config=/etc/ckan/default/production.ini
   
===================
GA-Report extension
===================

For creating detailed reports of CKAN analytics, including totals per group.

.. hint::
   Repo page at https://github.com/datagovuk/ckanext-ga-report
   
* Latest versioned branch: stable (10 months ago)
* Latest commit 1 month ago on master


.. note:: 
   The reporting graphics is clearly aligned to older CKAN versions.  


Check out the extension::   

   pyenv/bin/activate
   pip install -e  git+https://github.com/datagovuk/ckanext-ga-report.git#egg=ckanext-ga-report

Install the ``gflags`` library needed for creating the token file ::

   easy_install --upgrade python-gflags

Edit the ``/etc/ckan/default/production.ini`` file, and set your Google Analytics info::
   
   ## GA-report settings

   googleanalytics.id = DEFINE HERE THE ID
   googleanalytics.account = DEFINE HERE THE ACCOUNT
   googleanalytics.token.filepath = /var/lib/ckan/googleanalytics.dat
   ga-report.period = monthly
   ga-report.bounce_url = /
   
Directory ``/var/lib/ckan`` should already exist, because it was created when configuring a 
previous plugin (filestore).

Init the DB tables for ga-report::

   paster --plugin=ckanext-ga-report initdb --config=/etc/ckan/default/production.ini
   
Enable the plugin in ``/etc/ckan/default/production.ini``::

   ckan.plugins = [...] ga-report

Now follow the instructions on the ref page to create your ``credentials.json`` file.
   
Once you have created the ``credentials.json`` file, make sure it's in the same dir you 
are going to launch the command to create the token file.

Run::

   paster --plugin=ckanext-ga-report getauthtoken   --config=/etc/ckan/default/production.ini

You will get some messages on the screen and a URL starting with ::
   
   https://accounts.google.com/o/oauth2/auth? [long list of params here]
   
Copy that URL in your browser. You will be requested to accept the access to google analytics by 
the CKAN instance. Accept it. You will be redirected to a URL in the format ::

   http://localhost:8090/?code=YOURLONGCODEHERE
   
Your computer won't be serving such page (you'll get an error on your browser), but the
``getauthtoken`` procedure is waiting for this call.
Open a new terminal on your server and issue the command::

   curl "http://localhost:8090/?code=YOURLONGCODEHERE" 
 
You'll get this output :: 
 
   <html><head><title>Authentication Status</title></head><body><p>The authentication flow has completed.</p></body></html>
   
and you'll find the file ``token.dat`` in your current directory.
Now copy the file to the location we set in the ``googleanalytics.token.filepath`` property ::
 
   cp token.dat /var/lib/ckan/googleanalytics.dat

and everything should be set.

You can import the stats using the lines in the ref page::

   paster --plugin=ckanext-ga-report loadanalytics latest  --config=/etc/ckan/default/production.ini

You may get an error if no stats have been yet gathered.
       
You can check the stats in the page ``http://your_site/data/site-usage`` 
(e.g. http://84.33.2.79/data/site-usage). 
   
.. note:: 
   The reporting graphics is clearly aligned to older CKAN versions.  

Setting tracking code
---------------------

You'll have to setup up manually the javascript code needed for the access tracking.

Create a snippet containing the javascript code provided by Google Analytics::
 
   vim /usr/lib/ckan/default/src/ckan/ckan/templates/snippets/ga.html
   
It will contain something like this::   
   
   <script type="text/javascript">
   
     var _gaq = _gaq || [];
     _gaq.push(['_setAccount', 'YOUR-CODE-HERE']);
     _gaq.push(['_trackPageview']);
   
     (function() {
       var ga = document.createElement('script'); ga.type = 'text/javascript'; ga.async = true;
       ga.src = ('https:' == document.location.protocol ? 'https://ssl' : 'http://www') + '.google-analytics.com/ga.js';
       var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(ga, s);
     })();
   
   </script>

Then include this snippet in the CKAN main page::

   vim /usr/lib/ckan/default/src/ckan/ckan/templates/base.html

and add ::

   {% snippet 'snippets/ga.html' %}

just before ::

   </head>

===============
Issue extension
===============

Allows users to report issues with datasets and resources they find on CKAN.

.. hint::
   Repo page at https://github.com/datagovuk/ckanext-issues
   
* Status: beta
* Latest commit a year ago

Check out the extension::
   
   pip install git+https://github.com/datagovuk/ckanext-issues
   
Add plugin in ``/etc/ckan/default/production.ini``::
      
   ckan.plugins = [...] issues

.. warning::
   This plugin is not yet compatible with CKAN 2.2 and is currently disabled. 
   
   
This is the error returned when restarting CKAN:: 

   Traceback (most recent call last):
     File "/usr/lib/ckan/default/bin/paster", line 9, in <module>
       load_entry_point('PasteScript==1.7.5', 'console_scripts', 'paster')()
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/paste/script/command.py", line 104, in run
       invoke(command, command_name, options, args[1:])
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/paste/script/command.py", line 143, in invoke
       exit_code = runner.run(args)
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/paste/script/command.py", line 238, in run
       result = self.command()
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/paste/script/serve.py", line 284, in command
       relative_to=base, global_conf=vars)
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/paste/script/serve.py", line 321, in loadapp
       **kw)
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/paste/deploy/loadwsgi.py", line 247, in loadapp
       return loadobj(APP, uri, name=name, **kw)
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/paste/deploy/loadwsgi.py", line 272, in loadobj
       return context.create()
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/paste/deploy/loadwsgi.py", line 710, in create
       return self.object_type.invoke(self)
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/paste/deploy/loadwsgi.py", line 146, in invoke
       return fix_call(context.object, context.global_conf, **context.local_conf)
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/paste/deploy/util.py", line 56, in fix_call
       val = callable(*args, **kw)
     File "/usr/lib/ckan/default/src/ckan/ckan/config/middleware.py", line 57, in make_app
       load_environment(conf, app_conf)
     File "/usr/lib/ckan/default/src/ckan/ckan/config/environment.py", line 232, in load_environment
       p.load_all(config)
     File "/usr/lib/ckan/default/src/ckan/ckan/plugins/core.py", line 134, in load_all
       load(*plugins)
     File "/usr/lib/ckan/default/src/ckan/ckan/plugins/core.py", line 149, in load
       service = _get_service(plugin)
     File "/usr/lib/ckan/default/src/ckan/ckan/plugins/core.py", line 255, in _get_service
       return plugin.load()(name=plugin_name)
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/pkg_resources.py", line 2029, in load
       entry = __import__(self.module_name, globals(),globals(), ['__name__'])
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/ckanext/issues/plugin.py", line 11, in <module>
       from ckanext.issues.lib import util
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/ckanext/issues/lib/util.py", line 1, in <module>
       import ckanext.issues.model as issue_model
     File "/usr/lib/ckan/default/lib/python2.6/site-packages/ckanext/issues/model.py", line 9, in <module>
       from ckan.model.meta import types, Table, ForeignKey, DateTime
   ImportError: cannot import name types


===================
Hierarchy extension
===================

Provides a new field on the organization edit form to select a parent organization.

.. hint::
   Repo page at https://github.com/datagovuk/ckanext-hierarchy

* Latest commit 1 month ago
* Claims compatibility with CKAN since 2.2.

Check out the extension::

   . /usr/lib/ckan/default/bin/activate
   cd default/src/
   pip install -e git+https://github.com/datagovuk/ckanext-hierarchy.git#egg=ckanext-hierarchy
   
Add plugin in ``/etc/ckan/default/production.ini``::
      
   ckan.plugins = [...] hierarchy_display hierarchy_form
  
For using this plugin in customized layout as snippets refer to the documentation
on the repo page.    


==================
Document changelog
==================

+---------+------------+--------+------------------+
| Version | Date       | Author | Notes            |
+=========+============+========+==================+
| 1.0     | 2014-02-14 | ETj    | Initial revision |
+---------+------------+--------+------------------+
