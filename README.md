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

----------------------------
    COPY DIRECTORY TO VM
----------------------------

1. Run vagrant up for relevant VMs to be started
2. ssh into VM with vagrant ssh <VM name>
3. Copy the folder to VM by using cp -R /vagrant <name of folder>
4. Enter the new folder with cd <name of folder>
5. Run the playbook with ansible-playbook site.yml



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

4. Update package list and install Logstash
sudo apt-get update && sudo apt-get install logstash

NOTE
Logstash installpath is /etc/logstash. Here is logstash.yml which is the primary configuration file. 
A new .conf file needs to be created for the pipeline to Filebeat. This goes in the /etc/logstash/conf.d directory. 




#############
DOCKER ENGINE
#############

Docker Engine is installed in order to download and install Elasticsearch and Kibana in a faster, more effecient way with pre-set configurations. We opted to go for this in this project as we want easy and consistent automation of installations; the functionality of the SIEM stack is proof-of-concept only. 

# Add Docker's official GPG key:
sudo apt update         # Says they're already updated to latest version
sudo apt install ca-certificates curl       
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources (there gotta be an easier way to automate this later):
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update

# Install docker packages
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin




######################
ELASTICSEARCH + KIBANA
######################

# NOTE!!!
At this point, docker runs as root, and we're vagrant user when ssh'd in. This mean we don't actually have all permissions for the next step. 
AS OF MANUAL INSTALL TESTS, this is how we've solved it:

1. Give vagrant user permissions
sudo usermod -aG docker vagrant

# Before step 2, exit and re-log in as user

2. Install Elasticsearch and Kibana through Docker Engine
curl -fsSL https://elastic.co/start-local | sh


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
sudo apt-get install apt-transport-https -y

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



# SIEM-in-a-box
An automated security monitoring lab with three VMs provisioned via Vagrant and configured with Ansible, featuring an Nginx web server, Logstash log pipeline, and an ELK stack for log analysis.

## Table of contents
1. Architecture
2. Environments and IP addresses
3. Folder structure
4. Components
5. Requirements and prerequisites
6. Getting started
7. Secrets
8. Security measures
9. Security analysis
10. Verification
11. Design choices and motivation

## 1. Architecture



![alt text](<image (1).png>)


## 2. Environments and IP addresses

| VM | Role | IP address | Ports in use | Description |
|---|---|---|---|---|
| elk | SIEM node | 192.168.56.10 | 9200 5601 | Elasticsearch + Kibana, stores and visualizes logs |
| logstash | Ansible controller + Log pipeline | 192.168.56.11 | 5044 | Runs Ansible, receives logs from Filebeat and forwards to Elasticsearch |
| webserver | Monitored web server | 192.168.56.12 | 80 | Nginx + Filebeat, sends logs to Logstash |

## 3. Folder structure
```
project-siem/
├── .gitignore
├── ansible.cfg                    # Ansible configuration (inventory, host key checking)
├── ansible_id_ed25519.pub         # Public SSH key used by Ansible to connect to VMs
├── inventory.ini                  # Defines which servers Ansible manages and in which groups
├── README.md                      # Project documentation
├── secrets.yml                    # Credentials for Elasticsearch and Kibana (gitignored)
├── site.yml                       # Master playbook — runs all roles in correct order
├── Vagrantfile                    # Defines all VMs and network settings
├── .vagrant/                      # Auto-generated by Vagrant (do not edit)
│   ├── bundler/
│   ├── machines/
│   │   ├── elk/
│   │   │   └── virtualbox/
│   │   ├── logstash/
│   │   │   └── virtualbox/
│   │   └── webserver/
│   │       └── virtualbox/
│   ├── provisioners/
│   │   └── ansible/
│   │       └── inventory/
│   │           └── vagrant_ansible_inventory
│   └── rgloader/
│       └── loader.rb
├── group_vars/
│   └── all.yml                    # Variables shared across all hosts
├── host_vars/
│   ├── 192.168.56.10.yml          # Variables specific to elk node
│   ├── 192.168.56.11.yml          # Variables specific to logstash node
│   └── 192.168.56.12.yml          # Variables specific to webserver node
└── roles/
    ├── docker/                    # Installs Docker Engine on the elk node
    │   └── tasks/
    │       └── main.yml
    ├── elk/                       # Installs Elasticsearch and Kibana via Docker
    │   └── tasks/
    │       └── main.yml
    ├── filebeat/                  # Installs and configures Filebeat to ship Nginx logs
    │   ├── handlers/
    │   │   └── main.yml
    │   └── tasks/
    │       └── main.yml
    ├── logstash/                  # Installs and configures Logstash pipeline
    │   ├── handlers/
    │   │   └── main.yml
    │   └── tasks/
    │       └── main.yml
    └── nginx/                     # Installs Nginx web server
        ├── files/
        │   └── index.html
        └── tasks/
            └── main.yml
```
## 4. Components

