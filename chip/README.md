# CHIP Playbooks

This playbook will provision a [CHIP](https://getchip.com) for use as a headless server using [Ansible](https://www.ansible.com).

### Table of Contents

- [Provisioning CHIP](#provisioning-chip)
	- [Flash CHIP](#flash-chip)
	- [Configure WiFi](#configure-wifi)
	- [Configure SSH](#configure-ssh)
	- [Upgrade to Debian 10 (Buster)](#upgrade-to-debian-10--buster-)
	- [Provision with Ansible](#provision-with-ansible)
- [Bonus Knowledge](#bonus-knowledge)
	- [Auto-Mount a USB Device](#auto-mount-a-usb-device)

## Provisioning CHIP

### Flash CHIP

Use the [CHIP Flasher](http://flash.getchip.com) to install the latest headless Debian image to the CHIP. Following the [Control CHIP Using a Serial Terminal](https://docs.getchip.com/chip.html#control-chip-using-a-serial-terminal) instructions, connect to CHIP:

```sh
screen /dev/tty.usbmodem<xxxx> 115200
```

### Configure WiFi

Edit `/etc/network/interfaces`:

```
auto lo

iface lo inet loopback
iface eth0 inet dhcp

allow-hotplug wlan0
iface wlan0 inet dhcp

wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```

Edit `/etc/wpa_supplicant/wpa_supplicant.conf`:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="<SSID>"
    psk="<password>"
}
```

`exit` the interactive shell and kill the screen session with `ctrl-a k`.

### Configure SSH

You should now be able to SSH into the CHIP:

```sh
ssh chip@<ip-address>
```

Copy the appropriate public key over to the CHIP:

```sh
ssh-copy-id -i ~/.ssh/id_rsa_chip.pub chip@<ip-address>
```

Double check that you can SSH to the CHIP without requiring a password.

You may also want to prevent `/etc/rc.local` from regenerating SSH host keys on reboot. There's possibly a gnarly script in `/etc/rc.local` that causes this rather unexpected behavior. To contend with that, do:

```sh
sudo mv /etc/rc.local{,.bak}
sudo mv /etc/rc.local{.orig,}
```

### Upgrade to Debian 10 (Buster)

SSH to CHIP and prepare for the upgrade by doing:

```sh
sudo apt update
sudo apt install locales tmux

sudo dpkg-reconfigure locales
```

Edit `/etc/apt/sources.list`:

```text
deb http://deb.debian.org/debian buster main contrib non-free
deb-src http://deb.debian.org/debian buster main contrib non-free

deb http://deb.debian.org/debian buster-updates main contrib non-free
deb-src http://deb.debian.org/debian buster-updates main contrib non-free

deb http://deb.debian.org/debian stretch-backports main contrib non-free
deb-src http://deb.debian.org/debian stretch-backports main contrib non-free

deb http://security.debian.org/debian-security/ buster/updates main contrib non-free
deb-src http://security.debian.org/debian-security/ buster/updates main contrib non-free

deb http://opensource.nextthing.co/chip/debian/repo jessie main
```

Start a tmux session (with `tmux`) and run the following commands to upgrade Debian:

```sh
sudo apt update
sudo apt dist-upgrade
```

Restart CHIP with `sudo reboot`.

After the restart, make sure everything's starting up properly. You might also want to run `sudo apt autoremove` to clean up outdated packages.

### Provision with Ansible

Before running the Ansible playbooks, SSH to CHIP and install Python:

```sh
sudo apt update
sudo apt install python
```

Run the Ansible playbook to provision the CHIP:

```sh
ansible-playbook -K -v playbook.yml # or run individual playbooks
```

## Bonus Knowledge

### Auto-Mount a USB Device

```sh
sudo mkdir -p <mount-path> # (e.g. `/media/foo`)
sudo mount /dev/sda1 <mount-path>
```

Update `/etc/fstab` using the device's UUID (`sudo blkid /dev/sda1`):

```
UUID=<device-uuid>    <mount-path>    hfsplus    defaults,nofail    0    0
```
