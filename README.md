Ansible role proxmox_create_kvm
=========

[![Galaxy](https://img.shields.io/badge/galaxy-UdelaRInterior.proxmox__create__kvm-blue.svg)](https://galaxy.ansible.com/udelarinterior/proxmox_create_kvm) ![GitHub tag (latest by date)](https://img.shields.io/github/v/tag/udelarinterior/ansible-role-proxmox-create-kvm?label=release&logo=github&style=social) ![GitHub stars](https://img.shields.io/github/stars/udelarinterior/ansible-role-proxmox-create-kvm?style=social) ![GitHub forks](https://img.shields.io/github/forks/udelarinterior/ansible-role-proxmox-create-kvm?style=social)

 A complete role for KVM virtual machines creation in a Proxmox Virtual Environment (PVE) cluster, with multiple network interfaces and storage units. The role wraps the [community `proxmox` collection](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_module.html#parameter-description)

Requirements
------------

You must act on a Proxmox VE node or cluster already configured, i.e. you need PVE node already installed (tested with PVE 5 and 6), and a Proxmox user with VM creation rights.


Installation
------------

```shell
$ ansible-galaxy install udelarinterior.proxmox_create_lxc
```

To be able to update later and eventually to modify it, prefer using `requirements.yml` with the git source:

```yaml
- name: udelarinterior.proxmox_create_kmv
  src: https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm.git
  ```
And then download it with `ansible-galaxy`:

```shell
$ ansible-galaxy install -r requirements.yml
```

Using `git`, you'll have to be carefull to folder name :

```shell
$ git clone https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm.git udelarinterior.proxmox_create_kvm
```

Role Variables
--------------

The `defaults` variables define the VM parameters. To be specified by host under `host_vars/host_fqdn/vars` and eventually encrypted in `host_vars/host_fqdn/vault`

New interface introduced in v2.0.0 is maintained, with role's variables defined in the `pve_*` namespace when they are shared between several Proxmox roles, and in the `pve_kvm_*` namespace when they are specific to th present one. The role is no longer backward's compatible with [v1.X.Y previous interface](https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/blob/v1.2.0/README.md#role-variables).

```yaml
#########################################################
### Proxmox API connection and authentication section ###
#########################################################

# Flag to determine the behavior of the variables default values
  # Various module options used to have default values. This cause problems when user expects different behavior from proxmox by default or fill
  # options which cause problems when they have been set. The default value is "compatibility", which will ensure that the default values are used
  # when the values are not explicitly specified by the user. From community.general 4.0.0 on, the default value will switch to "no_defaults".
  # To avoid deprecation warnings, please set proxmox_default_behavior to an explicit value. This affects the acpi, autostart, balloon, boot,
  # cores, cpu, cpuunits, force, format, kvm, memory, onboot, ostype, sockets, tablet, template, vga, options.
  # See: https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_kvm_module.html#parameter-proxmox_default_behavior
  # Choices: compatibility - no_defaults
pve_default_behavior: compatibility

# Proxmox node hostname where we create or manage a KVM
pve_node: mynode

# FQDN or IP of the Proxmox API endpoint where we manage the cluster or node
pve_api_host: mynode.mycluster.org

# User to use to connect to the Proxmox cluster API
pve_api_user: automations@pam    # Optional if token based authentication is used
# Password for the previous API user (BETTER PUT THIS IN A VAULT, this dummy example can cause security issues)
pve_api_password: PaSsWd_f0r-AuToMaTi0nS    # Optional if token authentication is used

# pve_api_token_id: automations@pam!ansible                       # Optional if user-password based authentication is used
# pve_api_token_secret: 0b029a23-1ca3-a93f-8d90-5a4c9d064199      # Optional if user-password based authentication is used

# Validate the node's certificates when creating the virtual machine
pve_validate_certs: no    # Optional.

# Timeout (in seconds) for operations, both clone and create. (Cloning can take a while)
pve_kvm_timeout: 500


#####################################################
### VM resources and behaviour definition section ###
#####################################################

# By default, we suppose that `inventory_hostname` is the FQDN or the hostname of the VM to create,
# so we set the variable to the hostname. You can arbitrarly define this hostname
pve_hostname: "{{ inventory_hostname.split('.')[0] }}"

pve_kvm_started_after_provision: false # It will ensure that the VM is turned on when provisioning finishes

# FALSE: create a new VM  -  TRUE: clone an existing VM
pve_kvm_clone_from_existing: false

# If yes, the VM will be updated with new value. Cause of the operations of the API and security reasons, the update of the
# following parameters was disabled: net, virtio, ide, sata, scsi. Per example updating net update the MAC address and virtio
# create always new disk. Update of pool is disabled. It needs an additional API endpoint not covered by this module.
pve_kvm_update: no


#####################################################
#### To create a KVM by cloning a pre-existing one
###

## IMPORTANT: If the VM is cloned, the other parameters are not used,
## the assigned hardware characteristics are inherited from the cloned VM

# Create a full copy of all disk. This is always done when you clone a normal VM. For VM templates, we try to create a linked clone by default.
# FALSE: linked clone  -  TRUE: full clone ## See https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/issues/2
pve_kvm_clone_full: true

# ID of VM to be cloned
# pve_kvm_clone_vmid: 100

# VMID for the clone. If newid is not set, the next available VMID will be fetched from ProxmoxAPI.
# pve_kvm_clone_newid: 400

# Name of VM to be cloned. If vmid is set with the pve_kvm_clone_vmid variable hereafter, which takes precedence over
# the hostname, pve_kvm_clone_vm can take arbitrary value but it is required for initiating the clone.
pve_kvm_clone_vm: DebianBusterTemplate

# The name of the snapshot to clone (Don't use/define it at the same time as pve_kvm_clone_vm)
# pve_kvm_clone_snapname: DebianBusterConfigured

# Target storage for full clone.
pve_kvm_clone_storage: local-lvm

# Target drive's backing file's data format. Use format=unspecified and full=false for a linked clone.
# Choices: cloop - cow - qcow - qcow2 (default) - qed - raw - vmdk
pve_kvm_clone_format: raw

# Target node. Only allowed if the original VM is on shared storage.
pve_kvm_clone_target: "{{ pve_node }}"


#####################################################
#### To create a KVM from scratch
###

###
#### Most important/used vars
###

# Specifies whether a VM will be started during system bootup.
pve_onboot: no    # Optional

# Specify the description for the VM. Only used on the configuration web interface. This is saved as comment inside the configuration file.
pve_kvm_description: LAMP Server created with Ansible    # Optional

# Enable/disable KVM hardware virtualization.
pve_kvm_hardware_virtualization: yes

# Specifies guest operating system. This is used to enable special optimization/features for specific operating systems.
pve_kvm_ostype: l26   # Optional. Choices: l24 - l26 (default) - other - wxp - w2k - w2k3 - w2k8 - wvista - win7 - win8 - solaris

# Sets the number of CPU sockets. (1 - N).
pve_kvm_sockets: 1    # Optional

# Specify number of cores per socket.
pve_kvm_cores: 1    # Optional

# Specify CPU weight for a VM. You can disable fair-scheduler configuration by setting this to 0
pve_kvm_cpuunits: 1000    # Optional

# Specify if CPU usage will be limited. Value 0 indicates no CPU limit. If the computer has 2 CPUs, it has total of '2' CPU time
pve_kvm_cpulimit: 0    # Optional

# Memory size in MB for instance.
pve_kvm_memory: 512    # Optional

# Specify the amount of RAM for the VM in MB. Using zero disables the balloon driver.
pve_kvm_balloon: 0    # Optional

# Amount of memory shares for auto-ballooning. (0 - 50000). The larger the number is, the more memory this VM gets.
# Using 0 disables auto-ballooning, this means no limit. The number is relative to weights of all other running VMs.
pve_kvm_shares: 0    # Optional

# Select VGA type. If you want to use high resolution modes (>= 1280x1024x16) then you should use option 'std' or 'vmware'.
pve_kvm_vga: std      # Optioanl. Choices: std (default) - cirrus - vmware - qxl - serial0 - serial1 - serial2 - serial3 - qxl2 - qxl3 - qxl4

# Custom structure to define the VM network interfaces in a more human-readable way
pve_kvm_net_interfaces:
- id: net0
  model: virtio                 # Optional. Available models: virtio - e1000 - rtl8139 - vmxnet3
  hwaddr: 46:70:B7:26:87:4F     # Optional. If not indicated, Proxmox will assign one automatically.
  bridge: vmbr0
  firewall: true                # Optional
  disconnect: true              # Optional
  multiqueue: 2                 # Optional
  rate_limit: 250               # Optional (In MB/s)
  vlan_tag: 400                 # Optional

- id: net1
  bridge: vmbr0

# Custom structure to define the VM hard disks in a more human-readable way
pve_kvm_hard_disks:
- id: 0                         #  0 ≤ n ≤ 15
  bus: virtio                   # Optional. Available buses: virtio (default) - ide - sata - scsi
  storage: local-lvm
  size: 32
  format: raw                   # Optional. Available formats: qcow2 - raw (default) - subvol
  backup: true                  # Optional
  skip_replication: false       # Optional
  cache: none                   # Optional. Available types: none (default) - directsync - writethrough - writeback - unsafe
  io_thread: false              # Optional
  ssd_emulation: false          # Optional

- id: 1
  bus: sata
  storage: local-lvm
  size: 16
  ssd_emulation: true
  backup: false

# A hash/dictionary of volume used as IDE hard disk or CD-ROM
# pve_kvm_ide:    # Optional
#   ide2: 'local:cloudinit,format=qcow2'    # Useful example to add Cloud-Init Drive


###
#### Cloud-Init related vars
###

# Specifies the cloud-init configuration format. The default depends on the configured
# operating system type (ostype): nocloud format for Linux, and configdrive2 for Windows
# pve_kvm_citype: nocloud

# Specify custom files to replace the automatically generated ones at start
# pve_kvm_cicustom:

# Username of default user to create.
# pve_kvm_ciuser: default

# Password of default user to create.
# pve_kvm_cipassword: super-secret

# SSH key to assign to the default user. NOT TESTED with multiple keys but a multi-line value should work
# pve_kvm_sshkeys: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}" # Ansible controller default public key

# Set the IP configuration. A hash/dictionary of network ip configurations. ipconfig='{"key":"value", "key":"value"}'
# pve_kvm_ipconfig:
#   ipconfig0: 'ip=192.168.1.1/24,gw=192.168.1.1'

# DNS server IP address(es). If unset, PVE host settings are used.
# pve_kvm_nameservers:
#   - '1.1.1.1'
#   - '8.8.8.8'

# Sets DNS search domain(s). If unset, PVE host settings are used.
# pve_kvm_searchdomains: mycluster.org


###
#### Other vars
###

# See the module documentation: https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_kvm_module.html#parameters
# And the create_kvm.yml tasks: https://github.com/UdelaRInterior/ansible-role-proxmox-create-kvm/blob/master/tasks/create_kvm.yml
```

Dependencies
------------

We need Ansible version > 2.10 and `community.general` collection to have the appropriate API of Proxmox modules.

Proxmox VE 5 or higher installed in a server node or a cluster of several nodes.

Example Playbook
----------------

Let's say, as the vars example, `mynode.mycluster.org` is our PVE node NS (API on port 8006), `mynode` it's PVE node name, `deploy` our Proxmox user in this node and `pve_virtuals_group` an Ansible group of the machines to define.

Given that:
* virtual machines are named `<machine>.node.mycluster.org`,
* Name resolutions of PVE machines and node are configured,
* machine's variables are defined (for example in each `host_vars/<machine>/vars/`),
* All new IPs are allocated and routed,

the following playbook creates all the virtual machines declared in the `pve_virtuals_group`,

    - name: create virtual machines declared in  pve_virtuals_group
      hosts: pve_virtuals_group
      remote_user: deploy
      become: yes
      gather_facts: no

      roles:
        - udelarinterior.proxmox_create_kvm


License
-------

(c) Universidad de la República (UdelaR), Red de Unidades Informáticas de la UdelaR en el Interior.

Licenced under GPL-v3

Author Information
------------------

[@ulvida](https://github.com/ulvida)
[@santiagomr](https://github.com/santiagomr)
[@UdelaRInterior](https://github.com/UdelaRInterior)
https://proyectos.interior.edu.uy/
