# Host provisioning  

The playbook and roles are mostly generic settings once would set for a docker host. You may not need this if you already are provisioning your own infrastructure. 

To start the configuration of the Debian VM, you must first configure the VM interactively. For convenience, there is a Debian preseed file included. This file can be loaded pre-boot to automatically provision the VM so it's ready to log in and configure.

## Preseed

In the root of the repository there is a file called `preseed` that can be used for provisioning a Debian host from stratch. It will configure a user called `badger`. 

Some notes about it:
- Configured to use DHCP at boot. 
- Creates a single partition with LVM and swap
- Uses default credentials, which should be changed after installation
- Runs `sshd` on start

### Changing the password in the preseed file 

You can change the password which is hardcoded in the preseed file by doing:

```bash
printf "r00tme" | mkpasswd -s -m md5
$1$CHp7HkQW$Z2ZTY5cZMurbwbqU1zaS.1
```
And replacing it in the line `d-i passwd/user-password-crypted password`. 

### How to use the preseed file 
On how to use the preseed file depends on your virtualization system but one way is to: 

In the debian setup screen, press `e` to be able to edit the kernel parameters at boot and add:

`auto=true priority=critical preseed/url=http://<host>/preseed.cfg`

You have to host the preseed file somewhere on an endpoint directly accessible by the VM. When finished, you should be ready with a host that is ready to be configured by Ansible or your setup scripts. 


## Getting Started with Ansible

The playbook is designed to be run from another host. First, you must have `ansible-core` and `ansible` installed, tested with at least `ansible-core 2.17.1`.

## Playbook Variables 

Some key variables to look after: 

- `update_system: true` - Update system packages if first install or when needed
- `update_docker: true` - Update Docker
- `createorupdate_main_user: false` - Update user if any changes needed

We also have environment specific variables in for each host group in `group_vars/`. 

### Steps Before Running the Playbook

1. **Generate a Public Key Pair**: Put the public key in the `public_keys/` directory in the root of the repository. Only the public key is needed as `.pub`.
2. **Customize Variables**: Change the variables in `group_vars/all` as per your needs.
3. **Generate a `hosts.ini` File**: Follow the example to include host information.

### Running the Playbook

You can start the playbook with:

```sh
ansible-playbook -i hosts.ini -k playbook.yaml --ask-become-pass
```

## Roles 

### System

Installs base packages, configures time and journald logging.

### User

Creates the user and its initial groups. Will also add SSH key authentication. The password for the user is asked at the start of the script.

### UFW 

We use UFW for firewalling. An OpenSSH rule is added by default via Ansible. Due to how Docker and iptables interact, it is currently not possible to block Docker-exposed ports by default. Exposed ports should be used only when needed. More details at [ufw-docker](https://github.com/chaifeng/ufw-docker/blob/master/ufw-docker).

### Docker

Role for installing Docker. This also includes a Docker system prune service and custom logging driver. By default, the Docker prune service runs every day at 4 AM host time.

### Filesystem

Currently, it just creates a directory for the Docker Compose app. Future enhancements may be added.

### Keycloak

The keycloak role, if ran again, will remove all uncommited changes to tracked files on the remote host. 


### SSH

Adds public keys and configures `sshd` with sane defaults. 

### BadgeHub API

Creates needed directories to deploy and will clone the repository.
