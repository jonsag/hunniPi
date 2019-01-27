# hunniPi

A tutorial and perhaps a script to get a honey pot running on Raspberry Pi

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
-----------------------------
Connect to Rpi via ssh 
>$ ssh 1<IP> -u pi
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


Install requisites/good programs
-----------------------------
>$ sudo apt-get install git python-virtualenv libssl-dev libffi-dev build-essential libpython-dev python2.7-minimal authbind python-pip  

I like emacs, so I install it now  
Also screen is very useful  
>$ sudo apt-get install emacs screen  


Configure sshd
-----------------------------
Change listening port for sshd  
>$ sudo emacs /etc/ssh/sshd_config  

Add/edit line so that:
  
	Port 22222  

Restart ssh service  
>$ sudo service ssh restart  

Disconnect and connect at new port  
>$ exit  
>$ ssh 192.168.10.48 -l pi -p 22222  

Check that port look allright  
>$ sudo netstat -nap | grep 22


Installing cowrie, ssh honeypot
-----------------------------
Following TAKHION's tutorial at https://null-byte.wonderhowto.com/how-to/use-cowrie-ssh-honeypot-catch-attackers-your-network-0181600/  

Create cowrie user  
>$ sudo adduser --disabled-password cowrie  

Log in as new user  
>$ sudo su - cowrie  

Clone cowrie repository  
>$ git clone https://github.com/micheloosterhof/cowrie

Move into cowrie directory  
>$ cd cowrie  

Create a virtual environment for the tool  
>$ virtualenv cowrie-env  

Activate environment  
>$ source cowrie-env/bin/activate  

Use pip to install additional requirements  
>$ pip install --upgrade pip  
>$ pip install --upgrade -r requirements.txt  

Copy config file  
Settings which are set in cowrie.cfg will be assigned priority  
>$ cp ./etc/cowrie.cfg.dist ./etc/cowrie.cfg  

Now we edit the file  
>$ emacs ./etc/cowrie.cfg  

Line 29: Change hostname = srv04 to some other name  
Line 168: Comment out auth_class = UserDB  
Line 175-176: Remove comment from both lies, enabling random access  
Line 430: Telnet, change enabled = false to true  

Start cowrie server  
>$ bin/cowrie start  

To start cowrie as normal user  
>$ sudo su cowrie -c '/home/cowrie/cowrie/bin/cowrie start'  

Exit su cowrie  
>$ exit

Create iptables routing  
>$ sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222  
>$ sudo iptables -t nat -A PREROUTING -p tcp --dport 23 -j REDIRECT --to-port 2223

Save rules  
>$ sudo iptables-save > iptables.rules  
>$ sudo mv iptables.rules /home/cowrie/  

Add to rc.local to autostart  
>$ sudo emacs /etc/rc.local  

Add before line 'exit 0':  

	sudo su cowrie -c '/home/cowrie/cowrie/bin/cowrie start'  
	iptables-restore -n < /home/cowrie/iptables.rules  

Testing
-----------------------------
From another computer  
>$ sudo nmap <IP> -p 22,23,2222,2223,9022 -sS  

On honey pot:  
>$ tail -f /home/cowrie/cowrie/var/log/cowrie/cowrie.log  

Play logs  
>$ /home/cowrie/cowrie/bin/playlog /home/cowrie/cowrie/var/lib/cowrie/tty/...

A way to view logs:  
>$ grep 'succeed\|CMD\|TTY' /home/cowrie/cowrie/var/log/cowrie/cowrie.log


Files of interest
-----------------------------
Configuration file  
/home/cowrie/cowrie/etc/cowrie.cfg  

Log file  
/home/cowrie/cowrie/var/log/cowrie/cowrie.log  

Fake filesystem  
/home/cowrie/cowrie/share/cowrie/fs.pickle  

Files transferred from the attacker to the honeypot are stored here  
/home/cowrie/cowrie/var/lib/cowrie/downloads/  

