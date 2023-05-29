# ansible_packer role

[![License: GPLv3](https://img.shields.io/badge/license-GPLv3-brightgreen.svg)](https://www.gnu.org/licenses/gpl-3.0)

Please see the collection main page for a higher level description.

## Configuration

Below are the role default values from defaults/main/:

<pre>
---
# ISO builder configuration which is not using Packer

packer_target_pretty: Custom {{ __target_fullname }}

iso_boot_parameters: inst.geoloc=0 ip=

output_directory: /tmp/iso_images

image_checksum_file: "{{ image_name | default(__target_fullname + '.iso', true) + '.sha256' }}"

iso_volume_id: "{{ packer_target | upper | replace('_', '-') + '-0-Custom-' + ansible_facts.architecture }}"

---
# https://www.packer.io/docs/builders/qemu
qemu_binary: /usr/libexec/qemu-kvm
qemu_args: '["-cpu", "host,+nx"]'
qemu_accelerator: kvm

# Used when vm_type is set to uefi or uefi-secure
qemu_uefi_code: /usr/share/OVMF/OVMF_CODE.secboot.fd
qemu_uefi_vars: /usr/share/OVMF/OVMF_VARS.secboot.fd

output_directory: /tmp/packer_images

---
# https://www.packer.io/docs/builders/vsphere/vsphere-iso
#vcenter_credentials:
#  vcenter_server:
#  vcenter_username:
#  vcenter_password:
vcenter_insecure_connection: false

vcenter_datacenter:
vcenter_folder:

# https://www.packer.io/docs/builders/vsphere/vsphere-iso#working-with-clusters-and-hosts
#vcenter_host:
vcenter_cluster:

#vcenter_resource_pool:

vcenter_datastore:
vcenter_network:

vm_guest_os_type: "{{ __base_target.split('-')[0] + __base_version }}_64Guest"

# https://www.packer.io/plugins/builders/vsphere/vsphere-iso#content-library-import-configuration
vcenter_convert_to_template: true
#vcenter_content_library:
#  library:
#  #name:
#  ovf:
#  destroy:

---
# Set checksum to "none" to skip installation media checksum check by Packer
iso:
  centos_8:
    url: file:///VirtualMachines/boot/CentOS-Stream-8-x86_64-latest-dvd1.iso
    checksum: sha256:7120e48d53471713f0ec41c5193c01edcbf84843b81073b09e33925361c3adbc

  centos_9:
    url: file:///VirtualMachines/boot/CentOS-Stream-9-latest-x86_64-dvd1.iso
    checksum: sha256:6ca7523a8e06d404d08c80075004fbdb4a1546e13dcb6f6f34f3e8c6e585fe3a

  rhel_7:
    url: file:///VirtualMachines/boot/rhel-server-7.9-x86_64-dvd.iso
    checksum: sha256:2cb36122a74be084c551bc7173d2d38a1cfb75c8ffbc1489c630c916d1b31b25

  rhel_8:
    url: file:///VirtualMachines/boot/rhel-8.8-x86_64-dvd.iso
    checksum: sha256:517abcc67ee3b7212f57e180f5d30be3e8269e7a99e127a3399b7935c7e00a09

  rhel_8_7:
    url: file:///VirtualMachines/boot/rhel-8.7-x86_64-dvd.iso
    checksum: sha256:a6a7418a75d721cc696d3cbdd648b5248808e7fef0f8742f518e43b46fa08139

  rhel_8_8:
    url: file:///VirtualMachines/boot/rhel-8.8-x86_64-dvd.iso
    checksum: sha256:517abcc67ee3b7212f57e180f5d30be3e8269e7a99e127a3399b7935c7e00a09

  rhel_9:
    url: file:///VirtualMachines/boot/rhel-9.2-x86_64-dvd.iso
    checksum: sha256:a18bf014e2cb5b6b9cee3ea09ccfd7bc2a84e68e09487bb119a98aa0e3563ac2

  rhel_9_2:
    url: file:///VirtualMachines/boot/rhel-9.2-x86_64-dvd.iso
    checksum: sha256:a18bf014e2cb5b6b9cee3ea09ccfd7bc2a84e68e09487bb119a98aa0e3563ac2

---
# https://www.packer.io/plugins/builders/qemu#boot-configuration
boot_wait: 10s

# The OS installer boot_command will be appended by boot_parameters and boot_start
# RHEL-compatible boot_command on BIOS
boot_command: <up><wait><tab><wait> ip=dhcp inst.ks=hd:/dev/sr1:inst.cfg inst.geoloc=0 inst.nosave=all
# RHEL-compatible boot_command on UEFI
#boot_command: <up><wait>e<wait><down><down><leftCtrlOn>e<leftCtrlOff> ip=dhcp inst.ks=cdrom:inst.cfg inst.geoloc=0 inst.nosave=all
# NB. With RHEL on BIOS use '<enter>' to boot, on UEFI must use '<leftCtrlOn>x<leftCtrlOff>'
boot_start: <enter><wait>

boot_parameters: net.ifnames.prefix=net quiet systemd.show_status=yes

boot_password:
#root_password:

# Autopartitioning does not support
# creating BIOS/UEFI-compatible images
bios_uefi_boot: true
# One of: auto/custom/default/single
#partitioning:
disable_ipv6: false
selinux: enforcing
fips_enable: false
#security_profile:

serial_console_setup: false
kdump_update_disable: false
# Ask installer to setup DHCP networking for the image
network_setup_dhcp: true
# Remove static network configuration setup by installer
network_setup_purge: true

hostname: localhost.localdomain
ntp_servers: time.cloudflare.com
timezone: UTC
keyboard: us

install_single_lang: true
install_exclude_docs: false
install_exclude_weak_deps: false

create_admin: false
#admin_user:
#  uid: 4444
#  gid: 4444
#  name: admin
#  group: admin
#  groups: wheel
#  gecos: Admin User
#  ssh_key: ssh-rsa ... admin@image
#  passwordless_sudo: false

root_permit_local: true
root_permit_ssh: "prohibit-password"
root_permit_override_security_policy: false
#root_ssh_key: "ssh-rsa ... root@image"

# Define to temporarily register and update RHEL
# To be replaced with rhsm directive once available
#rh_register_command:
# Install Satellite tools when updating packages
rh_satellite_tools_install: true

#custom_partitioning: |
#  reqpart
#  part /boot    --fstype xfs      --size 1024 --grow
#  part /        --fstype xfs      --size 1024 --grow
#  part swap     --fstype swap     --size 2048

#custom_packages: |
#  cloud-init
#  cloud-utils-growpart
#  insights-client
#  rhc
#  rhc-worker-playbook
#  sos
#  unzip

#custom_section_early: |
#  %post --erroronfail
#  cat /etc/resolv.conf > /var/tmp/.inst-resolv-conf
#  %end

#custom_section_late: |
#  %post --erroronfail
#  touch /.unconfigured
#  %end

# RHEL Edge parameters
ostree_url: http://builder.example.com/repo/
ostree_href: rhel/9/x86_64/edge

---
do_cleanup: true
use_force: false

packer_binary: packer.io

# Set no_log when running Packer, should be set if
# using password for vCenter or WinRM after tests
packer_build_no_log: false

communicator: "none"

# Example Packer Ansible provisioner conf for RHEL
# NB. Writes cleartext root password in build.json
#
#communicator: "ssh"
#
## Needs to have root_permit_ssh set accordingly
#custom_json_builder_section: |
#  "ssh_username": "root",
#  "ssh_password": "{{ image_password }}",
#  "ssh_timeout": "15m",
#
#custom_json_provisioner: |
#  {
#    "type": "ansible",
#    "use_proxy": false,
#    "playbook_file": "/tmp/rhel_packer.yml",
#    "user": "admin"
#  }

---
vm_name: "{{ image_name | default(__target_fullname + '_image.qcow2', true) }}"
# Either bios, uefi, or uefi-secure
vm_type: bios
vm_tpm: false
vm_cpus: 2
vm_memory: 4096
vm_disk_size: "{{ disk_size | default(30720) }}"

---
# https://www.packer.io/plugins/builders/qemu#boot-configuration
win_boot_wait: 3s
# Must be set when using UEFI
#win_boot_command: <enter>FS1:<enter>EFI\\BOOT\\bootx64.efi<enter>

# Full path for additional drivers for the target Windows version
# For QEMU/KVM example, see the virtio-win-drivers-prepare script
# https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso
#win_drivers_path:

# Windows OS edition to install from the ISO
# Interactive installation will show editions
win_os_edition: 1

# Windows ISO label
win_iso_volume_id: WINDOWS_ISO

# Disk with PowerShell scripts
win_pss_disk: "{{ 'A:' if packer_builder == 'vmware' else 'E:' }}"
win_src_disk: "{{ win_pss_disk if packer_builder != 'iso' else 'D:' }}"

# Administrator account will also use this password with ISO,
# account will disabled and password reset during VM sysprep.
#win_winrm_password:

# Either bios, uefi, or custom
# See below for custom example
win_partitioning: bios

win_timezone: UTC

# System locale setting
win_locale_system: en-US
# UI locale setting
win_locale_ui: en-US
# User locale setting
win_locale_user: en-US
# Input locale setting
# https://docs.microsoft.com/windows-hardware/manufacture/desktop/default-input-locales-for-windows-language-packs
win_locale_input: en-US

#win_product_key:
#win_registered_owner:
#win_registered_organization:

# System customization script, runs before enabling WinRM
#win_customize_script: |

win_winrm_setup: |
  # configure-winrm.ps1 always ensures the service is running,
  # this part should configure WinRM per the local requirements
  Set-Item -Path WSMan:\localhost\Service\Auth\Basic -Value $true
  Set-Item -Path WSMan:\localhost\Service\AllowUnencrypted -Value $true

# Additional Packer ansible-playbook call parameters with VM template
win_ansible_arguments: '"--extra-vars", "ansible_winrm_scheme=http"'

# List of tasks or roles to run on the VM,
# all roles must be included with full path
win_provisioner_playbook: |2
    #vars:
    #  boot_configuration_timeout: 5
    #  dotnet_optimize_all_assemblies: true
    tasks:
      - win_ping:
    #roles:
    #  - /tmp/roles/accounts_local
    #  - /tmp/roles/boot_configuration
    #  #- /tmp/roles/system_update
    #  - /tmp/roles/dotnet_optimize
    #  - /tmp/roles/performance_tuning
    #  - /tmp/roles/performance_tuning
    #  - /tmp/roles/service_enablement
    #  - /tmp/roles/service_recovery

#win_custom_partitioning_create: |
#    <CreatePartition wcm:action="add">
#        <Order>1</Order>
#        <Type>Primary</Type>
#        <Extend>true</Extend>
#    </CreatePartition>

#win_custom_partitioning_modify: |
#    <ModifyPartition wcm:action="modify">
#        <Order>1</Order>
#        <PartitionID>1</PartitionID>
#        <Label>Windows</Label>
#        <Format>NTFS</Format>
#        <Letter>C</Letter>
#    </ModifyPartition>

#win_custom_partitioning_install_to: 1
</pre>

## License

GPLv3+
