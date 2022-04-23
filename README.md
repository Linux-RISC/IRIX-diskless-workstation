# IRIX diskless workstation using Reanimator
<br>
<b>Purpose</b><br>
The purpose of this guide is to show how to boot an IRIX diskless workstation using Reanimator http://irix.mersisl.com/<br>
<br>
<b>Requeriments</b><br>
<ul>
  <li>Read the guide "Diskless Workstation Administration Guide" https://irix7.com/techpubs/007-0855-080.pdf and understand the complete process. This guide has too much literature and it is not very concrete, you can explain the same in 10 pages.</li>
  <li>1. sgi computer to generate diskless tree.</li>
  <li>2. sgi computer to work as <b>diskless workstation</b>, it could be the same than 1.</li>
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
<b>Procedure</b><br>
1. Creating a directory to store the diskless tree<br>
c1. Create a directory on /home/irix/i named diskless<br>
c2. The path changes depending on your usb device and mounting point, if you use Reanimator's menus the path is /home/iris/i/sda1. Create there a directony named diskless.
c3. The path changes depending on your drive device and mounting point, let's suppose the drive is mounted on /media/sda1. Create there a directony named diskless.<br>
<br>
- Modify directory permissions: chmod 777 diskless<br>

