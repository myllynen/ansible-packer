# Ansible Packer Playbook

[![License: GPLv2](https://img.shields.io/badge/license-GPLv2-brightgreen.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)
[![License: GPLv3](https://img.shields.io/badge/license-GPLv3-brightgreen.svg)](https://www.gnu.org/licenses/gpl-3.0)

Simple Ansible setup to build RHEL images with Packer.

## Quick Usage Example

```
ansible-playbook packer.yml \
  -e packer_builder=qemu -e packer_target=rhel_8 \
  -e boot_password=foobar -e root_password=foobar \
  -e partitioning=auto -e disable_ipv6=true \
  -e security_profile=cis_server_l1 \
  -e image_name=test_image
```

## Introduction

A simple [Ansible](https://www.ansible.com/) setup to allow quick
building of RHEL (and other) images with
[Packer](https://www.packer.io/) using either
[Qemu](https://www.packer.io/docs/builders/qemu) or
[VMware vSphere](https://www.packer.io/docs/builders/vsphere/vsphere-iso)
as builders.

Adding support for other builders and operating systems as well as
further customization and parametrization of images is made easy by the
suitable split of configuration files and templates. Review the files
under [vars/](vars/) and [templates/](templates/) to see how to
customize or add support for new builders and operating systems.

These templates leave _root_ SSH login permitted! In most environments
this should be changed after initial development and testing.

The following table shows currently supported user-provided variables.
The first two paramaters, _packer\_builder_ and _packer\_target_ are
mandatory, the rest are optional.

| Variable         |  Allowed Values  |  Default  |
|------------------|:----------------:|:---------:|
| packer_builder   |  qemu, vmware    |   Unset   |
| packer_target    |  See 1) below    |   Unset   |
| packer_output    |      Path        |    Set    |
| boot_password    |  Bootloader pw   |   Unset   |
| root_password    |  root password   |   foobar  |
| partitioning     |  See 2) below    |   See 2)  |
| disable_ipv6     |  true or false   |   false   |
| security_profile |  See 3) below    |   Unset   |
| image_name       |  Name for image  |  OS name  |


1. Use [vars/content_iso.yml](vars/content_iso.yml) to add support
   for additional OS versions.
2. Adjust installer configuration files such as
   [templates/cfg-rhel_8.j2](templates/cfg-rhel_8.j2)
   to add support for different partitioning layouts.
3. The value is passed as-is to the installer
   [OpenSCAP](https://www.open-scap.org/) module.

NB. Mostly tested with
[RHEL](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux).
Some [CentOS](https://www.centos.org/) versions seem to fail if using
certain partition layouts or applying available updates during build.

## License

GPLv2+
