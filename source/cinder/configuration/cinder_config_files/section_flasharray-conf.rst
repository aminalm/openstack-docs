.. _flasharray_conf:

Pure Storage Driver for FlashArray
==================================

FlashArray Cinder Driver Configuration
--------------------------------------

The OpenStack Cinder driver enables communication between OpenStack
and FlashArray systems. The user can use information from
the FlashArray to configure the driver by modifying the
``/etc/cinder/cinder.conf`` service file on the controller host.
For more information on the configuration and best practices for 
specific OpenStack releases please visit
the following link: http://support.purestorage.com/Solutions/OpenStack

Table 4.18 lists the required storage system attributes used in the
``/etc/cinder/cinder.conf`` configuration file.

.. _table-4.18:

+--------------------------------------+----------------------------+---------------------------------------------+
| FlashArray Attribute                 | Default                    | Description                                 |
+======================================+============================+=============================================+
| ``san_ip``                           | None                       | FlashArray Management VIP                   |
+--------------------------------------+----------------------------+---------------------------------------------+
| ``pure_api_token``                   | None                       | FlashArray authorization API token          |
+--------------------------------------+----------------------------+---------------------------------------------+

Table 4.18. Required FlashArray Attributes

Add the following lines to the file, replacing login and password with
the cluster admin login credentials

::


    [DEFAULT]
    enabled_backends=pure

    [pure]
    volume_backend_name=pure
    volume_driver=PURE_VOLUME_DRIVER
    san_ip=192.168.1.34
    pure_api_token=

For ``PURE_VOLUME_DRIVER`` use either ``cinder.volume.drivers.pure.PureISCSIDriver`` for iSCSI or
``cinder.volume.drivers.pure.PureFCDriver`` for Fibre Channel connectivity.

Optional Cinder Configuration Attributes
----------------------------------------
You can optionally use the following attributes specific to FlashArray
in the ``[pure]`` section of the ``/etc/cinder/cinder.conf``
configuration file to control the interaction between the storage
system and the OpenStack Cinder service. (See Table 4.19.)

.. _table-4.19:

+------------------------------------------------------+----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| FlashArray Attribute                                 | Default                    | Description                                                                                                                                                                                                     |
+======================================================+============================+=================================================================================================================================================================================================================+
| ``pure_eradicate_on_delete``                         | False                      | Enable auto-eradication of deleted volumes, snapshots and consistency groups on deletion.                                                                                                                       |
+------------------------------------------------------+----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``pure_host_personality``                            | None                       | Set the host personality to tune the communication protocol between the FlashArray and the hypervisors. Recommended to leave this at the default setting.                                                       |
+------------------------------------------------------+----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``driver_ssl_cert_verify``                           | False                      | Set verification of FlashArray SSL certificates.                                                                                                                                                                |
+------------------------------------------------------+----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``driver_ssl_cert_path``                             | None                       | Non-default directory path to ``CA_Bundle`` file with certificates of trusted CAs.                                                                                                                              |
+------------------------------------------------------+----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``pure_automatic_max_oversubscription_ratio``        | True                       | Allow FlashArray to calculate the array oversubscription ratio.                                                                                                                                                 |
+------------------------------------------------------+----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``replication_device``                               | None                       | FlashArray Target for Replication. This option uses the format ``backend_id:<backend-id>,san_ip:<target-vip>,api_token:<target-api-token>,type:<replication-type>``                                             |
+------------------------------------------------------+----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``pure_replica_interval_default``                    | 3600                       | Snapshot replication interval in seconds.                                                                                                                                                                       |
+------------------------------------------------------+----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``pure_replica_retention_short_term_default``        | 14400                      | Retain all snapshots on target for this time (in seconds).                                                                                                                                                      |
+------------------------------------------------------+----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``pure_replica_retention_long_term_per_day_default`` | 3                          | Retain how many snapshots for each day.                                                                                                                                                                         |
+------------------------------------------------------+----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``pure_replica_retention_long_term_default``         | 7                          | Retain snapshots per day on target for this time (in days).                                                                                                                                                     |
+------------------------------------------------------+----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``pure_replication_pg_name``                         | ``cinder-group``           | Pure Protection Group name to use for async replication (will be created if it does not exist).                                                                                                                 |
+------------------------------------------------------+----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``pure_replication_pod_name``                        | ``cinder-pod``             | Pure Pod name to use for sync replication (will be created if it does not exist).                                                                                                                               |
+------------------------------------------------------+----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``pure_iscsi_cidr``                                  | ``0.0.0.0/0``              | CIDR of FlashArray iSCSI targets hosts are allowed to connect to. Default will allow connection to any IPv4 address.                                                                                            |
+------------------------------------------------------+----------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Table 4.19. Optional FlashArray Attributes

FlashArray Replication Setup
----------------------------

In order to use FlashArray with Replication enabled you must have a secondary
target backend configured and being referenced by primary host under
``replication_device`` attribute. Example:

::

    [pure]
    volume_backend_name=pure
    volume_driver=cinder.volume.drivers.pure.PureISCSIDriver
    san_ip=192.168.1.55
    pure_api_token=login
    replication_device=backend_id:pure-2,san_ip:192.168.1.32,api_token::type:async

    [pure-2]
    volume_backend_name=pure2
    volume_driver=cinder.volume.drivers.pure.PureISCSIDriver
    san_ip=192.18.1.32
    pure_api_token=

    [DEFAULT]
    enabled_backends=pure

.. note::

   The secondary FlashArray is not required to be in the ``enabled_backends``
   like in the example above.

   The secondary FlashArray is not required to be managed by OpenStack at all.

The value for the ``type`` key can be either ``sync`` or ``async``.

If the ``type`` is ``sync`` volumes will be created in a stretched ActiveCluster Pod. This
requires two arrays preconfigured with ActiveCluster enabled. You can
optionally specify ``uniform`` as ``true`` or ``false``, which will instruct
the driver that data paths are uniform between arrays in the cluster and data
connections should be made to both upon attaching.

Note that more than one ``replication_device`` line can be added to allow for
multi-target device replication.

A volume is only replicated if the volume is of a volume-type that has
the extra spec ``replication_enabled`` set to ``<is> True``. You can optionally
specify the ``replication_type`` key to specify ``<in> sync`` or ``<in> async``
to choose the type of replication for that volume. If not specified it will
default to ``async``.

To create a volume type that specifies replication to remote back ends with
async replication:

.. code-block:: console

   $ openstack volume type create ReplicationType
   $ openstack volume type set --property replication_enabled='<is> True' ReplicationType
   $ openstack volume type set --property replication_type='<in> async' ReplicationType

Refer to Table 4.19 for optional configuration parameters available
for async replication configuration.
