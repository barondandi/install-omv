# install-omv
OpenMediaVault Installation (Including unsupported and not recommended)

> Procedures to do an OpenMediaVault installation including procedures both unsupported (on a RAID protected filesystem and sharing OS space with data) and not recommended (using and USB flash drive).

# \*\*\* WIP \*\*\*

![](/images/omv_logo.png)

OpenMediaVault is available at  [https://www.openmediavault.org/](https://www.openmediavault.org/) and installation procedure at [https://openmediavault.readthedocs.io/en/5.x/installation/index.html](https://openmediavault.readthedocs.io/en/5.x/installation/index.html).

I will be using version 5.x and only specifying here the steps which are not reflected in the documentation.

1.  [Hardware used](#1.-Hardware-used)
2.  [Some initial recommendations and issues](#2.-some-initial-recommendations-and-issues)
    -   [USB is not recommended, so why?](#USB-is-not-recommended,-so-why?)
    -   [Generic recommendations](#Generic-recommendations)
    -   [USB specific recommendations](#USB-specific-recommendations)
3.  [USB Flash Drive installation](#3.-USB-Flash-Drive-installation)
    -   [Post installation tasks](#Post-installation-tasks)
    -   [Backing up configuration](#Backing-up-configuration)
4.  [RAID installation using an USB flash drive](#4.-RAID-installation-using-an-USB-flash-drive)
5.  [RAID installation using only hard drives](#5.-RAID-installation-using-only-hard-drives)
6.  [References and Credits](#6.-references-and-credits)
7.  [Summary](#7.-summary)


I already went through several NAS builds over the years. From a tailored made FreeBSD kernel to spin FreeNAS, openfiler, Ubuntu Server or a QNAP system.

In the end, I turned to OpenMediaVault, as it saves me a lot of pain for the everyday administration (and I am becoming lazy), and still gives you the full power of a Debian under the hood, that allows you any sort of tailoring, or the option to tinker if anything goes really wrong. This last past is a must for me, as I already had the experience of _**losing all the data**_ in two of my previous NASes (ZFS corruption due to lack of memory and an under-powered Annapurna Labs CPU based QNAP).

You might want to go through this tutorial for some of the following reasons:
-   You have no extra/free SATA ports in your NAS and want to make sure you do a proper installation on a USB drive, wihtout it dying after few weeks or months.
-   You install on a drive with plenty of space. As OMV does not support using OS drive for data storage (as noted in OMV installation guide) you waste a lot of free space on that device.
-   You want to RAID protect your OS to make sure you keep operation even if one of your boot drives fail.


## 1. Hardware used
In case it's of any interest, this is the hardware I will be using for the installation:

-   **Motherboard:** ASRock H370M-ITX/ac Mini ITX LGA1151 Motherboard
    > Mini ITX format and with 6 SATA 6Gb connectors on the board.

-   **CPU:** Intel Pentium Gold G5400 3.7 GHz Dual-Core Processor
    > There is an update of this model in 2019 to Intel Pentium Gold G5420

-   **Memory:** Corsair Vengeance LPX 8 GB (1 x 8 GB) DDR4-2400 Memory
    > One single DIMM as I might upgrade with a second one if I start using the system to spin containers or implement ZFS.
    > I would have preferred Crucial Ballistix but it was not available when ordering at a reasonable price.
    > Make sure to verify that it's supported in the motherboard documentation. Also, check latencies.

-   **Fan:** Noctua NH-L9i
    > Very quiet and low profile.
    > I make it low profile, as when I upgrade the NAS hardware, I can easily reuse and turn the old board into a very small mediacenter.

-   **Case:** Fractal Design Array R2 Mini ITX NAS Case with 300 Watt SFX Power Supply Unit
    > No longer available. Now replaced for the Fractal Design Node 304 Mini ITX Tower Case.

-   **USB Drive:** Samsung Fit Plus 32Gb USB 3.1
    > Avg. Sustained Write Speed 21.8MB/s. Avg. Sequential Write Speed 26.7MB/s ([https://usb.userbenchmark.com/SpeedTest/38059/Samsung-Flash-Drive-FIT](https://usb.userbenchmark.com/SpeedTest/38059/Samsung-Flash-Drive-FIT))

    > NOTE: _If possible, during installation, use a drive with LED activity, to make sure that after finishing installation and specific USB customization, activity is minimal over the drive. Thus, we increase the lifespan of the drive. During installation I used a bigger Sandisk Extreme USB 3.0. Once finished, I reduced the OS partition and cloned it to the final drive (procedure described later)._  

-   **Hard drives #1:** 2 x	SEAGATE 1.5 TB 3.5" 5400RPM
-   **Hard drives #2:** 4 x	Western Digital Red 8 TB 3.5" 5400RPM

## 2. Some initial recommendations and issues

### USB is not recommended, so why?

### Generic recommendations

### USB specific recommendations


-   In motherboards with several Monitor ports, usually only one is enabled initially. Find it and connect the monitor to follow with installation. Later on this can be changed in the BIOS.
    > In my case it was one of the 2 HDMI provided.
-   Get inside the BIOS menu and make sure that:
    -   All the fans are working properly. Proper ventilation of the drives is a must and the main reasons for drives failures are temperature and vibration.
        > In my case, the chassis fan was not working. I had to change the control mode from "Auto" to "PWM Mode".
    -   Disable the hardware that will not be used.
        > I disabled the audio and the WiFi
    -   Make sure S.M.A.R.T. is enabled for the hard drives.


- During installation I was given the choice of using one of the two 1Gb Intel ethernet port, I219V or I211AT. As initially I was going to use one only, I selected the I219-V the other one goes over PCIE (that's what I read at least).
- After automaticaly assigning IP using DNS, I pressed "ESC" and I was given the option to manually configure the network. This is ideal, as I need to set a fixed IP, and I can do that from the begining.
  > I also used this step to specify 3 name servers separating the IPs by spaces: Open DNS (208.67.220.220), Google (8.8.8.8) and a third one for my local Internet provider.

- You get a warning about UEFI mode installation. Force it with no problem.

  > NOTE: If you run into any trouble during installation, you can press "ESC" key and go back to a previous step.  

- If you get a "No root file system" error when partitioning the disks, there are several things you can do:
  - Enter the BIOS menu, and check the USB configuration. In my case I had to change the "Legacy USB Support" to "UEFI Setup Only" for the installation to work.
  - Format the USB Flash drive in EXT4 and make sure you con mount it and write into it. I had to first format it to FAT32 in a Windows system, and then move to a Linux system and overwrite the partition with EXT4 (no idea of the reason, but for me it was the only way to make it work).
- You might get installation giving an error after some percentage during the "Installing the system..." phase. In my case it was due to an USB drive without enough sustained write speed. I had to update to a faster one. You can check USB drives speed tests here: https://usb.userbenchmark.com/. Also, make sure you are connecting it to an USB 3.x connector on the motherboard.

- After the first reboot IPs are not properly configured. Please reboot again, this second time, the IP shoild have been set to the fixed IP specified during installation.

From here, you can login with root/<root_password> and type the command
```shell
omv-firstaid
```
You get into a menu where you can configure the network interface or change the GUI "admin" user password among other things.

Now you can shutdown the array to follow with the procedure:
```shell
shutdown -h now
```
Now we plug the USB drive onto a linux system and use GParted over it to reduce the OS partition size. I will be leaving it at 8Gb, as it is more than enough and leave space to expand afterwards. This way the OS partition to backup will be smaller and take less time and space.

![GParted: Original Layout](/images/gparted_1.png)

-   We right click and Unmount the /dev/sdx2 partition.
-   We can now resize it to the desired amount (I used 8192GiB)

![GParted: Resize](/images/gparted_2.png)

-   And apply the changes before leaving GParted

![](/images/gparted_4.png)
![GParted: Apply changes](/images/gparted_5.png)

-   We eject the USB drive and plug it back on the NAS. We boot again and see what happens. If there are no issues, we log on to the GUI and check that the filesystems are fine:

![GParted: Final Layout](/images/gparted_6.png)

-   First we go to System \ Update Management and we apply the available updates.
-   Then we go to System \ Date and Time and enable NTP service after specifiying the timezone.
> NOTE: Don't forget to SAVE the changes and then APPLY de configuration in the menu that appears. Othenwise changes will be lost.

- To make it easier to power down the system, go to System \ Power button, and change it to "Shutdown" so that we can power down the NAS without any monitor and keyboard.

![Power button](/images/power-button.png)


## 3. USB Flash Drive installation

### Post installation tasks

### Backing up configuration

## 4. RAID installation using an USB flash drive


First install OMV on a USB drive, and boot for the first time. Shut down OMV and insert two drives you actually intend on using, and boot OMV from USB device. Once OMV is up and running, login as root and start SSH service with following command:

service ssh start

omv-install-2

this way you can connect to OMV remotely via SSH, it’s easier to work that way. Now that you are logged in via SSH you need to determine your drives, I use lsscsi for that, but you can use whatever you want. In my demonstration I have OMV installed on /dev/sda, while /dev/sdb and /dev/sdc are SATA drives.

First we need to create partitions on SATA drives, I will use parted for creating 3 partitions, first for GRUB, second for OMV root, and third for data.

```shell
parted -a optimal /dev/sdb mklabel gpt
parted -a optimal /dev/sdb mkpart grub ext2 2048s 12M
parted -a optimal /dev/sdb mkpart root ext4 12M 4096M
parted -a optimal /dev/sdb mkpart data ext4 4096M 100%
parted -a optimal /dev/sdb set 1 bios_grub on
parted -a optimal /dev/sdb set 2 boot on
```

You can change sizes of partitions, I’m using 4G for OMV root partition, in my case that is enough you can change it to better suit your needs. You should end up with something like this:

root@openmediavault:~# parted /dev/sdb u s p
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 83886080s
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number Start End Size File system Name Flags
1 2048s 22527s 20480s grub bios_grub
2 22528s 7999487s 7976960s ext4 root boot
3 7999488s 83884031s 75884544s ext4 data

Now we clone this partition table to second drive /dev/sdc with sgdisk:

sgdisk -R=/dev/sdc /dev/sdb
sgdisk -G /dev/sdc

Now that we have two drives with identical partition setup we can create mdadm RAID arrays on those partitions. First array will be for OMV root drive on /dev/sd(b|c)2 partitions and other array on /dev/sd(b|c)3 partitions.

mdadm --create /dev/md0 --level=1 --raid-devices=2 --metadata=0.90 /dev/sdb2 /dev/sdc2
mdadm --create /dev/md1 --level=1 --raid-devices=2 --metadata=0.90 /dev/sdb3 /dev/sdc3

Create ext4 filesystem on /dev/md0:

mkfs.ext4 /dev/md0

With filesystem on /dev/md0 we can mount it on /mnt/root and sync OMV to /dev/md0.

mkdir /mnt/root
mount /dev/md0 /mnt/root/
rsync -avx / /mnt/root

One more thing, we must add new arrays to the /mnt/root/etc/mdadm/mdadm.conf, and change UUID of / mountpoint in chrooted OMV instalation. You can add new arrays to mdadm.conf with following command:

mdadm --detail --scan >> /mnt/root/etc/mdadm/mdadm.conf

Find UUID of /dev/md0 with blkid command and change UUID for / mount point in /mnt/root/etc/fstab.

Now you are ready to bind mount /dev /sys and /proc to our new root drive and chroot there:

mount --bind /dev /mnt/root/dev
mount --bind /sys /mnt/root/sys
mount --bind /proc /mnt/root/proc
chroot /mnt/root/

Congrats, you are now in your OMV installation on RAID array, there is only GRUB setup left to do and you are golden. Install GRUB on both new drives, update grub configuration and update initramfs and that is it.

grub-install --recheck /dev/sdb
grub-install --recheck /dev/sdc
grub-mkconfig -o /boot/grub/grub.cfg
update-initramfs -u

Exit chroot with exit and shutdown OMV. Remove USB device and boot into OMV on RAID1 array, once boot is done login as root via SSH and create filesystem on /dev/md1 with

mkfs.ext4 -m 1 -L DATA /dev/md1

And this is it, login to web interface, mount /dev/md1 and start using OMV installed on RAID1 array.

## 5. RAID installation using only hard drives

I did not need a pendrive. I used 4 x 4GB hard drives each. I installed the system on the first disk, then created a smaller partition on the other 3 disks. Created raid volume. Then I copied (rsync) the systems to the minor partition (raid) and functioned perfectly.

Finally, I deleted Disk 1 and included it in the raid scheme.

For those who need something similar:

Create disk sceme

```shell
parted -a optimal /dev/sdb mklabel gpt
yes
parted -a optimal /dev/sdb mkpart grub ext2 2048s 12M
parted -a optimal /dev/sdb mkpart swap linux-swap 12M 4096M
parted -a optimal /dev/sdb mkpart root ext4 4096M 10240M
parted -a optimal /dev/sdb mkpart data ext4 10240M 100%
parted -a optimal /dev/sdb set 1 bios_grub on
parted -a optimal /dev/sdb set 2 boot on
```

Copy to other discs (don’t copy to actualy system disc)

```shell
sgdisk -R=/dev/sdc /dev/sdb
sgdisk -G /dev/sdc

sgdisk -R=/dev/sdd /dev/sdb
sgdisk -G /dev/sdd
```

Create the raid w/ 4 discs (frist disc is not in the raid in this moment)
We create only 2 raids in this moment. Swap area and System area

```shell
mdadm –create /dev/md1 –level=10 –raid-devices=4 missing /dev/sdb2 /dev/sdc2 /dev/sdd2
mdadm –create /dev/md2 –level=1 –raid-devices=4 missing /dev/sdb3 /dev/sdc3 /dev/sdd3
```

Create file systems

```shell
mkswap /dev/md1
mkfs.ext4 /dev/md2
mkfs.ext4 -m 1 -L DATA /dev/md3
```

Mount and copy system

```shell
mkdir /mnt/root
mount /dev/md2 /mnt/root/
rsync -avx / /mnt/root
```

update the raid config file

```shell
mdadm –detail –scan >> /mnt/root/etc/mdadm/mdadm.conf
```

Find the UUID of md2 and update /mnt/root/etc/fstab

```shell
blkid
```

Mount system

```shell
mount –bind /dev /mnt/root/dev
mount –bind /sys /mnt/root/sys
mount –bind /proc /mnt/root/proc
chroot /mnt/root/
```

Update grub

```shell
grub-install –recheck /dev/sdb
grub-install –recheck /dev/sdc
grub-install –recheck /dev/sdd
grub-mkconfig -o /boot/grub/grub.cfg
update-initramfs -u
```

Ok, your system are OK. Reboot OMV.


When system up, copy the partition schema to the frist disc (system disc)

```shell
sgdisk -R=/dev/sda /dev/sdb
sgdisk -G /dev/sda
```

Add this disc to raid

```shell
mdadm /dev/md2 –add /dev/sda3
```

Create the raid for the DATA

```shell
mdadm –create /dev/md3 –level=10 –raid-devices=4 /dev/sda4 /dev/sdb4 /dev/sdc4 /dev/sdd4
```

Ok, finaly, update the grub on frist disc

```shell
grub-install –recheck /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
update-initramfs -u
```

Finish!

> Be careful to not use “sfdisk” command to recover partition table from the healthy disk, sfdisk does not support GPT.
Use “sgdisk” instead.
If new disk is /dev/sdb and healthy disk is /dev/sda, then do
> ```shell sgdisk -R=/dev/sdb /dev/sda ```
> to replicate partition table from sda to sdb.
Finally, use OMV GUI to recover RAID mirror.

## 6. References and Credits

This document is based on the following resources. I really thank the authors for sharing their knowledge and trouble:

- Installing OpenMediaVault on RAID-1 array - https://lazic.info/josip/post/installing-openmediavault-on-raid-device/


## 7. Summary

- __**Objetive:**__ Cover OpenMediaVault unsupported and not recommended installation procedures, and hopefully save you some time!
