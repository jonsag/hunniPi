# CUPS printer server, with Brother DCP-770CW, on Raspberry Pi  

Installation  
=============================

Install and configure CUPS  
-----------------------------
>$ sudo apt-get install cups  

Add lpadmin group to pi  
>$ sudo usermod -a -G lpadmin pi  

Configure CUPS  
>$ sudo emacs /etc/cups/cupsd.conf  

Comment out line 16 and add another below  
  
	# Listen localhost:631
	Port 631
	
Add/edit from line 30 to 47  

	# Restrict access to the server...                                                            
	<Location />
	  Order allow,deny
	  Allow @local
	</Location>
	
	# Restrict access to the admin pages...                                                       
	<Location /admin>
	  Order allow,deny
	  Allow @local
	</Location>
	
	# Restrict access to configuration files...                                                   
	<Location /admin/conf>
	  AuthType Default
	  Require user @SYSTEM
	  Order allow,deny
	  Allow @local
	</Location>
	
Restart server  
>$ sudo /etc/init.d/cups restart  

Add Brother DCP-770CW printer  
=============================  

LPR printer driver   
=============================  
Install prequisites  
>$ sudo apt-get install lib32stdc++  

Create directory  
>$ sudo mkdir /var/spool/lpd  

Download deb package  
>$ cd ~  
>$ wget https://download.brother.com/welcome/dlf005475/dcp770cwlpr-1.0.1-1.i386.deb  

Install  
>$ sudo dpkg -i --force-all dcp770cwlpr-1.0.1-1.i386.deb  

Edit config file  
>$ sudo emacs /etc/printcap  

Edit line 5, change from  

	:lp=/dev/usb/lp0:\
	
to  

	:rm=\<IP of printer\>\
	:rp=lp\
	
CUPSwrapper printer driver
=============================  
First install LPR printer driver as above  

Download deb package  
>$ wget https://download.brother.com/welcome/dlf005477/dcp770cwcupswrapper-1.0.1-1.i386.deb    

Install  
>$ sudo dpkg -i --force-all dcp770cwcupswrapper-1.0.1-1.i386.deb  


Add printer to CUPS  
============================= 

Add printer from cli  
-----------------------------
>$ sudo lpadmin -p 'Brother_DCP-770CW' -v 'socket://\<IP\>' -m 'Brother DCP-770CW‘ -P '/usr/share/ppd/brdcp770cw.ppd' -L '\<Location\>' -E  

Add printer to CUPS, manual entry  
-----------------------------
Go to \<IP\>:631  
Click 'Administration'  
Click 'Add printer' button. Enter username/password for pi if needed
Mark 'Other Network Printers:' 'LPD/LPR Host or Printer' and click 'Continue'  
In 'Connection:' add 'lpd://\<Your printer's IP address\>/binary_p1' and click 'Continue'  
Enter name, description and so on  
Mark 'Share' and click 'Continue'  
Set 'Make'->'Model' accordingly  
Click 'Add Printer'  
Click 'Set Default Options'  

Now you can do a test print, in web interface:  
Click 'Maintenance'->'Print Test Page'  


Add printer from web interface
-----------------------------
Goto \<IP\>:631  
Click 'Administration'  
Click 'Add printer' button. Enter username/password for pi if needed  
Mark 'Discovered Network Printers:' 'Brother DCP-770CW (Brother DCP-770CW)' and click 'Continue'  
Enter name, description and so on. Connection should be 'dnssd://Brother%20DCP-770CW._pdl-datastream._tcp.local/'  
Mark 'Share' and click 'Continue'  
Check settings and model etc. and click 'Continue'  

Now you can do a test print, in web interface:  
Click 'Maintenance'->'Print Test Page'  


Modify printer  
-----------------------------
Goto \<IP\>:631  
Click 'Administration'  
Click 'Manage printers' button. Enter username/password for pi  
Click 'DCP770CW'  
Click 'Administration'->'Modify Printer'  
Move mark to 'Discovered Network Printers:' 'Brother DCP-770CW (Brother DCP-770CW)' and click 'Continue'  
Enter description and so on. Connection should be 'dnssd://Brother%20DCP-770CW._pdl-datastream._tcp.local/'  
Mark 'Share' and click 'Continue'  
Check that model corresponds  
Click 'Modify Printer'  

Now you can do a test print, in web interface:  
Click 'Maintenance'->'Print Test Page'  


