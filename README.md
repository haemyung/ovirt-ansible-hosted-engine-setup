# ovirt-ansible-hosted-engine-setup

## This role has been migrated to [oVirt Ansible Collection](https://github.com/ovirt/ovirt-ansible-collection), please use latest version from there. This repository is now readonly and no longer used for active development.

Ansible role for deploying oVirt Hosted-Engine

# Requirements

 * Ansible version 2.9.11 or higher
 * Python SDK version 4.2 or higher
 * Python netaddr library on the ansible controller node

# Dependencies

No.

# Prerequisites

* A fully qualified domain name prepared for your Engine and the host. Forward and reverse lookup records must both be set in the DNS.
* `/var/tmp` has at least 5 GB of free space.
* Unless you are using Gluster, you must have prepared storage for your Hosted-Engine environment (choose one):
    * [Prepare NFS Storage](https://ovirt.org/documentation/admin-guide/chap-Storage/#preparing-nfs-storage)
    * [Prepare ISCSI Storage](https://ovirt.org/documentation/admin-guide/chap-Storage/#preparing-iscsi-storage)
* Install additional oVirt ansible role:

    ```bash
    $ ansible-galaxy install ovirt.engine-setup # case-sensitive
    ```
# Role variables

## General Variables

| Name                            | Default value         |  Description                                              |
|---------------------------------|-----------------------|-----------------------------------------------------------|
| he_bridge_if | eth0 | The network interface ovirt management bridge will be configured on |
| he_fqdn | null | The engine FQDN as it configured on the DNS |
| he_mem_size_MB | max | The amount of memory used on the engine VM |
| he_reserved_memory_MB | 512 | The amount of memory reserved for the host |
| he_vcpus | max | The amount of CPUs used on the engine VM |
| he_disk_size_GB | 61 | Disk size of the engine VM |
| he_vm_mac_addr | null | MAC address of the engine vm network interface. |
| he_domain_type | null | Storage domain type. available options: *nfs*, *iscsi*, *glusterfs*, *fc* |
| he_storage_domain_addr | null | Storage domain IP/DNS address |
| he_ansible_host_name | localhost | hostname in use on the first HE host (not necessarily the Ansible controller one) |
| he_restore_from_file | null | a backup file created with engine-backup to be restored on the fly |
| he_pki_renew_on_restore | false | Renew engine PKI on restore if needed |
| he_cluster | Default | name of the cluster with hosted-engine hosts |
| he_cluster_cpu_type | null | cluster CPU type to be used in hosted-engine cluster (the same as HE host or lower) |
| he_cluster_comp_version | null | Compatibility version of the hosted-engine cluster. Default value is the latest compatibility version |
| he_data_center | Default | name of the datacenter with hosted-engine hosts |
| he_data_center_comp_version | null | Compatibility version of the hosted-engine data center. Default value is the latest compatibility version |
| he_host_name | $(hostname -f) | name used by the engine for the first host |
| he_host_address | $(hostname -f) | address used by the engine for the first host |
| he_bridge_if | null | interface used for the management bridge |
| he_apply_openscap_profile | false | apply a default OpenSCAP security profile on HE VM |
| he_network_test | dns | the way of the network connectivity check performed by ovirt-hosted-engine-ha and ovirt-hosted-engine-setup, available options: *dns*, *ping*, *tcp* or *none*.  |
| he_tcp_t_address | null | hostname to connect if he_network_test is *tcp*  |
| he_tcp_t_port | null | port to connect if he_network_test is *tcp* |
| he_pause_host | false | Pause the execution to let the user interactively fix host configuration |
| he_offline_deployment | false | If `True`, updates for all packages will be disabled |
| he_additional_package_list | [] | List of additional packages to be installed on engine VM apart from ovirt-engine package |
| he_debug_mode | false | If `True`, HE deployment will execute additional tasks for debug |
| he_db_password | UNDEF | Engine database password |
| he_dwh_db_password | UNDEF | DWH database password |

## NFS / Gluster Variables

| Name                            | Default value         |  Description                                              |
|---------------------------------|-----------------------|-----------------------------------------------------------|
| he_mount_options | '' | NFS mount options
| he_storage_domain_path | null | shared folder path on NFS server |
| he_nfs_version | auto | NFS version.  available options: *auto*, *v4*, *v3*, *v4_0*, *v4_1*, *v4_2*
| he_storage_if | null | the network interface name that is connected to the storage network, assumed to be pre-configured|


## ISCSI Variables

| Name                            | Default value         |  Description                                              |
|---------------------------------|-----------------------|-----------------------------------------------------------|
| he_iscsi_username | null | iscsi username |
| he_iscsi_password | null | iscsi password |
| he_iscsi_target | null | iscsi target |
| he_lun_id | null | Lun ID |
| he_iscsi_portal_port | null | iscsi portal port |
| he_iscsi_portal_addr | null | iscsi portal address (just for interactive iSCSI discovery, use he_storage_domain_addr for the deployment) |
| he_iscsi_tpgt | null | iscsi tpgt |
| he_discard | false |  Discard the whole disk space when removed. more info [here](https://ovirt.org/develop/release-management/features/storage/discard-after-delete/)

## Static IP configuration Variables

DHCP configuration is used on the engine VM by default. However, if you would like to use static ip instead,
define the following variables:

| Name                            | Default value         |  Description                                              |
|---------------------------------|-----------------------|-----------------------------------------------------------|
| he_vm_ip_addr | null | engine VM ip address |
| he_vm_ip_prefix | null | engine VM ip prefix |
| he_dns_addr | null | engine VM DNS server |
| he_default_gateway | null | engine VM default gateway |
| he_vm_etc_hosts | false | Add engine VM ip and fqdn to /etc/hosts on the host |

# Example Playbook
This is a simple example for deploying Hosted-Engine with NFS storage domain.

This role can be used to deploy on localhost (the ansible controller one) or on a remote host (please correctly set he_ansible_host_name).
All the playbooks can be found inside the `examples/` folder.

## hosted_engine_deploy_localhost.yml

```yml
---
- name: Deploy oVirt hosted engine
  hosts: localhost
  connection: local
  roles:
    - role: ovirt.hosted_engine_setup
```

## hosted_engine_deploy_remotehost.yml

```yml
---
- name: Deploy oVirt hosted engine
  hosts: host123.localdomain
  roles:
    - role: ovirt.hosted_engine_setup
```

## passwords.yml

```yml
---
# As an example this file is keep in plaintext, if you want to
# encrypt this file, please execute following command:
#
# $ ansible-vault encrypt passwords.yml
#
# It will ask you for a password, which you must then pass to
# ansible interactively when executing the playbook.
#
# $ ansible-playbook myplaybook.yml --ask-vault-pass
#
he_appliance_password: 123456
he_admin_password: 123456
```

## Example 1: extra vars for NFS deployment with DHCP - he_deployment.json

```json
{
    "he_bridge_if": "eth0",
    "he_fqdn": "he-engine.example.com",
    "he_vm_mac_addr": "00:a5:3f:66:ba:12",
    "he_domain_type": "nfs",
    "he_storage_domain_addr": "192.168.100.50",
    "he_storage_domain_path": "/var/nfs_folder"
}
```

## Example 2: extra vars for iSCSI deployment with static IP, remote host - he_deployment_remote.json

```json
{
    "he_bridge_if": "eth0",
    "he_fqdn": "he-engine.example.com",
    "he_vm_ip_addr": "192.168.1.214",
    "he_vm_ip_prefix": "24",
    "he_gateway": "192.168.1.1",
    "he_dns_addr": "192.168.1.1",
    "he_vm_etc_hosts": true,
    "he_vm_mac_addr": "00:a5:3f:66:ba:12",
    "he_domain_type": "iscsi",
    "he_storage_domain_addr": "192.168.1.125",
    "he_iscsi_portal_port": "3260",
    "he_iscsi_tpgt": "1",
    "he_iscsi_target": "iqn.2017-10.com.redhat.stirabos:he",
    "he_lun_id": "36589cfc000000e8a909165bdfb47b3d9",
    "he_mem_size_MB": "4096",
    "he_ansible_host_name": "host123.localdomain"
}
```

### Test iSCSI connectivity and get LUN WWID before deploying

```
[root@c75he20180820h1 ~]# iscsiadm -m node --targetname iqn.2017-10.com.redhat.stirabos:he -p 192.168.1.125:3260 -l
[root@c75he20180820h1 ~]# iscsiadm -m session -P3
iSCSI Transport Class version 2.0-870
version 6.2.0.874-7
Target: iqn.2017-10.com.redhat.stirabos:data (non-flash)
	Current Portal: 192.168.1.125:3260,1
	Persistent Portal: 192.168.1.125:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.1994-05.com.redhat:6a4517b3773a
		Iface IPaddress: 192.168.1.14
		Iface HWaddress: <empty>
		Iface Netdev: <empty>
		SID: 1
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 5
		Target Reset Timeout: 30
		LUN Reset Timeout: 30
		Abort Timeout: 15
		*****
		CHAP:
		*****
		username: <empty>
		password: ********
		username_in: <empty>
		password_in: ********
		************************
		Negotiated iSCSI params:
		************************
		HeaderDigest: None
		DataDigest: None
		MaxRecvDataSegmentLength: 262144
		MaxXmitDataSegmentLength: 131072
		FirstBurstLength: 131072
		MaxBurstLength: 16776192
		ImmediateData: Yes
		InitialR2T: Yes
		MaxOutstandingR2T: 1
		************************
		Attached SCSI devices:
		************************
		Host Number: 3	State: running
		scsi3 Channel 00 Id 0 Lun: 2
			Attached scsi disk sdb		State: running
		scsi3 Channel 00 Id 0 Lun: 3
			Attached scsi disk sdc		State: running
Target: iqn.2017-10.com.redhat.stirabos:he (non-flash)
	Current Portal: 192.168.1.125:3260,1
	Persistent Portal: 192.168.1.125:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.1994-05.com.redhat:6a4517b3773a
		Iface IPaddress: 192.168.1.14
		Iface HWaddress: <empty>
		Iface Netdev: <empty>
		SID: 4
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 5
		Target Reset Timeout: 30
		LUN Reset Timeout: 30
		Abort Timeout: 15
		*****
		CHAP:
		*****
		username: <empty>
		password: ********
		username_in: <empty>
		password_in: ********
		************************
		Negotiated iSCSI params:
		************************
		HeaderDigest: None
		DataDigest: None
		MaxRecvDataSegmentLength: 262144
		MaxXmitDataSegmentLength: 131072
		FirstBurstLength: 131072
		MaxBurstLength: 16776192
		ImmediateData: Yes
		InitialR2T: Yes
		MaxOutstandingR2T: 1
		************************
		Attached SCSI devices:
		************************
		Host Number: 6	State: running
		scsi6 Channel 00 Id 0 Lun: 0
			Attached scsi disk sdd		State: running
		scsi6 Channel 00 Id 0 Lun: 1
			Attached scsi disk sde		State: running
[root@c75he20180820h1 ~]# lsblk /dev/sdd
NAME                                MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sdd                                   8:48   0  100G  0 disk
└─36589cfc000000e8a909165bdfb47b3d9 253:10   0  100G  0 mpath
[root@c75he20180820h1 ~]# lsblk /dev/sde
NAME                                MAJ:MIN RM SIZE RO TYPE  MOUNTPOINT
sde                                   8:64   0  10G  0 disk
└─36589cfc000000ab67ee1427370d68436 253:0    0  10G  0 mpath
[root@c75he20180820h1 ~]# /lib/udev/scsi_id --page=0x83 --whitelisted --device=/dev/sdd
36589cfc000000e8a909165bdfb47b3d9
[root@c75he20180820h1 ~]# iscsiadm -m node --targetname iqn.2017-10.com.redhat.stirabos:he -p 192.168.1.125:3260 -u
Logging out of session [sid: 4, target: iqn.2017-10.com.redhat.stirabos:he, portal: 192.168.1.125,3260]
Logout of [sid: 4, target: iqn.2017-10.com.redhat.stirabos:he, portal: 192.168.1.125,3260] successful.
```

# Usage
1. Check all the prerequisites and requirements are met.
2. Encrypt passwords.yml
```sh
$ ansible-vault encrypt passwords.yml
```

3. Execute the playbook

Local deployment:
```sh
$ ansible-playbook hosted_engine_deploy.yml --extra-vars='@he_deployment.json' --extra-vars='@passwords.yml' --ask-vault-pass
```

Deployment over a remote host:
```sh
ansible-playbook -i host123.localdomain, hosted_engine_deploy.yml --extra-vars='@he_deployment.json' --extra-vars='@passwords.yml' --ask-vault-pass
```

Deploy over a remote host from Ansible AWX/Tower
---

The flow creates a temporary VM with a running engine to use for configuring and bootstrapping the whole environment.
The bootstrap engine VM runs over libvirt natted network so, in that stage, is not reachable from outside the host where it's running on.

When the role dynamically adds the freshly created engine VM to the inventory, it also configures the host to be used as an ssh proxy and this perfectly works directly running the playbook with ansible-playbook.
On the other side, Ansible AWX/Tower by defaults relies on PRoot to isolate jobs and so the credentials supplied by AWX/Tower will not flow to the jump host configured with ProxyCommand.

[This can be avoided disabling job isolation in AWX/Tower](https://docs.ansible.com/ansible-tower/latest/html/administration/tipsandtricks.html#setting-up-a-jump-host-to-use-with-tower)

Please notice that *job isolation* can be configured system wise but not only for the HE deploy job and so it's not a recommended practice on production environments.

Deployment time improvements
---

To significantly reduce the amount of time it takes to deploy a hosted engine __over a remote host__, add the following lines to `/etc/ansible/ansible.cfg` under the `[ssh_connection]` section:

```
ssh_args = -C -o ControlMaster=auto -o ControlPersist=30m
control_path_dir = /root/cp
control_path = %(directory)s/%%h-%%r
pipelining = True
```

Make changes in the engine VM during the deployment
---
In some cases, a user may want to make adjustments to the engine VM
during the deployment process. There are 2 ways to do that:

**Automatic:**

Write ansible playbooks that will run on the engine VM before or after the engine VM installation.

Add the playbooks that will run __before__ the engine setup to
```hooks/enginevm_before_engine_setup``` and the playbooks that will run __after__ the engine setup to ```hooks/enginevm_after_engine_setup```.

These playbooks will be consumed automatically by the role when you execute it.

**Manual:**

To make manual adjustments you can set the variable ```he_pause_host``` to true. This will pause the deployment after the engine has been setup and create a lock-file at /tmp that ends with ```_he_setup_lock``` on the machine the role was executed on. The deployment will continue after deleting the lock-file, or after 24 hours ( if the lock-file hasn't been removed ).

In order to proceed with the deployment, before deleting the lock-file, make sure that the host is on 'up' state at the engine's URL.

Both of the lock-file path and the engine's URL will be presented during the role execution.

Demo
----
Here a demo showing a deployment on NFS configuring the engine VM with static IP.
[![asciicast](https://asciinema.org/a/205639.png)](https://asciinema.org/a/205639)

# License

Apache License 2.0.
