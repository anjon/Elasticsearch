# ELK/EFK
Elasticsearch,Logstash,Kibana / Elasticsearch,Filebeat,Kibana
![alt text](https://github.com/anjon/Elasticsearch/blob/main/elk_stack.png "ELK/EFK")

### Install ELK Stack
```sh
###### Install ELK Stack ######
# Install Java
sudo apt-get update
sudo apt-get install openjdk-11-jdk

# Install nginx 
sudo apt-get update && sudo apt-get install nginx
sudo systemctl enable nginx

# Install ELK 
sudo apt-get install apt-transport-https
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/oss-7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update && sudo apt-get install elasticsearch-oss kibana-oss logstash-oss filebeat
```
### Configure Elasticsearch
```sh
# Uncomment/modify the below line form the elasticsearch configuration
sudo vim /etc/elasticsearch/elasticsearch.yaml
cluster.name: my-application
node.name: node-1
network.host: localhost
http.port: 9200

# Now start the Elasticsearch 
sudo systemctl start elasticsearch.service
sudo systemctl status elasticsearch.service
```
### Configure kibana
```sh
# Uncomment the below line form the elasticsearch configuration
sudo vim /etc/kibana/kibana.yml
server.port: 5601
server.host: "localhost"

# Start the Kibana Service
sudo systemctl start kibana.service
sudo systemctl status kibana.service
```
### Confire Nginx for Kibana Access
```sh
sudo apt install apache2-utils
sudo htpasswd -c /etc/nginx/htpasswd.users kibadmin
cat /etc/nginx/htpasswd.users
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bkp
sudo vim /etc/nginx/sites-available/default
---------------
server {
        listen 80;

        server_name 192.168.1.110;

        auth_basic "Restricted Access";
        auth_basic_user_file /etc/nginx/htpasswd.users;

        location / {
            proxy_pass http://localhost:5601;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
    }
----------------
sudo systemctl restart nginx.service
sudo systemctl status nginx.service
```
*Now we should be able to access our kibana dasahboard via the nginx reverse proxy using the ip*
**http://192.168.1.110**
### Analyze 1st Sample apache data and Filter it with logstash
```sh
# Download the sample data
sudo wget https://logz.io/sample-data
sudo cp sample-data apache.log

# Configure logshash for filtering the data
vim /etc/logstash/conf.d/apachelog.conf
---------------------------------------
input {
  file {
    path => "/home/anjon/apache.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}

filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }

  date {
    match => [ "timestamp" , "dd/MMM/YYYY:HH:mm:ss Z" ]
  }

  geoip {
    source => "clientip"
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "petclinic-prd-1"
  }
}
-----------------------------------------------
# Start the logshash service
sudo systemctl start logstash.service
sudo systemctl status logstash.service
```
### Analyze 2nd Sample CSV data and Filter it with logstash
```sh
# Download the sample CSV 
sudo curl -O https://raw.githubusercontent.com/PacktPublishing/Kibana-7-Quick-Start-Guide/master/Chapter02/crimes_2001.csv

# Configure the filter for logstash
sudo vim /etc/logstash/conf.d/crimes.conf
-----------------------------------------
input {
  file {
    path => "/home/anjon/crimes_2001.csv"
    start_position => beginning
  }
}

filter {
  csv {
    columns => [
      "ID", "Case Number", "Date", "Block", "IUCR", "Primary Type", "Description",
      "location Description", "Arrest", "Domestic", "Beat", "District", "Ward",
      "Community Area", "FBI Code", "X Coordinate", "Y Coordinate", "Year",
      "Updated On", "lattitude", "longitude", "location"
    ]
    separator => ","
  }
}

output {
  elasticsearch {
    action => "index"
    hosts => ["localhost"]
    index => "crimes"
  }
}
------------------------------------------------
sudo systemctl restart logstash.service
sudo systemctl status logstash.service
```
*Now form the kibana dashboard we can index this data for analyze*
### Analyze realtime data(nginx logs) with filebeat
```sh
sudo filebeat modules list
sudo filebeat modules enable nginx
sudo filebeat modules enable system
sudo filebeat modules list

# Now as we enable 2 modules in filebeat let's configure those 2 modules 
vim /etc/filebeat/modules.d/nginx.yml
-------------------------------------
# Module: nginx
# Docs: https://www.elastic.co/guide/en/beats/filebeat/7.10/filebeat-module-nginx.html

- module: nginx
  # Access logs
  access:
    enabled: true
    var.paths: ["/var/log/nginx/access.log*"]           ///this is the logs path for analyze nginx access log
    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    #var.paths:

  # Error logs
  error:
    enabled: true
    var.paths: ["/var/log/nginx/error.log*"]            ///this is the logs path for analyze nginx error log
    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    #var.paths:

  # Ingress-nginx controller logs. This is disabled by default. It could be used in Kubernetes environments to parse ingress-nginx logs
  ingress_controller:
    enabled: false

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    #var.paths:
------------------------------------------------
# Again for the system logs to be analyze 
sudo vim /etc/filebeat/modules.d/system.yml
-------------------------------------------
# Module: system
# Docs: https://www.elastic.co/guide/en/beats/filebeat/7.10/filebeat-module-system.html

- module: system
  # Syslog
  syslog:
    enabled: true
    var.paths: ["/var/log/syslog*"]
    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    #var.paths:

  # Authorization logs
  auth:
    enabled: true
    var.paths: ["/var/log/auth.log*"]
    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    #var.paths:
--------------------------------------------------
# start the filebeat service 
sudo systemctl start filebeat.service
sudo systemctl status filebeat.service

# Also we can do the automatic filtering/indexing for filebeat with the below command
sudo filebeat setup -e
```
