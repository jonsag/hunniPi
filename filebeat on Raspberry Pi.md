# filebeat on Raspberry Pi

Install filebeat       
-----------------------------     
Download and install Go     
>$ cd ~/  
>$ wget https://redirector.gvt1.com/edgedl/go/go1.9.2.linux-armv6l.tar.gz     
     
Unpack     
>$ sudo tar -C /usr/local -xvzf go1.9.2.linux-armv6l.tar.gz     
     
Add to PATH and check version     
>$ export PATH=$PATH:/usr/local/go/bin     
>$ go version     
     
Download and build filebeat     
>$ export GOPATH=$HOME/go     
>$ mkdir go     
>$ mkdir -p ${GOPATH}/src/github.com/elastic     
>$ cd ${GOPATH}/src/github.com/elastic     
     
>$ git clone https://github.com/elastic/beats.git     
>$ cd beats     
>$ git checkout 23b9e27     
>$ cd filebeat     
>$ make     
>$ make update     
     
Finishing the installation     
>$ sudo mkdir /usr/share/filebeat /usr/share/filebeat/bin /etc/filebeat /var/log/filebeat /var/lib/filebeat     
>$ sudo mv filebeat /usr/share/filebeat/bin     
>$ sudo mv module /usr/share/filebeat/     
>$ sudo mv modules.d/ /etc/filebeat/     
>$ sudo cp filebeat.yml /etc/filebeat/     
>$ sudo chmod 750 /var/log/filebeat     
>$ sudo chmod 750 /etc/filebeat/     
>$ sudo chown -R root:root /usr/share/filebeat/*     
     
>$ sudo emacs /lib/systemd/system/filebeat.service    

	[Unit]     
	Description=filebeat     
	Documentation=https://www.elastic.co/guide/en/beats/filebeat/current/index.html     
	Wants=userwork-online.target     
	After=network-online.target     
	     
	[Service]     
	ExecStart=/usr/share/filebeat/bin/filebeat -c /etc/filebeat/filebeat.yml -path.home /usr/share/filebeat -path.config /etc/filebeat -path.data /var/lib/filebeat -path.logs /var/log/filebeat     
	Restart=always     
	     
	[Install]     
	WantedBy=multi-user.target     
	     
>$ sudo systemctl enable filebeat.service     
>$ sudo service filebeat start     
     
Setting filebeat up for cowrie     
>$ sudo emacs /etc/filebeat/filebeat.yml 
    
	filebeat.modules:
	
	filebeat.prospectors:
	- input_type: log
	  type: log
	  enabled: true
	  paths:
	    - /home/cowrie/cowrie/var/log/cowrie/cowrie.json*
	  encoding: plain
	  fields:
	    document_type: cowrie
	
	registry_file: /var/lib/filebeat/registry
	
	output.logstash:
	  hosts: ["192.168.10.6:5044"]  
	  index: hunniPi_cowri	
	  
	shipper:
	
	logging:
	  to_syslog: false
	  to_files: true
	  files:
	    path: /var/log/filebeat/
	    name: mybeat
	    rotateeverybytes: 10485760 # = 10MB
	    keepfiles: 7
	  level: info


Edit config     
>$ sudo emacs /etc/filebeat/filebeat.yml     

Line 5: /home/cowrie/cowrie/var/log/cowrie/cowrie.json*     
Line 12: hosts: ["\<IP of ELK server\>:5045"]     
     
Restart server     
>$ sudo service filebeat restart  

Useful commands  
-----------------------------     
Test filebeat config       
>$ sudo /usr/share/filebeat/bin/filebeat test config -c /etc/filebeat/filebeat.yml       
    