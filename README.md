# Ansible Packer Role for VM/ISO Image Creation

[![License: GPLv3](https://img.shields.io/badge/license-GPLv3-brightgreen.svg)](https://www.gnu.org/licenses/gpl-3.0)

Ansible role for building VM images with Packer.

The role also supports creating custom ISO installer images.

Support for RHEL Edge is currently under development.

## Introduction

This role builds custom Linux and Windows VM template images using
[Packer](https://www.packer.io/). Many OS installation parameters can be
set as Ansible variables to allow for a high degree of customization.
Same customizations can be applied to both VM template images and
unattended BIOS/UEFI-compatible ISO installer images.

See [this example](packer.yml) how a playbook could look like.

Currently (2024-09) tested Packer builders (platforms) are:

* [QEMU](https://www.packer.io/plugins/builders/qemu) (for KVM/libvirt/RHV/etc)
* [VMware vSphere](https://www.packer.io/plugins/builders/vsphere/vsphere-iso)

Currently (2024-09) tested OS variants and versions are:

* [RHEL](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux) 7, 8, 9
* [RHEL Edge](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux/edge-computing) 9 (experimental)
* [Windows Server](https://www.microsoft.com/en-us/windows-server) 2019, 2022, 2025
* [Windows](https://www.microsoft.com/en-us/windows/) 10, 11

VM images are modified to be used as templates at the end of automated
installation, RHEL with
[kickstart post-scripts](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/performing_an_advanced_rhel_installation/index)
and Windows with
[sysprep](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview).

VM images (templates) can also be further customized and updated
automatically at the end of the installation either by using custom
installer directives or by using the Packer
[Ansible provisioner](https://www.packer.io/plugins/provisioners/ansible/ansible).
VMware VM images are
[VMware Tools](https://docs.vmware.com/en/VMware-Tools) enabled so the
hypervisor can customize the OS in VMs created from these images.

Both RHEL and Windows custom ISO installer images embed and load the
installer configuration automatically on boot to proceed to install the
OS completely unattended.

Less tested but verified to work in the past OS variants include:

* [CentOS](https://www.centos.org/) 8, 9
* [Fedora](https://getfedora.org/) 34, 35, 36

## Quick Usage Example

To build a VM image [install Packer](https://www.packer.io/downloads) on
a build host, install this role, and run a playbook as shown below.

To install this collection from GitHub:

```
ansible-galaxy collection install git+https://github.com/myllynen/ansible-packer,master
```

Then, create a [playbook](packer.yml) to use this role.

This is a basic playbook for building an image with Qemu:

```
---
- name: Build image with Packer on Qemu
  hosts: all
  vars:
    packer_binary: packer.io
    packer_builder: qemu
    packer_target: rhel_9

    disk_size: 8192

    root_password: "{{ image_password }}"
    partitioning: auto

    hostname: localhost.localdomain
    ntp_servers: time.cloudflare.com
    timezone: Europe/Helsinki
    keyboard: fi

    # Builder: qemu
    # https://www.packer.io/docs/builders/qemu
    qemu_binary: /usr/libexec/qemu-kvm
    output_directory: /tmp/packer_images

    #
    # OS installer configuration
    #
    iso:
      rhel_9:
        url: file:///VirtualMachines/boot/rhel-9.4-x86_64-dvd.iso
        checksum: "none"

  roles:
    - myllynen.ansible_packer.ansible_packer
```

This is a more complete playbook for building an image on VMware:

```
---
- name: Build image with Packer on VMware vSphere
  hosts: all
  vars:
    #
    # Base configuration
    #
    packer_binary: /usr/local/sbin/packer.io
    packer_builder: vmware
    packer_target: rhel_9_4
    image_name: rhel9-template

    vm_type: uefi-secure
    vm_tpm: true
    disk_size: 30720

    do_cleanup: true
    use_force: true

    #
    # OS configuration
    #
    boot_password: "{{ image_password }}"
    root_password: "{{ image_password }}"
    partitioning: auto
    disable_ipv6: true
    #fips_enable: true
    #security_profile: cis
    boot_parameters: net.ifnames.prefix=net quiet

    hostname: localhost.localdomain
    ntp_servers: time.cloudflare.com
    timezone: Europe/Helsinki
    keyboard: fi

    create_admin: true
    admin_user:
      uid: 4444
      gid: 4444
      name: admin
      group: admin
      groups: wheel
      home: /home/admin
      gecos: Admin User
      ssh_key: "ssh-rsa ... admin@image"
      passwordless_sudo: true

    root_permit_local: true
    root_permit_ssh: "no"

    custom_packages: |
      cloud-init
      cloud-utils-growpart

    # Builder: vmware
    # https://www.packer.io/docs/builders/vsphere/vsphere-iso
    # These should come from vault
    vcenter_credentials:
      vcenter_server: vcenter.example.com
      vcenter_username: vcuser
      vcenter_password: vcpass
    vcenter_insecure_connection: false
    vcenter_datacenter: DC1
    vcenter_folder: Linux/templates
    vcenter_cluster: DC1-C1
    vcenter_datastore: DC1-C1-LUN-1
    vcenter_network: DC1-C1-VLAN-123
    vcenter_convert_to_template: true

    #
    # OS installer configuration
    #
    iso:
      rhel_9_4:
        url: file:///VirtualMachines/boot/rhel-9.4-x86_64-dvd.iso
        checksum: "none"

  roles:
    - myllynen.ansible_packer.ansible_packer
```

This example has additional options for building RHEL Edge image:

```
    packer_target: rhel_edge_9
    packer_target_pretty: RHEL 9 Edge
    ostree_url: http://10.1.1.10/repo/
    ostree_href: rhel/9/x86_64/edge

    iso:
      rhel_edge_9:
        url: file:///VirtualMachines/rhel-9.4-x86_64-dvd.iso
        checksum: "none"
```

See the example [playbook](packer.yml) for a more complete example and
[roles/ansible_packer/defaults/main](roles/ansible_packer/defaults/main)
for all the supported variables.

Finally, build image on a build host:

```
# Build latest RHEL 9 image with playbook defaults
ansible-playbook -c local -i localhost, packer.yml \
  -e packer_target=rhel_9

# Build RHEL 9.4 image on VMware vSphere with customizations
ansible-playbook -i 192.168.122.123, -u builder packer.yml \
  -e packer_builder=vmware -e packer_target=rhel_9_4 \
  -e bios_uefi_boot=true -e partitioning=single \
  -e disable_ipv6=true -e security_profile=cis \
  -e fips_enable=true -e image_name=test_image
```

## Custom ISO Installer Image Creation

A custom ISO installer image can be created to perform unattended OS
installation based on the configuration provided in a playbook. Users
would then only need to provide a static IP address (RHEL) for the host
on the boot prompt if not using DHCP (Windows). For the RHEL installer
(Anaconda) the static network boot parameter format is
ip=_ip::gateway:netmask:hostname:interface:none_, see
[RHEL documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/boot_options_for_rhel_installer/kickstart-and-advanced-boot-options_boot-options-for-rhel-installer#network-boot-options_kickstart-and-advanced-boot-options)
for more details. Note that the double-semicolon is required.

Root privileges are not needed for building ISOs with this role except
when RHEL ISOs are to be booted from USB on UEFI, see the role defaults
for more details. The required packages for RHEL ISO building are
listed in the [bindep.txt](bindep.txt) file. When building Windows ISOs
then also the _p7zip-plugins_ package from EPEL must be installed.

A playbook to build custom ISO installer image is almost identical to
the above playbooks used to build VM images with Packer:

```
---
- name: Build custom ISO
  hosts: all
  vars:
    # Using packer_ variables for compatibility,
    # xorrisofs not Packer is used to build image
    packer_builder: iso
    packer_target: rhel_9_4
    packer_target_pretty: Custom RHEL 9.4
    image_name: custom.iso

    root_password: "{{ image_password }}"
    partitioning: auto

    hostname: localhost.localdomain
    ntp_servers: time.cloudflare.com
    timezone: Europe/Helsinki
    keyboard: fi

    # Builder: ISO (not a Packer builder)
    iso_boot_parameters: inst.geoloc=0 ip=dhcp
    output_directory: /tmp/iso_images

    iso:
      rhel_9_4:
        url: file:///VirtualMachines/boot/rhel-9.4-x86_64-dvd.iso
        checksum: "none"

  roles:
    - myllynen.ansible_packer.ansible_packer
```

To build custom ISO on a build host:

```
# Build latest RHEL image with playbook defaults
ansible-playbook -c local -i localhost, build_iso.yml \
  -e image_password=Foobar_12
```

## Role Description

[Ansible](https://www.ansible.com/) role to allow for quickly building
of RHEL, Windows, and other OS images with
[Packer](https://www.packer.io/) using either
[Qemu](https://www.packer.io/docs/builders/qemu) or
[VMware vSphere](https://www.packer.io/docs/builders/vsphere/vsphere-iso)
as builders.

Linux builds do not save unencrypted passwords on disk at any point,
however Windows unattend files do have obscured passwords in them. This
is unavoidable as the Windows installer does not support encrypted
passwords.

Three paramaters, _packer\_builder_, _packer\_target_ and
_root\_password_ for Linux or _win\_winrm\_password_ for Windows are
mandatory, the rest are optional. See
[roles/ansible_packer/defaults/main](roles/ansible_packer/defaults/main)
for all the supported variables.

See
[roles/ansible_packer/defaults/main/content_iso.yml](roles/ansible_packer/defaults/main/content_iso.yml)
how to define OS versions and ISO locations.

See
[roles/ansible_packer/defaults/main/builder_qemu.yml](roles/ansible_packer/defaults/main/builder_qemu.yml)
and
[roles/ansible_packer/defaults/main/builder_vmware.yml](roles/ansible_packer/defaults/main/builder_vmware.yml)
for Packer builder related variables and their default values. ISO
related variables are in
[roles/ansible_packer/defaults/main/builder_iso.yml](roles/ansible_packer/defaults/main/builder_iso.yml).

Create BIOS/UEFI bootable image by setting _bios_uefi\_boot_ to `true`,
otherwise the VM image supports only the platform used for building the
image.

See the provided partitioning alternatives in
[roles/ansible_packer/templates/cfg-rhel_9.j2](roles/ansible_packer/templates/cfg-rhel_9.j2),
specify _custom\_partition_ and set `partitioning: custom` to use
custom partitioning layout.

The value of _security\_profile_ is passed as-is to the Linux installer
[OpenSCAP](https://www.open-scap.org/) module.

To create local admin user on the VM for Ansible etc, see
[roles/ansible_packer/defaults/main/os.yml](roles/ansible_packer/defaults/main/os.yml)
for admin user data specification (Linux). Windows admin user is
_winrm_, see
[roles/ansible_packer/defaults/main/windows.yml](roles/ansible_packer/defaults/main/windows.yml)
for details.

Vaulted password variables are supported. Note that the provided CentOS
templates do not support all the variables as the latest RHEL templates.

Additionally, `-e do_cleanup=true` will delete anything left behind on
the build host by earlier builds. Using `-e use_force=true` adds the
`-force` parameter to the Packer command line.

## See Also

See also
[https://github.com/myllynen/rhel-image](https://github.com/myllynen/rhel-image).

See also
[https://github.com/myllynen/rhel-ansible-roles](https://github.com/myllynen/rhel-ansible-roles).

See also
[https://github.com/myllynen/windows-ansible-roles](https://github.com/myllynen/windows-ansible-roles).

See also
[Red Hat Knowledge Base article on custom ISO with a kickstart file](https://access.redhat.com/solutions/60959).

See also
[https://weldr.io/lorax/mkksiso.html](https://weldr.io/lorax/mkksiso.html).

## License

GPLv3+
