Deploying Pure Storage FlashArray Cinder driver in a Red Hat OpenStack Platform 16.0
====================================================================================

.. _purestorage-flsharray-rhosp:

Overview
--------

This guide shows how to configure and deploy the Pure Storage FlashArray Cinder driver in a
**Red Hat OpenStack Platform (RHOSP) 16.0** Overcloud, using RHOSP Director.
After reading this, you'll be able to define the proper environment files and
deploy single or multiple FlashArray Cinder back ends in RHOSP Overcloud Controller
nodes.

.. note::

  For more information about RHOSP, please refer to its `documentation pages
  <https://access.redhat.com/documentation/en-us/red_hat_openstack_platform>`_.

.. warning::

  RHOSP16.0 is based on OpenStack Train release. Features included after Train
  release are not available in RHOSP16.0.

Requirements
------------

In order to deploy Pure Storage FlashArray Cinder back ends, you should have the
following requirements satisfied:

- Pure Storage FlashArrays deployed and ready to be used as Cinder
  back ends. See :ref:`cinder_flasharray_prerequisites` for more details.

- RHOSP Director user credentials to deploy Overcloud.

- RHOSP Overcloud Controller nodes where Cinder services will be installed.


Deployment Steps
----------------

Prepare the environment files
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

RHOSP makes use of **TripleO Heat Templates (THT)**, which allows you to define
the Overcloud resources by creating environment files.

To ensure that your RHOSP environment is correctly configured for using
Pure Storage FlashArrays obtain a copy of `pure-temp.yaml <https://raw.githubusercontent.com/PureStorage-OpenConnect/tripleo-deployment-configs/master/RHOSP16/pure-temp.yaml>`__
and save this in the ``/home/stack/templates`` directory. This will be required when
deploying the Overcloud.

Multiple back end configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Define Pure Storage Cinder back ends using Custom THT Configuration syntax.
It's possible to define all the back ends in a single environment file, but for
sake of clarity, the following example organizes the back ends in multiple
smaller environment files:

- ``/home/stack/templates/cinder-flasharray-backend1.yaml``

  This file defines the first Cinder volume back end
  ``tripleo_pure_1`` and its parameters:

  .. code-block:: yaml
    :name: cinder-flasharray-backend1.yaml

    parameter_defaults:
      ControllerExtraConfig:
        cinder::config::cinder_config:
          tripleo_pure_1/CinderPureBackendName:
            value: tripleo_pure_1
          tripleo_pure_1/CinderPureStorageProtocol:
            value: 'iSCSI'
          tripleo_pure_1/CinderPureSanIp:
            value: san_ip_1
          tripleo_pure_1/CinderPureAPIToken:
            value: api_token_1

- ``/home/stack/templates/cinder-purestorage-backend2.yaml``

  This file defines the second Cinder volume back end
  ``tripleo_pure_2`` and its parameters:

  .. code-block:: yaml
    :name: cinder-flasharray-backend2.yaml

    parameter_defaults:
      ControllerExtraConfig:
        cinder::config::cinder_config:
          tripleo_pure_2/CinderPureBackendName:
            value: tripleo_pure_2
          tripleo_pure_2/CinderPureStorageProtocol:
            value: 'iSCSI'
          tripleo_pure_2/CinderPureSanIp:
            value: san_ip_2
          tripleo_pure_2/CinderPureAPIToken:
            value: api_token_2

  Modify the parameter values according to your Pure Storage back end
  configuration.

- ``/home/stack/templates/cinder-enabled-backends.yaml``

  This file defines which back ends will be enabled. In this example, two
  back ends ``tripleo_pure_1`` and ``tripleo_pure_2`` will be
  enabled:

  .. code-block:: yaml
    :name: cinder-enabled-backends.yaml

    parameter_defaults:
      ControllerExtraConfig:
        cinder_user_enabled_backends:
          - 'tripleo_pure_1'
          - 'tripleo_pure_2'

.. note::

  You can define arbitrary Custom THT Configurations using the following syntax:

  .. code-block:: yaml
      :name: custom-config.yaml

      parameter_defaults:
        ControllerExtraConfig:
          cinder::config::cinder_config:
            <backend_name>/<configuration_name>:
              value: <value>

  Each configuration will be rendered in ``cinder.conf`` file as:

  .. code-block::
      :name: cinder.conf

      [backend_name]
      configuration_name=value

  See `Optional Cinder Configuration Attributes (RHOSP16)
  <./section_flasharray-conf-train.html#optional-cinder-configuration-attributes>`_
  for a complete list of the available Cinder Configuration Options.

