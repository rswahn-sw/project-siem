# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
 
  # Use Ubuntu 22.04 (Jammy Jellyfish) as base image for all VMs.
  config.vm.box = "ubuntu/jammy64"

  # --- FILTER AND CONTROLVM (Logstash) ---
  # This VM will filter logs and send it to the SIEM VM.
  # It will also run Ansible to set up the other machines, so it is created first.
  config.vm.define "logstash" do |ls|

    ls.vm.hostname = "logstash"

    ls.vm.network "private_network", ip: "192.168.56.11"

    ls.vm.provider "virtualbox" do |vb|

      vb.name = "project-logstash"

      vb.memory = "8192" # Souped up to 8 GB RAM for the Ansible playbook to run smoothly.

      vb.cpus = 2

    end

    # Installs the latest version of Ansible from the official Ansible PPA. 
    # This is necessary for the ansible-playbook command to work on this VM.
    ls.vm.provision "shell", inline: <<-SHELL

      export DEBIAN_FRONTEND=noninteractive
      
      apt-get update
      
      apt-get install -y software-properties-common
      
      apt-add-repository --yes --update ppa:ansible/ansible
      
      apt-get install -y ansible
    SHELL

    # Creates SSH key pair for Vagrant and copies them to the shared folder.
    ls.vm.provision "shell", inline: <<-SHELL

      sudo -u vagrant ssh-keygen -t ed25519 \ -f /home/vagrant/.ssh/id_ed25519 \ -N "" -C "logstash-node"

      cp /home/vagrant/.ssh/id_ed25519.pub \ /vagrant/ansible_id_ed25519.pub

      # Clones the project repository to the home directory of the logstash VM, so that Ansible can access the playbooks and configuration files.
      git clone https://github.com/rswahn-sw/project-siem
    SHELL

  end
  

  # --- WEBSERVER VM ---
  # This VM hosts the webserver to be monitored, and sends logs through Filebeat.
  config.vm.define "webserver" do |web|

    web.vm.hostname = "webserver"

    web.vm.network "private_network", ip: "192.168.56.12"

    web.vm.provider "virtualbox" do |vb|

      vb.name = "project-webserver"

      vb.memory = "512"

      vb.cpus = 1

    end

    # Adds the SSH key from the shared folder. 
    web.vm.provision "shell", inline: <<-SHELL

      # Create directory if it doesn't exist

      mkdir -p /home/vagrant/.ssh

      # Add the public key from the shared folder to authorized_keys, and change permissions so that Ansible can SSH in.

      cat /vagrant/ansible_id_ed25519.pub \ >> /home/vagrant/.ssh/authorized_keys

      chmod 700 /home/vagrant/.ssh

      chmod 600 /home/vagrant/.ssh/authorized_keys

      chown -R vagrant:vagrant /home/vagrant/.ssh
  SHELL

  

  end

  # --- "ELK" VM (Elasticsearch & Kibana) ---
  # This VM runs the SIEM stack (Elasticsearch and Kibana) and receives logs from the Logstash VM.
  config.vm.define "elk" do |elk|

    elk.vm.hostname = "elk"

    elk.vm.network "private_network", ip: "192.168.56.10"

    elk.vm.provider "virtualbox" do |vb|

      vb.name = "project-elk"

      vb.memory = "4096" # ELK needs significant RAM - changed to 1 from 4 GB until programs are set up

      vb.cpus = 1 # Changed to 1 from 2, for now

    end

    # Adds the SSH key from the shared folder.
    elk.vm.provision "shell", inline: <<-SHELL

      # Create directory if it doesn't exist

      mkdir -p /home/vagrant/.ssh

      # Add the public key from the shared folder to authorized_keys, and change permissions so that Ansible can SSH in.

      cat /vagrant/ansible_id_ed25519.pub \ >> /home/vagrant/.ssh/authorized_keys

      chmod 700 /home/vagrant/.ssh

      chmod 600 /home/vagrant/.ssh/authorized_keys

      chown -R vagrant:vagrant /home/vagrant/.ssh
  SHELL

  end

end