### .gitignore
Holds secret.txt and ansible_id_ed25519.pub that holds credentials and SSH keys.

### ansible.cfg
Disables host key checking and specifies inventory.ini as the default inventory file.

### inventory.ini
Groups the servers into [logstash], [elk] and [webserver]. Specifies IP addresses and vagrant as the Ansible user. Logstash uses ansible_connection=local since Ansible runs directly on that node.

### Secret.txt
Holds the login credentials for Elastic SIEM. 

### site.yml
Playbook that installs and configures all components in order: 
1. **Update** — runs apt update and upgrade on all nodes
2. **ELK** — installs Docker, Elasticsearch and Kibana on the elk node
3. **Logstash** — installs and configures Logstash on the logstash node
4. **Nginx** — installs Nginx web server on the webserver node
5. **Filebeat** — installs and configures Filebeat to ship Nginx logs to Logstash

### Vagrantfile
Defines three virtual machines in VirtualBox with a shared host-only network (192.168.56.0/24). Logstash starts first so that Ansible can SSH into the other VMs.

### Docker Role
Installs Docker Engine on the elk node. Adds the official Docker GPG key and repository, installs Docker Engine with all required plugins, and adds the vagrant user to the docker group so that Docker commands can be run without sudo. Elasticsearch and Kibana run as Docker containers.

### ELK Role 
Docker engine installs Elasticsearch and Kibana on the elk node. The docker install comes with pre-set configurations, like file paths and necessary dependencies. Elasticsearch stores and indexes logs received from Logstash.  
Kibana visualizes the logs stored in Elasticsearch for search and analysis.

### Filebeat Role
Installs Filebeat on the webserver node, and configures it to collect Nginx logs and forwards them to port 5044 on the logstash node.

### Logstash Role
Installs Java (required by Logstash), adds the Elastic repository and installs Logstash 9.3.3. Configures a pipeline that:
- **Input** — listens for incoming logs from Filebeat on port 5044
- **Filter** — tags incoming logs as `web_server_logs` for easier searching in Kibana
- **Output** — forwards processed logs to Elasticsearch on port 9200

### Nginx Role
Installs Nginx on the webserver node and brings a sample website online.

## 5. Requirements and prerequisites
Software that must be installed on the Windows host:

