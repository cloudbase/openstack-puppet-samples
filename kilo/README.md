OpenStack Kilo Puppet manifests
===============================

This is a collection of Puppet resources meant to be useful for proof of
concepts and similar scenarios where the goal is to deploy OpenStack in a
quick and painless way, often on limited hardware resources.

All in One
----------

Use this manifest to deploy OpenStack with:

* One single host
* One single NIC
* KVM hypervisor

The host can be even a virtual machine, as long as it supports nested
virtualization (e.g. VMWare Workstation / Player / Fusion / ESXi, KVM, etc).

### OpenStack release

Kilo

### OpenStack components

* Keystone
* Glance
* Nova / KVM
* Neutron / Open vSwitch
* Cinder / LVM
* Horizon

To be added soon:

* Swift
* Heat
* Ceilometer

### Supported operating system

Ubuntu 14.04 LTS

### Prerequisites

    sudo apt-get install puppet -y
    sudo puppet module install openstack/keystone --version ">=6.0.0 <7.0.0"
    sudo puppet module install openstack/glance --version ">=6.0.0 <7.0.0"
    sudo puppet module install openstack/cinder --version ">=6.0.0 <7.0.0"
    sudo puppet module install openstack/nova --version ">=6.0.0 <7.0.0"
    sudo puppet module install openstack/neutron --version ">=6.0.0 <7.0.0"
    sudo puppet module install example42/network
    sudo puppet module install saz/memcached

### Manifest

Modify the variables at the beginning of _openstack-aio-ubuntu-single-nic.pp_
to match your environment where needed and then run:

    sudo puppet apply --verbose openstack-aio-ubuntu-single-nic.pp

### OpenStack environment

After applying the manifest, the environment is fully deployed and ready to be
used. A Cirros image is included in Glance.

OpenStack Dahsboard (Horizon):

    http://<openstack_host>/horizon

Login with either the "admin" or "demo" credentials. Default password is
"Passw0rd" for both.

When booting instances, attach a nic to the "private" network.

Command line environment variables:

    source /root/keystonerc_admin

or

    source /root/keystonerc_demo

