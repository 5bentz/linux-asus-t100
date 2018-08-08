>  This work is licensed under the Creative Commons Attribution 4.0 International License. To view a copy of this license, visit http://creativecommons.org/licenses/by/4.0/ or send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.

# Installing Ubuntu on ASUS T100 TA

As of August 2018. (20180802)

John Brodie said
> The problem with step by step guides. The information is only accurate for up to a few months.

Follow this guide with a grain of salt. Check if something is working before trying to repair it. After fixing it, verify if it is *really* working.

### Resources
- Asus T100 Ubuntu group. Ask your questions here! https://plus.google.com/communities/117853703024346186936
- Various tutorials with screenshots https://tutorials.ubuntu.com/tutorial/
- Linuxium and Isorespin: customize Ubuntu ISOs! https://linuxiumcomau.blogspot.com/2017/06/customizing-ubuntu-isos-documentation.html

### Other sources that made this guide possible
- (old) Latest steps to install Ubuntu on the Asus T100TA: http://www.jfwhome.com/2016/01/04/latest-steps-to-install-ubuntu-on-the-asus-t100ta/
- (old) Installing Debian On AsusT100TA  https://wiki.debian.org/InstallingDebianOn/Asus/T100TA

## The guide starts here!

### 1. Download Ubuntu 18.04.1
Download the ISO file you prefer. I personally like Xubuntu for its lightweight, yet powerful & customizable desktop environment.
- Ubuntu: https://www.ubuntu.com/download/desktop
- Xubuntu: https://xubuntu.org/download
- Other torrents: http://torrent.ubuntu.com/

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
- Copy the [bootia32.efi](https://github.com/jfwells/linux-asus-t100ta/raw/master/boot/bootia32.efi) file in the `EFI/BOOT` directory. This directory should already contain various EFI files: probably  `BOOTx64.EFI` and `grubx64.efi`.

If, like jfwells, you would like to build `bootia32.efi` by yourself, follow his guide (primarily for Linux Ubuntu and other Linux Debian-derivatives): https://github.com/jfwells/linux-asus-t100ta/tree/master/boot

### 4. Boot
It is time to boot the installation medium!

- Power on your ASUS T100
- Press repetitively the *ESC* button at boot to prompt the boot menu
- Select your installation medium, in our case: *UBUNTU1804*
- Try Ubuntu without installing
- You might need to turn keyboard's numeric lock (NumLock) off
  * FN + numLock (or FN + Inser) on the keyboard
  * or `numlockx off` in a terminal
- Optionally, change your keyboard layout
  * `setxkbmap countryCode` (*de* for German, *fr* for French, etc)
- In a terminal, run `ubiquity -b` # Does not install a bootloader (Thanks Steven Andrew Mielke!)

For novice users, follow Ubuntu's tutorial. But do **not reboot** at the end of the installation. Press the button *Continue testing* instead.
https://tutorials.ubuntu.com/tutorial/tutorial-install-ubuntu-desktop#4
 When you are done with Ubuntu's tutorial, jump to the section *Install bootloader* in this document.

For more advanced users, choose the last installation type: *Something else*.

### 5. Partitioning
2 scenarios: keep Windows or ditch Windows.

The changes done in this section are not effectively written on the disk.
The actual partitioning will happen when we'll run the installation. Therefore, you can go back at any time and try again.

Note: ESP's filesystem is displayed as `ext4` in ubiquity when partitioning, before install. This is a display bug. The ESP is a VFAT partition.

ESP stands for EFI System Partition.

#### Keep Windows
- You should have already shrunk Windows's partition in Windows (Disks)
  * https://technet.microsoft.com/en-us/library/gg309169.aspx
- Select the *Free Space*
- Press the `+` button to add a new partition
  * *Mountpoint* dropdown-menu: `/`
  * OK

#### Ditch Windows
- Select the first line: `/dev/mmcblk2` (your device might be named differently!)
- New Partition table
- Select the *Free Space*
- Press the `+` button to add a new partition
  * Size: 100 MB (50 MB is enough, ubiquity requires 35 MB)
  * Use as: EFI system partition
- Select the *Free Space*
- Press the `+` button to add a new partition
  * Size: the rest (this is the default)
  * Use as: ext4 journaling file system
  * Mountpoint: `/`

Note: If you want to keep the current EFI system partition, delete the other partitions instead of creating a new partition table.

### 6. Installation
- Make sure 'Device for bootloader installation' is `/dev/mmcblk2`
- *Install now*
- ...Installation...
- When finished, *Continue testing*

### 7. Install bootloader

#### Enable WiFi
/!\ Theses filenames are for T100TA only. Other T100's (T100CHI, etc) has other brcmfmac numbers
- `cp /sys/firmware/efi/efivars/nvram-* /lib/firmware/brcm/brcmfmac43241b4-sdio.txt` #useful now
- `cp /sys/firmware/efi/efivars/nvram-* /target/lib/firmware/brcm/brcmfmac43241b4-sdio.txt` #useful after reboot
- `modprobe -r brcmfmac`
- `modprobe brcmfmac`

Now, you should be able to connect your ASUS T100 to your network.

#### Chroot steps
- Find the EFI System Partition. This should be the VFAT partition next to `/target`
  * `lsblk -f`
```
lsblk -f
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
  * In this example, it is `mmcblk2p1`
  * If you are unsure, check its size with `lsblk`, it should be about 100M.
- `mount /dev/mmcblk2p1 /target/boot/efi`
- Then, we have to mount some filesystems before chrooting:
```
for dir in /dev /dev/pts /proc /run /sys;
  do mount --bind "$dir" /target/"$dir";
done
```
- `chroot /target /bin/bash`

#### Bootloader Installation
- `apt update`
- `apt install grub-efi-ia32 # grub-pc removed is normal behavior`
- `grub-install --efi-directory /boot/efi`
- `update-grub`

- Check `efibootmgr` to see if `ubuntu` is in *BootCurrent* and first in *BootOrder*, as shown below:
```
 BootCurrent: 0001
 Timeout: 1 seconds
 BootOrder: 0001,0002
 Boot0001* ubuntu
 Boot0002* UEFI: USB stick
```

### 8. Boot options
#### Power saving
- Edit kernel boot options to add `intel_idle.max_cstate=1` before `quiet`
  * `nano /etc/default/grub`
```
GRUB_CMDLINE_LINUX_DEFAULT="intel_idle.max_cstate=1 quiet splash"
```
- cstate <= 1 is STABLE in 2018.
- I don't know if cstate >= 2 is stable.

#### GRUB boot screen
- If you want the system to boot faster, let's say 1 second after the GRUB boot screen
```
GRUB_DEFAULT=0
GRUB_TIMEOUT=1
```
- Update the grub configuration file in `/boot/efi/grub/grub.cfg`
  * `update-grub`

### 9. Feel free to do other things in the chroot environment
- When you are done. Just execute `exit`.
- Before the reboot
  * `umount /target/boot/efi`

### 10. Sound (T100TA)
/!\ T100TA only. Other T100's (T100CHI, etc) has other audio device numbers
- Download the following folder
  * https://drive.google.com/drive/folders/0B4DiU2o72FbuOXdwRXhfZ3ZmOFE?tid=0B9C1WK1FQhjfcXNrbzN6djQzajg
- Extract it and enter the folder
- I'm following the instructions from the file README.txt

- `sudo rm /var/lib/alsa/asound.state`
- `sudo mkdir /usr/share/alsa/ucm/bytcr-rt5640`
- `sudo cp HiFi bytcr-rt5640.conf /usr/share/alsa/ucm/bytcr-rt5640`
```
ll /usr/share/alsa/ucm/bytcr-rt5640
total 16
-rw-r--r-- 1 root root 8552 Aug  1 21:35 HiFi
-rw-r--r-- 1 root root  118 Aug  1 21:35 bytcr-rt5640.conf
```
- `sudo alsactl restore`
  * We have sound devices in Pulseaudio now :3 But still no sound.
  * Lower the sound volume, just in case.
- reboot
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

### 11. Backlit control
Use xbacklight. Working for kernel >= 4.13 (Ubuntu 1804 has 4.15)
- `xbacklight -inc 1` and `xbacklight -dec 1`
- xbacklight requires to configure Xorg: `/etc/X11/xorg.conf`
```
Section "Device"
    Identifier  "Card0"
    Driver      "intel"
    Option      "Backlight"  "intel_backlight"
EndSection
```

### 12. Hardware video decoding
- `apt install ubuntu-restricted-addons`
- `reboot`
- `apt install vainfo`
  * `vainfo`
- Parole should use around 25% CPU when playing a 720p, h264 video fullscreen, instead of 70-100% without hardware decoding.

### 13. Disable numlock at boot
- Numlock is especially annoying in the login screen, when typing the password...since we do not see the actual characters.
  * `apt remove numlockx`
