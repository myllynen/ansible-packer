# Ansible Packer Playbook

[![License: GPLv2](https://img.shields.io/badge/license-GPLv2-brightgreen.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)
[![License: GPLv3](https://img.shields.io/badge/license-GPLv3-brightgreen.svg)](https://www.gnu.org/licenses/gpl-3.0)

Ansible role for building OS images with Packer.

Also support creating custom ISO installer images.

## Quick Usage Example

This role builds custom operating system images with
[Packer](https://www.packer.io/).

To build an image install Packer on a build host, install this role, and
run a playbook as shown below.

For completeness sake, this role can also be used to create custom ISO
installer images (without Packer). See the example for creating custom
ISO later in this page.

After [installing Packer](https://www.packer.io/downloads), install this
role:

```
mkdir roles
cat << EOF > roles/requirements.yml
---
roles:
  - src: https://github.com/myllynen/ansible-packer.git
    type: git
    version: master
EOF
ansible-galaxy role install -p roles -r roles/requirements.yml
```

Then, create a [playbook](./packer.yml) to use this role.

This is a basic playbook for building an image with Qemu:

```
---
- name: Build image with Packer on Qemu
  hosts: all
  vars:
    packer_binary: packer.io
    packer_builder: qemu
    packer_target: rhel_8
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
      rhel_8:
        url: file:///VirtualMachines/boot/rhel-8.5-x86_64-dvd.iso
        checksum: sha256:1f78e705cd1d8897a05afa060f77d81ed81ac141c2465d4763c0382aa96cadd0

  roles:
    - ansible-packer
```

This is a more complete playbook for building an image with VMware
vSphere:

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
    packer_target: rhel_8_5
    image_name: rhel8-template
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
    #security_profile: cis_server_l1
    boot_parameters: net.ifnames.prefix=net quiet systemd.show_status=yes

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
    #vcenter_credentials:
    #  vcenter_server: vcenter.example.com
    #  vcenter_username: vcuser
    #  vcenter_password: vcpass
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
      rhel_8_5:
        url: file:///VirtualMachines/boot/rhel-8.5-x86_64-dvd.iso
        checksum: sha256:1f78e705cd1d8897a05afa060f77d81ed81ac141c2465d4763c0382aa96cadd0

  roles:
    - ansible-packer
```

See the example [playbook](./packer.yml) for more complete example and
[defaults/main](defaults/main) for all the supported variables.

Finally, build image on a build host:

```
# Build latest RHEL 8 image with playbook defaults
ansible-playbook -c local -i localhost, packer.yml \
  -e packer_target=rhel_8

# Build RHEL 8.5 image on VMware vSphere with customizations
ansible-playbook -i 192.168.122.123, -u builder packer.yml \
  -e packer_builder=vmware -e packer_target=rhel_8_5 \
  -e bios_uefi_boot=true -e partitioning=single \
  -e disable_ipv6=true -e security_profile=cis_server_l1 \
  -e image_name=test_image
```

## Custom ISO Installer Image Creation 

A playbook to build custom ISO is almost identical to the above
playbooks used to build OS images with Packer:

```
---
- name: Build custom ISO
  hosts: all
  vars:
    # Using packer_ variables for compatibility,
    # genisoimage not Packer used to build image
    packer_builder: iso
    packer_target: rhel_8_5
    packer_target_pretty: Custom RHEL 8.5
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
      rhel_8_5:
        url: file:///VirtualMachines/boot/rhel-8.5-x86_64-dvd.iso
        checksum: sha256:1f78e705cd1d8897a05afa060f77d81ed81ac141c2465d4763c0382aa96cadd0

  roles:
    - ansible-packer
```

To build custom ISO on a build host:

```
# Build latest RHEL 8 image with playbook defaults
ansible-playbook -c local -i localhost, build_iso.yml \
  -e image_password=foobar
```

## Role Description

[Ansible](https://www.ansible.com/) role to allow quick building of
[RHEL](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux)
and other images with [Packer](https://www.packer.io/) using either
[Qemu](https://www.packer.io/docs/builders/qemu) or
[VMware vSphere](https://www.packer.io/docs/builders/vsphere/vsphere-iso)
as builders.

These templates do not save cleartext passwords on disk at any point.

Three paramaters, _packer\_builder_, _packer\_target_ and
_root\_password_ are mandatory, the rest are optional. See
[defaults/main](defaults/main) for all the supported variables.

See [defaults/main/content_iso.yml](defaults/main/content_iso.yml) how
to define OS versions and ISO locations.

See [defaults/main/builder_qemu.yml](defaults/main/builder_qemu.yml) and
[defaults/main/builder_vmware.yml](defaults/main/builder_vmware.yml) for
Packer builder related variables and their default values. ISO related
variables are in
[defaults/main/builder_iso.yml](defaults/main/builder_iso.yml).

Create BIOS/UEFI bootable image setting _bios_uefi\_boot_ to `true`,
otherwise the image supports only the platform used for building the
image.

See the provided partitioning alternatives at
[templates/cfg-rhel_8.j2](templates/cfg-rhel_8.j2), specify
_custom\_partition_ and set `partitioning: custom` to use custom
partitioning layout.

The value of _security\_profile_ is passed as-is to the installer
[OpenSCAP](https://www.open-scap.org/) module.

To create local admin user on the VM for Ansible etc, see
[defaults/main/os.yml](defaults/main/os.yml) for admin user data
specification.

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

## License

GPLv2+
