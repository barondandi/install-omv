# install-omv
OpenMediaVault Installation (Including unsupported and not recommended)

> Procedures to do an OpenMediaVault installation including procedures both unsupported (on a RAID protected filesystem and sharing OS space with data) and not recommended (using and USB flash drive).

![](/images/omv_logo.png)

OpenMediaVault is available at  [https://www.openmediavault.org/](https://www.openmediavault.org/) and installation procedure at [https://openmediavault.readthedocs.io/en/5.x/installation/index.html](https://openmediavault.readthedocs.io/en/5.x/installation/index.html).

I will be using version 5.x and only specifying here the steps which are not reflected in the documentation.

1.  [Hardware used](#1.-Hardware-used)
2.  [Some initial recommendations and issues](#2.-some-initial-recommendations-and-issues)
    -   [USB is not recommended, so why?](#USB-is-not-recommended,-so-why?)
    -   [Generic recommendations](#Generic-recommendations)
    -   [USB specific recommendations](#USB-specific-recommendations)
3.  [USB Flash Drive installation](#3.-USB-Flash-Drive-installation)
    -   [Reduce OS partition size](#Reduce-OS-partition-size)
    -   [Post installation tasks](#Post-installation-tasks)
    -   [Enable flashmemory plugin](#Enable-flashmemory-plugin)
    -   [Finish configuration](#Finish-configuration)
    -   [Backing up USB](#Backing-up-USB)
4.  [RAID installation using an USB flash drive](#4.-RAID-installation-using-an-USB-flash-drive)
5.  [RAID installation using only hard drives](#5.-RAID-installation-using-only-hard-drives)
6.  [Data migration](#6.-Data-migration)
7.  [Converting RAID5 to RAID6](#7.-Converting-RAID5-to-RAID6)
    -   [Power down during a RAID rebuild](#Power-down-during-a-RAID-rebuild)
8.  [References and Credits](#8.-references-and-credits)
9.  [Summary](#9.-summary)

I already went through several NAS builds over the years. From a tailored made FreeBSD kernel to spin FreeNAS, openfiler, Ubuntu Server or a QNAP system.

In the end, I turned to OpenMediaVault, as it saves me a lot of pain for the everyday administration (and I am becoming lazy), and still gives you the full power of a Debian under the hood, that allows you any sort of tailoring, or the option to tinker if anything goes really wrong. This last past is a must for me, as I already had the experience of _**losing all the data**_ in two of my previous NASes (ZFS corruption due to lack of memory and an under-powered Annapurna Labs CPU based QNAP).

You might want to go through this tutorial for some of the following reasons:
-   You have no extra/free SATA ports in your NAS and want to make sure you do a proper installation on a USB drive, without it dying after few weeks or months.
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

I would like to free all SATA ports in the motherboard so that the NAS can be expanded to the fullest.

> Though initially I will start with 4 disks, I want to be able to expand to 6 in the future (as 8TB drive prices decrease).

The **protection** I will be using is RAID 6 meaning I will be investing 2 drives in parity. If I want to be space effective I need as many drives as possible with the same layout as the protection ones (meaning same size and partitioning _-thus, OS in a RAID protected partition is discarded-_):
-   RAID 5 is too risky in big drives.
-   RAID 1+0 is less space effective as we go over 4 drives.

To compensate for the USB chances of failure I will be taking the following measures:
-   Enable the openmediavault-flashmemory plugin. This lowers the amount of writes to the USB flash drive. Also swap partition will be disabled on the drive (this will surely impact performance, but minimize drive wear).
-   Automate USB backups, so that recovery is easy if anything happens.

### Generic recommendations

In motherboards with several Monitor ports, usually only one is enabled initially. Find it and connect the monitor to follow with installation. Later on this can be changed in the BIOS.
    > In my case it was one of the 2 HDMI provided.

Get inside the BIOS menu and make sure that:

-   All the fans are working properly. Proper ventilation of the drives is a must. The main reasons for drives failures are temperature and vibration.
    > In my case, the chassis fan was not working. I had to change the control mode from "Auto" to "PWM Mode".

-   Disable the hardware that will not be used.
    > I disabled the audio and the WiFi

-   Make sure S.M.A.R.T. is enabled for the hard drives.

During installation I was given the choice of using one of the two 1Gb Intel Ethernet port, I219V or I211AT. As initially I was going to use one only, I selected the I219-V the other one goes over PCIE (_that's what I read at least_).


### USB specific recommendations

If you get a "_No root file system_" error when partitioning the disks, there are several things you can do:

-   Enter the BIOS menu, and check the USB configuration. In my case I had to change the "Legacy USB Support" to "UEFI Setup Only" for the installation to work.
-   Format the USB Flash drive in EXT4 and make sure you con mount it and write into it. I had to first format it to FAT32 in a Windows system, and then move to a Linux system and overwrite the partition with EXT4 (no idea of the reason, but for me it was the only way to make it work).

You might get installation giving an error after some percentage during the "Installing the system..." phase. In my case it was due to an USB drive without enough sustained write speed.

> I had to update to a faster one. You can check USB drives speed tests here: [https://usb.userbenchmark.com/](https://usb.userbenchmark.com/). Also, make sure you are connecting it to an USB 3.x connector on the motherboard.

## 3. USB Flash Drive installation

We follow installation instructions in the OMV guide, "Installation on USB" ([https://openmediavault.readthedocs.io/en/5.x/installation/on_usb.html](https://openmediavault.readthedocs.io/en/5.x/installation/on_usb.html)), that simply state to follow the "Installation using an ISO image" ([https://openmediavault.readthedocs.io/en/5.x/installation/via_iso.html](https://openmediavault.readthedocs.io/en/5.x/installation/via_iso.html)) and afterwards enable the openmediavault-flashmemory plugin as we will be detailing. Some comments and issues follow:

-   After automaticaly assigning IP using DNS, I pressed "ESC" and I was given the option to manually configure the network. This is ideal, as I need to set a fixed IP, and I can do that from the begining.
    > I also used this step to specify 3 name servers separating the IPs by spaces: Open DNS (208.67.220.220), Google (8.8.8.8) and a third one for my local Internet provider.

-   You get a warning about UEFI mode installation. Force it with no problem.
    > **NOTE: If you run into any trouble during installation, you can press "ESC" key and go back to a previous step.**  

-   After the first reboot IPs are not properly configured. Please reboot again, this second time, the IP shoild have been set to the fixed IP specified during installation.

From here, you can login with root/<root_password> and type the command
```shell
omv-firstaid
```
You get into a menu where you can configure the network interface or change the GUI "admin" user password among other things.

Now you can shutdown the array to follow with the procedure:
```shell
shutdown -h now
```

### Reduce OS partition size

Now we plug the USB drive onto a linux system and use GParted over it to reduce the OS partition size. I will be leaving it at 8Gb, as it is more than enough and leave space to expand afterwards. This way the OS partition to backup will be smaller and take less time and space (_allowing recovery over smaller USBs if needed_).

![GParted: Original Layout](/images/gparted_1.png)

We right click and `unmount` the /dev/sdx2 partition.

We can now resize it to the desired amount (I used 8192GiB):

![GParted: Resize](/images/gparted_2.png)

And apply the changes before leaving GParted:

![](/images/gparted_4.png)
![GParted: Apply changes](/images/gparted_5.png)

We eject the USB drive and plug it back on the NAS. We boot again and see what happens. If there are no issues, we log on to the GUI and check that the filesystems are fine:

![GParted: Final Layout](/images/gparted_6.png)

In case we get an "UNEXPECTED INCONSISTENCY" over any of the partitions and we are told to run `fsck` manually, we simply need to do that and reboot. For example if we get an "The root filesystem on /dev/sda1 requires a manual fcsk" message, we would run:

```shell
fcsk /dev/sda1
```

### Post installation tasks

-   First we go to System \ Update Management and we apply the available updates.
-   Then we go to System \ Date and Time and enable NTP service after specifying the timezone.
    > NOTE: Don't forget to SAVE the changes and then APPLY de configuration in the menu that appears. Otherwise changes will be lost.

-   We go to Services \ SSH and enable the service. This way you we can connect to OMV remotely via SSH, it’s easier to work that way.

-   To make it easier to power down the system, go to System \ Power button, and change it to "Shutdown" so that we can power down the NAS without any monitor and keyboard.

![Power button](/images/power-button.png)

### Enable flashmemory plugin

To enable the openmediavault-flashmemory plugin, first we need to enable OMV-Extras.
-   The guide to do it is here: [https://forum.openmediavault.org/index.php?thread/5549-omv-extras-org-plugin/](https://forum.openmediavault.org/index.php?thread/5549-omv-extras-org-plugin/)
-   The list of plugins that it enables is here (for OMV 5.x): [https://bintray.com/beta/#/openmediavault-plugin-developers/usul?tab=packages](https://bintray.com/beta/#/openmediavault-plugin-developers/usul?tab=packages)

To install the preferred method is to run from the command line as root (you can check in the guide an alternative method from OMV web interface if needed):
> NOTE:  Remember that the SSH should be enabled, and it is easy to copy and and paste the commands from a terminal.

```shell
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/packages/raw/master/install | bash
```

After running this command if we go to the web interface System \ Plugins we see that the openmediavault-omvextrasorg is enabled.
![Plugins: OMV Extras](/images/plugins_1.png)

Several new plugins have appeared, among them openmediavault-flashmemory. We install the plugin:
![Plugins: Install flashmemory](/images/plugins_2.png)

A new category also appears: System \ OMV-Extras. If we are not going to use it, we disable the Backports.

Just in case, to avoid issues, we reboot the NAS. Then we follow the optional procedures specified in Storage \ Flash Memory (openmediavault-flashmemory plugin). With this we disable the swap partition on the USB drive. Very likely this will impact performace, but there was activity over the drive with it enabled.

Fstab (/etc/fstab) needs to be changed manually. Following these steps to change:

1.  Login as root locally or via ssh
2.  Execute the following command: `nano /etc/fstab`
3.  Add noatime and nodiratime to root options. See before and after example lines:
    -   BEFORE: `UUID=ccd327d4-a1ed-4fd2-b356-3b492c6f6c34 / ext4 errors=remount-ro 0 1`
    -   AFTER: `UUID=ccd327d4-a1ed-4fd2-b356-3b492c6f6c34 / ext4 noatime,nodiratime,errors=remount-ro 0 1`
4.  Comment out the swap partition. See before and after example lines (only need to add a # to beginning of the line):
    -   BEFORE: `UUID=a3c989d8-e12b-41d3-b021-098155d6b21b none swap sw 0 0`
    -   AFTER: `#UUID=a3c989d8-e12b-41d3-b021-098155d6b21b none swap sw 0 0`
5.  Ctrl-o to save
6.  Ctrl-x to exit
7.  If you disable swap, initramfs resume should fixed to avoid mdadm messages.
    -   Remove the resume file: `rm /etc/initramfs-tools/conf.d/resume`
    -   Update initramfs: `update-initramfs -u`
8.  On systems using GPT, swap will still be auto-detected. The swap partition should be turned off and wiped.
    -   Find the swap partition with: `blkid | grep swap`
    -   Example output where /dev/sda2 is the swap partition: `/dev/sda2: UUID="01c2d1ab-354b-4e2b-8d7c-ca35793f5fe7" TYPE="swap" PARTUUID="4f64854c-02"`
    -   Disable the swap partition with swapoff: `swapoff /dev/sda2`
    -   Wipe the swap partition with wipefs: `wipefs -a /dev/sda2`
9.  Reboot

### Finish configuration

We power down the NAS, connect the hard disk drives, and restart the system.

Fist we **enable S.M.A.R.T.** over the added drives. Thus we can check when is likely that one of the drives fails. We can do that from _Storage \ S.M.A.R.T._ (not supported for the USB flash drive). First we enable it on the _Settings_ tab, and after on the individual drives on the _Devices_ tab:

![SMART: Devices](/images/smart_1.png)

We configure the RAID protection we desire in _Storage \ RAID Management_:

![RAID: Created](/images/raid_1.png)

Be aware that depending on the dives speed and size, the process could take several hours (for example, building a RAID 5 group with 8 TB drives, took me close to 23 hours):

![RAID: Build time](/images/raid_2.jpg)

Then we create and mount the filesystems over the created RAID devices from _Storage \ File Systems_:

![FILESYSTEM: Created](/images/filesystem_1.png)

We create a user for accessing the shares in _Access Rights Management \ User_. By default it belongs to users group.

![USER: Created](/images/user_1.png)

Now we proceed to create the shared folders for the different protocols. I will use NFS to write content, and also use CIFS (Windows) to share in read -only mode.

I will start activating NFS from _Services \ NFS \ Settings_. Then we move to the _Shares_ tab, and from there we create the different exports. We press Add button, and from the menu that appears, first we need to _Add shared folder_:

![NFS: Add shared folder](/images/nfs_1.png)

Then we to to fill the rest of the fields in _Add share_:

![NFS: Add share](/images/nfs_2.png)

We repeat the process for all the exports we want to create:

![NFS: Shares](/images/nfs_3.png)

Once we are finished we move to _Services \ SMB/CIFS \ Settings_ in order to activate CIFs. There we can enable the service and change the default Workgroup among other things. Then we move to the _Shares_ tab, and from there we add the different shares. We press Add button, and from the menu that appears, first we no longer need need to _Add shared folder_, as we already did it for the NFS protocol. I mark that I want the shares to be read-only:

![CIFS: Add share](/images/cifs_1.png)

We repeat the process for all the shares we want to create:

![CIFS: Created](/images/cifs_2.png)

Now we move back to _Access Rights Management \ User_ and click on the _Privileges_ button for the user we created. If we want the access user we created to be able to write into the shares we created, we need to specify it here:

![USER: Privileges](/images/user_2.png)

With this we are done and we should be able to access the created shares. I will test it in a Linux host, where I hae created the _\mnt\MediaVault_ folder. I will mount over this folder the root share to check the contents. I can do that by using the command:

```shell
sudo mount -t nfs -v -o proto=tcp <omv_ip>:/ <folder_to_mount>
```

In my case this is the output (I also wrote a test file and unmounted the export afterwards):

```shell
[user@fedora ~]$ sudo mount -t nfs -v -o proto=tcp 192.168.1.2:/ /mnt/MediaVault
[sudo] password for user:
mount.nfs: timeout set for Sun May 17 12:06:13 2020
mount.nfs: trying text-based options 'proto=tcp,vers=4.2,addr=192.168.1.2,clientaddr=192.168.1.101'

[user@fedora ~]$ cd /mnt/MediaVault/

[user@fedora MediaVault]$ ls -alF
total 28
drwxr-xr-x. 7 root root  4096 May 17 00:53 ./
drwxr-xr-x. 4 root root  4096 Jul 25  2019 ../
drwxrwsr-x. 4 root users 4096 May  4 23:42 BackUp/
drwxrwsr-x. 7 root users 4096 May 17 10:37 GAMEs/
drwxrwsr-x. 2 root users 4096 May  5 23:33 MOVIEs_HD/
drwxrwsr-x. 2 root users 4096 May  5 23:35 MOVIEs_Series/
drwxrwsr-x. 2 root users 4096 May  5 23:36 MUSIC_HD/

[user@fedora MediaVault]$ cd BackUp/

[user@fedora BackUp]$ echo "Write test" > test.txt

[user@fedora BackUp]$ ls -ahlF
total 20K
drwxrwsr-x. 4 root    users 4.0K May 17 12:06 ./
drwxr-xr-x. 7 root    root  4.0K May 17 00:53 ../
-rw-rw-r--. 1 user users   11 May 17 12:06 test.txt

[user@fedora BackUp]$ sync

[user@fedora BackUp]$ cd

[user@fedora ~]$ sudo umount /mnt/MediaVault

```

Now we are ready to start rocking... :)

### Backing up USB

We power down the NAS, disconnect the USB drive, and connect it to a Linux box. _If you use other OS, you can easily search other tools and tutorials in Internet._

The tool we’ll use to create the bootable drive clone from the command line is the `dd` command.
> NOTE: If the USB drive gives any error you could try replacing `dd` by `ddrescue` command.

> **Warning**: `dd` command must be used very carefully. dd will do exactly what you tell it to, as soon as you tell it. There are no “Are you sure” questions or chances for backing out. This means that you could overwrite a drive you don't intend to if you are not careful.

We need to know what device your USB drive is associated with. That way you know for sure what device identity to pass to dd on the command line.

In a terminal window type the following command. The lsblk command lists the block devices on your computer. Each drive has a block device associated with it.

```shell
lsblk
```

This is an output example:

```shell
[user@fedora ~]$ lsblk
NAME                                            MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sdb                                               8:16   1  29.9G  0 disk  
├─sdb1                                            8:17   1   512M  0 part  
├─sdb2                                            8:18   1     8G  0 part  /run/media/user/7e4ce9ae-1ed5-4041-8dbd-5f2f8b04375a
└─sdb3                                            8:19   1   7.7G  0 part  
nvme0n1                                         259:0    0 894.3G  0 disk  
├─nvme0n1p1                                     259:1    0   200M  0 part  /boot/efi
├─nvme0n1p2                                     259:2    0     1G  0 part  /boot
└─nvme0n1p3                                     259:3    0 893.1G  0 part  /  

```

The output from lsblk will show the drives currently connected to your computer. There is one usb drive called `sdb` and there is one internal hard drive on this machine called `nvme0n1`.

We can see the three partitions we previously saw on GParted.

If you are uncertain of which is the USB drive, you can run the `lsblk` command **before** inserting the USB, and then again, **after** inserting it, and check which is the new drive that appeared.

The identifier we need to use is the one representing the drive, not either of the partitions. In our example this is sdb. Regardless of how it is named on your computer, the device that was not in the previous lsblk listing must be the USB drive.

The command we are going to issue to `dd` is as follows:

```shell
sudo dd if=/dev/sdb of=Downloads/YYYY-MM-DD_omv.img conv=fdatasync bs=4M status=progress
```

Where the explanation of the different parameters follow:
-   **sudo**: You need to be a superuser to issue dd commands. You will be prompted for your password.
-   **dd**: The name of the command we’re using.
-   **bs=4M**: The -bs (blocksize) option defines the size of each chunk that is read from the input file and wrote to the output device. 4 MB is a good choice because it gives decent throughput and it is an exact multiple of 4 KB, which is the blocksize of the ext4 filesystem. This gives an efficient read and write rate.
-   **if=/dev/sdb**: The -if (input file) option requires the path and name of the USB drive you are using as the input file. This is the value we identified by using the lsblk command previously. in our example it is sdb, so we are using /dev/sdb. Your USB drive might have a different identifier. Make sure you provide the correct identifier.
-   **of=Downloads/YYYY-MM-DD_omv.img**: The -of (output file) is the path and name of the file you are using as the output file. In our case we are placing the clone in a file named "YYYY-MM-DD_omv.img" on the Download folder.
-   **conv=fdatasync**: The conv parameter dictates how dd converts the input file as it is written to the output device. dd uses kernel disk caching when it writes to the USB drive. The fdatasync modifier ensure the write buffers are flushed correctly and completely before the creation process is flagged as having finished. This parameter is not relevant when doing the backups, but it is very important for the restoration processes
-   **status=progress**: To see progress while making an image with dd.

> NOTE: If you get read errors, you can use the parameter `conv=noerror` to tell `dd` to continue operation, ignoring all read errors. Or even better, install and use `ddrescue` instead of `dd`.

When the bootable USB drive image has been created dd reports the amount of data that was written to the USB drive, the elapsed time in seconds and the average data transfer rate.

This is an output example:

```shell
[user@fedora ~]$ sudo dd if=/dev/sdb of=Downloads/YYYY-MM-DD_omv.img conv=fdatasync bs=4M status=progress
32019316736 bytes (32 GB, 30 GiB) copied, 416 s, 77.0 MB/s
7648+1 records in
7648+1 records out
32080200192 bytes (32 GB, 30 GiB) copied, 423.676 s, 75.7 MB/s

[user@fedora ~]$ ls -ahlF Downloads/YYYY*
-rw-r--r--. 1 root root 30G May 10 17:28 Downloads/YYYY-MM-DD_omv.img
```

As you can see the output is a bit by bit copy of the USB drive, meaning there are lots of zeros in it and thus wasted space. We can easily sort this out by compressing the image afterwards, or doing everything on the same command:

```shell
sudo dd if=/dev/sdb conv=fdatasync bs=4M status=progress | gzip -c > Downloads/YYYY-MM-DD_omv.gz
```

If we check the output, we really saved a lot of space:

```shell
[user@fedora ~]$ ls -ahlF Downloads/YYYY*
-rw-r--r--. 1 user users  30G May 10 17:28 Downloads/YYYY-MM-DD_omv.img
-rw-r--r--. 1 user users 1.3G May 10 17:28 Downloads/YYYY-MM-DD_omv.img.gz

[user@fedora ~]$ gzip -l Downloads/YYYY-MM-DD_omv.img.gz
         compressed        uncompressed  ratio uncompressed_name
         1372693831          2015429120  31.9% Downloads/YYYY-MM-DD_omv.img
```

If we need to **build another USB drive** from the created image the command would be (if we did not compress it):

```shell
sudo dd if=Downloads/YYYY-MM-DD_omv.img of=/dev/sdb conv=fdatasync bs=4M status=progress
```

If we used gzip to save space this would be the command:
gunzip -c IMAGE.HERE-GZ | dd of=/dev/OUTPUT/DEVICE-HERE

```shell
sudo gzip -c Downloads/YYYY-MM-DD_omv.gz | dd of=/dev/sdb conv=fdatasync bs=4M status=progress
```

You can check the bootable USB drive works by rebooting your computer and booting from the USB drive, or you can try booting from it in another computer.


## 4. RAID installation using an USB flash drive

> NOTE: I describe here Josip Lazić procedure (link in References at the end), just adding some comments. I did not follow this porcedure as I went for the USB installation in the end (for the reasons already given).

First install OMV on a USB drive, as previously described, and boot for the first time. Do not enable flashmemory plugin. Shut down OMV and insert two drives you actually intend on using, and boot OMV from USB device.

 Now that you are logged in via SSH you need to determine your drives, I use `lsscsi` for that, but you can use whatever you want. In my demonstration I have OMV installed on `/dev/sda`, while `/dev/sdb` and `/dev/sdc` are SATA drives.

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

```shell
root@openmediavault:~# parted /dev/sdb u s p
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 83886080s
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number Start End Size File system Name Flags
1 2048s 22527s 20480s grub bios_grub
2 22528s 7999487s 7976960s ext4 root boot
3 7999488s 83884031s 75884544s ext4 data
```

> NOTE: I would recommend reproducing the partition sizes that you have on the USB drive. The space reserved for GRUB is 512M (instead of 12M) and the filesystem type vfat. If you plan to install Docker or other software, you might want to increase the size of the OMV root partition to 8GB. And finally you might also want to add the swap partition of the size of your memory (or the one that you expect to have if expanding it is in the horizon). As in the example, place the data partition at the end, and make it size to fill !00% of the remaining space.

Now we clone this partition table to second drive /dev/sdc with sgdisk:

```shell
sgdisk -R=/dev/sdc /dev/sdb
sgdisk -G /dev/sdc
```

Now that we have two drives with identical partition setup we can create mdadm RAID arrays on those partitions. First array will be for OMV root drive on /dev/sd(b|c)2 partitions and other array on /dev/sd(b|c)3 partitions.

```shell
mdadm --create /dev/md0 --level=1 --raid-devices=2 --metadata=0.90 /dev/sdb2 /dev/sdc2
mdadm --create /dev/md1 --level=1 --raid-devices=2 --metadata=0.90 /dev/sdb3 /dev/sdc3
```

Create ext4 filesystem on /dev/md0:

```shell
mkfs.ext4 /dev/md0
```

With filesystem on /dev/md0 we can mount it on /mnt/root and sync OMV to /dev/md0.

```shell
mkdir /mnt/root
mount /dev/md0 /mnt/root/
rsync -avx / /mnt/root
```

One more thing, we must add new arrays to the /mnt/root/etc/mdadm/mdadm.conf, and change UUID of / mountpoint in chrooted OMV instalation. You can add new arrays to mdadm.conf with following command:

```shell
mdadm --detail --scan >> /mnt/root/etc/mdadm/mdadm.conf
```

Find UUID of /dev/md0 with blkid command and change UUID for / mount point in /mnt/root/etc/fstab.

Now you are ready to bind mount /dev /sys and /proc to our new root drive and chroot there:

```shell
mount --bind /dev /mnt/root/dev
mount --bind /sys /mnt/root/sys
mount --bind /proc /mnt/root/proc
chroot /mnt/root/
```

Congrats, you are now in your OMV installation on RAID array, there is only GRUB setup left to do and you are golden. Install GRUB on both new drives, update grub configuration and update initramfs and that is it.

```shell
grub-install --recheck /dev/sdb
grub-install --recheck /dev/sdc
grub-mkconfig -o /boot/grub/grub.cfg
update-initramfs -u
```

Exit chroot with exit and shutdown OMV. Remove USB device and boot into OMV on RAID1 array, once boot is done login as root via SSH and create filesystem on /dev/md1 with

```shell
mkfs.ext4 -m 1 -L DATA /dev/md1
```

And this is it, login to web interface, mount /dev/md1 and start using OMV installed on RAID1 array.

![FILESYSTEM: RAID1](/images/filesystem_2.png)

> NOTE: Now, if a drive in RAID1 array fails you can replace it following this guide: [https://www.howtoforge.com/replacing_hard_disks_in_a_raid1_array](https://www.howtoforge.com/replacing_hard_disks_in_a_raid1_array)

## 5. RAID installation using only hard drives

> Later, in the same guide, Igor Novikoff, tailors the previous procedure to avoid the installation on a USB drive first. He directly installs it on one of the hard drives meant to be part of the RAID group, creates the RAID specifying that a device is missing, and once the installation is complete, re-purposes the drive and adds it to the RAID group.

> NOTE: In his installation the uses 4 drives, so I will try to modify the flow to do it with 2 drives, so that it matches the previous example.

I did not need a pendrive. I used 2 hard drives:

-   I installed the system on the first disk, then created a smaller partition on the other disk.
-   Created a RAID volume.
-   Then I copied (rsync) the systems to the minor partition (raid) and functioned perfectly.
-   Finally, I deleted Disk 1 and included it in the raid scheme.

For those who need something similar the procedure follows.

Create disk scheme:

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

In case you had more than 2 disks on the RAID, you would need to copy this partition table to the other discs that will be part of the RAID (except for the current system disc).

> NOTE: In this example you need to replace /dev/sdx for the drives identifier

```shell
sgdisk -R=/dev/sdx /dev/sdb
sgdisk -G /dev/sdx

```

Create the raid with 2 discs (first disc is not in the RAID in this moment). At this moment, we create only 2 partition RAIDs: _swap area_ and _system area_.

```shell
mdadm –create /dev/md1 –level=1 –raid-devices=2 missing /dev/sdb2
# mdadm –create /dev/md1 –level=10 –raid-devices=x missing /dev/sdb2 /dev/sdx2 /dev/sdy2
mdadm –create /dev/md2 –level=1 –raid-devices=2 missing /dev/sdb3
# mdadm –create /dev/md2 –level=1 –raid-devices=x missing /dev/sdb3 /dev/sdx3 /dev/sdy3
```

Create file systems:

```shell
mkswap /dev/md1
mkfs.ext4 /dev/md2
mkfs.ext4 -m 1 -L DATA /dev/md3
```

Mount and copy system:

```shell
mkdir /mnt/root
mount /dev/md2 /mnt/root/
rsync -avx / /mnt/root
```

Update the raid config file:

```shell
mdadm –detail –scan >> /mnt/root/etc/mdadm/mdadm.conf
```

Find the UUID of md2 and update /mnt/root/etc/fstab:

```shell
blkid
```

Mount system:

```shell
mount –bind /dev /mnt/root/dev
mount –bind /sys /mnt/root/sys
mount –bind /proc /mnt/root/proc
chroot /mnt/root/
```

Update grub:

```shell
grub-install –recheck /dev/sdb
# grub-install –recheck /dev/sdx
grub-mkconfig -o /boot/grub/grub.cfg
update-initramfs -u
```

The system partition in the RAID group is now OK. Reboot OMV. When the system is up, copy the partition schema to the first disc (previous system disc):

```shell
sgdisk -R=/dev/sda /dev/sdb
sgdisk -G /dev/sda
```

Add this disc to RAID group:

```shell
mdadm /dev/md2 –add /dev/sda3
```

Create the RAID group for the DATA partition:

```shell
mdadm –create /dev/md3 –level=1 –raid-devices=2 /dev/sda4 /dev/sdb4
# mdadm –create /dev/md3 –level=10 –raid-devices=x /dev/sda4 /dev/sdb4 /dev/sdx4 /dev/sdy4
```

Ok, finaly, update the grub on first disc:

```shell
grub-install –recheck /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
update-initramfs -u
```

Finish!

> NOTE: **Be careful to not use “sfdisk” command to recover partition table from the healthy disk**, sfdisk does not support GPT. **Use “sgdisk” instead**. If new disk is /dev/sdb and healthy disk is /dev/sda, then do: `sgdisk -R=/dev/sdb /dev/sda` to replicate partition table from sda to sdb. Finally, use OMV GUI to recover RAID mirror.

## 6. Data migration

One of the great things of OMV is that it has a full fledged Debian, and that allows us to install any software and directly connect any drive in a readable format.

As I needed to migrate data  onto the NAS, I though that the fastest and most reliable way of doing it, was to directly connect the drive containing the original data on to the motherboard, rather than using an USB drive or the network.

We power down the NAS, connect the drive with the data, and power it back again.

We can easily detect the added drive, as it's the one that has not S.M.A.R.T. enabled:

![MIGRATION: Connected drive](/images/migration_1.png)

Then we move to mount the filesystem from _Storage \ File Systems_:

![MIGRATION: Mount drive](/images/migration_2.png)

Once mounted we see the data occupied in the filesystem:

![MIGRATION: Drive mounted](/images/migration_3.png)

Now we connect using ssh, and install Midnight Commander (`mc`) from the command line:

> NOTE: We could also use cp, rsync or even a plugin to copy files from an internet browser.

```shell
apt-get install mc
```

The mounted partitions can be easily accessed from /srv/ folder:

```shell
root@mediavault:~# cd /srv

root@mediavault:/srv# ls -alF
total 32
drwxr-xr-x  8 root  root    4096 May 17 08:20 ./
drwxrwxr-x 20 root  root    4096 May  6 03:31 ../
drwx------  8 vault users   4096 May  1 11:17 dev-disk-by-label-NAS_8TB/
drwxr-xr-x  4 root  root    4096 May  4 21:43 dev-disk-by-label-r1small/
drwxr-xr-x  8 root  root    4096 May 17 11:07 dev-disk-by-label-r6big/
```

I used `mc` to copy the files from the source drive containing the data, to the destination RAID protected share in the NAS:

![MIGRATION: Data migration](/images/migration_4.png)

Once finished,  we can check that both mounted filesystems contain the same amount of data:

![MIGRATION: Size comparison](/images/migration_5.png)

We go back to _Storage \ File Systems_ and unmount the source data filesystem. Then power down the NAS, and remove the drive.

Data migration done!

## 7. Converting RAID5 to RAID6

RAID6 is the new industry standard for storage arrays that are comprised of high capacity disks. An array that is built as RAID6 can support the failure of 2 devices without data loss. That fault tolerance level increases the probability of a successful rebuild in the event of a lost volume.

> RAID 6 requires a minimum of 4 disks, and I had to use one of the 4 big drives I bought to hold the older NAS data and transition to the new one. Thus, I started with a 3 drives RAID 5 protected group. Then I migrated the data from the source drive onto the RAID 5 and liberated the 4th drive. Last, I converted the RAID 5 group onto a RAID 6 protection, by using this drive and the following procedure.

Log in to your system using ssh with root account.

Review the current status of the array with `cat /proc/mdstat`:

```shell
root@mediavault:/srv# cat /proc/mdstat
Personalities : [raid1] [raid6] [raid5] [raid4] [linear] [multipath] [raid0] [raid10]
md127 : active raid5 sdb[1] sda[0] sdc[2]
      15627788288 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/3] [UUU]
      bitmap: 0/59 pages [0KB], 65536KB chunk

unused devices: <none>
```

> NOTE: In this, and the following outputs, some data was removed for clarity.

The array volume `md127` has 3 disks, all are active, it is RAID5, and no unused devices.

Review the detailed information of the array (`mdadm --detail /dev/mdx`):

```shell
root@mediavault:/srv# mdadm --detail /dev/md127
/dev/md127:
        Raid Level : raid5
        Array Size : 15627788288 (14903.82 GiB 16002.86 GB)
      Raid Devices : 3
     Total Devices : 3
             State : clean
    Number   Major   Minor   RaidDevice State
       0       8        0        0      active sync   /dev/sda
       1       8       16        1      active sync   /dev/sdb
       2       8       32        2      active sync   /dev/sdc

```

Each disk has 8TB capacity for 16TB total capacity in the RAID5 array. The array appears to be clean and fully functional.

Beforehand, we have added the hard disk `/dev/sdd`, and booted the system.

As this drive was already partitioned and contained data, the first thing we will do now is to create the exact same partitioning as on `/dev/sda` (or `/dev/sdb` or `/dev/sdc`). We can clone the RAID partition table to the new drive `/dev/sdd` with `sgdisk`:

```shell
sgdisk -R=/dev/sdd /dev/sda
sgdisk -G /dev/sdd
```

> You can run `fdisk -l` to check if all hard drives have the same partitioning now.

Add an additional disk to the RAID5 array (`mdadm --add /dev/mdx /dev/sdy`):

```shell
root@mediavault:/srv# mdadm --add /dev/md127 /dev/sdd
mdadm: added /dev/sdd
```

Here `mdadm` is called, told it will be adding a disk, the target is the `/dev/md127` array, and the disk is `/dev/sdd`.

Verify the disk is available to the array (`mdadm --detail /dev/mdx`):

```shell
root@mediavault:/srv# mdadm --detail /dev/md127
/dev/md127:
        Raid Level : raid5
        Array Size : 15627788288 (14903.82 GiB 16002.86 GB)
      Raid Devices : 3
     Total Devices : 4
             State : clean
    Number   Major   Minor   RaidDevice State
       0       8        0        0      active sync   /dev/sda
       1       8       16        1      active sync   /dev/sdb
       2       8       32        2      active sync   /dev/sdc

       3       8       48        -      spare   /dev/sdd
```

The disk `/dev/sdd` is available to the array `md127` and listed as a spare. The total number of devices is increased to 6 while the total RAID devices is still 5 and total capcity has remained the same.

Configure the RAID5 array to be a RAID6 (`mdadm --grow /dev/mdx --level=6 --raid-devices=4 --backup-file=/root/raid5backup`):

```shell
root@mediavault:/srv# mdadm --grow /dev/md127 --level=6 --raid-devices=4 --backup-file=/root/raid5backup
mdadm: level of /dev/md127 changed to raid6
```

This calls `mdadm`, tells it to grow the array, the target is `/dev/md127`, the new RAID level is 6, the total devices is 4, and to backup the array configuration to `/root/raid5backup`:

-   The grow command is used because the total number of data disks is increasing from 5 to 6.
-   4 as the number of devices is the minimum required for RAID6.

Check the status of the array  (`cat /proc/mdstat`):

```shell
root@mediavault:/srv# cat /proc/mdstat
Personalities : [raid1] [raid6] [raid5] [raid4] [linear] [multipath] [raid0] [raid10]
md127 : active raid6 sdd[3] sdb[1] sda[0] sdc[2]
      15627788288 blocks super 1.2 level 6, 512k chunk, algorithm 18 [4/3] [UUU_]
      [>....................]  reshape =  0.0% (1515520/7813894144) finish=18633.4min speed=6987K/sec
      bitmap: 0/59 pages [0KB], 65536KB chunk
unused devices: <none>

root@mediavault:/srv# mdadm --detail /dev/md127
/dev/md127:
        Raid Level : raid6
        Array Size : 15627788288 (14903.82 GiB 16002.86 GB)
      Raid Devices : 4
     Total Devices : 4
             State : clean, degraded, reshaping
    Active Devices : 3
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 1
    Reshape Status : 0% complete
        New Layout : left-symmetric
    Number   Major   Minor   RaidDevice State
       0       8        0        0      active sync   /dev/sda
       1       8       16        1      active sync   /dev/sdb
       2       8       32        2      active sync   /dev/sdc
       3       8       48        3      spare rebuilding   /dev/sdd

```

The array is rebuilding to a RAID6 level array. Notice that the number of RAID devices is now 4, RAID level is 6. It also gives an estimation of the completion time (_around **13 days** -18633 minutes-_).

The next step is to wait until the rebuild is complete.

Verify the status of the array.

```shell
root@mediavault:/srv# cat /proc/mdstat

root@mediavault:/srv# mdadm --detail /dev/md127
```

Array is clean and rebuilt.

### Power down during a RAID rebuild

If during this time you need to power down the NAS, there should be no issues if you do it with an ordered shutdown. You will need to check though that everything is OK, as unexpected issues might have arisen.

Remember, that the procedure to check the RAID status is with `cat /proc/mdstat`:

```shell
root@mediavault:~# cat /proc/mdstat
Personalities : [raid1] [linear] [multipath] [raid0] [raid6] [raid5] [raid4] [raid10]
md126 : active raid6 sdb[1] sdc[2] sda[0]
      15627788288 blocks super 1.2 level 6, 512k chunk, algorithm 18 [4/3] [UUU_]
      [>....................]  reshape =  3.4% (272659404/7813894144) finish=1868.0min speed=67280K/sec
      bitmap: 2/59 pages [8KB], 65536KB chunk

md127 : active raid1 sde[0] sdf[1]
      1465006464 blocks super 1.2 [2/2] [UU]
      bitmap: 0/11 pages [0KB], 65536KB chunk

unused devices: <none>
```

In my case I had configured 2 RAID groups, one RAID 1, and the RAID 6 we just configured. After reboot, the assigned identifiers had swapped, as I have no static `mdadm` configuration and it's dynamically detected at boot time.

Now the older `md127` is now called `md126`. I will review the detailed information of the RAID 6 group (`mdadm --detail /dev/mdx`):

```shell
root@mediavault:~# mdadm --detail /dev/md126
/dev/md126:
        Raid Level : raid6
        Array Size : 15627788288 (14903.82 GiB 16002.86 GB)
      Raid Devices : 4
     Total Devices : 3
             State : clean, degraded, reshaping
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 0
    Reshape Status : 4% complete
    Number   Major   Minor   RaidDevice State
       0       8        0        0      active sync   /dev/sda
       1       8       16        1      active sync   /dev/sdb
       2       8       32        2      active sync   /dev/sdc
       -       0        0        3      removed
```

Now the last added drive `/dev/sdd` appears as "removed".

We can also check the detail with the `lsblk` command, as it lists the drives and the block device associated with it:

```shell
root@mediavault:~# lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda       8:0    0  7.3T  0 disk  
└─md126   9:126  0 14.6T  0 raid6 /srv/dev-disk-by-label-r6big
sdb       8:16   0  7.3T  0 disk  
└─md126   9:126  0 14.6T  0 raid6 /srv/dev-disk-by-label-r6big
sdc       8:32   0  7.3T  0 disk  
└─md126   9:126  0 14.6T  0 raid6 /srv/dev-disk-by-label-r6big
sdd       8:48   0  7.3T  0 disk  
sde       8:64   0  1.4T  0 disk  
└─md127   9:127  0  1.4T  0 raid1 /srv/dev-disk-by-label-r1small
sdf       8:80   0  1.4T  0 disk  
└─md127   9:127  0  1.4T  0 raid1 /srv/dev-disk-by-label-r1small
sdg       8:96   1 29.9G  0 disk  
├─sdg1    8:97   1  512M  0 part  /boot/efi
├─sdg2    8:98   1    8G  0 part  /
└─sdg3    8:99   1  7.7G  0 part  
```

I will place the removed drive back into the RAID 6 group with the `mdadm --assemble` command (`mdadm --assemble /dev/mdx /dev/sdy`), used to assemble one or more raid arrays from pre-existing components:

```shell
root@mediavault:~# mdadm --assemble /dev/md126 /dev/sdd
mdadm: Cannot assemble mbr metadata on /dev/sdd
mdadm: /dev/sdd has no superblock - assembly aborted
```

As the RAID was powered down in the middle of the rebuild process, even though the rebuild process continued at the point where it had left for the 3 original drives, it seems that **we need to restart the procedure from the beginning for the remaining drive** :(.

Let's go for it. First we add the additional disk to the RAID6 array (`mdadm --add /dev/mdx /dev/sdy`) and verify the disk is available to the array (`mdadm --detail /dev/mdx`):

```shell
root@mediavault:~# mdadm --add /dev/md126 /dev/sdd
mdadm: added /dev/sdd

root@mediavault:~# mdadm --detail /dev/md126
/dev/md126:
        Raid Level : raid6
        Array Size : 15627788288 (14903.82 GiB 16002.86 GB)
      Raid Devices : 4
     Total Devices : 4
    Active Devices : 3
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 1
    Reshape Status : 5% complete
    Number   Major   Minor   RaidDevice State
       0       8        0        0      active sync   /dev/sda
       1       8       16        1      active sync   /dev/sdb
       2       8       32        2      active sync   /dev/sdc
       -       0        0        3      removed

       4       8       48        -      spare   /dev/sdd
```

We see that the drive is added as an spare, and it's not replacing the removed drive. The reason for that is, that right now, the RAID 6 group is still doing a resync/recovery and we need to wait for it to finish. I could verify that by trying to re-run a `mdadm --grow` with the same parameters over the RAID 6 group:

```shell
root@mediavault:~# mdadm --grow /dev/md126 --level=6 --raid-devices=4 --backup-file=/root/raid6backup
mdadm: /dev/md126 is performing resync/recovery and cannot be reshaped

root@mediavault:~# cat /proc/mdstat
Personalities : [raid1] [linear] [multipath] [raid0] [raid6] [raid5] [raid4] [raid10]
md126 : active raid6 sdd[4](S) sdb[1] sdc[2] sda[0]
      15627788288 blocks super 1.2 level 6, 512k chunk, algorithm 18 [4/3] [UUU_]
      [=>...................]  reshape =  6.5% (509481684/7813894144) finish=1899.1min speed=64102K/sec
      bitmap: 2/59 pages [8KB], 65536KB chunk
unused devices: <none>
```

The conclusion: **we need to restart the procedure from the beginning (13 days) and wait for 32 extra hours** _-1899 minutes-_ for the current reshape process to finish. _So hopefully I will have the RAID 6 group clean after waiting for a little bit more than 14 days... fingers crossed_.


## 8. References and Credits

This document is based on the following resources. I really thank the authors for sharing their knowledge and trouble:
-   OpenMediaVault installation procedure - [https://openmediavault.readthedocs.io/en/5.x/installation/index.html](https://openmediavault.readthedocs.io/en/5.x/installation/index.html).
-   Installing OpenMediaVault on RAID-1 array - [https://lazic.info/josip/post/installing-openmediavault-on-raid-device/](https://lazic.info/josip/post/installing-openmediavault-on-raid-device/)
-   Converting RAID5 to RAID6 in mdadm - [http://ewams.net/?date=2013/05/02&view=Converting_RAID5_to_RAID6_in_mdadm](http://ewams.net/?date=2013/05/02&view=Converting_RAID5_to_RAID6_in_mdadm)

## 9. Summary

-   _**Objetive:**_ Cover OpenMediaVault unsupported and not recommended installation procedures, and hopefully save you some time!
