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
Line 168: Comment out auth_class = UserDB  
Line 175-176: Remove comment from both lies, enabling random access  
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

A way to view logs:  
$ grep 'succeede\|CMD\|TTY' /home/cowrie/cowrie/var/log/cowrie/cowrie.log


Files of interest
-----------------------------
Configuration file  
/home/cowrie/cowrie/etc/cowrie.cfg  

Fake filesystem  
/home/cowrie/cowrie/share/cowrie/fs.pickle  

Files transferred from the attacker to the honeypot are stored here  
/home/cowrie/cowrie/var/lib/cowrie/downloads/  

User database location, list of allowed user:pass  
/home/cowrie/cowrie/etc/userdb.txt  


Service file for cowrie
-----------------------------
Example file at:  
/home/cowrie/cowrie/docs/systemd/etc/systemd/system/cowrie.service  

Copy it to:  
/etc/systemd/system/cowrie.service  


Installing splunk log analyzer on server
-----------------------------
On the splunk server:  
# wget -O splunk-7.2.3-06d57c595b80-Linux-x86_64.tgz 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=linux&version=7.2.3&product=splunk&filename=splunk-7.2.3-06d57c595b80-Linux-x86_64.tgz&wget=true'  
# tar xvzf splunk-7.2.3-06d57c595b80-Linux-x86_64.tgz -C /opt  

Create links, etc  
# ln -s /opt/splunk/etc /etc/splunk  
# ln -s /opt/splunk/var/log/splunk /var/log/splunk  
# ln -s /opt/splunk/var/run/splunk /var/run/splunk  
# ln -s /opt/splunk/var/spool/splunk /var/spool/splunk  
# ln -s /opt/splunk/bin/splunk /usr/bin/splunk  

Create startscript  
# curl https://data.gpo.zugaina.org/argent-main/net-analyzer/splunk/files/splunk.initd --output /etc/init.d/splunk  
or
# emacs /etc/init.d/splunk  
	#!/sbin/runscript
	# Copyright 1999-2012 Gentoo Foundation
	# Distributed under the terms of the GNU General Public License v2
	
	depend() {
		after logger
	}
	
	start() {
		ebegin "Starting Splunk"
		/opt/splunk/bin/splunk start
		eend $?
	}
	
	stop() {
		ebegin "Stopping Splunk"
		/opt/splunk/bin/splunk stop
		eend $?
	}

Make executable  
# chmod 755 /etc/init.d/splunk  

Add as service  
# rc-update add splunk    

echo "LDPATH=/opt/splunk/lib" > "${T}/99splunk"
doenvd "${T}/99splunk"

Configure splunk  
$ cd /opt/splunk/bin  
$ ./splunk start  

Edit inputs  
# emacs /opt/splunk/etc/system/local/inputs.conf  
Add line:  
[tcp:9997]  

Scroll down and gree to license  
Enter admin user name: splunkadmin  
Enter passsword twice  

Go to the link at end of text  
http://<IP>:8080  
Log in with user/pass from installation  
Click Settings->Forwarding and receiving->Configure receiving + Add new  
Typ in port '9997'  and click 'Save'


Install Splunk Universal Forwarder on hunniPi
-----------------------------
$ wget -O splunkforwarder-7.2.3-06d57c595b80-Linux-arm.tgz 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=ARM&platform=linux&version=7.2.3&product=universalforwarder&filename=splunkforwarder-7.2.3-06d57c595b80-Linux-arm.tgz&wget=true'  

Install  
$ sudo tar xvzf splunkforwarder-7.2.3-06d57c595b80-Linux-arm.tgz -C /opt  

Enable start at boot  
$ sudo /opt/splunkforwarder/bin/splunk enable boot-start  

Scroll down and accept license  
Add user name: splunkadmin  
State password twice  

Configure the indexer that the forwarder will send its data to  
$ sudo /opt/splunkforwarder/bin/splunk add forward-server <IP>:9997 -auth splunkadmin:<PASSWORD>  

Add data to consume  
$ sudo /opt/splunkforwarder/bin/splunk add monitor /home/cowrie/cowrie/var/log/cowrie/cowrie.log -sourcetype linux_logs -index cowrie  

Start the forwarder  
$ sudo /opt/splunkforwarder/bin/splunk start  















