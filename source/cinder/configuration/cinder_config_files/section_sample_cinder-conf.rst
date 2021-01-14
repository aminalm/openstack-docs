.. _cinder-conf:

Sample cinder.conf
==================

This section provides an example Cinder configuration file
(``cinder.conf``) that contains one FlashArray backend.

::

    [DEFAULT]
    rabbit_password=password123
    rabbit_hosts=192.168.33.40
    rpc_backend=cinder.openstack.common.rpc.impl_kombu
    notification_driver=cinder.openstack.common.notifier.rpc_notifier
    periodic_interval=60
    enable_v2_api=false
    enable_v3_api=true
    lock_path=/opt/stack/data/cinder
    state_path=/opt/stack/data/cinder
    osapi_volume_extension=cinder.api.contrib.standard_extensions
    rootwrap_config=/etc/cinder/rootwrap.conf
    api_paste_config=/etc/cinder/api-paste.ini
    sql_connection=mysql://root:password123@127.0.0.1/cinder?charset=utf8
    my_ip=192.168.33.40
    verbose=True
    debug=True
    auth_strategy=keystone
    #ceilometer settings
    cinder_volume_usage_audit=True
    cinder_volume_usage_audit_period=hour
    control_exchange=cinder

    enabled_backends=pure

    [pure]
    volume_backend_name=pure
    volume_driver=cinder.volume.drivers.pure.PureISCSIDriver
    san_ip=12.168.1.32
    pure_api_token=c6033033-fe69-2515-a9e8-966bb7fe4b40
