# raspbian OS on Raspberry Pi  

Installing OS
=============================
Download Raspbian Stretch Lite from https://www.raspberrypi.org/downloads/raspbian/  
Choose the Light zip-file  

Cd to where your download is  
>$ unzip 2018-11-13-raspbian-stretch-lite.zip  

Insert SD-card and find out drive letter  
>$ dmesg  
For example /dev/mmcblk0 or /dev/sdb  

Unmount if mounted  
>$ umount /dev/mmcblk0p1  

Write image to SD-card  
>$ sudo dd bs=4M if=2018-11-13-raspbian-stretch-lite.img of=/dev/mmcblk0 conv=fsync status=progress 

Remove SD-card and insert it again to make new partitons visible     

Mount the first partition  
>$ sudo mount /dev/mmcblk0p1 /mnt/tmp  

Write empty file to boot partition to enable ssh at boot  
>$ sudo touch /mnt/tmp/ssh  

Remove SD-card and insert it a Rpi connected to your local network and boot it up 


Rpi configuration
=============================
Connect to Rpi via ssh 
>$ ssh \<IP\> -u pi
Login with user: pi and password:raspberry 

Update  
>$ sudo apt-get update && sudo apt-get upgrade  

Configure  
>$ sudo raspi-config   

1		Change password  
2 N1	Change hostname  
3 B1	Set to boot into console  
4 T1	Set locales  
4 T2	Set time zone  
4 T3	Choose keyboard layout    
4 T4	Set wifi country  
7 A1	Expand file system to use whole SD-card  
7 A3	Set memory split to 16  

Reboot to set new options 


Install good programs
=============================
I like emacs, so I install it now  
Also screen is very useful  
>$ sudo apt-get install emacs screen  






