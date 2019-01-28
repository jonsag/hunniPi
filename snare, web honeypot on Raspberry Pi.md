# snare, web honeypot on Raspberry Pi  


snare
=============================
Install prerequisites  
>$ sudo apt-get install python3-pip  

Download snare  
>$ cd ~/  
>$ git clone https://github.com/mushorg/snare.git  

Install requirements  
>$ cd snare  
>$ pip3 install -r requirements.txt  

I also had to run  
>$ sudo pip3 install -r requirements.txt  

Setup  
>$ sudo python3 setup.py install  

Clone a web page  
>$ sudo clone --target http://example.com  

Run snare  
>$ sudo snare --port 8080 --page-dir example.com --host-ip 0.0.0.0  

File locations:  
Config file: /home/pi/snare/docs/conf.py  
Debug log file: /opt/snare/snare.log  
Error log: /opt/snare/snare.err  

tanner, data analysis served by snare events  
=============================

redis
-----------------------------  
Install  
>$ sudo apt-get install redis-server  

phpox, PHP Sandbox  
-----------------------------
Download  
>$ cd ~/  
>$ git clone https://github.com/mushorg/phpox.git  

Install  
>$ cd phpox  
>$ sudo make  

Start  
>$ sudo python3 sandbox.py  

tanner  
----------------------------
Install docker  
>$ cd ~/  
>$ curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh  

Download tanner  
>$ cd ~/  
>$ git clone https://github.com/mushorg/tanner.git  

Install requirements  
>$ cd tanner  
>$ pip3 install -r requirements.txt  

Install  
>$ sudo python3 setup.py install  

Run tanner server  
>$ sudo tanner  

Run tanner web  
>$ sudo tannerweb  

Run tanner api  
>$ sudo tannerapi  

File locations:  
Config file: /home/pi/tanner/tanner/config.py  
Debug log file: /opt/tanner/tanner.log  
Error log: /opt/tanner/tanner.err  

Web locations  
=============================
snare: http://\<IP\>:8080  
tanner: http://\<IP\>:8090  
tannerweb: http://\<IP\>:8091  
tannerapi: http://\<IP\>:8092  


























