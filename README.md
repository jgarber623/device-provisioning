# Raspberry Pi: AirPlay

These playbooks will provision a [Raspberry Pi Zero W](https://www.raspberrypi.org/products/raspberry-pi-zero-w/) for use as an AirPlay server using a [HiFiBerry DAC+ Zero](https://www.hifiberry.com/shop/boards/hifiberry-dac-zero/).

## Getting Started

First, check out this repository and branch:

```sh
git clone -b raspberrypi/airplay https://github.com/jgarber623/device-provisioning
```

_Great start!_ 👏🏻

Before going much further, there are a few things worth looking at:

1. Look through the `vars/main.yml` files in the `roles` folder.
1. Update defaults to best match your setup (e.g. `hostname__hostname` in `roles/hostname/vars/main.yml`).
1. Configure your network to assign a static IP address to your Raspberry Pi based on its MAC address. This will make life a bit more predictable (and will give you consistent IP address to use in the `inventory` file).

## Preparing the Raspberry Pi

1. Download the latest version of [Raspbian Buster Lite](https://www.raspberrypi.org/downloads/raspbian/) from the Raspberry Pi website.
1. Using a tool like [balenaEtcher](https://www.balena.io/etcher/), write the downloaded `.img` file to an available SD card.
1. Remove `dtparam=audio=on` from `/Volumes/boot/config.txt`
1. Add `dtoverlay=hifiberry-dac` to `/Volumes/boot/config.txt`
1. Add an empty file to [enable SSH on the Raspberry Pi](https://www.raspberrypi.org/documentation/remote-access/ssh/) at boot: `touch /Volumes/boot/ssh`
1. Configure [wireless networking on the Raspberry Pi](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md): `touch /Volumes/boot/wpa_supplicant.conf`
1. Edit `wpa_supplicant.conf` and add the following content:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=<Insert country code here>

network={
  ssid="<Name of your WiFi>"
  psk="<Password for your WiFi>"
}
```

Insert the SD card into the Raspberry Pi, connect the device to power, and wait for the Raspberry Pi to come online.

## Provisioning the Raspberry Pi

### Configure SSH

Create a new SSH key pair for the Raspberry Pi, using a memorable file name like `~/.ssh/id_rsa_raspberry-pi-airplay`:

```sh
ssh-keygen -t rsa -b 4096 -C 'Raspberry Pi (AirPlay)'
```

Copy the newly-created SSH public key over to the Raspberry Pi using the Raspberry Pi's IP address on your network (`10.0.1.12` in this example):

```sh
ssh-copy-id -i ~/.ssh/id_rsa_raspberry-pi-airplay.pub pi@10.0.1.12
```

Update your SSH config file (`~/.ssh/config`):

```txt
Host RaspberryPiAirPlay
  User pi
  HostName 10.0.1.12
  IdentityFile ~/.ssh/id_rsa_raspberry-pi-airplay
```

You should now be able to connect to the Raspberry Pi without using a password:

```sh
ssh RaspberryPiAirPlay
```

`exit` out of the SSH session and proceed to the next section!

### Run Ansible Playbooks

Run `make provision` to provision the Raspberry Pi using the playbooks included in this repository.

## Acknowledgments

The following articles and wiki pages were useful in pulling together the software, commands, and configuration in the right order to get this project up and running:

- [Setting up a Raspberry Pi headless](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md)
- [Configuring Linux 4.x or Higher](https://www.hifiberry.com/docs/software/configuring-linux-3-18-x/)

## License

The code in this repository is freely available under the [MIT License](https://opensource.org/licenses/MIT). Use it, learn from it, fork it, improve it, change it, tailor it to your needs.
