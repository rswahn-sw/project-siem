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
- Architecture
- Environments and IP addresses
- Folder structure
- Components
- Requirements and prerequisites
- Getting started
- Secrets
- Security measures
- Security analysis
- Verification
- Design choices and motivation

## Architecture
 Instruktion: Lägg in ett arkitekturdiagram här. Du kan använda något av följande alternativ:

draw.io / diagrams.net — exportera som PNG och lägg filen i docs/architecture.png, referera sedan med bildsyntaxen nedan
draw.io XML — spara filen som docs/architecture.drawio (kan öppnas och redigeras direkt på GitHub)
ASCII-diagram — rita direkt i Markdown om du föredrar det, se exemplet nedan
Ta bort denna instruktionsruta när du är klar.

## Environments and IP addresses

| VM | Role | IP address | Ports in use | Description |
|---|---|---|---|---|
| elk | SIEM node | 192.168.56.10 | 9200 5601 | Elasticsearch + Kibana, stores and visualizes logs |
| logstash | Ansible controller + Log pipeline | 192.168.56.11 | 5044 | Runs Ansible, receives logs from Filebeat and forwards to Elasticsearch |
| webserver | Monitored web server | 192.168.56.12 | 80 | Nginx + Filebeat, sends logs to Logstash |

## Folder structure
```
project-siem/
├── Vagrantfile
├── inventory.ini
├── ansible.cfg
├── site.yml
├── README.md
├── .gitignore
└── roles/
    ├── docker/
    │   └── tasks/
    │       └── main.yml
    ├── elk/
    │   └── tasks/
    │       └── main.yml
    ├── filebeat/
    │   └── tasks/
    │       └── main.yml
    ├── logstash/
    │   ├── handlers/
    │   │   └── main.yml
    │   └── tasks/
    │       └── main.yml
    └── nginx/
        ├── files/
        │   └── index.html
        └── tasks/
            └── main.yml
```
## Components

### Vagrantfile
Defines three virtual machines in VirtualBox with a shared host-only network (192.168.56.0/24). Logstash starts first so that Ansible can SSH into the other VMs.

### ansible.cfg
Disables host key checking and specifies inventory.ini as the default inventory file.

### inventory.ini
Groups the servers into [logstash], [elk] and [webserver]. Specifies IP addresses and vagrant as the Ansible user. Logstash uses ansible_connection=local since Ansible runs directly on that node.

### site.yml
Playbook that installs and configures all components in order: 
1. **Update** — runs apt update and upgrade on all nodes
2. **ELK** — installs Docker, Elasticsearch and Kibana on the elk node
3. **Logstash** — installs and configures Logstash on the logstash node
4. **Nginx** — installs Nginx web server on the webserver node
5. **Filebeat** — installs and configures Filebeat to ship Nginx logs to Logstash

### Logstash Role
Installs Java (required by Logstash), adds the Elastic repository and installs Logstash 9.3.3. Configures a pipeline that:
- **Input** — listens for incoming logs from Filebeat on port 5044
- **Filter** — tags incoming logs as `web_server_logs` for easier searching in Kibana
- **Output** — forwards processed logs to Elasticsearch on port 9200

### Nginx Role
Installs Nginx on the webserver node and allows HTTP traffic through UFW on port 80.

### Filebeat Role
Installs Filebeat on the webserver node, (collects Nginx logs and forwards them to Logstash).

### Elasticsearch Role
Stores and indexes logs received from Logstash.

### Kibana Role
Web interface for visualizing and searching logs stored in Elasticsearch. 

### Docker Role
Installs Docker Engine on the elk node. Adds the official Docker GPG key and repository, installs Docker Engine with all required plugins, and adds the vagrant user to the docker group so that Docker commands can be run without sudo. Elasticsearch and Kibana run as Docker containers.

## Requirements and prerequisites
Software that must be installed on the Windows host:

* [VirtualBox](https://www.virtualbox.org/) — (tested with version 7.x)
* [Vagrant](https://www.vagrantup.com/) — (tested with version 2.x)
* [Git](https://git-scm.com/)

Hardware requirements:

* At least ? GB RAM (project uses approximately 6? GB in total)
* At least 10 GB free disk space ??

Secret.txt

## Getting started

1.  

2.

3.

4.


## Secrets


## Security measures

| Measure | Where | How to verify |
|---|---|---|
| Private network | All VMs | VMs only communicate on 192.168.56.0/24, not accessible from internet |
| GPG key verification | All VMs | Packages verified with official GPG keys before installation |
| Elasticsearch not exposed externally | ELK node | Only accessible on 192.168.56.10:9200 within private network |
| Logstash not exposed externally | Logstash node | Only accessible on 192.168.56.11:5044 within private network |
| Elasticsearch requires authentication | ElK node | curl http://192.168.56.10:9200 - should return 401 Unauthorized |



## Security analysis

### Remaining vulnerabilities

**Vulnerability 1: Unencrypted communication**

All traffic between VMs uses unencrypted HTTP. An attacker with access to the internal network could read or manipulate the traffic.

*Remediation*: Configure TLS certificates for communication between Filebeat, Logstash, Elasticsearch and Kibana.

*Accepted in this environment because*: The private network (192.168.56.0/24) is isolated from the internet and only accessible from the host machine. Risk is considered low in a lab environment.

**Vulnerability 2: Host key checking disabled**

`host_key_checking = False` in ansible.cfg means Ansible does not verify the identity of the hosts it connects to, which could allow man-in-the-middle attacks.

*Remediation*: Enable host key checking and manually distribute known host keys before running Ansible, or use a pre-configured known_hosts file.

*Accepted in this environment because*: In a local lab environment with no external access, the risk is considered acceptable.

**Vulnerability 3: No firewall**

No firewall (UFW) is configured on the VMs, meaning all ports are open within the private network.

*Remediation*: Configure UFW on all VMs to only allow necessary ports.

*Accepted in this environment because*: The private network is isolated from the internet and only accessible from the host machine.

**Vulnerability 4: Ansible runs as root**

Ansible runs all tasks with `become: yes` (root privileges) instead of a dedicated service user. If the playbook is compromised, an attacker would have full root access to all VMs.

*Remediation*: Create a dedicated Ansible user with only the necessary privileges.

*Accepted in this environment because*: In a local lab environment this is acceptable for simplicity.

**Vulnerability 5: No log rotation**

Nginx access logs can grow indefinitely and potentially fill the disk, causing services to stop working.

*Remediation*: Configure logrotate to automatically rotate and compress log files.

*Accepted in this environment because*: In a short-lived lab environment disk space is not a critical concern.

**Vulnerability 6: All logs forwarded without filtering**

Filebeat is configured to forward all Nginx logs to Logstash without any filtering or rate limiting, which could overwhelm the pipeline with unnecessary data.

*Remediation*: Configure Filebeat and Logstash filters to only forward relevant log entries and add rate limiting.

*Accepted in this environment because*: In a lab environment with low traffic, the volume of logs is not a concern.


## Verification


## Design choices and motivation
