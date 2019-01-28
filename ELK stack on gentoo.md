# ELK stack on gentoo  
    
Shut down all instances of ELK and nginx on RPi       
>\# sudo service elasticsearch stop         
>\# sudo systemctl disable elasticsearch       
>\# sudo service logstash stop           
>\# sudo systemctl disable logstash       
>\# sudo service kibana stop          
>\# sudo systemctl disable kibana       
>\# sudo service nginx stop        
>\# sudo systemctl disable nginx       
       
Verify java version       
>\# java -version       
  
Should be at least 1.8       
       
Elasticsearch       
-----------------------------       
Unmask       
>\# echo '=app-misc/elasticsearch-6.4.3 ~amd64' >> /etc/portage/package.keywords/use.keywords       
       
Install       
>\# emerge -av app-misc/elasticsearch       
       
Make sure you have proper permissions       
>\# chown root:elasticsearch /etc/elasticsearch && chmod 2750 /etc/elasticsearch       
>\# chown root:elasticsearch /etc/elasticsearch/elasticsearch.keystore && chmod 0660 /etc/elasticsearch/elasticsearch.keystore       
        
Configure       
>\# emacs /etc/elasticsearch/elasticsearch.yml         
  
Line 17: cluster.name: \<your name\>         
Line 55: network.host: 127.0.0.1         
Line 59: http.port: 9200        
       
Start         
>\# /etc/init.d/elasticsearch start         
       
Add to autostart         
>\# rc-update add elasticsearch default       
       
Logstash       
-----------------------------       
Unmask and mask       
>\# echo '=app-admin/logstash-bin-6.4.3 ~amd64' >> /etc/portage/package.keywords/use.keywords       
       
Install       
>\# emerge -av app-admin/logstash-bin       
       
Install plugin       
>\# /opt/logstash/bin/logstash-plugin install logstash-input-beats       
       
Start         
>\# /etc/init.d/logstash start         
       
Add to autostart         
>\# rc-update add logstash default       
       
Kibana       
-----------------------------       
Unmask and mask       
>\# echo '=www-apps/kibana-bin-6.4.3 ~amd64' >> /etc/portage/package.keywords/use.keywords       
       
Install       
>\# emerge -av www-apps/kibana-bin       
       
Configure       
>$ emacs /etc/kibana/kibana.yml         
  
Line 2: server.port: 5601         
Line 7: server.host: "127.0.0.1"         
Line 28: elasticsearch.url: "http://127.0.0.1:9200"       
       
Start         
>\# /etc/init.d/kibana start         
       
Add to autostart         
>\# rc-update add kibana default       
       
Nginx       
-----------------------------       
Install nginx       
       
Create user         
>\# htpasswd -c /etc/nginx/htpasswd.users kibana_admin       
       
Edit config file         
>\# emacs /etc/nginx/sites-available/default       
add:      
   
	server {       
	    listen 8088;       
	    server_name example.com;       
	    auth_basic "Restricted Access";       
	    auth_basic_user_file /etc/nginx/htpasswd.users;       
	    location / {       
	        proxy_pass http://127.0.0.1:5601;       
	        proxy_http_version 1.1;       
	        proxy_set_header Upgrade $http_upgrade;       
	        proxy_set_header Connection 'upgrade';       
	        proxy_set_header Host $host;       
	        proxy_cache_bypass $http_upgrade;       
	    }       
	    access_log /var/log/nginx/kibana.access_log main;       
	    error_log /var/log/nginx/kibana.error_log info;       
	}       
       
Start         
>\# /etc/init.d/nginx start         
       
Add to autostart         
>\# rc-update add nginx default       
       
Setup MaxMinds GeoIp databaSE       
>\# wget http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.mmdb.gz       
>\# mkdir /opt/logstash/vendor/geoip         
>\# gunzip GeoLite2-City.mmdb.gz       
>\# mv GeoLite2-City.mmdb /opt/logstash/vendor/geoip/       
       
       
Configure kibana       
-----------------------------       
>\# emacs /etc/kibana/kibana.yml       
  
