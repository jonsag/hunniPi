# nagios on Raspberry Pi  

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
>$ sudo systemstl enable nagios  

Restart apache  
>$ sudo service apache2 restart  

Now you can visit http://\<IP\>/nagios

Create dir for log files  
>$ sudo mkdir /var/log/nagios  

Create symbolic link to log  
>$ sudo ln -s /usr/local/nagios/var/nagios.log /var/log/nagios/nagios.log  

Create symbolic links to configs for comfort  
>$ sudo ln -s /usr/local/nagios/etc /etc/nagios  

Configuration files
objects/commands.cfg ->  
objects/contacts.cfg ->  
objects/timeperiods.cfg ->  
objects/templates.cfg ->  
objects/localhost.cfg -> definitions for monitoring the local host  

Add line to nagios.cfg  
>$ emacs /etc/nagios/nagios.cfg  

Add:  

	cfg_dir=/usr/local/nagios/etc/myconfigs 

My own configuration files  
myconfigs/hostgroups.cfg  
myconfigs/servicedefinitions.cfg  
























