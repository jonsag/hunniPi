# splunk, log analyzer on separate server  

On the splunk server:  
>$ cd ~/  
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
http://\<IP\>:8080  
Log in with user/pass from installation  
Click 'Settings'->'Forwarding and receiving'->'Configure receiving' + Add new  
Type in port '9997'  and click 'Save'  

Go into 'Settings'->'Access Controls'  
Then 'Roles'->'Admin'  
Then scroll all the way down to 'Indexes Searched by Default'  
Add 'cowrie' to the right-hand column  
Click 'Save'  


