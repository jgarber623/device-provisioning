# Pine64 Provisioning Playbook

This playbook will provision a [Pine64](https://www.pine64.org) for use as a headless local network server.

## Prerequisites

- [Ansible](https://www.ansible.com) (`brew install ansible`)
- [Etcher](https://etcher.io) (`brew cask install etcher`)
- A Pine64
- An SD card (at least 8GB)

## Provisioning

Download the latest Debian minimal release from [ayufan’s linux-build repository](https://github.com/ayufan-pine64/linux-build) and flash the image to an SD Card using Etcher. Insert the SD Card into the Pine64, connect an Ethernet cable, and power things up!

You should now be able to SSH into the Pine64:

```sh
ssh pine64@<ip-address>
```

Copy the appropriate public key over to the Pine64:

```sh
ssh-copy-id -i ~/.ssh/id_rsa_pine64.pub pine64@<ip-address>
```

SSH to the Pine64 and run the following commands to expand the root filesystem:

```sh
sudo /usr/local/sbin/resize_rootfs.sh
sudo reboot
```

Run the Ansible playbook to provision the Pine64:

```sh
ansible-playbook --ask-become-pass --verbose --inventory hosts playbook.yml
```