User database location, list of allowed user:pass  
/home/cowrie/cowrie/etc/userdb.txt  


Service file for cowrie - presently not working
-----------------------------
Add cowrie service  
>$ sudo cp /home/cowrie/cowrie/docs/systemd/etc/systemd/system/cowrie.service /etc/systemd/system/  

Start service  
>$ sudo service cowrie start  

Add to autostart  
>$ sudo systemctl enable cowrie

  
A log viewer - cowrie-logviewer
=============================
>$ sudo su - cowrie  
>$ cd ~/../cowrie  
>$ git clone https://github.com/mindphluxnet/cowrie-logviewer  
>$ cd cowrie-logviewer  

Make script executable  
>$ chmod +x cowrie-logviewer.py  

Install prerequisites  
>$ pip install -r requirements.txt  

MaxMind GeoLite 2 Country database setup  
>$ mkdir maxmind  
>$ cd maxmind  
>$ wget http://geolite.maxmind.com/download/geoip/database/GeoLite2-Country.mmdb.gz  
>$ gunzip GeoLite2-Country.mmdb.gz  
>$ rm GeoLite2-Country.mmdb.gz  

Configuration  
>$ emacs cowrie-logviewer.py  

Line 20: log_path = '/home/cowrie/cowrie/var/log/cowrie/'  
Line 21: dl_path = '/home/cowrie/cowrie/var/lib/cowrie/downloads/'  
Line 22: maxmind_path = '/home/cowrie/cowrie-logviewer/maxmind/GeoLite2-Country.mmdb'  


Another log viewer - kippo-graph
=============================
Enable output to mysql
-----------------------------
Install mysql  
>$ sudo apt-get install python-mysqldb mysql-server libmariadbclient-de  

Setup database  
>$ sudo mysql -u root -p  

Use same password as for user root in shell  
  
In mysql:  
>mysql> CREATE DATABASE cowriedb;  
>mysql> GRANT ALL ON cowriedb.* TO 'cowrieuser'@'localhost' IDENTIFIED BY '<cowrie db pass>';  
>mysql> exit  

>$ cd /home/cowrie/cowrie    
>$ sudo mysql -u cowrieuser -p cowriedb    

>mysql> source ./docs/sql/mysql.sql;  
>mysql> exit  

Halt cowrie  
>$ sudo /home/cowrie/cowrie/bin/cowrie stop  

Edit config file  
>$ sudo su - cowrie  
>$ emacs /home/cowrie/cowrie/etc/cowrie.cfg  

Uncomment lines 569-576  
Line 570: enabled = true  
Line 572: database = cowriedb  
Line 573: username = cowrieuser  
Line 574: password = <cowrie db pass>  

>$ pip install mysql mysql-python  

Now restart cowrie with logging to mysql  
>$ sudo su cowrie -c '/home/cowrie/cowrie/bin/cowrie start'  
>$ exit  

  
Install kippo-graph
-----------------------------
Install requirements  
>$ apt-get install -y libapache2-mod-php php-mysql php-gd php-curl php-bcmath php-gmp php-xml  

Start apache  
>$ sudo /etc/init.d/apache2 restart  

Clone repository  
>$ git clone https://github.com/ikoniaris/kippo-graph.git  

Setup   
>$ sudo cp -R kippo-graph /var/www/html/    
>$ cd /var/www/html/kippo-graph  
>$ sudo chmod 777 generated-graphs  
>$ sudo cp config.php.dist config.php  

Configure  
>$ sudo emacs config.php
  
Line 23: define('DB_USER', 'cowrieuser');  
Line 24: define('DB_PASS', '<cowrie db pass>');  
Line 25: define('DB_NAME', 'cowriedb');  

Set owner  
>$ sudo chown www-data:www-data -R /var/www  

Now visit kippo-graph at:  
http://<IP>/kippo-graph  


