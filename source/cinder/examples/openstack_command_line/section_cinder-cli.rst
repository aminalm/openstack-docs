.. _cinder-cli:

Cinder Command Line Interface (CLI)
===================================

Cinder Service Verification
---------------------------

In this section, we use the Cinder CLI to verify that the configuration
presented in the section called ":ref:`cinder-conf`"
has been properly initialized by Cinder.

::

    user@opstack:~/$ openstack volume service list
    +------------------+----------------+------+---------+-------+----------------------------+-----------------+
    | Binary           | Host           | Zone | Status  | State | Updated_at                 | Disabled Reason |
    +------------------+----------------+------+---------+-------+----------------------------+-----------------+
    | cinder-scheduler | opstack        | nova | enabled | up    | 2021-01-15T21:43:51.000000 | -               |
    | cinder-volume    | opstack@pure   | nova | enabled | up    | 2021-01-15T21:43:54.000000 | -               |
    | cinder-volume    | opstack@pure-2 | nova | enabled | up    | 2021-01-15T21:43:54.000000 | -               |
    +------------------+----------------+------+---------+-------+----------------------------+-----------------+


-  ``opstack@pure`` is the backend defined by the configuration
   stanza ``[pure]``.

-  ``opstack@pure-2`` is the backend defined by the configuration
   stanza ``[pure-2]``.

.. _create-volume:

Creating and Defining Cinder Volume Types
-----------------------------------------

In this section, we create a variety of Cinder volume Types that
leverage different capabilities of the driver.

