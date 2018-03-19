# Building bare metal images with Packer, VirtualBox and qemu-img  

This repository should demonstrate that it is possible to provision OS with Packer, then create raw disk image that can be booted directly on bare metal.  

I am using random [Alpine Packer builder](https://github.com/ketzacoatl/packer-alpine/tree/master/00-iso-install) from the GitHub (thanks!) because of Alpine small size.  

Firstly, run `packer build packer_template.json` to create VirtualBox `OVA` archive which contains disk and configuration files.  

Then decompress `OVA` tar archive and convert `VMDK` to `RAW`:  
```
$ cd output-virtualbox-iso
$ tar xvf alpine-clean-3.6.1.ova
$ qemu-img convert -f vmdk alpine-clean-3.6.1-disk001.vmdk -O raw alpine-clean-3.6.1.raw
```

Check that `RAW` file is actual disk file:   
```
$ fdisk -l alpine-clean-3.6.1.raw
Disk alpine-clean-3.6.1.raw: 2 GiB, 2147483648 bytes, 4194304 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x1de52f32

Device                  Boot   Start     End Sectors  Size Id Type
alpine-clean-3.6.1.raw1 *       2048  206847  204800  100M 83 Linux
alpine-clean-3.6.1.raw2       206848 1255423 1048576  512M 82 Linux swap / Solaris
alpine-clean-3.6.1.raw3      1255424 4194303 2938880  1.4G 83 Linux
```

Partition offset is `1255424 sectors` * `512 bytes` per sector, so if you want to mount it, you can use `mount` command with the appropriate offset:   
```
$ mkdir /tmp/loop
$ sudo mount -o ro,loop,offset=642777088 alpine-clean-3.6.1.raw /tmp/loop
$ mount | grep alpine-clean-3.6.1.raw
alpine-clean-3.6.1.raw on /tmp/loop type ext4 (ro,relatime,data=ordered)
```

If you want to create bootable USB, make sure that your USB is not mounted. If mounted, use `umount` command.    
```
$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
...
sda      8:0    0   477G  0 disk
├─sda1   8:1    0   512M  0 part /boot/efi
├─sda2   8:2    0 460.6G  0 part /
└─sda3   8:3    0  15.9G  0 part
sdb      8:16   1   7.3G  0 disk /media/user/ABCD-EFGH  <--- our mounted USB stick

$ sudo umount /media/user/ABCD-EFGH  
```
Finally, write raw disk to the USB memory stick
```
$ sudo dd if=./alpine-clean-3.6.1.raw of=/dev/sdb bs=4k

131072+0 records in
131072+0 records out
536870912 bytes (537 MB, 512 MiB) copied, 139.334 s, 3.9 MB/s
```

TODO:  
1. Shrink disk image using zerofree / other tools
2. ?
3. Profit


Sources:  
<https://major.io/2010/12/14/mounting-a-raw-partition-file-made-with-dd-or-dd_rescue-in-linux/>  
<https://blog.filippo.io/converting-a-partition-image-to-a-bootable-disk-image/>  
<https://wiki.hackzine.org/sysadmin/kvm-import-ova.html>
