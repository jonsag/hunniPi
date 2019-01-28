# filebeat on ubuntu     

Add the Beats repository     
>$ wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -     
     
Install the apt-transport-https package     
>$ sudo apt-get install apt-transport-https     
     
Save the repository definitio     
>$ echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list     
     
Install     
>$ sudo apt-get update && sudo apt-get install filebeat     
     
Run filebeat from cli     
>$ sudo filebeat -c /etc/filebeat/filebeat.yml -e -d "*"     
     
