# Raspberry Pi: Time Machine

These playbooks will provision a [Raspberry Pi 3 Model B](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/) for use as a [Time Machine](https://www.apple.com/in/support/timemachine/) backup server.

## Before You Get Started…

First, check out this repository and branch:

```sh
git clone -b raspberrypi/timemachine https://github.com/jgarber623/device-provisioning
```

_Great start!_ 👏🏻

Before going much further, there are a few things worth looking at:

1. Look through the various `defaults/main.yml` files in the `roles` folder.
1. Update the necessary defaults to best match your setup (e.g. the `fstab__mount_points` list in `roles/fstab/defaults/main.yml`)
1. Consider configuring your network to assign a static IP address to your Raspberry Pi based on its MAC address. This will make life a bit more predictable (and will give you an IP address to add to the `inventory` file).

## Prepare USB Hard Drive(s)

For each hard drive you wish to use as a networked Time Machine volume, connect the drive to a macOS computer, launch Disk Utility, and erase the drive using the "Mac OS Extended (Journaled)" format.

Next, in a new Finder window, click on the newly-formatted volume in the sidebar and choose "File > Get Info" (`cmd + i`). At the bottom of the Get Info window, under "Sharing & Permissions," change all options to "Read & Write" and make sure that "Ignore ownership on this volume" is selected.

## Prepare the Raspberry Pi

1. Download the latest version of [Raspbian Stretch Lite](https://www.raspberrypi.org/downloads/raspbian/) from the Raspberry Pi website.
1. Using a tool like [Etcher](https://etcher.io), write the downloaded `.img` file to an available SD card.
1. Before removing the SD card, add an empty file to [enable SSH on the Raspberry Pi](https://www.raspberrypi.org/documentation/remote-access/ssh/) at boot: `touch /Volumes/boot/ssh`
1. Optionally, configure the Raspberry Pi's GPU settings by appending `gpu_mem=16` to `/Volumes/boot/config.txt`. _(This allocates the minimum allowable memory to the Raspberry Pi's GPU which **should** be okay for a headless server.)_
1. Insert the SD card into the Raspberry Pi, connect your prepared USB hard drives to the Raspberry Pi, connect everything to power, and wait for the Raspberry Pi to come online.

At this point, you should be able to successfully SSH to the Raspberry Pi using a command like:

```sh
ssh pi@10.0.0.20
```

## Prepare macOS and Provision the Raspberry Pi

1. Create an SSH key pair for the Raspberry Pi and save it to `~/.ssh/id_rsa_raspberry-pi-time-machine`:
	```sh
	ssh-keygen -t rsa -b 4096 -C 'Raspberry Pi (Time Machine)'
	```
1. Update your SSH config file (`~/.ssh/config`):
	```txt
	Host RaspberryPiTimeMachine
	  User pi
	  HostName 10.0.0.20
	  IdentityFile ~/.ssh/id_rsa_raspberry-pi-time-machine
	```
1. Run `make boostrap` from the root of this project to install [Ansible](https://www.ansible.com).
1. Run `make provision` to provision the Raspberry Pi using the playbooks included in this repository.

## Configure Time Machine on macOS

1. Enable "unsupported" network volume support in Time Machine:
	```sh
	defaults write com.apple.systempreferences TMShowUnsupportedNetworkVolumes 1
	```
1. Connect to the networked Time Machine volume:
	1. From Finder, choose "Go > Connect to Server…" (`cmd + k`).
	1. Enter `afp://10.0.0.20` in the "Connect to Server" window. _(Note that your Raspberry Pi's IP address may be different.)_
	1. When prompted, enter the username and password for the Raspberry Pi and choose the appropriate shared drive.
1. Launch System Preferences and open the Time Machine preference pane.
1. Choose "Select Disk…" and select the networked Time Machine volume.

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
