# nagios on Raspberry Pi  

Installation
=============================

Install prerequisites  
>$ sudo apt-get install apache2 libapache2-mod-php build-essential libgd2-xpm-dev  

Create user  
>$ sudo useradd -m -s /bin/bash nagios  

Set new password for nagios user (optional)  
>$ sudo passwd nagios  

Create a group for allowing external comands via the WEB UI  
>$ sudo groupadd nagcmd  

Add nagios and apache user to nagcmd group  
>$ sudo usermod -a -G nagcmd nagios  
>$ sudo usermod -a -G nagcmd www-data  

Download nagios sources  
>$ cd ~  
>$ wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.3.tar.gz  
>$ wget http://www.nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz  

Unpack sources  
>$ tar zxvf nagios-4.4.3.tar.gz  
>$ tar zxvf nagios-plugins-2.2.1.tar.gz  

Configure, compile and install nagios core  
>$ cd nagios-4.4.3  
>$ ./configure --with-command-group=nagcmd  
>$ make all  
>$ sudo make install  
>$ sudo make install-init  
>$ sudo make install-config  
>$ sudo make install-commandmode  
>$ sudo make install-webconf  

Optional installs  
-----------------------------  
Install the Exfoliation theme for the Nagios web interface  
>$ sudo make install-exfoliation  

Install the classic theme for the Nagios web interface  
>$ sudo make install-classicui  

Create a user for logging in to nagios web ui  
>$ sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin    

Configure, compile and install plugins  
>$ cd ~/nagios-plugins-2.2.1  
>$ ./configure --with-nagios-user=nagios --with-nagios-group=nagios  
>$ make  
>$ make install  

Verify configuration for errors  
>$ sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg  

Start nagios service  
>$ sudo service nagios start  

Add nagios to autostart  
>$ sudo systemctl enable nagios  

Restart apache  
>$ sudo service apache2 restart  

Now you can visit http://\<IP\>/nagios

NRPE - Nagios Remote Plugin Executor
=============================

On server  
-----------------------------  
>$ cd ~  
>$ wget https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-3.2.1/nrpe-3.2.1.tar.gz  
>$ tar zxvf nrpe-3.2.1.tar.gz    
>$ cd nrpe-3.2.1  
>$ ./configure --with-ssl=/usr/bin/openssl --with-ssl-lib=/usr/lib/arm-linux-gnueabihf  
>$ make check_nrpe  
>$ sudo make install-plugin  

On client  
-----------------------------  
Make sure you are running version >3  

Install nrpe client on Raspberry Pi  
-----------------------------  
Create user  
>$ sudo useradd -m -s /bin/false nagios  

Install prerequisites  
>$ sudo apt-get install libssl-dev  

Download, configure and install    
>$ cd ~  
>$ wget https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-3.2.1/nrpe-3.2.1.tar.gz  
>$ wget http://www.nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz  
>$ tar zxvf nrpe-3.2.1.tar.gz  
>$ tar zxvf nagios-plugins-2.2.1.tar.gz  
>$ cd nrpe-3.2.1  
>$ ./configure --with-ssl=/usr/bin/openssl --with-ssl-lib=/usr/lib/arm-linux-gnueabihf  
>$ make nrpe  
>$ sudo make install-daemon  
>$ sudo make install-config  
>$ sudo make install-init  
>$ cd ~/nagios-plugins-2.2.1  
>$ ./configure --with-nagios-user=nagios --with-nagios-group=nagios  
>$ make  
>$ sudo make install  

Edit configuration file  
>$ sudo emacs /usr/local/nagios/etc/nrpe.cfg  

Add IP for machine running nagios server to line 106  

	allowed_hosts=\<IP\>,127.0.0.1,::1  

Add line (353):  

	include=/usr/local/nagios/etc/mynrpe.cfg  
	
Edit new config file  
>$ sudo emacs include=/usr/local/nagios/etc/mynrpe.cfg  

Add:  

	command[check_mmcblk0p1]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/mmcblk0p1  
	command[check_mmcblk0p2]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/mmcblk0p2  
	
Start service  
>$ sudo service nrpe start  

Add to autostart  
>$ sudo systemctl enable nrpe  


Configuration
=============================

