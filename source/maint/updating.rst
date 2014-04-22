.. _ckan_updating:

#############
Updating CKAN
#############

Updating extensions
===================

CKAN extensions (and CKAN itself) are downloaded into the server using `git <http://git-scm.com/>`_, 
a distributed version control system, and, so far, all extensions are hosted on `github <http://github.com>`_.

This means that there is a central repository for each extension 
(e.g. `harvest extension <http://github.com/ckan/ckanext-harvest>`_) where developers store and update the source code, 
and from where the users can download it. 
 
When an extension is installed from `github`, it includes all the latest updates that have been `pushed`
by the devs into the repository. After some time it is possible that the repository on github receives some updates.

In order for the installed version to be kept in synch, such updates should be `pulled` from the repo into the server
copy.

All the extensions that have been installed using the command ::

   git clone https://github.com/YOUR_EXTENSION.git
   
can be updated with the following procedure:

- Login as user ``ckan``.
- Enter the extension you need to update: e.g.::

   cd /usr/lib/ckan/default/ckanext-ecportal
    
- Pull the updates ::

   git pull
    
- Restart the service as user ``root`` ::

   service supervisord restart


.. hint::

   You may want to take a **backup** of the extension dir before upgrading.
   
   Even if all the changes are versioned and you can rollback to any previous version you wish, 
   you may be more confident using ``cp`` ``mv`` and ``tar`` rather than ``git``.  


Troubleshooting updates
=======================

In case you edited some files that have also been changed by the devs on the central repository, 
you'll get a *conflict* when pulling the updates, and the pull procedure will be aborted.
E.g.::

   $ git pull
   
   remote: Counting objects: 40, done.
   remote: Compressing objects: 100% (32/32), done.
   remote: Total 40 (delta 12), reused 33 (delta 8)
   Unpacking objects: 100% (40/40), done.
   From https://github.com/geosolutions-it/ckanext-mapstore
      a650148..c1df278  master     -> origin/master
   Updating a650148..c1df278
   error: Your local changes to the following files would be overwritten by merge:
      ckanext/mapstore/preview/preview_config.js
   Please, commit your changes or stash them before you can merge.
   Aborting
   
Probably you only made some minor editing that will be easy to port back on the latest version from the repository,
so the idea is to backup your file, restore the original file and then try the pull again.

The output tells us that the file ::

   ckanext/mapstore/preview/preview_config.js

is the only conflicting one, so let's backup it :: 

   cp ckanext/mapstore/preview/preview_config.js ckanext/mapstore/preview/preview_config.js_YOUR_BK_SUFFIX
   
You may check the changes in your repo (with respect to the central repository on github) using ``git status``::

   $ git status 
   
   # On branch master
   # Your branch is behind 'origin/master' by 6 commits, and can be fast-forwarded.
   #
   # Changes not staged for commit:
   #   (use "git add <file>..." to update what will be committed)
   #   (use "git checkout -- <file>..." to discard changes in working directory)
   # 
   #  modified:   ckanext/mapstore/preview/preview_config.js
   #
   # Untracked files:
   #   (use "git add <file>..." to include in what will be committed)
   #
   #  ckanext/mapstore/preview/preview_config.js_BK
   no changes added to commit (use "git add" and/or "git commit -a")
   
Since we created a backup copy of the file, we can "discard changes in working directory"::

   git checkout -- ckanext/mapstore/preview/preview_config.js

You can ask for a ``git status`` again to make sure there are no other modified files, and then issue the pull::

   git pull
   
Now reapply your changes if needed, before restarting ``supervisord``.


 

