.. _gic-fa-iscsi-or-fc:

Glance Image Cache for Cinder with FlashArray iSCSI/FC as a Cinder Backend
==========================================================================

The following checklist provides the steps necessary for configuration
of Glance Image Cache for Cinder with FlashArray for iSCSI or Fibre Channel:

+-----+-------------------------------------------------------+---------+
| Step| Description of Step                                   | Done?   |
+=====+=======================================================+=========+
| 1   | Configure internal tenant settings in cinder.conf     |         |
+-----+-------------------------------------------------------+---------+
| 2   | Configure Image-Volume cache setting in cinder.conf   |         |
+-----+-------------------------------------------------------+---------+
| 3   | Restart Cinder and Glance services                    |         |
+-----+-------------------------------------------------------+---------+
| 4   | Upload a Glance Image                                 |         |
+-----+-------------------------------------------------------+---------+
| 5   | Boot from Cinder                                      |         |
+-----+-------------------------------------------------------+---------+
| 6   | Verify functionality                                  |         |
+-----+-------------------------------------------------------+---------+

Table 5.1: Checklist of Steps for Enhanced Instance Creation

|

1) Configure internal tenant settings in cinder.conf

Obtain the ``cinder_internal_tenant_project_id``

::

    $ openstack service list
    +----------------------------------+-------------+----------------+
    | ID                               | Name        | Type           |
    +----------------------------------+-------------+----------------+
    | 185c30110a134e599b744e2d084504f6 | glance      | image          |
    | 455f9e2b1440450fb016619a08b57a0f | cinderv2    | volumev2       |
    | 541701f5857e4ba5ac8256ff45ce183b | neutron     | network        |
    | 54749405a293416da8156469e1673c06 | cinderv3    | volumev3       |
    | 7f133836fbb1458e9e4acac04269d5c8 | placement   | placement      |
    | 94cfcb74a6d04e5fae66e06dc05b77b2 | cinder      | block-storage  |
    | 94ddb7fa2daa4d1eac82814e6d1467f1 | keystone    | identity       |
    | 9ef1d17fe1154994bf0aaba6a05b8045 | nova        | compute        |
    | c80cdd3eff674a9fb52d26a9f04fdc6d | nova_legacy | compute_legacy |
    +----------------------------------+-------------+----------------+

Edit cinder.conf to contain the following entry in the DEFAULT stanza

::

    [DEFAULT]
    ...
    cinder_internal_tenant_project_id=6763e676132f4aaabb68cc1517b18d38
    ...

Obtain the ``cinder_internal_tenant_user_id``

::

    $ openstack user list
    +----------------------------------+----------+
    | ID                               | Name     |
    +----------------------------------+----------+
    | c008a02d553a4f9595d8437da67996b4 | admin    |
    +----------------------------------+----------+

Edit cinder.conf to contain the following entry in the DEFAULT stanza

::

    [DEFAULT]
    ...
    cinder_internal_tenant_user_id=a05232baaeda49b589b11a3198efb054
    ...

2) Configure Image-Volume cache settings in the FlashArray stanza in cinder.conf

::

    [FA-1]
    ...
    image_volume_cache_enabled = True
    ...

3) Restart Cinder services

::

    $ systemctl restart openstack-cinder-{api,scheduler,volume}

.. note::

    Depending on your deployment mechanism the cinder service names
    may vary. Refer to your deployment installation guide for more
    information.

4) Upload a Glance image

The following command uses an image that is publicly available. Please
use the image you prefer and replace the URL accordingly.

::

    $ wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64-disk-kvm.img | openstack image create ubutu-focal-image --container-format bare --disk-format qcow2 --file focal-server-cloudimg-amd64-disk-kvm.img --min-disk 3

5) Boot from Cinder

::

    $ nova boot --flavor m1.medium --key-name openstack_key --nic net-id=replace-with-neutron-net-id --block-device source=image,id=replace-with-glance-image-id,dest=volume,shutdown=preserve,bootindex=0,size=5  ubuntu-vm

6) Verify functionality

Please open /var/log/cinder/volume.log and look for a message similar to
the following to confirm that the image-volume was cached successfully::

    ...
    2016-09-30 16:38:52.211 DEBUG cinder.volume.flows.manager.create_volume [req-9ea8022f-1dd4-4203-b1f3-019f3c1b377a None None] Downloaded image 16d996d3-87aa-47da-8c82-71a21e8a06fb ((None, None)) to volume 6944e5be-7c56-4a7d-a90b-5231e7e94a6e successfully. from (pid=20926) _copy_image_to_volume /opt/stack/cinder/cinder/volume/flows/manager/create_volume.py
    ...

7) Additional Options

For additional configuration options and details refer to the
`Pure Glance Image Cache whitepaper. <https://support.purestorage.com/Solutions/OpenStack/z_Legacy_OpenStack_Reference/OpenStack%C2%AE_Liberty%3A_A_Look_at_the_Glance_Image-Cache_for_Cinder>`_
