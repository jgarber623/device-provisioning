# Raspberry Pi: Time Machine

These playbooks will provision a [Raspberry Pi 3 Model B](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/) for use as a macOS Time Machine backup server.

## Getting Started

First, check out this repository and branch:

```sh
git clone -b raspberrypi/timemachine https://github.com/jgarber623/device-provisioning
```

_Great start!_ 👏🏻

Before going much further, there are a few things worth looking at:

1. Look through the `vars/main.yml` files in the `roles` folder.
1. Update defaults to best match your setup (e.g. `fstab__mount_points` in `roles/fstab/vars/main.yml`).
1. Configure your network to assign a static IP address to your Raspberry Pi based on its MAC address. This will make life a bit more predictable (and will give you consistent IP address to use in the `inventory` file).

## Preparing the Hardware

### Configure macOS

First, enable "unsupported" network volume support in Time Machine:

```sh
defaults write com.apple.systempreferences TMShowUnsupportedNetworkVolumes 1
```

Next, install provisioning depencies by running `make boostrap` from the root of this project. This will install the [Ansible](https://www.ansible.com) command line utilities.

### Prepare USB Hard Drive(s)

For each hard drive you wish to use as a networked Time Machine volume, connect the drive to a macOS computer, launch Disk Utility, and erase the drive using the "Mac OS Extended (Journaled)" format.

Next, in a new Finder window, click on the newly-formatted volume in the sidebar and choose "File > Get Info" (`⌘I`). At the bottom of the Get Info window, under "Sharing & Permissions," change all options to "Read & Write" and make sure that "Ignore ownership on this volume" is selected.

### Prepare the Raspberry Pi

1. Download the latest version of [Raspbian Stretch Lite](https://www.raspberrypi.org/downloads/raspbian/) from the Raspberry Pi website.
1. Using a tool like [Etcher](https://etcher.io), write the downloaded `.img` file to an available SD card.
1. Before removing the SD card, add an empty file to [enable SSH on the Raspberry Pi](https://www.raspberrypi.org/documentation/remote-access/ssh/) at boot: `touch /Volumes/boot/ssh`
1. Optionally, configure the Raspberry Pi's GPU settings by appending `gpu_mem=16` to `/Volumes/boot/config.txt`. _(This allocates the minimum allowable memory to the Raspberry Pi's GPU which **should** be okay for a headless server.)_

Insert the SD card into the Raspberry Pi, connect your prepared USB hard drives to the Raspberry Pi, connect everything to power, and wait for the Raspberry Pi to come online.

## Provisioning the Raspberry Pi

### Configure SSH

Create a new SSH key pair for the Raspberry Pi, using a memorable file name like `~/.ssh/id_rsa_raspberry-pi-time-machine`:

```sh
ssh-keygen -t rsa -b 4096 -C 'Raspberry Pi (Time Machine)'
```

Copy the newly-created SSH public key over to the Raspberry Pi using the Raspberry Pi's IP address on your network (`10.0.0.20` in this example):

```sh
ssh-copy-id -i ~/.ssh/id_rsa_raspberry-pi-time-machine.pub pi@10.0.0.20
```

Update your SSH config file (`~/.ssh/config`):

```txt
Host RaspberryPiTimeMachine
  User pi
  HostName 10.0.0.20
  IdentityFile ~/.ssh/id_rsa_raspberry-pi-time-machine
```

You should now be able to connect to the Raspberry Pi without using a password:

```sh
ssh RaspberryPiTimeMachine
```

`exit` out of the SSH session and proceed to the next section!

### Run Ansible Playbooks

Run `make provision` to provision the Raspberry Pi using the playbooks included in this repository.

## Configure Time Machine on macOS

With the Raspberry Pi provisioned, it's time to configure macOS to recognize the Pi and its connected drive(s):

1. Launch System Preferences and open the Time Machine preference pane.
1. Choose "Select Disk…" and select the networked Time Machine volume.
1. When prompted, enter the username and password for the Raspberry Pi and choose the appropriate drive.

Time Machine should soon begin the initial backup!

## Acknowledgments

The following articles and wiki pages were useful in pulling together the software, commands, and configuration in the right order to get this project up and running:

- [DIY Time Capsule with a Raspberry Pi](https://www.calebwoods.com/2015/04/06/diy-time-capsule-raspberry-pi/) by [Caleb Woods](https://www.calebwoods.com)
- [Install Netatalk 3.1.11 on Debian 9 Stretch](http://netatalk.sourceforge.net/wiki/index.php/Install_Netatalk_3.1.11_on_Debian_9_Stretch)
- [How to Use a Raspberry Pi as a Networked Time Machine Drive For Your Mac](https://www.howtogeek.com/276468/how-to-use-a-raspberry-pi-as-a-networked-time-machine-drive-for-your-mac/) by [Justin Pot](https://www.howtogeek.com/author/justinpot/)
- [Build a $35 Time Capsule - Raspberry Pi Time Machine Backup Server](https://raymii.org/s/articles/Build_a_35_dollar_Time_Capsule_-_Raspberry_Pi_Time_Machine.html) by [Remy van Elst](https://raymii.org)
- [RaspberryPi Time Machine](https://scottrchristian.com/2017/09/08/raspberry-pi-time-machine/) by [Scott R. Christian](https://scottrchristian.com)
- [How to use a Raspberry Pi for your Time Machine backups](https://www.jannikarndt.de/blog/2018/01/how_to_use_a_raspberry_pi_for_your_time_machine_backups/) by [Jannik Arndt](https://www.jannikarndt.de)

## License

The code in this repository is freely available under the [MIT License](http://opensource.org/licenses/MIT). Use it, learn from it, fork it, improve it, change it, tailor it to your needs.
