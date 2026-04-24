# project-siem
"SIEM-in-a-box" project for virtualization and automation class

------------------------
    PROJECT BACKLOG
------------------------

1. Virtualization
    1.1. One ansible control VM, hostname logstash
    1.2. One VM running SIEM, hostname elk
    1.3. One VM running webserver, hostname webserver

2. Software
    2.1. Logstash
        2.1.1. Ansible
        2.1.2. Logstash
        2.1.3 Java Runtime Environment (installs through Logstash)
    2.2. Elk
        2.2.1. Elasticsearch
        2.2.2. Kibana
    2.3. Webserver
        2.3.1. Nginx
        2.3.2. Filebeat

3. Configurations
    3.1. Ansible playbook
    3.2. Logstash - Pipeline configuration file (.conf)
    3.3. Webserver - Filebeat configuration file (.yml)




---------------------
    INSTALLATIONS
---------------------

########
LOGSTASH
########

1. Download and install Public Signing Key
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic-keyring.gpg

2. Install apt-transport-https package (if it doesn't already exist)
sudo apt-get install apt-transport-https

3. Save the repository definition to /etc/apt/sources.list.d/elastic-9.x.list
echo "deb [signed-by=/usr/share/keyrings/elastic-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-9.x.list

4. Install Logstash
sudo apt-get install logstash


NOTE
Logstash installpath is /etc/logstash. Here is logstash.yml which is the primary configuration file. 
A new .conf file needs to be created for the pipeline to Filebeat. This goes in the /etc/logstash/conf.d directory. 

########
NGINX
########

1. Update the package list and install Nginx
sudo apt update && sudo apt install nginx -y

2. Enable Nginx to start automatically on reboot
sudo systemctl enable nginx

3. Start Nginx
sudo systemctl start nginx

########
FILEBEAT
########

1. Download and install Public Signing Key
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic-keyring.gpg

2. Install apt-transport-https package
sudo apt-get install-apt-transport-https -y

3. Save the repository definition to /etc/apt/sources.list.d/elastic-9.x.list
echo "deb [signed-by=/usr/share/keyrings/elastic-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-9.x.list

4. Update package list and install Filebeat
sudo apt-get update && sudo apt-get install filebeat -y

5. Enable Filebeat to start automatically on reboot
sudo systemctl enable filebeat

6. Start Filebeat
sudo systemctl start filebeat

--------------------
Summary of Data Flow
--------------------

1. Nginx writes a line to /var/log/nginx/access.log.
2. Filebeat detects the new line and sends it to Logstash (Port 5044).
3. Logstash receives the text, uses Grok to pull out the IP address, status code, and URL, then sends the JSON to Elasticsearch.
4. Elasticsearch stores the data for search and visualization.