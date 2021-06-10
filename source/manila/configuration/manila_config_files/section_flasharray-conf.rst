.. _manila_flasharray_conf:

Pure Storage Driver for FlashArray
==================================

FlashArray Manila Driver Configuration
--------------------------------------

The OpenStack Manila driver enables communication between OpenStack
and FlashArray systems with FA-Files enabled on the array. 
The user can use information from
the FlashArray to configure the driver by modifying the
``/etc/manila/manila.conf`` service file on the controller host.
For more information on the configuration and best practices for 
specific OpenStack releases please visit
the following link: http://support.purestorage.com/Solutions/OpenStack

Table 9.15 lists the required storage system attributes used in the
``/etc/manila/manila.conf`` configuration file.

.. _table-9.15:

+--------------------------------------+----------------------------+---------------------------------------------+
| FlashArray Attribute                 | Default                    | Description                                 |
+======================================+============================+=============================================+
| ``flasharray_mgmt_vip``              | None                       | FlashArray Management VIP                   |
+--------------------------------------+----------------------------+---------------------------------------------+
| ``flasharray_file_vip``              | None                       | FlashArray Files VIP                        |
+--------------------------------------+----------------------------+---------------------------------------------+
| ``flasharray_api``                   | None                       | FlashArray authorization API token          |
+--------------------------------------+----------------------------+---------------------------------------------+

Table 9.15. Required FlashArray Attributes

Add the following lines to the file, replacing login and password with
the cluster admin login credentials

::


    [DEFAULT]
    enabled_share_backends=flasharray

    [flasharray]
    driver_handles_share_servers = False
    share_backend_name=flasharray
    share_driver=manila.share.drivers.purestorage.flasharray.FlashArrayShareDriver
    flasharray_mgmt_ip=192.168.1.34
    flasharray_file_ip=192.168.1.35
    flasharray_api=<API token>

Optional Manila Configuration Attributes
----------------------------------------
You can optionally use the following attributes specific to FlashArray
in the ``[pure]`` section of the ``/etc/manila/manila.conf``
configuration file to control the interaction between the storage
system and the OpenStack Manila service. (See Table 9.16.)

.. _table-9.16:

+--------------------------------------------------+----------------------------+----------------------------------------------------+
| FlashArray Attribute                             | Default      | Description                                                      |
+==================================================+============================+====================================================+
| ``flasharray_eradicate``                         | True         | Enable auto-eradication of deleted shares and snapshots.         |
+--------------------------------------------------+----------------------------+----------------------------------------------------+
| ``pure_automatic_max_oversubscription_ratio``    | True         | Allow FlashArray to calculate the array oversubscription ratio.  |
+--------------------------------------------------+----------------------------+----------------------------------------------------+

Table 9.16. Optional FlashArray Attributes
