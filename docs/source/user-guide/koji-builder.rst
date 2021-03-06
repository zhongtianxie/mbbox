============
koji-builder
============

This page documents the usage of koji-builder crd.

Description
===========

MBox utilizes koji-builder to create a new repositories when needed.

Dependencies
============

`Koji-Builder Custom Resource Definition (CRD) <https://raw.githubusercontent.com/fedora-infra/mbbox/master/mbox-operator/config/crd/bases/apps.fedoraproject.org_mbkojibuilders.yaml>`_

Koji builder depends on `koji-hub <koji-hub.html#koji-hub>`_. This component is deployed as part of the operator deployment.

Parameters
==========

+----------------------+------------------------------------+----------+
| Name                 | Default Value                      | Type     |
+======================+====================================+==========+
| image                | quay.io/fedora/koji-builder:latest | string   |
+----------------------+------------------------------------+----------+
| replicas             | 1                                  | int      |
+----------------------+------------------------------------+----------+
| configmap            | koji-builder-configmap             | string   |
+----------------------+------------------------------------+----------+
| cacert_secret        | koji-hub-ca-cert                   | string   |
+----------------------+------------------------------------+----------+
| client_cert_secret   | koji-builder-client-cert           | string   |
+----------------------+------------------------------------+----------+
| koji_hub_user        | 'koji-builder.mbox.dev'            | string   |
+----------------------+------------------------------------+----------+
| koji_hub_host        | 'koji-hub'                         | string   |
+----------------------+------------------------------------+----------+
| koji_hub_port        | 8443                               | string   |
+----------------------+------------------------------------+----------+
| max_jobs             | 5                                  | int      |
+----------------------+------------------------------------+----------+
| vendor               | MBox                               | string   |
+----------------------+------------------------------------+----------+
| host_archs           | ["x86_64"]                         | [string] |
+----------------------+------------------------------------+----------+
| host_name            | koji-hub:8443                      | string   |
+----------------------+------------------------------------+----------+
| ssl_verify           | true                               | boolean  |
+----------------------+------------------------------------+----------+
| shared_pvc           | koji-hub-mnt-pvc                   | string   |
+----------------------+------------------------------------+----------+
| mbox                 | ""                                 | string   |
+----------------------+------------------------------------+----------+
| host_channels        | ["default", "createrepo"]          | [string] |
+----------------------+------------------------------------+----------+


image
-----

The the full qualified image name to pull koji-builder from.

replicas
--------

The amount of koji-builder replicas to deploy.

configmap
---------

The configmap name to use when deploying koji-builder.

This configmap object contains configuration files that are mounted in koji-builder pod filesystem.

cacert_secret
-------------

The root CA secret name to use.

If not provided it uses the one generated by koji-hub (self-signed).

client_cert_secret
------------------

The koji-hub client secret name to use or create.

It will skip its creation (self signed) if one is already present.

It needs to be created and signed using the root CA certificate and private key.

Secret format:

.. code-block:: yaml

  apiVersion: v1
  kind: Secret
  metadata:
    name: myservice
    namespace: default
    labels:
      app: koji-builder
  type: kubernetes.io/tls
  data:
    tls.crt: -|
      fillme
    tls.key: -|
      fillme
    tls.pem: -|
      This is a combination of tls.key and tls.crt separated by '\n' and encoded in base64
      Example: "{{ (lookup('file', 'client_key.pem') + '\n' + lookup('file', 'client_cert.pem')) | b64encode }}"

koji_hub_user
-------------

User to use when authenticating with koji-hub.

koji_hub_host
-------------

Hostname of the koji-hub server instance that koji-builder will connect to.

koji_hub_port
-------------

Port of the koji-hub server instance that koji-builder will connect to.

max_jobs
--------

Max concurrent jobs the koji-builder should run in parallel.

vendor
------

Koji-builder vendor used in rpm headers.

host_archs
----------

The list of supported koji builder host architectures.

Defaults to a single architecture of "x86_64".

host_name
---------

The koji host name to be used when creating a koji host in koji-hub.

The name should be a qualified hostname address.

This name should be unique in koji and is also used as the koji-build client
certificate CN field.

ssl_verify
----------

A boolean flag used to tell koji-builder to verify ssl certs when connectiong to koji-hub.

It should be set to false if using self-signed certs.

shared_pvc
----------

Name of the shared PersistentVolumeClaim koji-builder will use.

host_channels
-------------

A list of channels to add the koji-host to.

Defaults to the following channels: "default" and "createrepo".

mbox
----

A Mbox resource name to retrieve shared data from (pvc volume and shared certs).

Koji-builder will use the following vars if this property is missing:

* mnt_pvc_name (shared koji mnt volume)
* cacert_secret (root ca secret)

Usage
=====

Upstream file can be found `here <https://raw.githubusercontent.com/fedora-infra/mbbox/master/mbox-operator/config/samples/apps_v1alpha1_mbkojibuilder.yaml>`_

Create a file containing the following content (modify as needed):

.. code-block:: yaml

  apiVersion: apps.fedoraproject.org/v1alpha1
  kind: MBKojiBuilder
  metadata:
    name: example
    labels:
      app: mbox
  spec:
    image: quay.io/fedora/koji-builder:latest
    replicas: 1
    configmap: koji-builder-configmap
    cacert_secret: koji-hub-ca-cert
    client_cert_secret: koji-builder-client-cert
    koji_hub_user: 'koji-builder.mbox.dev'
    koji_hub_host: 'koji-hub'
    koji_hub_port: 8443
    max_jobs: 5
    vendor: MBox
    host_archs:
      - x86_64
    host_channels:
      - default
      - createrepo
    host_name: mbbox.default
    ssl_verify: false
    shared_pvc: koji-hub-mnt-pvc

Run the following command to create a koji-builder resource:
  
.. code-block:: shell

  kubectl apply -f koji-builder-cr.yaml

You can check its status by running:

.. code-block:: shell

  kubectl get mbkojibuilder/example -o yaml