Yet another log viewer - ELK/Elasticsearch, logstash, kibana 
=============================
Prerequisite - Java 8
-----------------------------
>$ sudo apt-get install dirmngr  
>$ echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" | sudo tee /etc/apt/sources.list.d/webupd8team-java.list  
>$ echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" | sudo tee -a /etc/apt/sources.list.d/webupd8team-java.list  
>$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886  
>$ sudo apt-get update  
>$ sudo apt-get install oracle-java8-jdk  

Create directory to download everything in  
>$ mkdir ~/ELK  
>$ cd ~/ELK  

Elasticsearch
-----------------------------
Download and install  
>$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.5.2.deb  
>$ sudo dpkg -i elasticsearch-5.5.2.deb  

>$ sudo emacs /etc/elasticsearch/elasticsearch.yml  

Line 17: cluster.name: hunnipi  
Line 55: network.host: 127.0.0.1  
Line 59: http.port: 9200  

>$ sudo emacs /etc/elasticsearch/jvm.options  

Line 23: -Xms200m  
Line 24: -Xms500m  

Start  
>$ sudo service elasticsearch start  

Add to autostart  
>$ sudo systemctl enable elasticsearch  

Logstash
-----------------------------
Download and install  
>$ wget https://artifacts.elastic.co/downloads/logstash/logstash-5.5.2.deb  
>$ sudo dpkg -i logstash-5.5.2.deb  

Setup the JFFI code for our ARM chip
>$ sudo apt-get install ant zip  
>$ sudo git clone https://github.com/jnr/jffi.git  
>$ cd jffi  
>$ sudo ant jar  
>$ sudo cp build/jni/libjffi-1.2.so /usr/share/logstash/vendor/jruby/lib/jni/arm-Linux  

(when the .so file is not generated, delete the complete jffi folder and reinstall again)  

>$ cd /usr/share/logstash/vendor/jruby/lib  
>$ sudo zip -g jruby-complete-1.7.11.jar jni/arm-Linux/libjffi-1.2.so  

Start  
>$ sudo service logstash start  

Add to autostart  
>$ sudo systemctl enable logstash  

Kibana
-----------------------------
Go to directory, download and unpack  
>$ cd ~/ELK
>$ wget https://artifacts.elastic.co/downloads/kibana/kibana-5.5.2-linux-x86.tar.gz  
>$ tar -xvzf kibana-5.5.2-linux-x86.tar.gz  

