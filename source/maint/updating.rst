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

   cd /usr/lib/ckan/default/ckanext-geonetwork

- Pull the updates ::

   git pull

- Restart the service as user ``root`` ::

   systemctl stop supervisord
   systemctl start supervisord


.. hint::

   You may want to take a **backup** of the extension dir before upgrading.

   Even if all the changes are versioned and you can rollback to any previous version you wish,
   you may be more confident using ``cp`` ``mv`` and ``tar`` rather than ``git``.
