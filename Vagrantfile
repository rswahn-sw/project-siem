# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
 
  # Use Ubuntu 22.04 (Jammy Jellyfish) as base image for all VMs
  config.vm.box = "ubuntu/jammy64"

  # --- FILTER VM (Logstash) ---
  # This VM will filter logs and send it to the SIEM VM
  config.vm.define "logstash" do |ls|

    ls.vm.hostname = "logstash-node"

    ls.vm.network "private_network", ip: "192.168.56.11"

    ls.vm.provider "virtualbox" do |vb|

      vb.name = "project-logstash"

      vb.memory = "512"

      vb.cpus = 1

    end

    ls.vm.provision "shell", inline: "apt-get update && apt-get install -y ansible"


    # Installing Ansible on ls node, making it the control VM which all ansible commands will run from
    # Vagrant will automatically install Ansible on this VM before running the playbook
    #ls.vm.provision "ansible_local" do |ansible|

      #ansible.playbook = "site.yml"

      #ansible.install = true # This tells Vagrant to install Ansible via apt for you

      #ansible.limit = "all"

      #ansible.groups = {
        #"elk_group"      => ["elk"],
        #"logstash_group" => ["logstash"],
        #"webserver_group"  => ["webserver"]
      #}

    # NY KOD - skapar SSH-nyckel och lägger den i delad mapp
    ls.vm.provision "shell", privileged: false, inline: <<-SHELL
      if [ ! -f ~/.ssh/id_ed25519 ]; then
        #ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
        sudo -u vagrant ssh-keygen -t ed25519 \ -f /home/vagrant/.ssh/id_ed25519 \ -N "" -C "logstash-node"
      fi
      cp /home/vagrant/.ssh/id_ed25519.pub \ /vagrant/ansible_id_ed25519.pub
      chmod 600 ~/.ssh/id_ed25519
    SHELL

  end
  



  # --- WEBSERVER VM ---
  # This VM is the target to be monitored
  config.vm.define "webserver" do |web|

    web.vm.hostname = "web-server"

    web.vm.network "private_network", ip: "192.168.56.12"

    web.vm.provider "virtualbox" do |vb|

      vb.name = "project-webserver"

      vb.memory = "512"

      vb.cpus = 1

    end

    # NY KOD - hämtar ls's nyckel från delad mapp
    web.vm.provision "shell", inline: <<-SHELL
    # Create directory if it doesn't exist


    # Lägg till kontrollnodens publika nyckel i

    # authorized_keys. Ansible kan nu SSH:a in.

    mkdir -p /home/vagrant/.ssh

    cat /vagrant/ansible_id_ed25519.pub \ >> /home/vagrant/.ssh/authorized_keys

    chmod 700 /home/vagrant/.ssh

    chmod 600 /home/vagrant/.ssh/authorized_keys

    chown -R vagrant:vagrant /home/vagrant/.ssh
  SHELL

  

  end

  # --- "ELK" VM (Elasticsearch & Kibana) ---
  # This is the ansible controller, and the one running the SIEM tools
  # Controller is created last so that it can use Ansible on the other already-created machines
  config.vm.define "elk" do |elk|

    elk.vm.hostname = "elk-node"

    elk.vm.network "private_network", ip: "192.168.56.10"

    elk.vm.provider "virtualbox" do |vb|

      vb.name = "project-elk"

      vb.memory = "1024" # ELK needs significant RAM - changed to 1 from 4 GB until programs are set up

      vb.cpus = 1 # Changed to 1 from 2, for now

    end

    # NY KOD - hämtar elk's nyckel från delad mapp
    elk.vm.provision "shell", inline: <<-SHELL
    # Create directory if it doesn't exist


    # Lägg till kontrollnodens publika nyckel i

    # authorized_keys. Ansible kan nu SSH:a in.

    mkdir -p /home/vagrant/.ssh

    cat /vagrant/ansible_id_ed25519.pub \ >> /home/vagrant/.ssh/authorized_keys

    chmod 700 /home/vagrant/.ssh

    chmod 600 /home/vagrant/.ssh/authorized_keys

    chown -R vagrant:vagrant /home/vagrant/.ssh
  SHELL

  end

end