Create directory and move files  
>$ sudo mkdir /opt/kibana  
>$ sudo mv kibana-5.5.2-linux-x86/* /opt/kibana/  

Install latest ARM nodejs version  
>$ wget https://nodejs.org/download/release/v6.10.2/node-v6.10.2-linux-armv6l.tar.gz  
>$ tar -xvzf node-v6.10.2-linux-armv6l.tar.gz  

Move and create links  
>$ sudo cp node-v6.10.2-linux-armv6l/bin/node /usr/local/bin/node  
>$ sudo cp node-v6.10.2-linux-armv6l/bin/npm /usr/local/bin/npm  
>$ sudo mv /opt/kibana/node/bin/node /opt/kibana/node/bin/node.orig  
>$ sudo mv /opt/kibana/node/bin/npm /opt/kibana/node/bin/npm.orig  
>$ sudo ln -s /usr/local/bin/node /opt/kibana/node/bin/node  
>$ sudo ln -s /usr/local/bin/npm /opt/kibana/node/bin/npm  

Check settings  
>$ sudo emacs /opt/kibana/config/kibana.yml  

Line 2: server.port: 5601  
Line 7: server.host: "127.0.0.1"  
Line 21: elasticsearch.url: "http://127.0.0.1:9200"  

Add Kibana to our systemd folder  
>$ sudo emacs /etc/systemd/system/kibana.service  

	[Unit]
	Description=Kibana
	
	[Service]
	ExecStart=/opt/kibana/bin/kibana
	StandardOutput=null
	
	[Install]
	WantedBy=multi-user.target

Start  
>$ sudo service kibana start  

Add to autostart  
>$ sudo systemctl enable kibana  

Nginx
-----------------------------
Install  
>$ sudo apt-get install nginx apache2-utils  

Create user  
>$ sudo htpasswd -c /etc/nginx/htpasswd.users kibana_admin  

Edit config file  
>$ sudo emacs /etc/nginx/sites-available/default  

Replace  

	server {
	        listen 80 default_server;
	        listen [::]:80 default_server;
	
	        # SSL configuration
	        #
	        # listen 443 ssl default_server;
	        # listen [::]:443 ssl default_server;
	        #
	        # Self signed certs generated by the ssl-cert package
	        # Don't use them in a production server!
	        #
	        # include snippets/snakeoil.conf;
	        root /var/www/html;
	        # Add index.php to the list if you are using PHP
	        index index.html index.htm index.nginx-debian.html;
	        server_name _;
	        location / {
	                # First attempt to serve request as file, then
	                # as directory, then fall back to displaying a 404.
	                try_files $uri $uri/ =404;
	        }

with  

	server {
	    listen 8080;
	    server_name example.com;
	    auth_basic "Restricted Access";
	    auth_basic_user_file /etc/nginx/htpasswd.users;
	    location / {
	        proxy_pass http://127.0.0.1:5601;
	        proxy_http_version 1.1;
	        proxy_set_header Upgrade $http_upgrade;
	        proxy_set_header Connection 'upgrade';
	        proxy_set_header Host $host;
	        proxy_cache_bypass $http_upgrade;
	    }
	}
	
Restart everything  
>$ sudo service logstash restart && sudo service elasticsearch restart && sudo service kibana restart  

Start nginx service  
>$ sudo service nginx start  

Add to autostart  
>$ sudo systemctl enable nginx  

Now visit your new Kibana at:  
http://<IP>:8080  

Configure Kibana for cowrie
-----------------------------
Create log directory  
>$ sudo mkdir /var/log/kibana  
>$ sudo chown kibana:kibana /var/log/kibana)  

Edit config file  
>$ sudo emacs /opt/kibana/config/kibana.yml  

Line 18: server.name: "hunniPi"  
Line 86: logging.dest: /var/log/kibana/kibana.log  
Add line: tilemap.url: https://tiles.elastic.co/v2/default/{z}/{x}/{y}.png?elastic_tile_service_tos=agree&my_app_name=kibana  

Download GeoIP data and extract  
>$ wget http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.mmdb.gz  
>$ gunzip GeoLite2-City.mmdb.gz  

Create dir and move  
>$ sudo mkdir /usr/share/logstash/vendor/geoip/  
>$ sudo mv GeoLite2-City.mmdb /usr/share/logstash/vendor/geoip  
>$ sudo chown logstash:logstash -R /usr/share/logstash/vendor/geoip   

Copy config  
>$ sudo cp /home/cowrie/cowrie/docs/elk/logstash-cowrie.conf /etc/logstash/conf.d/  

Edit conf  
>$ sudo emacs /etc/logstash/conf.d/logstash-cowrie.conf  

Line 4: path => ["/home/cowrie/cowrie/var/log/cowrie/cowrie.json"]
Line 49: database => "/usr/share/logstash/vendor/geoip/GeoLite2-City.mmdb"  
Line 69: path => "/var/log/logstash/cowrie-logstash.log"  

Validate logstash config  
>$ /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/logstash-cowrie.conf -t  

Restart  
>$ sudo service logstash restart  

Test whether data is loaded into ElasticSearch  
>$ curl 'http://localhost:9200/_search?q=cowrie&size=5'  

If this gives output, your data is correctly loaded into ElasticSearch  

List indexes  
>$ curl 'http://localhost:9200/_cat/indices?v'  

To check that all services are running  
>$ sudo service --status-all  


Installing splunk log analyzer on server
=============================
On the splunk server:  
>\# wget -O splunk-7.2.3-06d57c595b80-Linux-x86_64.tgz 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=linux&version=7.2.3&product=splunk&filename=splunk-7.2.3-06d57c595b80-Linux-x86_64.tgz&wget=true'  
>\# tar xvzf splunk-7.2.3-06d57c595b80-Linux-x86_64.tgz -C /opt  

Create links, etc  
>\# ln -s /opt/splunk/etc /etc/splunk  
>\# ln -s /opt/splunk/var/log/splunk /var/log/splunk  
>\# ln -s /opt/splunk/var/run/splunk /var/run/splunk  
>\# ln -s /opt/splunk/var/spool/splunk /var/spool/splunk  
>\# ln -s /opt/splunk/bin/splunk /usr/bin/splunk  

Create startscript  
>\# curl https://data.gpo.zugaina.org/argent-main/net-analyzer/splunk/files/splunk.initd --output /etc/init.d/splunk  

or

>\# emacs /etc/init.d/splunk  

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
>\# chmod 755 /etc/init.d/splunk  

Add as service  
>\# rc-update add splunk    

Create an account on virustotal.com  
Find your API-key on yot community profile  
Add the key to line 11 in:  
/opt/splunk/etc/apps/tango/bin/vt.py  

Install python requests  
>\# cd ~/src  
>\# git clone https://github.com/kennethreitz/requests/  
>\# cp -R requests/requests /opt/splunk/etc/apps/tango/bin/    

Edit inputs  
>\# emacs /opt/splunk/etc/system/local/inputs.conf  

Add line:  

	[tcp:9997]  

Start and configure splunk  
>$ cd /opt/splunk/bin  
>$ ./splunk start  

Scroll down and agree to license  
Enter admin user name: splunkadmin  
Enter passsword twice  

Go to the link at end of text  
http://<IP>:8080  
Log in with user/pass from installation  
Click 'Settings'->'Forwarding and receiving'->'Configure receiving' + Add new  
Type in port '9997'  and click 'Save'  

Go into 'Settings'->'Access Controls'  
Then 'Roles'->'Admin'  
Then scroll all the way down to 'Indexes Searched by Default'  
Add 'cowrie' to the right-hand column  
Click 'Save'  

Install Splunk Universal Forwarder on hunniPi
-----------------------------
>$ wget -O splunkforwarder-7.2.3-06d57c595b80-Linux-arm.tgz 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=ARM&platform=linux&version=7.2.3&product=universalforwarder&filename=splunkforwarder-7.2.3-06d57c595b80-Linux-arm.tgz&wget=true'  

Install  
>$ sudo tar xvzf splunkforwarder-7.2.3-06d57c595b80-Linux-arm.tgz -C /opt  

Enable start at boot  
>$ sudo /opt/splunkforwarder/bin/splunk enable boot-start  

Scroll down and accept license  
Add user name: splunkadmin  
State password twice  

Configure the indexer that the forwarder will send its data to  
>$ sudo /opt/splunkforwarder/bin/splunk add forward-server <IP>:9997 -auth splunkadmin:<PASSWORD>  

Add data to consume  
>$ sudo /opt/splunkforwarder/bin/splunk add monitor /home/cowrie/cowrie/var/log/cowrie/cowrie.log -sourcetype linux_logs -index cowrie  
>$ sudo /opt/splunkforwarder/bin/splunk add monitor /home/cowrie/cowrie/var/log/cowrie/cowrie.json -sourcetype cowrie -index honeypot  

Download and configure Tango Intelligence app  
>$ git clone https://github.com/aplura/Tango.git  
>$ chmod +x Tango/tango_input/bin/input.sh  
>$ sudo cp -R Tango/tango* /opt/splunkforwarder/etc/apps/  

Add sensor  
>$ sudo /opt/splunkforwarder/bin/splunk add script /opt/splunkforwarder/etc/apps/tango_input/bin/input.sh -sourcetype sensor -index honeypot -interval 86400  


Start the forwarder  
>$ sudo /opt/splunkforwarder/bin/splunk start  















