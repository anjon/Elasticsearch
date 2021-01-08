# ELK/EFK
Elasticsearch,Logstash,Kibana / Elasticsearch,Filebeat,Kibana

#### Install ELK 
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
