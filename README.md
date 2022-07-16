Last update: 2022/07/16<br>
# IRIX diskless workstation using Reanimator (work in progress)
<br>
The purpose of this guide is to show how to boot an IRIX diskless workstation using Reanimator http://irix.mersisl.com/<br>
<br>
Note: for a classic diskless workstation suppport according to https://irix7.com/techpubs/007-0855-080.pdf and with limited functionality, please visit <a href="classic-IRIX-diskless-workstation.md" target="_blank">classic-IRIX-diskless-workstation</a>.<br>
<br>
<h2>Requirements</h2>
<ul>
  <li>1. IRIX/Linux administration and network administration skills.</li>
  <li>2. A sgi computer with a functional IRIX installation on the <b>primary</b> hard disk. I will use an Octane2.</li>
  <li>3. A <b>secondary</b> hard disk with a functional IRIX installation. This disk will work as "rescue disk" and must have enough free space to tar the entire primary hard disk.</li>
  <li>4. Raspberry Pi+Reanimator to work as <b>diskless server</b>, providing <b>bootp and NFS server</b> services, NFS will serve the IRIX files. Reanimator on VirtualBox should work too. I will use a Raspberry Pi.</li>
  <li>5. An optional but recommended NAS to improve performance.</li>
</ul>
<br>
I have tested three configurations:<br>
C1. RBPi/VirtualBox working as bootp server and NFS server. This is the easiest configuration to test a diskless workstation, if using RBPi keep in mind that the SD card will reduce its expected life time, due to the excess of write cycles.<br>
C2. (RBPi only) To avoid the SD problem and increase the throughput, you can connect an external hard disk to an USB port on RBPi and use it as storage instead the SD card.<br>
C3. RBPi/VirtualBox working as bootp server and using a separated NFS NAS as storage (NFS 4.x disabled). I have used as NAS (Network-attached storage https://en.wikipedia.org/wiki/Network-attached_storage) a think client with Debian GNU/Linux and NFS, you don't need to use a professional solution.<br>
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
Run in Command Monitor:<br>

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
Run as root (/diskless must exist):<br>

```
# mount /dev/dsk/dks0d1s0 /diskless
```

<h3>3. Copying and restoring diskless.tar on Reanimator</h3>
<br>
Example using configuration C1 and scp to copy Octane2-->RBPi:

```
# tar cvf /diskless.tar /diskless
# scp /diskless.tar pi@192.168.9.100:/home/irix/i
```
Example using configuration C2 and NFS to copy Octane2-->RBPi, the usb drive <b>must</b> be mounted:

```
# tar cvf /diskless.tar /diskless
# mount 192.168.9.100:/home/irix/i/sda1 /mnt
# cp /diskless.tar /mnt
# umount /mnt
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
