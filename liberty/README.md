OpenStack Liberty Puppet manifests
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

Liberty

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
    sudo puppet module install openstack/keystone --version ">=7.0.0 <8.0.0"
    sudo puppet module install openstack/glance --version ">=7.0.0 <8.0.0"
    sudo puppet module install openstack/cinder --version ">=7.0.0 <8.0.0"
    sudo puppet module install openstack/nova --version ">=7.0.0 <8.0.0"
    sudo puppet module install openstack/neutron --version ">=7.0.0 <8.0.0"
    sudo puppet module install example42/network
    sudo puppet module install saz/memcached

**IMPORTANT:** There is a bug (https://bugs.launchpad.net/puppet-glance/+bug/1483663/comments/3) in puppet-glance module v. 7.0.0 released Nov 26th 2015 which needs a patch before running the puppet.

    cd /etc/puppet/modules/glance/ && sudo wget https://patch-diff.githubusercontent.com/raw/thenoizz/puppet-glance/pull/1.patch && sudo patch -p1 -b < 1.patch && cd ~

### Manifest

Modify the variables at the beginning of _openstack-aio-ubuntu-single-nic.pp_
to match your environment where needed and then run:

    sudo puppet apply --verbose openstack-aio-ubuntu-single-nic.pp

### OpenStack environment

After applying the manifest, the environment is fully deployed and ready to be
used. A Cirros image is included in Glance.

OpenStack Dashboard (Horizon):

    http://<hostname>/horizon

Login with either the "admin" or "demo" credentials. Default password is
"Passw0rd" for both.

When booting instances, attach a NIC to the "private" network.

Command line environment variables:

    source /root/keystonerc_admin

or

    source /root/keystonerc_demo

### Troubleshooting

#### puppetlabs-rabbitmq: Dependency cycle

Latest master or anything after commit [28fc64a][0], causes dependency cycle when used with puppetlabs-apt 1.8.x version.

```
Error: Failed to apply catalog: Found 1 dependency cycle:
(Anchor[apt::source::rabbitmq] => Apt::Source[rabbitmq] => Class[Rabbitmq::Repo::Apt] => Class[Apt::Update] => Exec[apt_update] => Class[Apt::Update] => Anchor[apt::source::rabbitmq])
Try the '--graph' option and opening the resulting '.dot' file in OmniGraffle or GraphViz
```

It seems this happens because in commit [28fc64a][0] a requirement on Class['apt::update'] for ensuring updated repos was introduced which seems to be correctly handled only in puppetlabs-apt 2.x.

More details can be found on the following bug [report][1].

##### How to fix it ?

- Upgrade the `puppetlabs-apt` module

```bash
~ $ sudo puppet module install puppetlabs-apt --version "2.0.0" --force
```

```
Notice: Preparing to install into /etc/puppet/modules ...
Notice: Downloading from https://forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/etc/puppet/modules
└── puppetlabs-apt (v2.0.0)
```

- Apply the `Update apt::source to match with new method` patch


```bash
wget https://goo.gl/WsDtjU -O /tmp/apt.diff && cd /etc/puppet/modules/rabbitmq/manifests/repo/ && sudo git apply /tmp/apt.diff && cd ~
```

```bash
wget https://goo.gl/Na6V6o -O /tmp/rabbitmq_spec.diff && cd /etc/puppet/modules/rabbitmq/spec/classes/ && sudo git apply /tmp/rabbitmq_spec.diff && cd ~
```

```bash
wget https://goo.gl/fM7daR -O /tmp/metadata.diff && cd /etc/puppet/modules/rabbitmq && sudo git apply /tmp/metadata.diff && cd ~
```

More information related to this patch can be found on the following [link][2].

[0]: https://github.com/puppetlabs/puppetlabs-rabbitmq/commit/28fc64a7d536873daf2a93e6461611c7238e053e
[1]: https://tickets.puppetlabs.com/browse/MODULES-2995
[2]: https://github.com/puppetlabs/puppetlabs-rabbitmq/pull/423/commits/c6d3b3dda2ddcf6747dc6f8328a090f42a292a0e
