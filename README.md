# hunniPi

A tutorial and perhaps a script to get some honeypots running on Raspberry Pi


Work order
=============================

cowrie, ssh honeypot  
-----------------------------  
Install raspbian OS on Raspberry Pi  
https://github.com/jonsag/hunniPi/blob/master/raspbian%20OS%20on%20Raspberry%20Pi.md  

Install cowrie on Raspberry Pi  
https://github.com/jonsag/hunniPi/blob/master/cowrie%2C%20ssh%20honeypot%20on%20Raspberry%20Pi.md  

elasticsearch - logstash - kibana, analyzer  
-----------------------------  
Install ELK stack on server  
https://github.com/jonsag/hunniPi/blob/master/ELK%20stack%20on%20gentoo.md  

Install filebeat on Raspberry Pi  
https://github.com/jonsag/hunniPi/blob/master/filebeat%20on%20Raspberry%20Pi.md  

snare, web honeypot  
-----------------------------  
Install snare on Raspberry Pi  
https://github.com/jonsag/hunniPi/blob/master/snare%2C%20web%20honeypot%20on%20Raspberry%20Pi.md  


Credits
=============================
TAKHION: https://null-byte.wonderhowto.com/how-to/use-cowrie-ssh-honeypot-catch-attackers-your-network-0181600/  
mushmush: https://snare.readthedocs.io/en/latest/  
mushmush: https://tanner.readthedocs.io/en/latest/  
Fernando Domínguez: http://blog.fernandodominguez.me/logging-cowrie-logs-to-the-elk-stack/  
EVAL2A: https://eval2a.wordpress.com/2017/12/22/honeypot-part-2-visualizing-the-logs-using-the-elk-stack/  
elastic.co: https://discuss.elastic.co/  
Mitchell Anicas: https://www.digitalocean.com/community/tutorials/how-to-map-user-location-with-geoip-and-elk-elasticsearch-logstash-and-kibana  
stackoverflow.com: https://stackoverflow.com  
linuxquestion.org: https://www.linuxquestions.org  
Hay Turla: https://resources.infosecinstitute.com/glastopf-pi-a-simple-yet-cool-web-honeypot-for-your-raspberry-pi/#gref  
Habilis: https://habilisbest.com/install-redis-on-your-raspberrypi  
Ryan Gordon: https://blog.hypriot.com/getting-started-with-docker-on-your-arm-device/  
Tyler: https://howchoo.com/g/nmrlzmq1ymn/how-to-install-docker-on-your-raspberry-pi  
Sean Mancini: https://www.seanmancini.com/2018/02/check-snare-web-application-honey-pot-successor-glastopf/  
yerry pi: http://droidtoo.blogspot.com/2013/05/setting-up-dionaea-on-raspberry-pi.html  
REal0day: https://0x00sec.org/t/run-the-trap-how-to-setup-your-own-honeypot-to-collect-malware-samples/7445  


Check your honepot
=============================

Scan interesting ports
-----------------------------  
>$ sudo nmap 192.168.10.48 -p 22,23,80,81,2222,2223,5044,5045,5601,8080,8088,8090,8091,8092,9200,22222 -sS  




  















