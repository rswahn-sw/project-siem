# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
 
  # Use Ubuntu 22.04 (Jammy Jellyfish) as base image for all VMs
  config.vm.box = "ubuntu/jammy64"

  # --- VM 1: THE BRAIN (Elasticsearch & Kibana) ---
  config.vm.define "elk" do |elk|

    elk.vm.hostname = "elk-node"

    elk.vm.network "private_network", ip: "192.168.56.10"

    elk.vm.provider "virtualbox" do |vb|

      vb.name = "project-elk"

      vb.memory = "1024" # ELK needs significant RAM - changed to 1 from 4 GB until programs are set up

      vb.cpus = 1 # Changed to 1 from 2, for now

    end

    # Installing Ansible on elk node, making it the control VM which all ansible commands will run from
    # Vagrant will automatically install Ansible on this VM before running the playbook
    elk.vm.provision "ansible_local" do |ansible|

      ansible.playbook = "site.yml"

      ansible.install = true # This tells Vagrant to install Ansible via apt for you

    end

  end

  # --- VM 2: THE FILTER (Logstash) ---
  config.vm.define "logstash" do |ls|

    ls.vm.hostname = "logstash-node"

    ls.vm.network "private_network", ip: "192.168.56.11"

    ls.vm.provider "virtualbox" do |vb|

      vb.name = "project-logstash"

      vb.memory = "1024"

      vb.cpus = 1

    end

  end

  # --- VM 3: THE TARGET (Webserver to be Monitored) ---
  config.vm.define "web_target" do |web|

    web.vm.hostname = "web-target"

    web.vm.network "private_network", ip: "192.168.56.12"

    web.vm.provider "virtualbox" do |vb|

      vb.name = "project-webserver"

      vb.memory = "1024"

      vb.cpus = 1

    end

  end


  # Calling the ansible playbook, grayed out for now cause of errors

  #  ansible.limit = "all"

  #  ansible.groups = {
  #    "elk"      => ["elk_node"],
  #    "logstash" => ["logstash_node"],
  #    "targets"  => ["web_target"]
  #  }

  # end

end