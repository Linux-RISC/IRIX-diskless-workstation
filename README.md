Last update: 2022/07/17<br>
# IRIX diskless workstation using Reanimator
<br>
The purpose of this guide is to show how to boot an IRIX diskless workstation using Reanimator http://irix.mersisl.com/<br>
<br>
Notes:<br>
- For a classic diskless workstation suppport according to https://irix7.com/techpubs/007-0855-080.pdf and with limited functionality, please visit <a href="classic-IRIX-diskless-workstation.md" target="_blank">classic-IRIX-diskless-workstation</a>.<br>
- This a <b>unofficial</b> guide, it ignores https://irix7.com/techpubs/007-0855-080.pdf but picks up some ideas from it.<br>
<br>
<h2>Requirements</h2>
<ul>
  <li>1. IRIX/Linux administration and network administration skills.</li>
  <li>2. A sgi computer with a functional IRIX installation on the <b>primary</b> hard disk. I will use an Octane2.</li>
  <li>3. A <b>secondary</b> hard disk with a functional IRIX installation. This disk will work as "rescue disk" and must have enough free space to save the entire primary hard disk.</li>
  <li>4. Raspberry Pi+Reanimator providing <b>bootp and NFS server</b> services, NFS will serve the IRIX files. Reanimator on VirtualBox should work too. I will use a Raspberry Pi.</li>
  <li>5. An optional but recommended NAS to improve performance.</li>
