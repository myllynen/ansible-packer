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

# Apply boot config fix needed on UEFI with RHEL ISO
# images booting from USB sticks. This requires sudo.
iso_boot_usb_fix: false

output_directory: /tmp/iso_images

image_checksum_file: "{{ image_name | default(__target_fullname + '.iso', true) + '.sha256' }}"

iso_volume_id: "{{ packer_target | upper | replace('_', '-') + '-0-BaseOS-' + ansible_facts.architecture }}"

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
# These should come from vault
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
  centos_9:
    url: file:///VirtualMachines/boot/CentOS-Stream-9-latest-x86_64-dvd1.iso
    checksum: "none"

  rhel_8:
    url: file:///VirtualMachines/boot/rhel-8-x86_64-dvd.iso
    checksum: "none"

  rhel_9:
    url: file:///VirtualMachines/boot/rhel-9-x86_64-dvd.iso
    checksum: "none"

---
# https://www.packer.io/plugins/builders/qemu#boot-configuration
boot_wait: 10s

# The OS installer boot_command will be appended by boot_parameters and boot_start
# RHEL-compatible boot_command on BIOS
#boot_command: <up><wait><tab><wait> inst.ks=hd:/dev/sr1:/inst.cfg inst.geoloc=0 inst.nosave=all ip=dhcp
# RHEL-compatible boot_command on UEFI
boot_command: <up><wait>e<wait><down><down><leftCtrlOn>e<leftCtrlOff> inst.ks=cdrom:/inst.cfg inst.geoloc=0 inst.nosave=all ip=dhcp

# Custom boot parameters to append
boot_parameters: net.ifnames.prefix=net quiet

# NB. With RHEL on BIOS use '<enter>' to boot, on UEFI must use '<leftCtrlOn>x<leftCtrlOff>'
boot_start: <leftCtrlOn>x<leftCtrlOff><wait>

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
#  ssh_key: ssh-ed25519 ... admin@image
#  passwordless_sudo: false

root_permit_local: true
root_permit_ssh: "prohibit-password"
root_permit_override_security_policy: false
#root_ssh_key: "ssh-ed25519 ... root@image"

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

# Timeout for Packer VM installation
shutdown_timeout: 15m

# Example Packer Ansible provisioner conf for RHEL
#
#communicator: "ssh"
#
## Needs to have root_permit_ssh set accordingly
#custom_json_builder_section: |
#  "pause_before_connecting": "10s",
#  "ssh_timeout": "15m",
#  "ssh_username": "root",
#  "ssh_private_key_file": "{{ ssh_private_key_file }}",
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
vm_type: uefi-secure
vm_tpm: true
vm_cpus: 2
vm_memory: 4096
vm_disk_size: "{{ disk_size | default(30720) }}"

---
# https://www.packer.io/plugins/builders/qemu#boot-configuration
win_boot_wait: 3s
# Must be set when using UEFI, use '<enter>' on BIOS
win_boot_command: <enter>FS1:<enter>EFI\\BOOT\\bootx64.efi<enter>

# Full path for additional drivers for the target Windows version
# For QEMU/KVM example, see the virtio-win-drivers-prepare script
# https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso
#win_drivers_path:

# Windows OS edition to install from the ISO
# Interactive installation will show editions
# The index reference should be either INDEX or NAME
win_os_index: INDEX
win_os_edition: 1

# Windows ISO label
win_iso_volume_id: WINDOWS_ISO

# Disk drive definitions: PowerShell scripts, Windows installer, VM tools
win_pss_disk: "{{ 'A:' if packer_builder == 'vmware' else 'E:' }}"
win_src_disk: "{{ win_pss_disk if packer_builder != 'iso' else 'D:' }}"
win_vm_tools_disk: "{{ 'E:' if packer_builder == 'vmware' else win_src_disk }}"

# Either bios, uefi, or custom
# See below for custom example
win_partitioning: uefi

win_timezone: UTC

# System locale setting
win_locale_system: en-US
# UI locale setting
win_locale_ui: en-US
# User locale setting
win_locale_user: en-US
# Input locale (language and keyboard) setting, see
# https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/default-input-locales-for-windows-language-packs
win_locale_input: 0409:00000409

# Password for Windows Administrator and remote management
# account that will be created during installation, see below.
# NB! This variable is used to determine whether to build RHEL
#     or Windows ISOs/VMs and for building RHEL must be unset.
#win_admin_password:

