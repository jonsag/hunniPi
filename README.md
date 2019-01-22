# hunniPi
A tutorial and perhaps a script to get a honey pot running on Raspberry Pi

Installing OS
=============================
Download Raspbian Stretch Lite from https://www.raspberrypi.org/downloads/raspbian/  
Choose the Light zip-file  

Cd to where your download is  
$ unzip 2018-11-13-raspbian-stretch-lite.zip  

Insert SD-card and find out drive letter  
$ dmesg  
For example /dev/mmcblk0 or /dev/sdb  

Unmount if mounted  
$ umount /dev/mmcblk0p1  

Write image to SD-card  
$ sudo dd bs=4M if=2018-11-13-raspbian-stretch-lite.img of=/dev/mmcblk0 conv=fsync status=progress 

Remove SD-card and insert it again to make new partitons visible     

Mount the first partition  
$ sudo mount /dev/mmcblk0p1 /mnt/tmp  

Write empty file to boot partition to enable ssh at boot  
$ sudo touch /mnt/tmp/ssh  

Remove SD-card and insert it a Rpi connected to your local network and boot it up 


Rpi configuration
-----------------------------
Connect to Rpi via ssh 
$ ssh 1<IP> -u pi
Login with user: pi and password:raspberry 

Update  
$ sudo apt-get update && sudo apt-get upgrade  

Configure  
$ sudo raspi-config   
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


Install requisites/good programs
-----------------------------
$ sudo apt-get install git python-virtualenv libssl-dev libffi-dev build-essential libpython-dev python2.7-minimal authbind  

I like emacs, so I install it now  
Also screen is very useful  
$ sudo apt-get install emacs screen  


Configure sshd
-----------------------------
Change listening port for sshd  
$ sudo emacs /etc/ssh/sshd_config  
Add/edit line so that:  
Port 22222  

Restart ssh service  
$ sudo service ssh restart  

Disconnect and connect at new port  
$ exit  
$ ssh 192.168.10.48 -l pi -p 22222  

Check that port look allright  
$ sudo netstat -nap | grep 22


Installing cowrie, ssh honeypot
-----------------------------
Following TAKHION's tutorial at https://null-byte.wonderhowto.com/how-to/use-cowrie-ssh-honeypot-catch-attackers-your-network-0181600/  

Create cowrie user  
$ sudo adduser --disabled-password cowrie  

Log in as new user  
$ sudo su - cowrie  

Clone cowrie repository  
$ git clone https://github.com/micheloosterhof/cowrie

Move into cowrie directory  
$ cd cowrie  

Create a virtual environment for the tool  
$ virtualenv cowrie-env  

Activate environment  
$ source cowrie-env/bin/activate  

Use pip to install additional requirements  
$ pip install --upgrade pip  
$ pip install --upgrade -r requirements.txt  

Copy config file  
Settings which are set in cowrie.cfg will be assigned priority  
$ cp ./etc/cowrie.cfg.dist ./etc/cowrie.cfg  

Now we edit the file  
$ emacs ./etc/cowrie.cfg  
Line 29: Change hostname = srv04 to some other name  
Line 430: Telnet, change enabled = false to true  
Save and exit editor  

Change routing  
From another shell:
$ sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222  
$ sudo iptables -t nat -A PREROUTING -p tcp --dport 23 -j REDIRECT --to-port 2223 
Start cowrie server  
$ bin/cowrie start  


Testing
-----------------------------
From another computer  

$ sudo nmap <IP> -p 22,2222,9022 -sS  

On honey pot:  
$ tail -f /home/cowrie/cowrie/var/log/cowrie/cowrie.log  

Play logs  
$ /home/cowrie/cowrie/bin/playlog /home/cowrie/cowrie/var/lib/cowrie/tty/...













