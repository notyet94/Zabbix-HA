# -*- mode: ruby -*-
# vi: set ft=ruby :
$script = <<SCRIPT
#!/bin/sh
echo "running script in the VM"
timedatectl set-timezone EST
sysctl vm.swappiness=0
echo "vm.swappiness=0" | tee --append /etc/sysctl.conf
yum clean all
yum check-update
# Vagrant's "change host name" sets the short host name.
# Before we undo the /etc/hosts silliness (see below) let's
# reset /etc/hostname to the *full* host name
#
#hostname --fqdn > /etc/hostname && hostname -F /etc/hostname

# Vagrant's "change host name" capability for Fedora
# maps hostname to loopback, conflicting with hostmanager.
# We must repair /etc/hosts
#
sed -ri 's/127\.0\.0\.1\s.*/127.0.0.1 localhost localhost.localdomain/' /etc/hosts
SCRIPT


# Builds custom test environment using JSON config file

# read vm and chef configurations from JSON files
nodes_config = (JSON.parse(File.read("nodes.json")))['nodes']

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  nodes_config.each do |node|
    node_name   = node[0] # name of node
    node_values = node[1] # content of node

    config.vbguest.auto_update = true
    config.vbguest.iso_path = "http://download.virtualbox.org/virtualbox/%{version}/VBoxGuestAdditions_%{version}.iso"
    
    config.vm.box = node_values[':box']

    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.hostmanager.ignore_private_ip = false
    config.hostmanager.include_offline = true
    config.ssh.insert_key = false
	config.ssh.private_key_path = ["~/.vagrant.d/insecure_private_key", "~/.ssh/id_rsa"]

    config.vm.define node_name do |config|
      # configures all forwarding ports in JSON array
      ports = node_values['ports']
      ports.each do |port|
        config.vm.network :forwarded_port,
          host:  port[':host'],
          guest: port[':guest'],
          id:    port[':id']
      end
      config.vm.hostname = node_name
      config.vm.network :private_network, ip: node_values[':ip']

      config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", node_values[':memory']]
        vb.customize ["modifyvm", :id, "--cpus", node_values[':cpus']]
        vb.customize ["modifyvm", :id, "--name", node_name]
      end
	  config.vm.synced_folder "CONFIGs/", "/CONFIGs"
      config.vm.provision "shell", inline: $script
	  config.vm.provision :shell, :path => node_values[':bootstrap']
    
	  
	end
  end
  
   
end

