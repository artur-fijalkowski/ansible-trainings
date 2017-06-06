# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|

  config.vm.box = "2.box"
  # config.vm.box = "ubuntu/xenial64"
  config.ssh.username = "ubuntu"
  # config.ssh.password = "5605ab75c7b6cc6baa497b18"
 
  config.vm.synced_folder "./data", "/vagrant_data"
  config.vm.boot_timeout = 900

  config.vm.define "controller", autostart: true do |host|
    host.vm.network "private_network", ip: "192.168.144.200"
	host.vm.network "forwarded_port", guest: 80, host: 8080 
#    host.vm.hostname = "controller.ansible.local"
    host.vm.provider "virtualbox" do |v|
      v.memory = 1024
      v.cpus = 2
    end
	host.vm.provision "shell", inline: <<-SHELL
      sudo -u ubuntu -H bash -e /vagrant_data/install_ansible.sh
	  echo "192.168.144.200 controller.ansible.local controller" >>/etc/hosts
	  echo "192.168.144.201 node-1.ansible.local node-1" >>/etc/hosts
	  echo "192.168.144.202 node-2.ansible.local node-2" >>/etc/hosts
    SHELL
  end

  [1,2].each do |i|
    config.vm.define "node-#{i}", autostart: true do |host|
      host.vm.network "private_network", ip: "192.168.144.20#{i}"
	  host.vm.network "forwarded_port", guest: 80, host: "808#{i}"
#      host.vm.hostname = "node-#{i}.ansible.local"
	  host.vm.provider "virtualbox" do |v|
        v.memory = 512
        v.cpus = 1
      end
    
      host.vm.provision "shell", inline: <<-SHELL
	    echo "192.168.144.200 controller.ansible.local controller" >>/etc/hosts
	    echo "192.168.144.201 node-1.ansible.local node-1" >>/etc/hosts
	    echo "192.168.144.202 node-2.ansible.local node-2" >>/etc/hosts
      SHELL
	end
  end

end