#win_product_key:
#win_registered_owner:
#win_registered_organization:

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

# Customization script, runs before enabling remote access
#win_customize_script: |

---
# Additional Packer Ansible parameters
# These are arguments to use with WinRM
# NB. These will be visible in ps(1) output
win_ansible_arguments: >
  "--extra-vars", "ansible_winrm_scheme=https",
  "--extra-vars", "ansible_winrm_transport=basic",
  "--extra-vars", "ansible_winrm_server_cert_validation=ignore",
  "--extra-vars", "ansible_become_method=runas"

# These are arguments to use with SSH/key
# NB. These will be visible in ps(1) output
# NB. Removing SSH host keys will not work using SSH
#win_ansible_arguments: >
#  "--extra-vars", "ansible_shell_type=powershell",
#  "--extra-vars", "ansible_become_method=runas"

# Private SSH key to use during VM installation
#ssh_private_key_file: ~/.ssh/id_ed25519_packer

# These are arguments to use with SSH/password
# NB. These will be visible in ps(1) output
# NB. Removing SSH host keys will not work using SSH
#win_ansible_arguments: >
#  "--extra-vars", "ansible_ssh_pass={% raw %}{{ user `win_admin_password` }}{% endraw %}",
#  "--extra-vars", "ansible_shell_type=powershell",
#  "--extra-vars", "ansible_become_method=runas"


# List of tasks or roles to run on the VM
# See packer_windows.yml for complete example
#win_provisioner_role_path: /src/roles.git/roles
win_provisioner_playbook: |2
    #gather_facts: false
    #vars:
    #  system_update_retry_count: 3
    #  system_update_retry_delay: 30
    #  system_update_retry_after_install: true
    #  system_update_categories: '*'
    #  system_update_skip_optional: true
    #  system_update_state: installed
    #  system_update_display_results: true
    #  system_update_reboot: true
    #  system_update_reboot_timeout: 1200
    #  system_update_compile_assemblies: true
    #  system_update_compile_filter: '.*'
    tasks:
      - ansible.windows.win_ping:
    #roles:
    #  - system_update

---
# Remote management account name
win_remote_user: winrm

# Optional administrators SSH key to install
#win_admin_ssh_key:

# Setup SSH for remote access
win_remote_setup_ssh: |
  Get-NetConnectionProfile | Set-NetConnectionProfile -NetworkCategory Private
  $missing = Get-WindowsCapability -Name 'OpenSSH.Server*' -Online | ? State -ne 'Installed'
  if ($missing) {
    Add-WindowsCapability -Name $missing.Name -Online
    if ((Get-WindowsCapability -Name $missing.Name -Online | ? State -ne 'Installed')) {
      Write-Host 'Failed to install OpenSSH.Server capability.'
      Exit 1
    }
  }
  Start-Service -Name sshd
  Set-Service -Name sshd -StartupType Automatic
  Set-ItemProperty -Path HKLM:\SOFTWARE\OpenSSH -Name DefaultShell -Value C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
  $rule = Get-NetFirewallRule -Name OpenSSH-Server-In-TCP -ErrorAction SilentlyContinue
  if (-not $rule) {
    New-NetFirewallRule `
      -Enabled True `
      -Name OpenSSH-Server-In-TCP `
      -DisplayName 'OpenSSH SSH Server (sshd)' `
      -Description 'Inbound rule for OpenSSH SSH Server (sshd)' `
      -Group 'OpenSSH Server' `
      -LocalPort 22 `
      -Action Allow `
      -Direction Inbound `
      -Protocol TCP `
      -Profile @('Domain', 'Private')
  }
  if ($rule -and ($rule.Enabled -ne $true -or $rule.Action -ne 'Allow')) {
    Set-NetFirewallRule -Name OpenSSH-Server-In-TCP -Enabled True -Profile @('Domain', 'Private') -Action Allow
  }
  $keyFile = 'C:\ProgramData\ssh\administrators_authorized_keys'
  $publicKey = '{{ win_admin_ssh_key | default("", true) }}'
  if (-not (Test-Path -Path $keyFile)) {
    New-Item -Path $keyFile -ItemType File
  }
  icacls.exe $keyFile /inheritance:r /grant ""Administrators:F"" /grant ""SYSTEM:F""
  if ($publicKey.StartsWith('ssh')) {
    Add-Content -Path $keyFile -Value $publicKey
  }

