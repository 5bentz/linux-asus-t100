>  This work is licensed under the Creative Commons Attribution 4.0 International License. To view a copy of this license, visit http://creativecommons.org/licenses/by/4.0/ or send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.

# Installing Ubuntu on ASUS T100 TA

As of November 2018. (20181125)

John Brodie said
> The problem with step by step guides. The information is only accurate for up to a few months.

Follow this guide with a grain of salt. Check if something is working before trying to repair it. After fixing it, verify if it is *really* working.

**2024 Update**: After many years, this is all the truer in view of the fact that many more systems work out of the box: WiFi, Bluetooth, Sound, Brightness control, Hardware video decoding. Many steps are now _outdated_. `[skip if it works]` is added to these sections.

Most importantly, **backup your data**. You already do it monthly, don't you?

### Source repository
Contribute to the guide here: https://github.com/5bentz/linux-asus-t100

### Resources
- Linux on Asus T100 group. Ask your questions here! https://www.facebook.com/groups/381498159345464/
- (Old) Asus T100 Ubuntu group ([Killed by Google](https://killedbygoogle.com/)) https://plus.google.com/communities/117853703024346186936
- Various tutorials with screenshots https://tutorials.ubuntu.com/tutorial/
- Linuxium and Isorespin: customize Ubuntu ISOs! https://linuxiumcomau.blogspot.com/2017/06/customizing-ubuntu-isos-documentation.html
- (2016) Guide for the T100 Ubuntu 16.04: https://drive.google.com/drive/folders/0B4s5KNXf2Z36QW9acnY4RXd3bW8
- I finally found why this Camera driver won't work on an unmodded ASUS laptop-BIOS https://github.com/jfwells/linux-asus-t100ta/issues/4

### Other sources that made this guide possible
- (2016) Latest steps to install Ubuntu on the Asus T100TA: http://www.jfwhome.com/2016/01/04/latest-steps-to-install-ubuntu-on-the-asus-t100ta/
- Installing Debian On AsusT100TA  https://wiki.debian.org/InstallingDebianOn/Asus/T100TA

## The guide starts here!

### 1. Download Ubuntu
Download the ISO file you prefer. I personally like Xubuntu for its lightweight, yet powerful & customizable desktop environment.
- Ubuntu: https://www.ubuntu.com/download/desktop
- Xubuntu: https://xubuntu.org/download
- Other torrents: http://torrent.ubuntu.com/

Note: Download the *64 bit version*. 32 bit versions may fail to boot.

### 2. Flash the installation media
Flash the installation media. We will need to write on it afterwards, so do not use `dd` or *DD mode*.

#### On Windows
Alternatively to this section's instructions, you can follow Ubuntu's tutorial.
https://tutorials.ubuntu.com/tutorial/tutorial-create-a-usb-stick-on-windows#0

- Rufus download: https://rufus.ie/ (Rufus or Rufus Portable)
- Run it

The defaults should be alright, I'd just recommend setting the partition scheme to GTP UEFI, since ASUS T100 and Windows 10 are compatible with it. Make sure you are flashing the *correct* device.

- Partition scheme: GPT UEFI
- Name: 11 characters max, for example *UBUNTU1804*
- File system: default (FAT32)
- Cluster size: default (8192 bytes)
- Image Mode: default (ISO).

### 3. Add the bootable GRUB file for our IA32-powered ASUS T100.
Once Rufus has finished to flash the media:
- Copy the [bootia32.efi](https://github.com/5bentz/linux-asus-t100/raw/master/boot/bootia32.efi) file in the `EFI/BOOT` directory. This directory should already contain various EFI files: probably  `BOOTx64.EFI` and `grubx64.efi`.

If, like jfwells, you would like to build `bootia32.efi` by yourself, follow his guide (primarily for Linux Ubuntu and other Linux Debian-derivatives): https://github.com/jfwells/linux-asus-t100ta/tree/master/boot

### 4. Boot

#### Disable Secure Boot
- Power on your ASUS T100
- Press repetitively the *F2* button at boot to prompt the UEFI menu, namely *Setup Utility*
- Go to the tab `Security`, then `Secure Boot menu`
- Make sure `Secure Boot Support` is `[Disabled]`

#### Boot the installation medium
It is time to boot the installation medium!

- Power on your ASUS T100
- Press repetitively the *ESC* button at boot to prompt the boot menu
- Select your installation medium, in our case: *UBUNTU1804*
- Try Ubuntu without installing (*)
- You might need to turn keyboard's numeric lock (NumLock) off
  * FN + numLock (or FN + Inser) on the keyboard
  * or `numlockx off` in a terminal
- Optionally, change your keyboard layout
  * `setxkbmap countryCode` (*de* for German, *fr* for French, etc)

(*) Some users have random freezes after 5 minutes on every attempt to install Ubuntu. Edit the GRUB parameters to disable the power saving feature in the installer as well.

  * Press *E* on the entry `Try Ubuntu without installing`
  * Add `intel_idle.max_cstate=1` before `quite splash`
  * Press *F10* to boot Ubuntu

#### Run the installer
- In a terminal, run `ubiquity -b`

Note: The flag `-b` is *necessary* in this tutorial. It tells ubiquity not to install a bootloader. Without this flag, ubiquity would crash when trying to install it (Thanks Steven Andrew Mielke!). The bootloader is installed in the section *Bootloader Installation* below.

#### For novice users
For novice users, follow [Ubuntu's tutorial](https://tutorials.ubuntu.com/tutorial/tutorial-install-ubuntu-desktop#4). But do **not reboot** at the end of the installation. Press the button *Continue testing* instead.
 When you are done with Ubuntu's tutorial, jump to the section *Bootloader Installation* in this document.

#### For more advanced users
For more advanced users, choose the last installation type: *Something else*. And jump to the next section *Partitioning*.

### 5. Partitioning
The changes done in this section are not effectively written on the disk.
The actual partitioning will happen when we'll run the installation. Therefore, you can go back at any time and try again.

Note: A new ESP's filesystem is displayed as `ext4` in ubiquity when partitioning, before install. This is a display bug. The ESP is a VFAT or FAT32 partition.

ESP stands for EFI System Partition.

Note: Ignore the device with a single partition of 8014 MB, namely `/dev/mmcblk0`.

#### Make up space for Ubuntu
2 scenarios: keep Windows or ditch Windows.

##### Keep Windows
- You should have already shrunk Windows's partition in Windows (Disks)
  * Windows 7: https://technet.microsoft.com/en-us/library/gg309169.aspx
  * Windows 10: https://docs.microsoft.com/en-us/windows-server/storage/disk-management/shrink-a-basic-volume#BKMK_WINUI

##### Ditch Windows
- Delete each partition, except the ESP. The ESP is probably one of the first partition, its size is 100 MB, and may be labeled `SYSTEM`.
  * Select the partition you want to delete
  * Press the `-` button to delete it.

Note: Alternatively, if you know what you are doing, you can create a new partition table and a new ESP. Backup the old ESP, just in case.

#### Create the partition for Ubuntu
- Select the *Free Space*
- Press the `+` button to add a new partition
  * Size: the rest (this is the default)
  * Use as: ext4 journaling file system
  * *Mountpoint* dropdown-menu: `/`
  * OK

### 6. Installation
- Make sure 'Device for bootloader installation' is the right device, probably `/dev/mmcblk2`
- *Install now*
- ...Installation...
- When finished, *Continue testing*

### 7. Bootloader Installation
- From now onward, we will run the commands as root. To obtain superuser privileges, execute
 * `sudo -s`
- Do *not* use `sudo` for each command, since it fails with some commands (`for` and `>`).

#### Enable WiFi [skip if it works]
/!\ Theses filenames are for T100TA and T100CHI only. Other T100's (T100TAF and 100H\*) have other brcmfmac numbers. See the troubleshooting section *No WiFi* at the end of this document.

/!\ The filename ends with **sdio.txt**. Do not overwrite the file ending with **sdio.bin**.

- `cp /sys/firmware/efi/efivars/nvram-* /lib/firmware/brcm/brcmfmac43241b4-sdio.txt` #useful now
- `cp /sys/firmware/efi/efivars/nvram-* /target/lib/firmware/brcm/brcmfmac43241b4-sdio.txt` #useful after reboot
- `modprobe -r brcmfmac`
- `modprobe brcmfmac`

Now, you should be able to connect your ASUS T100 to your network.

#### Chroot in the new system
- Find the EFI System Partition. This should be the VFAT partition next to `/target`
  * In the example below, it is `mmcblk2p1`
  * If you are unsure, check its size with `lsblk`, it should be about 100M.
  * `lsblk -f`
```
$ lsblk -f
NAME         FSTYPE   LABEL       UUID                  MOUNTPOINT
loop0        squashfs                                   /rofs
sda
└─sda1       ntfs     Restore     0A32F68B32F67AD1
sdb
└─sdb1       vfat     XUBUNTU_18_ D85F-FC95             /cdrom
mmcblk2
├─mmcblk2p1  vfat                 1DA4-A881
└─mmcblk2p2  ext4                 a1994fa2-ddf3-...ff   /target
mmcblk2boot0
mmcblk2boot1
mmcblk0
└─mmcblk0p1  vfat                 9016-4EF8             /media/xubuntu/9016-4EF8
```
- Mount the EFI System Partition on the new system
  * `mount /dev/mmcblk2p1 /target/boot/efi`
- Then, we have to mount some other filesystems before chrooting:
```
for dir in /dev /dev/pts /proc /run /sys /sys/firmware/efi/efivars;
  do mount --bind "$dir" /target/"$dir";
done
```
- Here we go!
  * `chroot /target /bin/bash`

#### Install the Bootloader
- Verify that `/dev` `/dev/pts` `/proc` `/run` `/sys` `/sys/firmware/efi/efivars` and `/boot/efi` are mounted
  * `findmnt`
```
TARGET
/
├─/dev
│ └─/dev/pts
├─/proc
├─/run
├─/sys
│ └─/sys/firmware/efi/efivars
└─/boot/efi
```
- Install grub for EFI-IA32 architecture, and update its config file
  * `apt update`
  * `apt install grub-efi-ia32` #grub-pc removed is normal behavior
  * `grub-install --efi-directory /boot/efi`
  * `update-grub`
- Run `efibootmgr` to see if `ubuntu` is in *BootCurrent* and if it is first in *BootOrder*, as shown below:
```
$ efibootmgr
 BootCurrent: 0001
 Timeout: 1 seconds
 BootOrder: 0001,0002
 Boot0001* ubuntu
 Boot0002* UEFI: USB stick
```

### 8. Boot options
- Boot options must be edited in the file `/etc/default/grub`
  * `nano /etc/default/grub`

#### Power saving
- Edit kernel command-line parameters to add `intel_idle.max_cstate=1` before `quiet`
```
GRUB_CMDLINE_LINUX_DEFAULT="intel_idle.max_cstate=1 quiet splash"
```
- cstate <= 1 is STABLE in 2018.
- cstate >= 2 is NOT stable.

#### GRUB boot screen
- If you want the system to boot faster, let's say 1 second after the GRUB boot screen
```
GRUB_DEFAULT=0
GRUB_TIMEOUT=1
```

#### Finally
- Update the grub configuration file `/boot/efi/grub/grub.cfg`
  * `update-grub`

### 9. Feel free to do other things in the chroot environment, then reboot
- When you are done. Just execute `exit`.
- Before the reboot
  * `umount /target/boot/efi`
- Reboot on the new system.

### 10. Sound [skip if it works]
/!\ T100TA and T100CHI only. Other T100's (T100TAF and T100H\*) has other audio device numbers. You will find files for your device on the [Asus T100 group drive](https://drive.google.com/drive/folders/0B4s5KNXf2Z36VVJDQnY5NEltdmc).
- Download the following folder
  * https://drive.google.com/drive/folders/0B4DiU2o72FbuOXdwRXhfZ3ZmOFE
- Extract it and enter the folder
- Follow the instructions from the file README.txt
  * `sudo rm /var/lib/alsa/asound.state`
  * `sudo mkdir /usr/share/alsa/ucm/bytcr-rt5640`
  * `sudo cp HiFi bytcr-rt5640.conf /usr/share/alsa/ucm/bytcr-rt5640`
- Verify the file are correctly installed, as shown below
```
$ ll /usr/share/alsa/ucm/bytcr-rt5640
total 16
-rw-r--r-- 1 root root 8552 Aug  1 21:35 HiFi
-rw-r--r-- 1 root root  118 Aug  1 21:35 bytcr-rt5640.conf
```
- `sudo alsactl restore`
  * We have sound devices in Pulseaudio now :3 But still no sound.
  * Lower the sound volume, just in case.
- Reboot
  * a new asound file is generated (created before or after reboot), but still no sound
- `sudo cp kernel4.5.xand4.4.x.asound.state /var/lib/alsa/asound.state`
- `sudo alsactl restore`
  * Now we have sound!

If you have no sound, make sure Pulseaudio is correctly set:
- `pavucontrol`
  * Configuration pane
    * Card Name: off
    * Built-in audio: Play HiFi quality music
  * Input device: ignore it, this is for your micro.
  * Output device
    * Port: Headphones or speaker playback
- You are good!

If you still have no sound, see the troubleshooting section *No Sound* at the end of this document.

### 11. Backlit control [skip if it works]
Use xbacklight. Working for kernel >= 4.13 (Ubuntu 1804 has kernel 4.15)
- `xbacklight -inc 1` and `xbacklight -dec 1`
- xbacklight requires to configure Xorg: `/etc/X11/xorg.conf`
```
Section "Device"
    Identifier  "Card0"
    Driver      "intel"
    Option      "Backlight"  "intel_backlight"
EndSection
```

### 12. Hardware video decoding [skip if it works]
With hardware video decoding, a video player should use around 25% CPU when playing a 720p, h264 video fullscreen, instead of 70-100% without hardware decoding.
- `apt install ubuntu-restricted-addons`
- `reboot`
- `apt install vainfo`
  * `vainfo`

### 13. Disable numlock at boot
- Numlock is especially annoying in the login screen, when typing the password...since we do not see the actual characters.
  * `apt remove numlockx`

### 14. Bluetooth [skip if it works]
/!\ Same warning as Sound and WiFi, the following file is for T100TA and T100CHI only. Other T100's (T100TAF and T100H\*) have other Bluetooth device numbers.

Bluetooth should already partially work. For a better support, e.g. *pairing* and *bonding*, we need the firmware file `BCM4324B3.hcd` in the folder `/lib/firmware/brcm/`.
- This file can be found in Windows' partition, in the folder `C:\Windows\system32\drivers`.
- Or, download it from: https://launchpad.net/asust100-ubuntu/+milestone/bluetooth-t100ta
- Install it
  * `mv BCM4324B3.hcd /lib/firmware/brcm/`
- Reboot.

### 15. Sound beep/freeze when playing video in less than 5 minutes with Ubuntu 22.04, 24.04+
See [thesofproject/sof#6384](https://github.com/thesofproject/sof/issues/6384)
```
echo 'options snd_sof sof_debug=1' | sudo tee /etc/modprobe.d/fix-sound-sof.conf
reboot
```

## Troubleshooting

### 1. No WiFi
We have to find out which file your system needs.
- Run dmesg
 * `sudo dmesg`
- Find the following line, ignore the `...`
  * `brcmfmac ...: Direct firmware load for brcm/brcmfmacVWXYZ-sdio.bin for chip ...`
- For example, a T100TAF needs `brcmfmac43340-sdio.txt`.
- You can download it on the ASUS group: https://drive.google.com/drive/folders/0B4s5KNXf2Z36cUpzSURqaTk1TE0

### 2. No Sound

#### Solution A: Turn `realtime-scheduling` off for pulseaudio
For any software, the rule of thumb is to override the configuration by creating a new .conf file in `/etc/software/directory.conf.d/` instead. In this way, the system won't complain during an upgrade of the configuration file (Here `daemon.conf` for pulseaudio).

- Obtain superuser privilege (root)
  * `sudo -s`
- Create a new directory for our configuration file
  * `mkdir -p /etc/pulse/daemon.conf.d/`
- Create the configuration file
  * `echo 'realtime-scheduling = no' > /etc/pulse/daemon.conf.d/50-fix_pulseaudio.conf`
  * You can change the name of the file, provided you keep the .conf extension though.
- For more information
  * `man pulse-daemon.conf`

#### Solution B: Disable sound over HDMI
- Obtain superuser privilege (root)
  * `sudo -s`
- `echo 'blacklist snd_hdmi_lpe_audio' > /etc/modprobe.d/fix_audio.conf`
- Reboot

### 3. T100TAM Touchscreen
Add these kernel command-line parameters: `tsc=reliable clocksource=tsc`

### 4. T100TA does not always boot
Sometimes after loading GRUB, the system gets stuck with the last message being `Loading initial ramdisk ...`.
Usually this happens after a reboot. And most of the time, the second boot then works. But sometimes you have to try many times without success.
All the suggestions which can be found on the internet like adding `GRUB_CMDLINE_LINUX_DEFAULT=debug --verbose initcall_debug eralycon=efifb` in `/etc/default/grub` do not produce any additional output when it's not booting.

One first workaround is to install memtest86+, then first boot into the memtest, wait 3 seconds, reboot and then boot into Linux.

One better workaround seems to work:
- Edit `/etc/default/grub`
  * Enable the line `GRUB_TERMINAL=console`
  * Call `update-grub`

After this there will be the new message `Trying to terminate efi services again` and the system seems to boot and reboot every time successfully.

# History

##### 20240913
* 2024 update!
* Disable power saving feature in the installer as well (cstate=1)
* Use newer bootia32.efi for Ubuntu 24.04
* Sound beep/freeze when playing video in less than 5 minutes
* Mount /sys/firmware/efi/efivars
* Workaround when T100TA does not always boot

##### 20181125
* Verify the mounts before installing bootloader
* Add an audio fix: Disable sound over HDMI
* Add a fix for T100TAM Touchscreen
* C-state >= 2 is not stable
* Minor changes

##### 20181024
* Mention the source repository
* Improve the style
* Create the file *README.md*

##### 20181007
* Advise to become root instead of using `sudo`
* Add a *Bluetooth* section
* Add a *Troubleshooting* chapter with 2 sections: *No WiFi* and *No Sound*

##### 20180815
* Use 64 bit ISOs
* Disable Secure Boot
* Keep the ESP instead of wiping it

##### 20180802
First version
