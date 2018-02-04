# CHIP Playbooks

This playbook will provision a [CHIP](https://getchip.com) for use as a headless server using [Ansible](https://www.ansible.com).

### Table of Contents

- [Provisioning CHIP](#provisioning-chip)
	- [Flash CHIP](#flash-chip)
	- [Configure WiFi with nmcli](#configure-wifi-with-nmcli)
	- [Configure SSH](#configure-ssh)
	- [Upgrade to Debian 9 (Stretch)](#upgrade-to-debian-9--stretch-)
	- [Provision with Ansible](#provision-with-ansible)
- [Bonus Knowledge](#bonus-knowledge)
	- [Auto-Mount a USB Device](#auto-mount-a-usb-device)
	- [SSH Host Key Regeneration](#ssh-host-key-regeneration)

## Provisioning CHIP

### Flash CHIP

Use the [CHIP Flasher](http://flash.getchip.com) to install the latest headless Debian image to the CHIP. Following the [Control CHIP Using a Serial Terminal](https://docs.getchip.com/chip.html#control-chip-using-a-serial-terminal) instructions, connect to CHIP:

```sh
# Connect to CHIP
screen /dev/tty.usbmodem<xxxx> 115200
```

### Configure WiFi with nmcli

```sh
# Connect to WiFi
sudo nmcli device wifi connect '<SSID>' password '<password>' ifname wlan0

# Display active connections
nmcli connection show --active

# Test the connection
ping -c 4 8.8.8.8
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

### Upgrade to Debian 9 (Stretch)

SSH to CHIP and prepare for the upgrade by doing:

```sh
sudo apt update
sudo apt install apt-transport-https locales tmux

sudo dpkg-reconfigure locales
```

Edit `/etc/apt/sources.list`:

```text
deb http://deb.debian.org/debian stretch main contrib non-free
deb-src http://deb.debian.org/debian stretch main contrib non-free

deb http://deb.debian.org/debian stretch-updates main contrib non-free
deb-src http://deb.debian.org/debian stretch-updates main contrib non-free

deb http://deb.debian.org/debian stretch-backports main contrib non-free
deb-src http://deb.debian.org/debian stretch-backports main contrib non-free

deb http://security.debian.org/debian-security/ stretch/updates main contrib non-free
deb-src http://security.debian.org/debian-security/ stretch/updates main contrib non-free

deb http://opensource.nextthing.co/chip/debian/repo jessie main
```

Run the following commands to upgrade Debian:

```sh
# Grab updated package versions
sudo apt update

# Start a new tmux session
tmux

# UPGRADE!
sudo apt dist-upgrade
```

Before rebooting, update WiFi settings to use WPA Supplicant instead of Network Manager.

```sh
# Delete nmcli's connection information
sudo nmcli connection delete id '<SSID>'
```

Edit `/etc/network/interfaces`:

```
auto lo

iface lo inet loopback
iface eth0 inet dhcp

allow-hotplug wlan0
auto wlan0

iface wlan0 inet dhcp
wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```

Create and/or edit `/etc/wpa_supplicant/wpa_supplicant.conf`:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="<SSID>"
    psk="<password>"
}
```

Restart CHIP with `sudo reboot`.

### Provision with Ansible

Before running the Ansible playbooks, install Python:

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

### SSH Host Key Regeneration

Keep an eye on `/etc/rc.local` if SSH host keys are regenerating after reboot. There might be some gnarly scripts in there that cause this rather unexpected behavior. A reversable fix:

```sh
sudo mv /etc/rc.local{,.bak}
sudo mv /etc/rc.local{.orig,}

sudo reboot
```
