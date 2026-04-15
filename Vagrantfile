# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # Use Ubuntu 22.04 (Jammy Jellyfish) as base image for all VMs
  config.vm.box = "ubuntu/jammy64"

  # --- VM 1: THE BRAIN (Elasticsearch & Kibana) ---
  config.vm.define "elk_node" do |elk|
    elk.vm.hostname = "elk-node"
    elk.vm.network "private_network", ip: "192.168.56.10"
    elk.vm.provider "virtualbox" do |vb|
      vb.memory = "4096" # ELK needs significant RAM
      vb.cpus = 2
    end
  end

  # --- VM 2: THE FILTER (Logstash) ---
  config.vm.define "logstash_node" do |ls|
    ls.vm.hostname = "logstash-node"
    ls.vm.network "private_network", ip: "192.168.56.11"
    ls.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 1
    end
  end

  # --- VM 3: THE TARGET (Webserver to be Monitored) ---
  config.vm.define "web_target" do |web|
    web.vm.hostname = "web-target"
    web.vm.network "private_network", ip: "192.168.56.12"
    web.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 1
    end
  end

  # --- ANSIBLE PROVISIONING ---
  # This runs from your HOST machine. 
  # It targets all VMs defined above using the 'site.yml' playbook.
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "site.yml"
    ansible.limit = "all"
    ansible.groups = {
      "elk"      => ["elk_node"],
      "logstash" => ["logstash_node"],
      "targets"  => ["web_target"]
    }
  end
end