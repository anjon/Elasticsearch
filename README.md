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
* Now we should be able to access our kibana dasahboard via the nginx reverse proxy using the ip*
**http://192.168.1.110**