</ul>
<br>
I have tested three configurations:<br>
<b>C1</b>. RBPi/VirtualBox working as bootp server and NFS server. This is the easiest configuration to test a diskless workstation, if using RBPi keep in mind that the SD card will reduce its expected life time, due to the excess of write cycles.<br>
<b>C2</b>. (RBPi only) To avoid the SD problem and increase the throughput, you can connect an external hard disk to an USB port on RBPi and use it as storage instead the SD card.<br>
<b>C3</b>. RBPi/VirtualBox working as bootp server and using a separated NFS NAS as storage (NFS 4.x disabled). I have used as NAS (Network-attached storage https://en.wikipedia.org/wiki/Network-attached_storage) a think client with Debian GNU/Linux and NFS, you don't need to use a professional solution.<br>
<br>
I assume that you are using a file system that is compatible with GNU/Linux file permissions, such as ext4.<br>
<br>
You can use any of the three configurations, the procedure is the same, you only need to modify the directory paths.<br>
<br>
According to my experience, copying the files with rsync over network is much slower, it's faster to backup an IRIX disk with tar and restore it on the destination machine.<br>
<br>
<h2>Procedure</h2>
<h3>1. Boot from secondary hard disk</h3>
Example: secondary hard disk is SCSI ID 2.<br>
<br>
Run in Command Monitor:

```
>>setenv SystemPartition dksc(0,2,8)
>>setenv OSLoadPartition dksc(0,2,0)
```
Return to main menu and boot IRIX.<br>
<br>
<h3>2. Mount primary disk</h3>
Assuptions:<br>
- primary hard disk is SCSI ID 1.<br>
- both disks are partitioned as rootdrive.<br> 
<br>
Run as root (/diskless must exist):

```
# mount /dev/dsk/dks0d1s0 /diskless
```

<h3>3. Copying and restoring diskless.tar on Reanimator</h3>
You can trasfer the file to the NAS using NFS, scp, Samba, ... some examples are provided.<br>
<br>
Example using configuration C1 and scp to copy Octane2-->RBPi:

```
# tar cvf /diskless.tar /diskless
# scp /diskless.tar pi@192.168.9.100:/home/irix/i
```

Restoring on Reanimator:
```
pi@rbpi:/home/irix/i $ sudo rm -r diskless
pi@rbpi:/home/irix/i $ sudo tar xvf diskless.tar
pi@rbpi:/home/irix/i $ sudo chmod 777 diskless
```

Example using configuration C2 and NFS to copy Octane2-->RBPi, the usb drive <b>must</b> be mounted:
```
# tar cvf /diskless.tar /diskless
# mount 192.168.9.100:/home/irix/i/sda1 /mnt
# cp /diskless.tar /mnt
# umount /mnt
```
Restoring on Reanimator:

```
pi@rbpi:/home/irix/i/sda1 $ sudo rm -r diskless
pi@rbpi:/home/irix/i/sda1 $ sudo tar xvf diskless.tar
pi@rbpi:/home/irix/i/sda1 $ sudo chmod 777 diskless
```

Example using configuration C3, creating the file diskless.tar on a NAS shared resource. This is the <b>fastest method</b> according to my experience:

```
# mount NAS_IP:/path /mnt
# tar cvf /mnt/diskless.tar /diskless
# umount /mnt
```

In my case, I used sgug's tar for compression, but Nekoware's tar or sgi Freeware's tar should work too:
```
# mount NAS_IP:/path /mnt
# /usr/sgug/bin/tar czvf /mnt/diskless.tar /diskless
# umount /mnt
```

Of course, you can use tar on the secondary hard disk and trasfer the file to the NAS:

```
# tar cvf /diskless.tar /diskless
# mount NAS_IP:/path /mnt
# cp /diskless.tar /mnt
# umount /mnt
```

Restore on the NAS using local tools, in case of a GNU/Linux box:

```
$ sudo rm -r diskless
$ sudo tar xvf diskless.tar
$ sudo chmod 777 diskless
$ sudo mv diskless/* .
$ sudo rmdir diskless
```
Note that the "diskless" share must be the root directory, after unpacking diskless.tgz you should move all the contents to the parent directory and delete the empty "diskless" directory.

<h3>4. Configuring Reanimator and diskless workstation</h3>
Reanimator provides <b>preconfigured</b> /etc/bootparams and /etc/exports. Edit them using Reanimator's menus according to your configuration.<br>
This is an example for using Octane2 as diskless workstation:<br>
<br>
<b>C1 configuration</b><br>
/etc/bootparams on Reanimator:

```
# C1. RBPi/VirtualBox(change IP to 192.168.9.101) working as bootp server and NFS server
IRIS2   root=192.168.9.100:/home/irix/i/diskless
```
/etc/exports on Reanimator:

```
/home/irix/i/diskless                   *(rw,no_root_squash,no_subtree_check)
```
Command Monitor configuration on diskless workstation:<br>

```
>>setenv verbose on
>>setenv diskless 1
>>setenv netaddr 192.168.9.2
>>setenv OSLoader /unix
>>setenv SystemPartition bootp():diskless
>>setenv OSLoadPartition bootp():diskless
```

<b>C2 configuration</b><br>
/etc/bootparams on Reanimator:

```
# C2. (RBPi only) bootp+NFS+external hard disk connected to an USB port on RBPi
IRIS2   root=192.168.9.100:/home/irix/i/sda1/diskless
```
/etc/exports on Reanimator:

```
/home/irix/i/sda1/diskless                   *(rw,no_root_squash,no_subtree_check)
```
Command Monitor configuration on diskless workstation:<br>

```
>>setenv verbose on
>>setenv diskless 1
>>setenv netaddr 192.168.9.2
>>setenv OSLoader /unix
>>setenv SystemPartition bootp():sda1/diskless
>>setenv OSLoadPartition bootp():sda1/diskless
```

<b>C3 configuration</b><br>
/etc/bootparams on Reanimator:

```
# C3. RBPi/VirtualBox working as bootp server and using a separated NFS NAS as storage
# This path must be a NFS share defined on the NAS configuration
IRIS2   root=NAS_IP:/path/diskless
```
/etc/exports on Reanimator:

```
/home/irix/i/diskless                   *(rw,no_root_squash,no_subtree_check)
```
Command Monitor configuration on diskless workstation:<br>

```
>>setenv verbose on
>>setenv diskless 1
>>setenv netaddr 192.168.9.2
>>setenv OSLoader /unix
>>setenv SystemPartition bootp():diskless
>>setenv OSLoadPartition bootp():diskless
```
Don't <b>forget</b> to copy the <b>unix</b> file from "diskless" directory on the NAS to /home/irix/i/diskless on Reanimator. If you have several kernels, use a directory structure on "diskless" directory like:<br>
<ul>
  <li>6.5.22/unix</li>
  <li>6.5.30/unix</li>
  <li>...</li>
</ul>
And modify the configuration on Command Monitor, depending on the kernel used:

```
>>setenv SystemPartition bootp():diskless/6.5.22
>>setenv OSLoadPartition bootp():diskless/6.5.22
```

If you are an experienced user, you can copy the kernels to /home/irix/i/diskless on Reanimator using different names (for example 6.5.22 and 6.5.30) and modify OSLoader variable:<br>

```
>>setenv verbose on
>>setenv diskless 1
>>setenv netaddr 192.168.9.2
>>setenv OSLoader /6.5.22
>>setenv SystemPartition bootp():diskless
>>setenv OSLoadPartition bootp():diskless
```
<br>
The diskless workstation will boot without virtual memory, "Virtual Swap Space" can be added using Swap Manager.<br>
<br>
To set default configuration, run in Command Monitor:<br>

```
>>resetenv
```
<br>
<h3>5. Real examples</h3>
<b>Configuration for only one diskless client</b><br>
/etc/bootparams on Reanimator:<br>

```
IRIS2   root=192.168.9.13:/media/sdb1/NAS/diskless
```
/etc/exports on NAS: note that it is not necessary to share /media/sdb1/NAS/diskless directory via NFS, just share /media/sdb1/NAS<br>

```
/media/sdb1/NAS     192.168.9.*(rw,no_root_squash,no_subtree_check)
```
IRIX / on /media/sdb1/NAS/diskless
<br>
Octane2:<br>

```
>>setenv verbose on
>>setenv diskless 1
>>setenv netaddr 192.168.9.2
>>setenv OSLoader /unix
>>setenv SystemPartition bootp():diskless
>>setenv OSLoadPartition bootp():diskless
```
<br>
<h3>6. Possible use cases:</h3>
1. <b>Virtual</b> rescue disk:<br>
<ul>
  <li>in case of IRIX boot fail or disk failure, instead using a physical disk</li>
  <li>Automated network backup</li>
  <li>...</li>
</ul>2. Multiple IRIX versions on diskless directory to boot different IRIX versions for a specific machine/software, for example:<br>
Content of directory "diskless" on NAS:
<ul>
  <li>6.5.22</li>
  <li>6.5.30</li>
  <li>Octane2</li>
  <li>Indy_5.3</li>
  <li>Indy_5.3_LightWave3</li>
  <li>...</li>
</ul>
Select the IRIX version to boot modifying Command Monitor variables:

```
>>setenv verbose on
>>setenv diskless 1
>>setenv netaddr 192.168.9.2
>>setenv OSLoader /unix
>>setenv SystemPartition bootp():diskless/Indy_5.3_LightWave3
>>setenv OSLoadPartition bootp():diskless/Indy_5.3_LightWave3
```

3. Test software on multiple machines without reinstalling<br>
4. Portable farm of sgi machines<br>

<h3>7. Possible improvements:</h3>
- Online repository containing a collection of .tgz files with different versions, machines and software<br>
- Use disk images (how, using dd?) to avoid thousands of files in a directory and mount those .img files on the NFS server<br>
- Is there a way to copy the primary disk without using a secondary disk? Yes!, using a preconfigured IRIX to boot as diskless workstation<br>
- Boot from network and use a local disk for swap, data or both<br>
- Share IRIX directories between machines (using Unix soft links) with the same IRIX version to save disk space, the concept is the same than "shared trees" of sgi diskless workstation manual
