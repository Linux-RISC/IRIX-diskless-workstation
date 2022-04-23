# IRIX diskless workstation using Reanimator
<br>
<b>Purpose</b><br>
The purpose of this guide is to show how to boot an IRIX diskless workstation using Reanimator http://irix.mersisl.com/<br>
<br>
<b>Requeriments</b><br>
<ul>
  <li>Read the guide "Diskless Workstation Administration Guide" https://irix7.com/techpubs/007-0855-080.pdf and understand the complete process.</li>
  <li>sgi computer to generate client directories.</li>
  <li>sgi computer to work as <b>diskless workstation</b>, could be the same.</li>
  <li>Raspberry Pi+Reanimator to work as diskless workstation <b>server</b>. Reanimator on VirtualBox should work too.</li>
</ul>
<br>
I have tested three configurations:<br>
1. RBPi working as bootp server and NAS. This is the easiest configuration to test a diskless workstation, keep in mind that the SD card will reduce its expected life time, due to the excess of write cycles.
2. To avoid this problem, you can connect an external hard disk to an USB port on RBPi and use it as storage instead the SD card.<br>
3. RBPi working as bootp server and using a separated NAS as storage. This is the easiest configuration to test a diskless workstation, keep in mind that the SD card will reduce the expected life time, due to the excess of write cycles.<br>
<br>
I have used as NAS (Network-attached storage https://en.wikipedia.org/wiki/Network-attached_storage) a think client with Debian GNU/Linux, it's not necessary a professioal solution.<br>