* [VirtualBox](https://www.virtualbox.org/) — (tested with version 7.x)
* [Vagrant](https://www.vagrantup.com/) — (tested with version 2.x)
* [Git](https://git-scm.com/)

Hardware requirements:

* At least 8,5 GB free RAM 
* At least ? GB free disk space 


## 6. Getting started

**1.   Clone Github repo**

git clone git@github.com:rswahn-sw/project-siem.git
cd project-siem

**2. Start VMs**

vagrant up

**3. Connect remotely to the Ansible control VM (runs on logstash node) with SSH**

vagrant ssh logstash

 **4. Copy repo files to VM (this works since vagrantfile is in root folder)**

cp -R /vagrant project-siem
cd project-siem

**5. Run playbook**

ansible-playbook site.yml

**6. Navigate to SIEM**

open a web browser (host computer) and navigate to http://192.168.56.10:5601 (Kibana port at elk node VM)

**7. Fetch elastic credentials from secrets.yml (root folder) and log into Kibana**

username=elastic, password=ES_LOCAL_PASSWORD from secrets.yml

### Expected result
You should be able to find an automated http request from a curl source in the logs. You can create more logs by navigating to http://192.168.56.12 (nginx sample page).

## 7. Secrets
This files contains information about Elasticsearch and Kibana, including credentials. This can be encrypted with Ansible vault after logging in.

## 8. Security measures

| Measure | Where | How to verify |
|---|---|---|
| Private network | All VMs | VMs only communicate on 192.168.56.0/24, not accessible from internet |
| GPG key verification | All VMs | Packages verified with official GPG keys before installation |
| Elasticsearch and Kibana requires authentication | ElK node | curl http://192.168.56.10:9200 and curl http://192.168.56.10:5601 - should return 401 Unauthorized |
| All credentials and SSH keys are in seperate files and generated locally | Host | Old SSH keys and passwords will not work when you generate VMs again | 



## 9. Security analysis

### Remaining vulnerabilities

**Vulnerability 1: Unencrypted communication**

All traffic between VMs uses unencrypted HTTP. An attacker with access to the internal network could read or manipulate the traffic.

*Remediation*: Configure the network to use HTTPS traffic for communication between Filebeat, Logstash, Elasticsearch and Kibana.

*Accepted in this environment because*: The private network (192.168.56.0/24) is isolated from the internet and only accessible from the host machine. This lab environment is for proof of concept only and should not contain any sensitive information.  

**Vulnerability 2: Host key checking disabled**

`host_key_checking = False` in ansible.cfg means Ansible does not verify the identity of the hosts it connects to, which could allow man-in-the-middle attacks.

*Remediation*: Enable host key checking and manually distribute known host keys before running Ansible, or use a pre-configured known_hosts file.

*Accepted in this environment because*: In a local lab environment with no external access, the risk is considered acceptable.

**Vulnerability 3: No firewall**

No firewall (UFW) is configured on the VMs, meaning all ports are open within the private network.

*Remediation*: Configure UFW on all VMs to only allow necessary ports.

*Accepted in this environment because*: The private network is isolated from the internet and only accessible from the host machine.


**Vulnerability 4: All logs forwarded without filtering**

Filebeat is configured to forward all Nginx logs to Logstash without any filtering or rate limiting, which could overwhelm the pipeline with unnecessary data.

*Remediation*: Configure Filebeat and Logstash filters to only forward relevant log entries and add rate limiting.

*Accepted in this environment because*: In a lab environment with low traffic, the volume of logs is not a concern.


## 10. Verification
If the logs shows up in Kibana it means that everything is configured and working correctly. You can check that the logstash port is open and listening and that filebeat knows where to send the logs, but if it does not the logs will never show up in Kibana.

## 11. Design choices and motivation
### Logstash as Ansible control and starts first
The logstash node is in the middle of the data flow and already need connections to both other VMs. This makes it suitable to run Ansible as well since the playbook will only run once. The logstash node is created first because it generates the SSH keys that Ansible will use for the other VMs.

### SSH key distribution via shared folder
Instead of manually distributing SSH keys, Logstash generates an ed25519 key pair during provisioning and copies the public key to the shared `/vagrant` folder. Webserver and elk automatically pick up the key and add it to `authorized_keys`. This makes the setup fully automated as well as never shares the sensitive private SSH key.

### ELK stack via Docker
Elasticsearch and Kibana are installed via Docker using Elastic's official start-local script instead of installing them directly on the VM. This simplifies the installation and ensures a consistent and supported configuration.

### Private network
All VMs communicate on an isolated host-only network. This isolates the lab from the internet and avoids port conflicts with the host machine.

### Separate Ansible roles
Each component has its own Ansible role (docker, elk, logstash, nginx, filebeat). This makes every individual role easy to maintain, as well as allowing scalability by adding new roles.

### Specific versions
Logstash, Elasticsearch and Filebeat are set to specific version numbers to ensure compatibility.

### Variables
The variables supports scalability by allowing a value to be used multiple times but can be edited in just one place.