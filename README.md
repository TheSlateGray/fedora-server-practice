<!--
reference: https://www.makeareadme.com/
reference: https://commonmark.org/
-->

[![GitHub issues](https://img.shields.io/github/issues/TheSlateGray/fedora-server-practice)](https://github.com/TheSlateGray/fedora-server-practice/issues)
[![GitHub pull requests](https://img.shields.io/github/issues-pr/TheSlateGray/fedora-server-practice)](https://github.com/TheSlateGray/fedora-server-practice/pulls)
[![GitHub license](https://img.shields.io/github/license/TheSlateGray/fedora-server-practice)](https://github.com/TheSlateGray/fedora-server-practice/blob/main/LICENSE)

# Fedora Server Practice

Kickstart and Ansible setup of a Fedora Server install.

## Motivation

My home server has grown and expanded over the last few years from Ubuntu to Debian and then stablizing with OMV last year. OMV is a good home server OS, however I would like to standardize my configuration with code rather than rely on remember changes made in the GUI. I found the great [blog series](https://blog.while-true-do.io/fedora-home-server-intro-concept/) published by while-true-do and decided to adapt it to my needs.

## Description

### OS Installation

[Kickstart](https://pykickstart.readthedocs.io/en/latest/) is for the
initial deployment of my homeserver. Kickstart automates all the steps needed in the installer and you can also provide additional packages, scripts and files.

- [kickstart.cfg](./kickstart/kickstart.cfg) is the base configuration for the server. At this time it is only customized for use in a VM before I commit to moving away from OMV.

Starting an installation via Kickstart is pretty easy:

1. Boot from USB media
2. At the below screenshot, interrupt the boot sequence by hitting a key and editing the first entry.
3. Add `inst.ks=https://your.kickstart.url/path/to/ks.cfg`
4. Continue and the installer will do its work.
5. Wait until the system is rebooted and login with the preconfigured credentials.

### Customization

Instead of providing and maintaining an inventory, this uses a [manifest.yml](./ansible/manifest.yml).
In the future this will could be changed to run from an inventory file.

### Configuration

Before configuring with Ansible ssh keys need to be copied to the new server.

```bash
# Copy public rsa key to created admin account.
ssh-copy-id -i ~/.ssh/id_rsa.pub admin@IP_ADDRESS
# The server will promt for the preconfigured password.
# Verify key copy worked.
ssh admin@IP_ADDRESS
exit
```

The [Ansible Playbook "configure.yml"](./ansible/playbooks/configure.yml) will:

- Configure the hostname and timezone
- Install and configure chrony (NTP)
- Install and configure some CLI tools
- Install and configure tuned (power profiles)
- Install and configure Avahi (mDNS provider)
- Install and configure Cockpit (Admin Web UI)
- Install and configure Performance Co-Pilot (performance metrics system)
- Install and configure KVM and libvirt
- Configure a bridge network for the virtual machines

To apply the playbook:

```bash
# Run the playbook (dry-run)
$ ansible-playbook -u admin -K -i IP_ADDRESS, --check --diff ansible/playbooks/configure.yml

# Run the playbook (for real)
$ ansible-playbook -u admin -K -i IP_ADDRESS, ansible/playbooks/configure.yml
```

### Update

The [Ansible Playbook "update.yml"](./ansible/playbooks/update.yml) will:

- Update the system
- Reboot if necessary

To appl the playbook:

```bash
# Run the playbook (dry-run)
$ ansible-playbook -u admin -K -i IP_ADDRESS, --check ansible/playbooks/update.yml

# Run the playbook (for real)
$ ansible-playbook -u admin -K -i IP_ADDRESS, ansible/playbooks/update.yml
```

### VM Creation

In case you are using the virtualization, I have provided a playbook to create
Virtual Maschines. It will prompt you for some information and create the VM.

```bash
# Create a VM
$ ansible-playbook -u admin -K -i IP_ADDRESS, ansible/playbooks/create_vm.yml
```

Afterwards, you will be able to log in to the new machine.

## License

This project is modified from the [fedora-homeserver](https://github.com/dschier-wtd/fedora-homeserver).
Except otherwise noted, all work is [licensed](LICENSE) under a [BSD-3-Clause License](https://opensource.org/licenses/BSD-3-Clause).
