.. _ckan_geoserver:

############################################
WMS Previews for protected data in GeoServer
############################################

Configure the GeoServer location for Auth/Auth
''''''''''''''''''''''''''''''''''''''''''''''

For protected data in GeoServer the CKAN user must be authenticated when WMS requests are performed from the CKAN's WMS preview.
This implies some requirements:

* GeoNetwork metadata that refer to the private data in GeoServer must be harvested using the LaMMA-private harvest source. This source harvests the dataset inside the CKAN’s LaMMA Organizzation as private datasets, so this implies that only the CKAN’s users of this Organizations can see the involved dataset.
* CKAN and GeoServer must have the same users (i.e. CKAN must replicate the same users of GeoServer). The user is who can access to the private the LaMMA Organizzation as described at the point one (so the user should be member of the Organizzation to see private datasets).
* A 'Request Header' authentication filter must be configured in GeoServer ad described `here <http://docs.geoserver.org/2.6.x/en/user/security/tutorials/httpheaderproxy/index.html>`_
* Apache HTTP must be configured with a Location that parses the Cookie Header for each request to GeoServer, creates the required Header containing the user name (to  use for authenticating into GeoServer). Below the configuration to use::

	  <Location "/geoserver"> 
		RequestHeader set X-Auth-Header "expr=%{req:Cookie}"
		RequestHeader edit X-Auth-Header "^.*auth_tkt=.(.{40})([^!]+)!.*$" "$2"
	  </Location>

           