.. _common-probs:

Common Problems
======================

Common problems listed below are followed by the ``cinder``,
or ``nova`` CLI command and possible reasons for the occurrence of the
problem.

1. Volume create operation fails
--------------------------------

::

    $ cinder create size_gb

-  Cinder API service is down.

-  Cinder scheduler service is down.

2. Volume create with volume-type operation fails
-------------------------------------------------

::

    $ cinder create --volume-type volume_type size_gb

-  All the reasons mentioned under Item 1 in this appendix.

-  The FlashArray backend(s) with available space do not support at least
   one of the extra-specs bound to the volume-type requested. Hence, it
   does not return the extra spec in volume stats call to the Cinder
   scheduler.

-  The configured API token does not have sufficient
   privileges on the FlashArray.

-  The configured management IP address/FQDN and API token for a FlashArray
   backend are incorrect.

3. Volume create from image-id operation fails
----------------------------------------------

::

    $ cinder create --image-id image-id size_gb

-  All the reasons mentioned under Item 1 in this appendix.

-  Glance related services are down.

-  The image could not be downloaded from glance because of download
   error.

4. Volume create from image-id with volume-type operation fails
---------------------------------------------------------------

::

    $ cinder create --image-id image-id --volume-type volume_type size_gb

-  All the reasons mentioned under Items 1, 2, and 3 in this appendix.

5. Volume create from snapshot operation fails
----------------------------------------------

::

    $ cinder create --snapshot-id snapshot-id size_gb

-  All reason mentioned under Items 1 in this appendix.

6. Create cloned volume operation fails
---------------------------------------

::

    $ cinder create --source-volid volume-id size_gb

-  All reason mentioned under Items 1 in this appendix.

7. Volume attach operation in nova fails
----------------------------------------

::

    nova volume-attach instance-id volume-id path size_gb

-  iSCSI drivers:

   -  The iSCSI service on the ``nova-compute`` host may not be running.

   -  The iSCSI network is not reachable due to firewall, configuration, or
      transient issues.

-  FC drivers:

   -  The FC zones are incorrectly configured.

   -  If using the Fibre Channel Zone Manager the SAN switch connectivity
      information is incorrect.
