# CHIP Provisioning Playbook

This playbook will provision a [CHIP](https://getchip.com) for use as a headless local network server.

## Prerequisites

- [Ansible](https://www.ansible.com) (`brew install ansible`)
- A CHIP
- An SD card (at least 8GB)

## Provisioning

Use the [CHIP Flasher](http://flash.getchip.com) to install the latest headless Debian image to the CHIP Following the [Control CHIP Using a Serial Terminal](https://docs.getchip.com/chip.html#control-chip-using-a-serial-terminal) instructions, connect to CHIP and configure WiFi:

```sh
# Connect to CHIP
screen /dev/tty.usbmodem<xxxx> 115200

# Connect to WiFi
sudo nmcli device wifi connect '<SSID>' password '<password>' ifname wlan0

# Display active connections
nmcli connection show --active

# Test the connection
ping -c 4 8.8.8.8
```

`exit` the interactive shell and kill the screen session with `ctrl-a k`.

You should now be able to SSH into the CHIP:

```sh
ssh chip@<ip-address>
```

Copy the appropriate public key over to the CHIP:

```sh
ssh-copy-id -i ~/.ssh/id_rsa_chip.pub chip@<ip-address>
```

SSH to CHIP, update/upgrade everything, and install Python:

```sh
sudo apt update
sudo apt dist-upgrade
sudo apt install python
```

Run the Ansible playbook to provision the CHIP:

```sh
ansible-playbook --ask-become-pass --verbose --inventory hosts playbook.yml
```
