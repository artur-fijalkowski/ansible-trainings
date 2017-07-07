# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

# number of nodes to manage (safe: 2)
nodes = 2

hosts = "192.168.144.200 controller.ansible.local controller\n"
inventory = "[controllers]\ncontroller.ansible.local ansible_connection=local\n[nodes]\n"
inventory_simple = "localhost ansible_connection=local\n"

(1..nodes).each do |i|
  hosts += "192.168.144.20#{i} node-#{i}.ansible.local node-#{i}\n"
  inventory += "node-#{i}.ansible.local ansible_host=192.168.144.20#{i}\n"
  inventory_simple += "node-#{i}\n"
end

open('./data/ansible/inventory.ini', 'w') do |f|
  f.puts inventory
end

open('./data/ansible/inventory_simple.ini', 'w') do |f|
  f.puts inventory_simple
end

Vagrant.configure("2") do |config|

  config.vm.box = "2.box"
  # config.vm.box = "ubuntu/xenial64"
  config.ssh.username = "ubuntu"
  # config.ssh.password = "5605ab75c7b6cc6baa497b18"
 
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder "./data", "/vagrant_data"
  config.vm.boot_timeout = 900

  config.vm.define "controller", autostart: true do |host|
    host.vm.network "private_network", ip: "192.168.144.200"
	host.vm.network "forwarded_port", guest: 80, host: 8080 
#    host.vm.hostname = "controller.ansible.local"
    host.vm.provider "virtualbox" do |v|
      v.memory = 1024
      v.cpus = 2
	  v.customize [ "modifyvm", :id, "--uartmode1", "file", File.join(Dir.pwd, "ubuntu-xenial-16.04-cloudimg-console.log") ]
    end

	host.vm.provision "shell", inline: <<-SHELL
      sudo -u ubuntu -H bash -e /vagrant_data/install_ansible.sh
	  echo "#{hosts}" >>/etc/hosts
    SHELL
  end

  (1..nodes).each do |i|
    config.vm.define "node-#{i}", autostart: true do |host|
      host.vm.network "private_network", ip: "192.168.144.20#{i}"
	  host.vm.network "forwarded_port", guest: 80, host: "808#{i}"
#      host.vm.hostname = "node-#{i}.ansible.local"
	  host.vm.provider "virtualbox" do |v|
        v.memory = 512
        v.cpus = 1
	    v.customize [ "modifyvm", :id, "--uartmode1", "file", File.join(Dir.pwd, "ubuntu-xenial-16.04-cloudimg-console.log") ]
      end
    
      host.vm.provision "shell", inline: <<-SHELL
        echo "#{hosts}" >>/etc/hosts
      SHELL
	end
  end

end