.. warning::

  RHOSP16.0 is based on OpenStack Train release. Features and Configuration
  Options included after Train release are not available in RHOSP16.0.


Use Certified Pure Storage Cinder Volume Container
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Red Hat requires that you utilize the Certified Pure Storage Cinder Volume
Container when deploying RHOSP16.0 with a Pure Storage FlashArray backend.

This container can be found in the `Red Hat Container Catalog <https://catalog.redhat.com/software/containers/search?q=pure&p=1>`__
and should be stored in a local registry.

Alternatively, you may build your own version of this container and store it
within a local registry.

Follow these steps to build your own version of the Pure Storage Cinder Volume
container:

 * Obtain a copy of the `Dockerfile <https://raw.githubusercontent.com/PureStorage-OpenConnect/tripleo-deployment-configs/master/RHOSP16/Dockerfile>`__

 * Login to the Red Hat registry

 .. code-block:: bash

    sudo buildah login registry.redhat.io

 * Build the podman image

 .. code-block:: bash
 
    sudo buildah bud . -t "openstack-cinder-volume-pure:latest"

 * Push the new image to a local registry

 .. code-block:: bash

    sudo openstack tripleo container image push --local <registry:port>/<directory>/openstack-cinder-volume-pure:latest

Create a Custom Environment File
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create a new environment file ``custom_container_pure.yaml`` in the directory
``/home/stack/templates`` with only the custom container parameter.

.. code-block:: bash

  parameter_defaults:
    DockerCinderVolumeImage: <registry:port>/<directory>/openstack-cinder-volume-pure:latest

Alternatively, you may edit the container images environment file (usually
``overcloud_images.yaml``, created when the ``openstack overcloud container
image prepare`` command was executed) and change the appropriate
parameter to use the custom container image.

Deploy Overcloud
^^^^^^^^^^^^^^^^

Now that you have the Cinder back end environment files defined, you can run
the command to deploy the RHOSP Overcloud. Run the following command as
the ``stack`` user in the RHOSP Director command line, specifying the
YAML file(s) you defined:

.. code-block:: bash
  :name: overcloud-deploy

   (undercloud) [stack@rhosp16-undercloud ~]$ openstack overcloud deploy \
   --templates \
   -e /home/stack/containers-prepare-parameter.yaml \
   -e /home/stack/templates/cinder-flasharray-backend1.yaml \
   -e /home/stack/templates/cinder-flasharray-backend2.yaml \
   -e /home/stack/templates/cinder-enabled-backends.yaml \
   -e /home/stack/templates/custom_container_pure.yaml \
   --stack overcloud

If you modified the container images environment file the
``custom_container_pure.yaml`` line is not required above.

.. note::
  Alternatively, you can use ``--environment-directory`` parameter and specify
  the whole directory to the deployment command. It will consider all the YAML
  files within this directory:

  .. code-block:: bash
    :name: overcloud-deploy-environment-directory

     (undercloud) [stack@rhosp16-undercloud ~]$ openstack overcloud deploy \
     --templates \
     -e /home/stack/containers-prepare-parameter.yaml \
     --environment-directory /home/stack/templates \
     --stack overcloud


Test the Deployed Back Ends
^^^^^^^^^^^^^^^^^^^^^^^^^^^

After RHOSP Overcloud is deployed, run the following command to check if the
Cinder services are up:

.. code-block:: bash
  :name: cinder-service-list

  [stack@rhosp16-undercloud ~]$ source ~/overcloudrc
  (overcloud) [stack@rhosp16-undercloud ~]$ cinder service-list


Run the following commands as ``stack`` user in the RHOSP Director command line
to create the volume types mapped to the deployed back ends:

.. code-block:: bash
  :name: create-volume-types

  [stack@rhosp16-undercloud ~]$ source ~/overcloudrc
  (overcloud) [stack@rhosp16-undercloud ~]$ cinder type-create pure1
  (overcloud) [stack@rhosp16-undercloud ~]$ cinder type-key pure1 set volume_backend_name=tripleo_pure_1
  (overcloud) [stack@rhosp16-undercloud ~]$ cinder type-create pure2
  (overcloud) [stack@rhosp16-undercloud ~]$ cinder type-key pure2 set volume_backend_name=tripleo_pure_2

Make sure that you're able to create Cinder volumes with the configured volume
types:

.. code-block:: bash
  :name: create-volumes

  [stack@rhosp16-undercloud ~]$ source ~/overcloudrc
  (overcloud) [stack@rhosp16-undercloud ~]$ cinder create --volume-type pure1 --name v1 1
  (overcloud) [stack@rhosp16-undercloud ~]$ cinder create --volume-type pure2 --name v2 1

