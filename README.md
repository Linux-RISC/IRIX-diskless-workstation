# IRIX diskless workstation using Reanimator
<br>
The purpose of this guide is to show how to boot an IRIX diskless workstation using Reanimator http://irix.mersisl.com/<br>
<br>
<h2>Requeriments</h2>
<ul>
  <li>Read the guide "Diskless Workstation Administration Guide" https://irix7.com/techpubs/007-0855-080.pdf and understand the complete process. This guide has too much literature and it is not very concrete, you can explain the same in 10 pages.</li>
  <li>1. sgi computer to generate diskless tree.</li>
  <li>2. sgi computer to work as <b>diskless workstation</b>, it could be the same than (1).</li>
  <li>3. Raspberry Pi+Reanimator to work as diskless workstation <b>server</b>. Reanimator on VirtualBox should work too.</li>
</ul>
<br>
I have tested three configurations:<br>
c1. RBPi working as bootp server and NAS. This is the easiest configuration to test a diskless workstation, keep in mind that the SD card will reduce its expected life time, due to the excess of write cycles.<br>
c2. To avoid this problem, you can connect an external hard disk to an USB port on RBPi and use it as storage instead the SD card.<br>
c3. RBPi working as bootp server and using a separated NAS as storage. I have used as NAS (Network-attached storage https://en.wikipedia.org/wiki/Network-attached_storage) a think client with Debian GNU/Linux and NFS, you don't need usign a professional solution.<br>
<br>
You can use any of the three configurations, the procedure is the same, you only need to modify the directory paths.<br>
<br>
<h2>Procedure</h2>
<h3>1. Creating a directory to store the diskless tree</h3>
c1. Create a directory on /home/irix/i named diskless<br>
c2. The path changes depending on your usb device and mounting point, if you use Reanimator's menus the path is /home/iris/i/sda1. Create there a directory named diskless.<br>
c3. The path changes depending on your drive device and mounting point, let's suppose the drive is mounted on /media/sda1. Create there a directory named diskless.<br>
<br>
- Modify directory permissions:<br>

```
% chmod 777 diskless

```
<h3>2. Preparing the IRIX server to generate the diskless tree</h3>
- verify it has installed the diskless server subsystem<br>

```
% versions nfs

```
If the subsystem is not installed, you can install it from the media "ONC3/NFS Version 3 for IRIX 6.2, 6.3, 6.4, and 6.5".<br>
Download this media on Reanimator(2. Download menu --> 7. Download IRIX 6.5.x) and proceed:<br>

```
% inst
1. open
Type irix@rbpi:i/IRIX/irix65x/nfs/dist
11. done
..
keep *
install nfs.sw.dskless_server
go
```
<h3>Run share_setup to create a share tree configuration file (share.dat).</h3>

```
% ./share_setup
----------
This program will help you create a share tree configuration file for a 
single IRIX version.  Create a separate share tree configuration file for
a 32-bit and a 64-bit version of the same IRIX release.

Retrieving system information...

Hostname is octane2
If you wish to use a different interface name for this host, enter it now
(Press return if octane2 is ok): 

Do you want client boot information stored on octane2 (y/n)? y --> SERVER_NIS="no"
----------
if "no":
Make sure octane2 is listed in the NIS server's /etc/updaters file.
----------
Enter the root directory of the diskless tree 
(e.g. /diskless) : /diskless

Enter the name of the share tree (e.g. IRIX65): 6.5.30

Share tree will be installed at /diskless/share/6.5.30

Enter the name of the share tree configuration file (no extension please): 6.5.30

Share tree configuration file will be written at /var/boot/6.5.30.dat
Press [Enter] to continue
----------
***CLIENT ARCHITECTURE INFORMATION***


*** PLEASE READ CAREFULLY ***
If you are using a 32 bit machine as your diskless server, you can only
install share trees for 32 bit platforms.  If you have a 64 bit server,
you can install either a 32 or a 64 bit share tree.

Do you want to install a share tree for all client platforms (y or n): y
Which version would you like:
        1. 32bit
        2. 64bit
Choose one (1 or 2): 2
Storing share tree configuration file at /var/boot/6.5.30.dat

/var/boot/6.5.30.dat
SERVER_NIS="yes" --> SERVER_NIS="no"

```