Line 25: server.name: "\<your server name\>"         
Line 96: logging.dest: /var/log/kibana/kibana.log         
Add line: tilemap.url: https://tiles.elastic.co/v2/default/{z}/{x}/{y}.png?elastic_tile_service_tos=agree&my_app_name=kibana         
       
       
Configure logstash       
-----------------------------       
>\# emacs /etc/logstash/conf.d/cowrie.conf       
  
add:     
    
	input {  
	  beats {  
	    port => 5044    # Pick an available port to listen on  
	    host => "0.0.0.0"  
	  }  
	}  
	  
	filter {  
	  if [fields][document_type] == "cowrie" {  
	    json {  
	      source => message  
	    }  
	    date {  
	      match => [ "timestamp", "ISO8601" ]  
	    }  
	    if [src_ip]  {  
	      dns {  
	        reverse => [ "src_host", "src_ip" ]  
	        action => "append"  
	      }  
	      geoip {  
	        source => "src_ip"  # With the src_ip field  
	        target => "geoip"   # Add the geoip one  
	        # Using the database we previously saved  
	        database => "/opt/logstash/vendor/geoip/GeoLite2-City.mmdb"  
	        add_field => [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]  
	        add_field => [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]  
	      }  
	      # Get the ASN code as well  
	      #geoip {  
	        #source => "src_ip"  
	        #database => "/opt/logstash/vendor/geoip/GeoIPASNum.dat"  
	      #}  
	      mutate {  
	        convert => [ "[geoip][coordinates]", "float" ]  
	      }  
	    }  
	  }  
	}  
	  
	output {  
	  if [fields][document_type] == "cowrie" {  
	    # Output to elasticsearch  
	    elasticsearch {  
	      hosts => ["127.0.0.1:9200"]  # Provided elasticsearch is listening on that host:port  
	      index => ["cowrie-%{+YYYY.MM.dd}"]  
	      #sniffing => true  
	      manage_template => false  
	      #index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"  
	      document_type => "%{[@metadata][type]}"  
	    }  
	    #file {  
	    #  path => "/tmp/cowrie-logstash.log"  
	    #  codec => json  
	    #}  
	    # For debugging  
	    stdout {  
	      codec => rubydebug  
	    }  
	  }  
	}         
       
       
Useful commands       
-----------------------------       
Check logstash config       
>\#        
       
Run logstash from cli       
>\# /opt/logstash/bin/logstash -f /etc/logstash/conf.d/cowrie.conf --log.level debug       
          
Check interesting ports       
>\# nmap 192.168.10.6 -p 80,81,5044,5045,5601,8080,8088,9200 -sS       
       
Telnet to logstash       
>$ telnet 192.168.10.6 5044       
       
Test whether data is loaded into ElasticSearch         
>$ curl 'http://localhost:9200/_search?q=cowrie&size=5'       
       
Check cluster health       
>$ curl -XGET 'localhost:9200/_cluster/health?pretty'       
       
List indexes         
>$ curl 'http://localhost:9200/_cat/indices?v'       
       
Create an index called 'test'       
>$ curl -XPUT 'localhost:9200/test'       
       
Get info on index       
>$ curl -XGET 'localhost:9200/test?pretty'       
       
Delete index 'test'       
>$ curl -XDELETE localhost:9200/test       

List templates  
>$ curl GET 'http://localhost:9200/_template'  
       
Delete all indexes, DANGEROUS       
>$ curl -X DELETE 'http://localhost:9200/_all'       
       
Add data manually       
>$ for JSON in cowrie/*.*; do ./bulk_index.sh $JSON; done       
       
Restart everything       
>\# /etc/init.d/elasticsearch restart && /etc/init.d/logstash restart && /etc/init.d/kibana restart && /etc/init.d/nginx restart       
       
       
       
       
       
       
       
       
       
       
       
       
       
       
