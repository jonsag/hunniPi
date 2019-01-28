# splunk universal forwarder on Raspberry Pi  

>$ cd ~/  
>$ wget -O splunkforwarder-7.2.3-06d57c595b80-Linux-arm.tgz 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=ARM&platform=linux&version=7.2.3&product=universalforwarder&filename=splunkforwarder-7.2.3-06d57c595b80-Linux-arm.tgz&wget=true'  

Install  
>$ sudo tar xvzf splunkforwarder-7.2.3-06d57c595b80-Linux-arm.tgz -C /opt  

Enable start at boot  
>$ sudo /opt/splunkforwarder/bin/splunk enable boot-start  

Scroll down and accept license  
Add user name: splunkadmin  
State password twice  

Configure the indexer that the forwarder will send its data to  
>$ sudo /opt/splunkforwarder/bin/splunk add forward-server \<IP\>:9997 -auth splunkadmin:\<PASSWORD\>  

Add data to consume  
>$ sudo /opt/splunkforwarder/bin/splunk add monitor /home/cowrie/cowrie/var/log/cowrie/cowrie.log -sourcetype linux_logs -index cowrie  
>$ sudo /opt/splunkforwarder/bin/splunk add monitor /home/cowrie/cowrie/var/log/cowrie/cowrie.json -sourcetype cowrie -index honeypot  

Download and configure Tango Intelligence app  
>$ cd ~/  
>$ git clone https://github.com/aplura/Tango.git  
>$ chmod +x Tango/tango_input/bin/input.sh  
>$ sudo cp -R Tango/tango* /opt/splunkforwarder/etc/apps/  

Add sensor  
>$ sudo /opt/splunkforwarder/bin/splunk add script /opt/splunkforwarder/etc/apps/tango_input/bin/input.sh -sourcetype sensor -index honeypot -interval 86400  

Start the forwarder  
>$ sudo /opt/splunkforwarder/bin/splunk start  








