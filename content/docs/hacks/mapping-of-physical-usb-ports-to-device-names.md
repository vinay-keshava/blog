---
type: docs
title: "Mapping of Physical USB ports to Device Name on GNU/Linux"
next: /docs/
---

## How to Map Physical USB ports to Device Names on GNU/Linux
```bash{filename="lsusb command"}
$ lsusb
Bus 002 Device 004: ID 04b3:3025 IBM Corp. NetVista Full Width Keyboard
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 003 Device 123: ID 0951:1666 Kingston Technology DataTraveler 100 G3/G4/SE9 G2/50
Bus 003 Device 002: ID 2109:0815 VIA Labs, Inc. USB3.0 Hub             

$ lsusb -t
/:  Bus 06.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/2p, 10000M
    |__ Port 2: Dev 2, If 0, Class=Hub, Driver=hub/7p, 5000M
/:  Bus 05.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/3p, 480M
    |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/8p, 480M
/:  Bus 04.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/2p, 480M
/:  Bus 03.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/4p, 5000M
    |__ Port 4: Dev 2, If 0, Class=Hub, Driver=hub/4p, 5000M
        |__ Port 2: Dev 123, If 0, Class=Mass Storage, Driver=usb-storage, 5000M
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/3p, 480M
    |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/6p, 480M
        |__ Port 3: Dev 4, If 0, Class=Human Interface Device, Driver=usbhid, 1.5M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/4p, 480M
    |__ Port 4: Dev 2, If 0, Class=Hub, Driver=hub/4p, 480M

```

"lsusb" is a utility of "usbutils" in GNU/Linux to display information about the USB Buses and the USB's attached to the buses.The output of the command displays the VendorID:ProductID and to which bus it is attached,the "Kingston Technology DataTraveler" bearing the VendorID:ProductID (0951:1666) attached to bus 003, the lsusb command with option "-t" provides a tree like output in a hierarchial structure.

"lsblk" is a tree-like structure to identify devices and their partitions,and also displays device name (/dev/sd* if its a hard disk,/dev/nvme0n1 in case of SSD),size of the drive/partition,  whether it is a disk/partition and the device's mountpoints,here the /dev/sda* is the hard disk and sda1/2/5 are the partitions of the hard disk and /dev/sdb is the Kingston USB drive connected.

```bash
$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0 119.2G  0 disk 
├─sda1   8:1    0 118.3G  0 part /
├─sda2   8:2    0     1K  0 part 
└─sda5   8:5    0   976M  0 part [SWAP]
sdb      8:16   1  28.9G  0 disk 
sr0     11:0    1  1024M  0 rom  
```

```Suppose if there are multiple USB sticks attached to mutiple physical USB ports how to know which device names is mapped to which USB port ?```

If there are multiple USB's attached how do we get to know USB drive's device file and to which port that USB is connected ?

The answer is by /dev/disk/* 
Everything in GNU/Linux is either a file or a directory,so even the disk are represented as files in GNU/Linux.
```bash
$  ls -ltrh /dev/disk/
total 0
drwxr-xr-x 2 root root 100 Aug 28 12:28 by-partuuid
drwxr-xr-x 2 root root 100 Aug 29 09:49 by-uuid
drwxr-xr-x 2 root root 260 Aug 29 09:49 by-path
drwxr-xr-x 2 root root  60 Aug 29 09:49 by-label
drwxr-xr-x 2 root root 160 Aug 29 09:49 by-id
drwxr-xr-x 2 root root 260 Aug 29 09:49 by-diskseq
```

The USB disks can be identified by the Partition UUID, UUID of the disk, by path, by label of the disk and also by disk sequence.

### By UUID(Universally Unique Identifier)
```bash{filename="by uuid"}
$ cat /etc/fstab 
UUID=3341b336-0c26-4079-b3aa-faca8e2dd8b6 /               ext4    errors=remount-ro 0       1
UUID=53b53d97-b5e4-43fe-b560-2a01f119b6cf none            swap    sw              0       0
/dev/sr0        /media/cdrom0   udf,iso9660 user,noauto     0       0

$ ls -ltrh /dev/disk/by-uuid/
total 0
lrwxrwxrwx 1 root root 10 Aug 22 10:40 3341b336-0c26-4079-b3aa-faca8e2dd8b6 -> ../../sda1
lrwxrwxrwx 1 root root 10 Aug 22 10:40 53b53d97-b5e4-43fe-b560-2a01f119b6cf -> ../../sda5
lrwxrwxrwx 1 root root  9 Aug 29 09:49 2010-10-06-11-43-04-00 -> ../../sdb
```
The device /dev/disk/by-uuid/3341b336-0c26-4079-b3aa-faca8e2dd8b6 is simply a symbolic link to actual an device,the reason being this is device name may change depending whether disk is attached or not,whereas these links will point to the same drive,so henceforth safer to use.



### By-Label
```bash{filename="by-label"}
$ ls -ltrh /dev/disk/by-label/
total 0
lrwxrwxrwx 1 root root 9 Aug 29 09:49 VINAY-USB -> ../../sdb
```
Labels are easy, it avoids confusion in identifying disk, instead of remembering /dev/sda device file names.

### By-Path
```bash{filename="by-path"}
$ ls -ltrh /dev/disk/by-path/
total 0
lrwxrwxrwx 1 root root  9 Aug 22 10:40 pci-0000:00:1f.2-ata-2.0 -> ../../sda
lrwxrwxrwx 1 root root  9 Aug 22 10:40 pci-0000:00:1f.2-ata-2 -> ../../sda
lrwxrwxrwx 1 root root 10 Aug 22 10:40 pci-0000:00:1f.2-ata-2-part2 -> ../../sda2
lrwxrwxrwx 1 root root 10 Aug 22 10:40 pci-0000:00:1f.2-ata-2.0-part2 -> ../../sda2
lrwxrwxrwx 1 root root 10 Aug 22 10:40 pci-0000:00:1f.2-ata-2-part1 -> ../../sda1
lrwxrwxrwx 1 root root 10 Aug 22 10:40 pci-0000:00:1f.2-ata-2.0-part1 -> ../../sda1
lrwxrwxrwx 1 root root 10 Aug 22 10:40 pci-0000:00:1f.2-ata-2-part5 -> ../../sda5
lrwxrwxrwx 1 root root 10 Aug 22 10:40 pci-0000:00:1f.2-ata-2.0-part5 -> ../../sda5
lrwxrwxrwx 1 root root  9 Aug 22 10:40 pci-0000:00:1f.2-ata-3.0 -> ../../sr0
lrwxrwxrwx 1 root root  9 Aug 22 10:40 pci-0000:00:1f.2-ata-3 -> ../../sr0
lrwxrwxrwx 1 root root  9 Aug 29 09:49 pci-0000:00:14.0-usb-0:4.2:1.0-scsi-0:0:0:0 -> ../../sdb
```
Here /dev/sdb is the USB device attached and "pci-0000:00:14.0-usb-0:4.2:1.0-scsi-0:0:0:0" file which represents the USB device describes that the USB device is connected from PCI bus to SCSI adapter."by-path" is the pci path of the disk device. this device file name is created depending on the shortest physical path to the device.
"/dev/sda" the first SCSI drive on the first SCSI bus,/dev/sdb is the second SCSI drive and /dev/sdc is the third SCSI drive and so on.

```
:wq
```

