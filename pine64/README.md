# Pine64 Provisioning Playbook

This playbook will provision a [Pine64](https://www.pine64.org) for use as a headless server using [Ansible](https://www.ansible.com).

### Table of Contents

- [Provisioning Pine64](#provisioning-pine64)
	- [Flash SD Card](#flash-sd-card)
	- [Configure SSH](#configure-ssh)
	- [Upgrade to Debian 10 (Buster)](#upgrade-to-debian-10--buster-)
	- [Provision with Ansible](#provision-with-ansible)
- [Bonus Knowledge](#bonus-knowledge)
	- [Working with Bluetooth](#working-with-bluetooth)
	- [Installing and Configuring Pi-hole](#installing-and-configuring-pi-hole)

## Provisioning Pine64

### Flash SD Card

Download the latest "minimal" Debian release from [ayufan’s linux-build repository](https://github.com/ayufan-pine64/linux-build) and flash the image to an SD Card using Etcher. Insert the SD Card into the Pine64, connect an Ethernet cable, and power things up!

### Configure SSH

You should now be able to SSH into the Pine64:

```sh
ssh pine64@<ip-address>
```

Copy the appropriate public key over to the Pine64:

```sh
ssh-copy-id -i ~/.ssh/id_rsa_pine64.pub pine64@<ip-address>
```

Double check that you can SSH to the Pine64 without requiring a password.

### Upgrade to Debian 10 (Buster)

SSH to the Pine64 and prepare for the upgrade by running:

```sh
sudo apt update
sudo apt install python tmux

sudo dpkg-reconfigure locales
sudo dpkg-reconfigure tzdata
```

Update `/etc/apt/sources.list`:

```sh
sudo tee /etc/apt/sources.list << EOF
deb http://deb.debian.org/debian buster main contrib non-free
deb-src http://deb.debian.org/debian buster main contrib non-free

deb http://deb.debian.org/debian buster-updates main contrib non-free
deb-src http://deb.debian.org/debian buster-updates main contrib non-free

deb http://security.debian.org/debian-security/ buster/updates main contrib non-free
deb-src http://security.debian.org/debian-security/ buster/updates main contrib non-free
EOF
```

Optionally, create/update files in `/etc/apt/sources.list.d/`:

```sh
sudo tee /etc/apt/sources.list.d/longsleep-ubuntu-pine64-flavour-makers.list << EOF
deb http://ppa.launchpad.net/longsleep/ubuntu-pine64-flavour-makers/ubuntu xenial main
deb-src http://ppa.launchpad.net/longsleep/ubuntu-pine64-flavour-makers/ubuntu xenial main
EOF

sudo tee /etc/apt/sources.list.d/ayufan-pine64-ppa.list << EOF
deb http://ppa.launchpad.net/ayufan/pine64-ppa/ubuntu xenial main
deb-src http://ppa.launchpad.net/ayufan/pine64-ppa/ubuntu xenial main
EOF

sudo tee /etc/apt/sources.list.d/ayufan-pine64.list << EOF
deb http://deb.ayufan.eu/orgs/ayufan-pine64/releases /
deb http://deb.ayufan.eu/orgs/ayufan-pine64/pre-releases /
EOF
```

Start a tmux session (with `tmux`) and run the following commands to upgrade Debian:

```sh
sudo apt update
sudo apt dist-upgrade
```

Restart the Pine64 with `sudo shutdown -r now`.

After the restart, make sure everything's starting up properly. You might also want to run `sudo apt autoremove` to clean up outdated packages.

### Provision with Ansible

Run the main Ansible playbook to provision the Pine64:

```sh
ansible-playbook -K -v playbook.yml # or run individual playbooks
```

## Bonus Knowledge

### Working with Bluetooth

The [first message in this Pine64 forum thread](https://forum.pine64.org/showthread.php?tid=2248&pid=21412) contains a helpful rundown of working with Bluetooth on the Pine64.

```sh
# Make sure nothing is blocked
sudo rfkill list

# Launch bluetoothctl
sudo bluetoothctl

[bluetooth] power on
[bluetooth] agent on
[bluetooth] default-agent
[bluetooth] pairable on
[bluetooth] discoverable on
[bluetooth] scan on
```

You should now see a list of devices with their device addresses. To pair with, connect to, or trust a particular device, run:

```sh
[bluetooth] pair <bluetooth-device-address>
[bluetooth] connect <bluetooth-device-address>
[bluetooth] trust <bluetooth-device-address>
```

### Installing and Configuring Pi-hole

[Pi-hole](https://github.com/pi-hole/pi-hole)'s interactive installer doesn't play nicely with Ansible, so these steps are probably best done manually. After connecting to the Pine64, run the following to begin installation:

```sh
curl -sSL https://install.pi-hole.net | bash
```

There's a Pi-hole playbook with some useful tasks that can be run with:

```sh
ansible-playbook -K -v pihole.yml
```

These tasks will add [hoshsadiq's addblock-nocoin-list](https://github.com/hoshsadiq/adblock-nocoin-list) to Pi-hole's blocklists, update Gravity, and change the temperature displays to use Fahrenheit.

Swap your router's DNS configuration to point to the Pine64's IP address (v4 and v6 if you want) and you should be all set!
