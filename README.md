# Ansible Packer Playbook

[![License: GPLv2](https://img.shields.io/badge/license-GPLv2-brightgreen.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)
[![License: GPLv3](https://img.shields.io/badge/license-GPLv3-brightgreen.svg)](https://www.gnu.org/licenses/gpl-3.0)

Simple Ansible setup to build RHEL images with Packer.

## Quick Usage Example

To build an image install [Packer](https://www.packer.io/), ensure
Packer is available as _packer.io_ command (create a sym link if
needed), then clone this repo and run the playbook as shown below:

```
# Build latest RHEL 8 image on Qemu with all defaults
ansible-playbook packer.yml \
  -e packer_builder=qemu -e packer_target=rhel_8

# Build RHEL 8.5 image on vSphere with customizations
ansible-playbook packer.yml \
  -e packer_builder=vmware -e packer_target=rhel_8_5 \
  -e boot_password=foobar -e root_password=foobar \
  -e bios_uefi_boot=true -e partitioning=auto \
  -e security_profile=cis_server_l1 \
  -e disable_ipv6=true \
  -e image_name=test_image
```

## Introduction

A simple [Ansible](https://www.ansible.com/) setup to allow quick
building of
[RHEL](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux).
(and other) images with [Packer](https://www.packer.io/) using either
[Qemu](https://www.packer.io/docs/builders/qemu) or
[VMware vSphere](https://www.packer.io/docs/builders/vsphere/vsphere-iso)
as builders.

Adding support for other builders and operating systems as well as
further customization and parametrization of images is made easy by the
suitable split of configuration files and templates. Review the files
under [vars/](vars/) and [templates/](templates/) to see how to
customize or add support for new builders and operating systems.

These templates do not save cleartext passwords on disk at any point.
_root_ SSH login with password is permitted while running the
provisioner script but then set as per the target configuration.

The following table shows currently supported user-provided variables.
The first two paramaters, _packer\_builder_ and _packer\_target_ are
mandatory, the rest are optional.

| Variable         |  Allowed Values  |  Default  |
|:-----------------|:----------------:|:---------:|
| packer_builder   |  qemu, vmware    |   Unset   |
| packer_target    |  See 1) below    |   Unset   |
| packer_output    |      Path        |   See 2)  |
| boot_password    |  Bootloader pw   |   Unset   |
| root_password    |  root password   |   foobar  |
| bios_uefi\_boot  |  true or false   |   See 3)  |
| partitioning     |  See 4) below    |   See 4)  |
| disable_ipv6     |  true or false   |   false   |
| security_profile |  See 5) below    |   Unset   |
| create_admin     |  See 6) below    |   Unset   |
| disable_root     |  See 7) below    |   Unset   |
| image_name       |  Name for image  |  OS name  |

1. Use [vars/content_iso.yml](vars/content_iso.yml) to add support
   for additional OS versions.
2. See [vars/builder_qemu.yml](vars/builder_qemu.yml) and
   [vars/builder_vmware.yml](vars/builder_vmware.yml) for default
   output values.
3. Create BIOS/UEFI bootable image, otherwise support only the
   platform mode used for building the image.
4. Adjust installer configuration files such as
   [templates/cfg-rhel_8.j2](templates/cfg-rhel_8.j2)
   to add support for different partitioning layouts.
5. The value is passed as-is to the installer
   [OpenSCAP](https://www.open-scap.org/) module.
6. Create local admin user on the VM for Ansible etc,
   [templates/sysprep-rhel_8.j2](templates/sysprep-rhel_8.j2) must be
   adjusted for local environment, at least SSH key and user details.
7. Completely disable root account. Caution needed. Requires that
   _create\_admin_ is also used. See
   [templates/sysprep-rhel_8.j2](templates/sysprep-rhel_8.j2).

In case providing cleartext passwords on the command line (which makes
them visible e.g. in process listing by
[ps(1)](https://man7.org/linux/man-pages/man1/ps.1.html))
is not appropriate vaulted password variables can be used as well.

Additionally, using `-e do_cleanup=true` will delete anything left
behind on the build host by earlier builds. Using `-e use_force=true`
adds the `-force` parameter to the Packer command line.

NB. CentOS/RHEL 7 fails to boot if initramfs is updated during build.

## See Also

See also
[https://github.com/myllynen/rhel-image](https://github.com/myllynen/rhel-image).

## License

GPLv2+