# Setup WinRM for remote access
win_remote_setup_winrm: |
  Get-NetConnectionProfile | Set-NetConnectionProfile -NetworkCategory Private
  Start-Service -Name WinRM
  Set-Service -Name WinRM -StartupType Automatic
  Set-Item -Path WSMan:\localhost\Service\AllowUnencrypted -Value $false
  Set-Item -Path WSMan:\localhost\Service\Auth\Basic -Value $true
  Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
  $user = '{{ win_remote_user }}'
  Set-ItemProperty -Path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System -Name LocalAccountTokenFilterPolicy -Value 1
  $sid = (New-Object -TypeName System.Security.Principal.NTAccount -ArgumentList $user).Translate([System.Security.Principal.SecurityIdentifier])
  $sddl = (Get-Item -Path WSMan:\localhost\Service\RootSDDL).Value
  $sd = New-Object -TypeName System.Security.AccessControl.CommonSecurityDescriptor -ArgumentList $false, $false, $sddl
  $sd.DiscretionaryAcl.AddAccess(
    [System.Security.AccessControl.AccessControlType]::Allow,
    $sid,
    [int]0x10000000,
    [System.Security.AccessControl.InheritanceFlags]::None,
    [System.Security.AccessControl.PropagationFlags]::None
    )
  $sddl = $sd.GetSddlForm([System.Security.AccessControl.AccessControlSections]::All)
  Set-Item -Path WSMan:\localhost\Service\RootSDDL -Value $sddl -Force
  $rule = Get-NetFirewallRule -Name WINRM-HTTP-In-TCP -ErrorAction SilentlyContinue
  if (-not $rule) {
    New-NetFirewallRule `
      -Enabled True `
      -Name WINRM-HTTP-In-TCP `
      -DisplayName 'Windows Remote Management (HTTP-In)' `
      -Description 'Inbound rule for Windows Remote Management via WS-Management. [TCP 5985]' `
      -Group '@FirewallAPI.dll,-30267' `
      -LocalPort 5985 `
      -Action Allow `
      -Direction Inbound `
      -Protocol TCP `
      -Profile @('Domain', 'Private')
  }
  if ($rule -and ($rule.Enabled -ne $true -or $rule.Action -ne 'Allow')) {
    Set-NetFirewallRule -Name WINRM-HTTP-In-TCP -Enabled True -Profile @('Domain', 'Private') -Action Allow
  }
  $friendlyName = 'WinRM over HTTPS'
  $cert = New-SelfSignedCertificate -CertStoreLocation Cert:\LocalMachine\My `
            -DnsName $env:COMPUTERNAME -NotAfter (get-date).AddYears(10) `
            -Provider 'Microsoft Enhanced RSA and AES Cryptographic Provider' `
            -KeyLength 4096
  $cert.FriendlyName = $friendlyName
  $listener = Get-WSManInstance winrm/config/Listener -Enumerate | ? Transport -eq HTTPS
  $command = if ($listener) { 'Set-WSManInstance' } else { 'New-WSManInstance' }
  & $command -ResourceURI winrm/config/Listener `
    -SelectorSet @{Address='*';Transport='HTTPS'} `
    -ValueSet @{CertificateThumbprint=$cert.Thumbprint;Enabled=$true}
  $rule = Get-NetFirewallRule -Name WINRM-HTTPS-In-TCP -ErrorAction SilentlyContinue
  if (-not $rule) {
    New-NetFirewallRule `
      -Enabled True `
      -Name WINRM-HTTPS-In-TCP `
      -DisplayName 'Windows Remote Management (HTTPS-In)' `
      -Description 'Inbound rule for Windows Remote Management via WS-Management. [TCP 5986]' `
      -Group '@FirewallAPI.dll,-30267' `
      -LocalPort 5986 `
      -Action Allow `
      -Direction Inbound `
      -Protocol TCP `
      -Profile @('Domain', 'Private')
  }
  if ($rule -and ($rule.Enabled -ne $true -or $rule.Action -ne 'Allow')) {
    Set-NetFirewallRule -Name WINRM-HTTPS-In-TCP -Enabled True -Profile @('Domain', 'Private') -Action Allow
  }

# Remote access method to configure
# By default both SSH and WinRM are enabled
win_remote_setup: |
  {{ win_remote_setup_ssh }}

  {{ win_remote_setup_winrm }}
</pre>

## License

GPLv3+
