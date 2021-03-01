.. _without-share:

FlashBlade Driver
=================

Overview
--------

The FlashBlade driver does not support share server
management ind is therefore considered a ``DHSS=False`` type
driver.

Installation
------------

.. important::

    The FlashBlade Manila driver is not currently upstream in OpenStack Manila
    and therefore needs to be manually installed in a running Manila
    configuration.
    This driver is only supported on a Best Efforts basis and there is
    currently no official support from Pure Storage.

Perform the following steps to install the driver.

- Locate the ``manila`` directory for your OpenStack deployment and create
  the following directory
  
  ``manila/share/drivers/purestorage``

- Copy the FlashBlade driver file into this new directory

  ``cd manila/share/drivers/purestorage``
  ``wget https://raw.githubusercontent.com/PureStorage-OpenConnect/OpenStack-Manila/master/manila/share/drivers/purestorage/flashblade.py .``

- Edit the file ``manila/opts.py`` and perform the following actions:

  - Near the beginning of the file with the ``import`` commands and add

    ``import manila.share.drivers.purestorage.flashblade``

  - Find the ``_global_opt_lists`` section and add the following lines

    ``manila.share.drivers.purestorage.flashblade.flashblade_auth_opts,``
    ``manila.share.drivers.purestorage.flashblade.flashblade_extra_opts,``
    ``manila.share.drivers.purestorage.flashblade.flashblade_connection_opts,``

Setup
-----

To set up the FlashBlade driver without Share Server,
the following stanza should be added to the Manila
configuration file (``manila.conf``)::

    [flashblade]
    share_backend_name = flashblade
    share_driver = manila.share.drivers.purestorage.flashblade.FlashBladeShareDriver
    driver_handles_share_servers = False
    flashblade_mgmt_vip = FlashBlade management VIP
    flashblade_data_vip = FlashBlade data VIP
    flashblade_api = FlashBlade API token
    flashblade_eradicate = { True | False }

-  Be sure that the value of the ``enabled_share_backends`` option in
   the ``[DEFAULT]`` stanza includes the name of the stanza you chose
   for the backend.

-  The value of ``driver_handles_share_servers`` **MUST** be set to
   ``False``.

After completing the configuration of the Manila configuration file you must
restart all of the Manila services to enable the FlashBlade driver.

Table 9.15, “Configuration options for FlashBlade”
lists the configuration options available for FlashBlade driver.

+-----------------------------------+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Option                            | Type       | Description                                                                                                                                                                                                                 |
+===================================+============+=============================================================================================================================================================================================================================+
| ``share_backend_name``            | Required   | The name used by Manila to refer to the Manila backend                                                                                                                                                                      |
+-----------------------------------+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``share_driver``                  | Required   | Set the value to manila.share.drivers.purestorage.flashblade.FlashBladeShareDriver                                                                                                                                          |
+-----------------------------------+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``driver_handles_share_servers``  | Required   | Denotes whether the driver should handle the responsibility of managing share servers. This must be set to ``false`` if the driver is to operate without managing share servers.                                            |
+-----------------------------------+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``flashblade_mgmt_vip``           | Required   | Denotes the Management IP address of the Pure Storage FlashBlade to manage with the Manila driver.                                                                                                                          |
+-----------------------------------+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``flashblade_data_vip``           | Required   | Denotes a Data VIP address on the managed Pure Storage FlashBlade. This should be within the same subnet, or accessible via routing, to the Neutron network used by the Nova instances requiring to use this FlashBlade.    |
+-----------------------------------+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``flashblade_api``                | Required   | Denotes an API token for a user on the FlashBlade. This user requires administrative role capabilities and can be either local or be managed under an external account control system, such as LDAP or AD.                  |
+-----------------------------------+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``flashblade_eradicate``          | Optional   | Enable auto-eradication of shares and snapshots on deletion (Default: ``False``)                                                                                                                                            |
+-----------------------------------+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Table 9.15. Configuration options for FlashBlade