-  The ``gold`` type provisions Cinder volumes onto a specified FlashArray
   (in this example, that would be ``[pure]``.

-  The ``replicated`` type provisions Cinder volumes onto any backend that
   has replicated enabled .

::

    $ openstack volume type create gold
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | description | None                                 |
    | id          | 562d2673-9778-45d0-a3b9-76399559ddab |
    | is_public   | True                                 |
    | name        | gold                                 |
    +-------------+--------------------------------------+

::

    $ openstack volume type create replicated
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | description | None                                 |
    | id          | c0e40b80-6b29-4a46-b0e6-18cc5bb82e6d |
    | is_public   | True                                 |
    | name        | replicated                           |
    +-------------+--------------------------------------+

::

    $ openstack volume type set --property "volume_backend_name=pure" gold

::

    $ openstack volume type set --property replication_enabled="<is> True" replicated

::

    $ openstack volume type list --long
    +--------------------------------------+-------------+-----------+---------------------+------------------------------------+
    | ID                                   | Name        | Is Public | Description         | Properties                         |
    +--------------------------------------+-------------+-----------+---------------------+------------------------------------+
    | c0e40b80-6b29-4a46-b0e6-18cc5bb82e6d | replicated  | True      | None                | replication_enabled='<is> True'    |
    | 562d2673-9778-45d0-a3b9-76399559ddab | gold        | True      | None                | volume_backend_name='pure'         |
    | dc2e583a-6ac3-4877-b8fa-a7abda06abe0 | __DEFAULT__ | True      | Default Volume Type |                                    |
    +--------------------------------------+-------------+-----------+---------------------+------------------------------------+

Creating Cinder Volumes with Volume Types
-----------------------------------------

In this section, we create volumes with no type, as well as each of the
previously defined volume types.

::

    $ openstack volume create --type gold --size 1 myGold
    +---------------------+--------------------------------------+
    | Field               | Value                                |
    +---------------------+--------------------------------------+
    | attachments         | []                                   |
    | availability_zone   | nova                                 |
    | bootable            | false                                |
    | consistencygroup_id | None                                 |
    | created_at          | 2021-01-20T20:10:03.000000           |
    | description         | None                                 |
    | encrypted           | False                                |
    | id                  | d0b0a399-0e56-4eb1-bbb5-51f7fa12fb7b |
    | migration_status    | None                                 |
    | multiattach         | False                                |
    | name                | myGold                               |
    | properties          |                                      |
    | replication_status  | None                                 |
    | size                | 1                                    |
    | snapshot_id         | None                                 |
    | source_volid        | None                                 |
    | status              | creating                             |
    | type                | gold                                 |
    | updated_at          | None                                 |
    | user_id             | db7c175b4f914538ad4c3240cdc72dc4     |
    +---------------------+--------------------------------------+

::

    $ openstack volume create --type replicated --size 1 myReplicated
    +---------------------+--------------------------------------+
    | Field               | Value                                |
    +---------------------+--------------------------------------+
    | attachments         | []                                   |
    | availability_zone   | nova                                 |
    | bootable            | false                                |
    | consistencygroup_id | None                                 |
    | created_at          | 2021-01-20T20:11:47.000000           |
    | description         | None                                 |
    | encrypted           | False                                |
    | id                  | f9c54059-ac1f-4118-851a-136c9fc0c2a1 |
    | migration_status    | None                                 |
    | multiattach         | False                                |
    | name                | myReplicated                         |
    | properties          |                                      |
    | replication_status  | None                                 |
    | size                | 1                                    |
    | snapshot_id         | None                                 |
    | source_volid        | None                                 |
    | status              | creating                             |
    | type                | replicated                           |
    | updated_at          | None                                 |
    | user_id             | db7c175b4f914538ad4c3240cdc72dc4     |
    +---------------------+--------------------------------------+

::

    $ openstack volume list
    +--------------------------------------+-------------------+-----------+------+-------------+
    | ID                                   | Name              | Status    | Size | Attached to |
    +--------------------------------------+-------------------+-----------+------+-------------+
    | f9c54059-ac1f-4118-851a-136c9fc0c2a1 | myReplicated      | available |    1 |             |
    | d0b0a399-0e56-4eb1-bbb5-51f7fa12fb7b | myGold            | available |    1 |             |
    +--------------------------------------+-------------------+-----------+------+-------------+

.. _cinder-manage:

Cinder Manage Usage
-------------------

In this section we import an FlashArray LUN by specifying it by
name or UUID.

::

    $ cinder get-pools
    +----------+-------------------+
    | Property | Value             |
    +----------+-------------------+
    | name     | opstack@pure#pure |
    +----------+-------------------+

::

    $ cinder --os-volume-api-version 3.8 manageable-list opstack@puredriver-1#puredriver-1
    +----------------------------------------------------------------+------+----------------+---------------------------------------------------------------------------+--------------------------------------+------------+
    | reference                                                      | size | safe_to_manage | reason_not_safe                                                           | cinder_id                            | extra_info |
    +----------------------------------------------------------------+------+----------------+---------------------------------------------------------------------------+--------------------------------------+------------+
    | {'name': 'volume-e898e997-1254-4c56-9136-239b1568fe0b-cinder'} | 1    | False          | Volume connected to host devstack-bbabda406f7846c6b0d9e1ec53761419-cinder | -                                    | -          |
    | {'name': 'volume-648bcd17-e36e-41b3-8087-391544e3b9ac-cinder'} | 1    | False          | Volume already managed                                                    | 648bcd17-e36e-41b3-8087-391544e3b9ac | -          |
    | {'name': 'volume-17eb768e-8758-44ff-afc1-45720c1b19f1-cinder'} | 1    | False          | Volume already managed                                                    | 17eb768e-8758-44ff-afc1-45720c1b19f1 | -          |
    | {'name': 'manage-me'}                                          | 2    | True           |                                                                           | -                                    | -          |
    +----------------------------------------------------------------+------+----------------+---------------------------------------------------------------------------+--------------------------------------+------------+

::

    $ cinder manage --id-type name --volume-type pure --name newly-managed opstack@pure#pure manage-me
    +--------------------------------+--------------------------------------+
    | Property                       | Value                                |
    +--------------------------------+--------------------------------------+
    | attachments                    | []                                   |
    | availability_zone              | nova                                 |
    | bootable                       | false                                |
    | consistencygroup_id            | None                                 |
    | created_at                     | 2021-01-20T20:26:36.000000           |
    | description                    | None                                 |
    | encrypted                      | False                                |
    | id                             | e8f6a986-0c4f-4330-a694-428cf396e66c |
    | metadata                       | {}                                   |
    | migration_status               | None                                 |
    | multiattach                    | False                                |
    | name                           | newly-managed                        |
    | os-vol-host-attr:host          | opstack@pure#pure                    |
    | os-vol-mig-status-attr:migstat | None                                 |
    | os-vol-mig-status-attr:name_id | None                                 |
    | os-vol-tenant-attr:tenant_id   | 90da20335bc546f7989bd1aa6e8c373c     |
    | replication_status             | None                                 |
    | size                           | 0                                    |
    | snapshot_id                    | None                                 |
    | source_volid                   | None                                 |
    | status                         | creating                             |
    | updated_at                     | 2021-01-20T20:26:36.000000           |
    | user_id                        | db7c175b4f914538ad4c3240cdc72dc4     |
    | volume_type                    | pure                                 |
    +--------------------------------+--------------------------------------+

::

    $ openstack volume list --long
    +--------------------------------------+-------------------+-----------+------+------------+----------+-------------+------------+
    | ID                                   | Name              | Status    | Size | Type       | Bootable | Attached to | Properties |
    +--------------------------------------+-------------------+-----------+------+------------+----------+-------------+------------+
    | e8f6a986-0c4f-4330-a694-428cf396e66c | newly-managed     | available |    2 | pure       | false    |             |            |
    | f9c54059-ac1f-4118-851a-136c9fc0c2a1 | myReplicated      | available |    1 | replicated | false    |             |            |
    | d0b0a399-0e56-4eb1-bbb5-51f7fa12fb7b | myGold            | available |    1 | gold       | false    |             |            |
    +--------------------------------------+-------------------+-----------+------+------------+----------+-------------+------------+

.. _cinder-unmanage:

Cinder Unmanage Usage
---------------------

In this section we unmanage a Cinder volume by specifying its ID.

::

    $ openstack volume list --long
    +--------------------------------------+-------------------+-----------+------+------------+----------+-------------+------------+
    | ID                                   | Name              | Status    | Size | Type       | Bootable | Attached to | Properties |
    +--------------------------------------+-------------------+-----------+------+------------+----------+-------------+------------+
    | e8f6a986-0c4f-4330-a694-428cf396e66c | newly-managed     | available |    2 | pure       | false    |             |            |
    | f9c54059-ac1f-4118-851a-136c9fc0c2a1 | myReplicated      | available |    1 | replicated | false    |             |            |
    | d0b0a399-0e56-4eb1-bbb5-51f7fa12fb7b | myGold            | available |    1 | gold       | false    |             |            |
    +--------------------------------------+-------------------+-----------+------+------------+----------+-------------+------------+

::

    $ cinder unmanage e8f6a986-0c4f-4330-a694-428cf396e66c

::

    $ openstack volume list --long
    +--------------------------------------+-------------------+-----------+------+------------+----------+-------------+------------+
    | ID                                   | Name              | Status    | Size | Type       | Bootable | Attached to | Properties |
    +--------------------------------------+-------------------+-----------+------+------------+----------+-------------+------------+
    | f9c54059-ac1f-4118-851a-136c9fc0c2a1 | myReplicated      | available |    1 | replicated | false    |             |            |
    | d0b0a399-0e56-4eb1-bbb5-51f7fa12fb7b | myGold            | available |    1 | gold       | false    |             |            |
    +--------------------------------------+-------------------+-----------+------+------------+----------+-------------+------------+

Applying Cinder QoS via the Command Line
----------------------------------------

In this section, we will configure a Cinder volume type, a Cinder QoS
spec, and lastly associate the QoS spec with the volume type.

::

    $ openstack volume type create vol_type_qos_demo
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | description | None                                 |
    | id          | a8664c72-0501-45a5-8e02-4ae682d4dc21 |
    | is_public   | True                                 |
    | name        | vol_type_qos_demo                    |
    +-------------+--------------------------------------+

::

    $ openstack volume qos create --consumer back-end --property "maxIOPS=100" qos_demo
    +------------+--------------------------------------+
    | Field      | Value                                |
    +------------+--------------------------------------+
    | consumer   | back-end                             |
    | id         | 6adea28c-926a-467b-99a9-cb7f138311d3 |
    | name       | qos_demo                             |
    | properties | maxIOPS='100'                        |
    +------------+--------------------------------------+

::

    $ openstack volume qos associate qos_demo vol_type_qos_demo

::

    $ openstack volume qos list
    +--------------------------------------+----------+----------+-------------------+---------------+
    | ID                                   | Name     | Consumer | Associations      | Properties    |
    +--------------------------------------+----------+----------+-------------------+---------------+
    | 6adea28c-926a-467b-99a9-cb7f138311d3 | qos_demo | back-end | vol_type_qos_demo | maxIOPS='100' |
    +--------------------------------------+----------+----------+-------------------+---------------+

::

    $ openstack volume create --type vol_type_qos_demo --size 1 qos_vol
    +---------------------+--------------------------------------+
    | Field               | Value                                |
    +---------------------+--------------------------------------+
    | attachments         | []                                   |
    | availability_zone   | nova                                 |
    | bootable            | false                                |
    | consistencygroup_id | None                                 |
    | created_at          | 2021-01-20T20:40:12.000000           |
    | description         | None                                 |
    | encrypted           | False                                |
    | id                  | 017a7c69-db2a-467b-add9-3a71af75096e |
    | migration_status    | None                                 |
    | multiattach         | False                                |
    | name                | qos_vol                              |
    | properties          |                                      |
    | replication_status  | None                                 |
    | size                | 1                                    |
    | snapshot_id         | None                                 |
    | source_volid        | None                                 |
    | status              | creating                             |
    | type                | vol_type_qos_demo                    |
    | updated_at          | None                                 |
    | user_id             | db7c175b4f914538ad4c3240cdc72dc4     |
    +---------------------+--------------------------------------+

After we associate the QoS spec with the volume type, we can use the
volume type just as we did in the section called
:ref:`“Creating and Defining Cinder volume Types”<create-volume>`.

Manipulating Cinder Consistency Groups via the Command Line
-----------------------------------------------------------

.. note::

   Support for Consistency groups has been deprecated in Block Storage V3
   API. Only Block Storage V2 API supports consistency groups. Future
   releases will involve a migration of existing consistency group operations
   to use generic volume group operations.

In this section, we will configure a Cinder volume type, associate the
volume type with a backend capable of supporting consistency groups,
create a Cinder consistency group, create a Cinder volume within the
consistency group, take a snapshot of the consistency group, and then
finally create a second consistency group from the snapshot of the first
consistency group.

::

    $ openstack volume type create consistency-group-support
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | description | None                                 |
    | id          | a65fab80-5898-4ea0-93e9-3c404783426e |
    | is_public   | True                                 |
    | name        | consistency-group-support            |
    +-------------+--------------------------------------+

::

    $ openstack volume type set --property volume_backend_name=<BACKEND_WITH_CG_SUPPORT> consistency-group-support

::

    $ openstack consistency group create --volume-type consistency-group-support cg1
    +-------+--------------------------------------+
    | Field | Value                                |
    +-------+--------------------------------------+
    | id    | 714e752d-500a-42b3-9711-93de2a5471e2 |
    | name  | cg1                                  |
    +-------+--------------------------------------+

::

    $ openstack volume create --consistency-group cg1 --size 1 --type consistency-group-support vol-in-cg1
    +---------------------+--------------------------------------+
    | Field               | Value                                |
    +---------------------+--------------------------------------+
    | attachments         | []                                   |
    | availability_zone   | nova                                 |
    | bootable            | false                                |
    | consistencygroup_id | 714e752d-500a-42b3-9711-93de2a5471e2 |
    | created_at          | 2021-01-20T20:50:33.000000           |
    | description         | None                                 |
    | encrypted           | False                                |
    | id                  | 5c2aea90-33c7-4aaf-a1a8-9e6b2f3775fc |
    | migration_status    | None                                 |
    | multiattach         | False                                |
    | name                | vol-in-cg1                           |
    | properties          |                                      |
    | replication_status  | None                                 |
    | size                | 1                                    |
    | snapshot_id         | None                                 |
    | source_volid        | None                                 |
    | status              | creating                             |
    | type                | consistency-group-support            |
    | updated_at          | None                                 |
    | user_id             | db7c175b4f914538ad4c3240cdc72dc4     |
    +---------------------+--------------------------------------+

::

    $ cinder cgsnapshot-create 2cc3d172-af05-421b-babd-01d4cd91078d --name snap-of-cg1
    +---------------------+--------------------------------------+
    |       Property      |                Value                 |
    +---------------------+--------------------------------------+
    | consistencygroup_id | 1e875dfe-e213-43c6-a365-12610b92341b |
    |      created_at     |      2016-02-29T16:01:30.000000      |
    |     description     |                 None                 |
    |          id         | cd3770e1-fa59-48a6-ba48-2f3581f2b03b |
    |         name        |             snap-of-cg1              |
    |        status       |               creating               |
    +---------------------+--------------------------------------+

::

    $ openstack consistency group snapshot create --consistency-group cg1 snap-of-cg1
    +-------+--------------------------------------+
    | Field | Value                                |
    +-------+--------------------------------------+
    | id    | b2bf7a53-d393-4abb-9ec6-b43cf7c50ee8 |
    | name  | snap-of-cg1                          |
    +-------+--------------------------------------+

To delete a consistency group, first make sure that any snapshots of the
consistency group have first been deleted, and that any volumes in the
consistency group have been removed via an update command on the
consistency group.

::

    $ openstack consistency group snapshot delete snap-of-cg1

::

    $ openstack consistency group remove volume cg1 vol-in-cg1

::

    $ openstack volume delete vol-in-cg1

::

    $ openstack consistency group delete cg1


Manipulating Cinder Groups via the Command Line
-----------------------------------------------------------
In this section, we will configure a Cinder volume type, associate the
volume type with a backend capable of supporting groups, create a Cinder
group type, create a Cinder group, create a Cinder volume within the group,
take a snapshot of the group, and then finally create a group from the
snapshot of the first group.

.. note::
   Currently only the Block Storage V3 API supports group operations. The
   minimum version for group operations supported by the FlashArray drivers is
   3.14. The API version can be specified with the following CLI flag
   ``--os-volume-api-version 3.14``. Optionally an environment variable can
   be set: ``export OS_VOLUME_API_VERSION=3.14``

.. note::
   The Cinder community plans to migrate existing consistency group operations
   to group operations in an upcoming release. Please review Cinder
   release notes for upgrade instructions prior to using group operations.

.. note::
   The FlashArray volume drivers support the consistent_group_snapshot_enabled
   group type. By default Cinder group snapshots take individual snapshots
   of each Cinder volume in the group. To enable consistency group snapshots set
   ``consistent_group_snapshot_enabled="<is> True"`` in the group type used.

::

    $ openstack volume type create volume-support
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | description | None                                 |
    | id          | 08750a07-2d28-44ff-96a6-77f0b7980414 |
    | is_public   | True                                 |
    | name        | volume-support                       |
    +-------------+--------------------------------------+

::

    $ openstack volume type set --property volume_backend_name=<BACKEND_WITH_CG_SUPPORT> volume-support

::

    $ cinder --os-volume-api-version 3.14 group-type-create group-support
    +--------------------------------------+---------------+-------------+
    | ID                                   | Name          | Description |
    +--------------------------------------+---------------+-------------+
    | bda7548f-18a2-4aed-bce9-f598a3d39312 | group-support | -           |
    +--------------------------------------+---------------+-------------+

::

    $ cinder --os-volume-api-version 3.14 group-type-key group-support set consistent_group_snapshot_enabled="<is> True"

::

    $ cinder --os-volume-api-version 3.14 group-create --name group1 group-support volume-support
    +-------------------+------------------------------------------+
    | Property          | Value                                    |
    +-------------------+------------------------------------------+
    | availability_zone | nova                                     |
    | created_at        | 2021-01-20T21:34:52.000000               |
    | description       | None                                     |
    | group_snapshot_id | None                                     |
    | group_type        | bda7548f-18a2-4aed-bce9-f598a3d39312     |
    | id                | a4ba4b81-2a38-445f-96db-f173d08a66fe     |
    | name              | group1                                   |
    | source_group_id   | None                                     |
    | status            | creating                                 |
    | volume_types      | ['08750a07-2d28-44ff-96a6-77f0b7980414'] |
    +-------------------+------------------------------------------+

::

    $ cinder --os-volume-api-version 3.14 create --name vol-in-group1 --group-id a4ba4b81-2a38-445f-96db-f173d08a66fe --volume-type volume-support 1
    +--------------------------------+--------------------------------------+
    | Property                       | Value                                |
    +--------------------------------+--------------------------------------+
    | attachments                    | []                                   |
    | availability_zone              | nova                                 |
    | bootable                       | false                                |
    | consistencygroup_id            | None                                 |
    | created_at                     | 2021-01-20T21:37:12.000000           |
    | description                    | None                                 |
    | encrypted                      | False                                |
    | group_id                       | a4ba4b81-2a38-445f-96db-f173d08a66fe |
    | id                             | 41e88376-b3e3-4978-8c77-3c9017361a6f |
    | metadata                       | {}                                   |
    | migration_status               | None                                 |
    | multiattach                    | False                                |
    | name                           | vol-in-group1                        |
    | os-vol-host-attr:host          | cinder2@puredriver-1#puredriver-1    |
    | os-vol-mig-status-attr:migstat | None                                 |
    | os-vol-mig-status-attr:name_id | None                                 |
    | os-vol-tenant-attr:tenant_id   | 90da20335bc546f7989bd1aa6e8c373c     |
    | replication_status             | None                                 |
    | size                           | 1                                    |
    | snapshot_id                    | None                                 |
    | source_volid                   | None                                 |
    | status                         | creating                             |
    | updated_at                     | 2021-01-20T21:37:12.000000           |
    | user_id                        | db7c175b4f914538ad4c3240cdc72dc4     |
    | volume_type                    | volume-support                       |
    +--------------------------------+--------------------------------------+

::

    $ cinder --os-volume-api-version 3.14 group-snapshot-create group1 --name group1-snapshot1
    +---------------+--------------------------------------+
    | Property      | Value                                |
    +---------------+--------------------------------------+
    | created_at    | 2021-01-20T21:38:06.000000           |
    | description   | None                                 |
    | group_id      | a4ba4b81-2a38-445f-96db-f173d08a66fe |
    | group_type_id | bda7548f-18a2-4aed-bce9-f598a3d39312 |
    | id            | 1e76e70a-258b-4fc3-b056-f98f4fa458a5 |
    | name          | group1-snapshot1                     |
    | status        | creating                             |
    +---------------+--------------------------------------+

::

    $ cinder --os-volume-api-version 3.14 group-create-from-src --group-snapshot group1-snapshot1 --name group2
    +----------+--------------------------------------+
    | Property | Value                                |
    +----------+--------------------------------------+
    | id       | 1b971923-64f2-45c0-864d-8e5f49315dca |
    | name     | group2                               |
    +----------+--------------------------------------+


Revert to Snapshot
------------------
In this section, we will create a new volume, take a snapshot from it and
revert to that last snapshot.

.. note::
   This command is only available in microversion 3.40 and above.

.. note::
   You can only revert the volume to the last snapshot taken. If you need to
   revert to an earlier snapshot, you have to delete snapshots until that one
   is the most recent.

.. note::
   The snapshot being reverted to must have the same size of the volume.

::

    $ openstack volume create --type gold --size 1 cinder-vol-1
    +---------------------+--------------------------------------+
    | Field               | Value                                |
    +---------------------+--------------------------------------+
    | attachments         | []                                   |
    | availability_zone   | nova                                 |
    | bootable            | false                                |
    | consistencygroup_id | None                                 |
    | created_at          | 2021-01-20T21:41:47.000000           |
    | description         | None                                 |
    | encrypted           | False                                |
    | id                  | 974198c4-149e-4d72-b0fb-962ea0123b9f |
    | migration_status    | None                                 |
    | multiattach         | False                                |
    | name                | cinder-vol-1                         |
    | properties          |                                      |
    | replication_status  | None                                 |
    | size                | 1                                    |
    | snapshot_id         | None                                 |
    | source_volid        | None                                 |
    | status              | creating                             |
    | type                | gold                                 |
    | updated_at          | None                                 |
    | user_id             | db7c175b4f914538ad4c3240cdc72dc4     |
    +---------------------+--------------------------------------+

::

    $ openstack volume snapshot create cinder-vol-1
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | created_at  | 2021-01-20T21:44:30.247286           |
    | description | None                                 |
    | id          | 29bdba4c-956c-46dd-9179-5a5931318375 |
    | name        | cinder-vol-1                         |
    | properties  |                                      |
    | size        | 1                                    |
    | status      | creating                             |
    | updated_at  | None                                 |
    | volume_id   | 974198c4-149e-4d72-b0fb-962ea0123b9f |
    +-------------+--------------------------------------+

::

    $ cinder --os-volume-api-version=3.40 revert-to-snapshot 29bdba4c-956c-46dd-9179-5a5931318375
