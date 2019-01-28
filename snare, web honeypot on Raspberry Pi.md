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
Download  

>$ git clone https://github.com/mushorg/tanner.git  

Install requirements  

>$ cd tanner  
>$ pip3 install -r requirements.txt  

Install  

>$ sudo python3 setup.py install  

Run  

>$ sudo tanner  


