Create dir for log files  
>$ sudo mkdir /var/log/nagios  

Create symbolic link to log  
>$ sudo ln -s /usr/local/nagios/var/nagios.log /var/log/nagios/nagios.log  

Create symbolic links to configs for comfort  
>$ sudo ln -s /usr/local/nagios/etc /etc/nagios  

Checks are located at  
/usr/local/nagios/libexec/  

Configuration files in /usr/local/nagios/etc/objects
-----------------------------  
commands.cfg ->  
contacts.cfg ->  
timeperiods.cfg ->  
templates.cfg ->  
localhost.cfg -> definitions for monitoring the local host  

Add own configuration files  
-----------------------------  

Edit nagios.cfg  
>$ emacs /etc/nagios/nagios.cfg  

Add:  

	cfg_dir=/usr/local/nagios/etc/myconfigs 

My own configuration files in /usr/local/nagios/etc/myconfigs   
-----------------------------  
hostgroups.cfg  
hosts.cfg  
mycommands.cfg  
myservicedefinitions.cfg  
servicedefinitions.cfg  


Misc plugins installation and configuration  
=============================

check_cpu_stats.sh  
=============================

On client  
-----------------------------  
>$ sudo apt-get install ksh sysstat  

>$ sudo wget "https://exchange.nagios.org/components/com_mtree/attachment.php?link_id=608&cf_id=30" -O /usr/local/nagios/libexec/check_cpu_stats.sh  
>$ sudo chown nagios:nagios /usr/local/nagios/libexec/check_cpu_stats.sh  
>$ sudo chmod a+x /usr/local/nagios/libexec/check_cpu_stats.sh  

TBA  

On server  
-----------------------------  
TBA  

nagisk.pl  
=============================

On client  
-----------------------------  

>$ wget https://github.com/nicolargo/nagisk/raw/master/nagisk.pl -O /usr/local/nagios/libexec/nagisk.pl  
>$ sudo chown nagios:nagios /usr/local/nagios/libexec/nagisk.pl  
>$ sudo chmod a+x /usr/local/nagios/libexec/nagisk.pl  

Edit config file  
>$ emacs /usr/local/nagios/etc/mynrpe.cfg  

Add: 

	command [check_asterisk_version]=/usr/local/nagios/libexec/nagisk.pl -c version  
	command [check_asterisk_peers]=/usr/local/nagios/libexec/nagisk.pl -c peers  
	command [check_asterisk_channels]=/usr/local/nagios/libexec/nagisk.pl -c channels  
	command [check_asterisk_zaptel]=/usr/local/nagios/libexec/nagisk.pl -c zaptel  
	command [check_asterisk_span]=/usr/local/nagios/libexec/nagisk.pl -c span -s 1  

On server  
-----------------------------

Edit config file  
>$ emacs myservicedefinitions.cfg

	define service { 
	use generic-Service 
	host_name raspbx 
	service_description Check SIP 
	servicegroups sip 
	check_command check_nrpe! check_asterisk_version 
	}
	
	define service { 
	use generic-service 
	host_name raspbx 
	service_description Check SIP peers 
	servicegroup sip 
	check_command check_nrpe! check_asterisk_peers 
	}
	
	define service { 
	use generic-service 
	host_name raspbx 
	service_description check SIP channels 
	servicegroup sip 
	check_command check_nrpe! check_asterisk_channels 
	}
	
	define service { 
	use generic-service 
	host_name raspbx 
	service_description Check Zaptel card 
	servicegroup sip 
	check_command check_nrpe! check_asterisk_zaptel 
	}
	
	define service { 
	use generic-service 
	host_name sip 
	service_description Check Zaptel Span 1 
	servicegroups raspbx 
	check_command check_nrpe! check_asterisk_span 
	}


Commands
=============================

$USER1$ = /usr/local/nagios/libexec/  

check_http -I $HOSTADDRESS$ $ARG1$  
check_http  

check_nrpe -H \<host\> -c \<command\>  
check_nrpe!check_load  
check_nrpe!check_users  
check_nrpe!check_total_procs  
check_nrpe!check_zombie_procs  
check_nrpe!check_mmcblk0p1  
check_nrpe!check_mmcblk0p2  





















