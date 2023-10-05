---
type: docs
title: "Customize .qcow2 image"
---

Tired of Debian Installer ):

On Proxmox VE i had to go through the Debian Installer if i want to spin up a new VM,taking a lot of time and effort.

1. Download the .qcow2 image from cloud.debian.org

https://cloud.debian.org/images/cloud/bookworm/20230910-1499/debian-12-genericcloud-amd64-20230910-1499.qcow2 ,this is a generic cloud image which can be easily imported onto proxmox.

2. Reset the root password on the disk image

```$ virt-customize -a debian-10-genericcloud-amd64.qcow2 --root-password password:debian```

By default the .qcow2 doesn't have any root password,so the disk image has be customized using virt-customize to add root password. 

3. Increase size of .qcow2 disk image.
By default the size of Debian Generic Cloud is 2GB, using qemu-img we can resize the disk image. 

```
qemu-img resize image.qcow2 +SIZE 
```

4. Import the disk image on Proxmox VE.

Copy the image to ```/var/lib/vz/template/qemu/```.
Create a VM on Proxmox VE without any media (do not attach any physical media) and delete any existing disk on proxmox.

```
qm importdisk 114 /var/lib/vz/template/qemu/debian-12-genericcloud-amd64-20230910-1499.qcow2 amogha -format qcow2
```

Execute the above ```qm importdisk``` on the proxmox server where ```114```is the VM id where in your case will be different.
Refreshing the Proxmox GUI on the browser,attach the ```unused Hard Disk``` under ```Hardware```, also add a cloudInit drive and set ```IP address to dhcp``` to automatically assign IP address for both IPv4 and IPv6.

Under ```Options``` update the boot order and check whether the hard disk which was added to be checklisted and prioritize it to first.

```bash
Another alternative way is to use Preseed file at boot which automates 
debian installer,haven't tried that yet.

:wq #for now
```
