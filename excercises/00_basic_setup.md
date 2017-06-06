## Start

1. install Oracle Virtual Box
2. install Vagrant from Hashicorp
3. install git for Windows (optional)

## Let's rock

1. go to directory with box file and Vagrantfile
2. execute `vagrant up`
3. (wait)
4. Log-in to controller host (`vagrant ssh controller` or use putty - ubuntu@127.0.0.1:2222, password is in Vagrantfile)
5. Create `~/.ssh/id_rsa` containing file `vagrant_private_key` from `.vagrant.d` on you host machine
6. Change mode to 600 for `~/.ssh/id_rsa` (`chmod 600 ~/.ssh/id_rsa`)
7. Turn on virtualenv environment `source ~/venv/bin/activate`
8. Check on your web browser following urls: [controller](http://localhost:8080) [node-1](http://localhost:8081) [node-2](http://localhost:8082)

## Command your ships

1. Go to directory `/vagrant_data/ansible`
2. Check if you can ping all hosts using different inventories:
  * `ansible -i inventory.ini -m ping all`
  * `ansible -i inventory_simple.ini -m ping all`

When connecting for the first time ssh will ask you to confirm host key when connecting via Ansible.
3. Try restarting apache2 service on node-2 using ansible
  * `ansible -i inventory.ini -m command -a 'service apache2 restart' -b node-2.ansible.local`