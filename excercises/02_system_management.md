## Creating new LVM disk

With this excercise new LVM VG/ LVM LV will be created, formatted and mounted as /var/tmp

Following modules are used:
* `lvg` - management of LVM VolumeGroups (creating VGs, adding PVs, etc.)
* `lvol` - Logical Volume management
* `filesystem` - formatting disks
* `file` - disk path modification
* `mount` - mount points management

Run following command
* `ansible-playbook -i inventory.ini playbooks/do_lvm.yml`

## Creating new user with groups and password

Do yourself a playbook using following modules:
* `group` - (http://docs.ansible.com/ansible/group_module.html)[groups management]
* `user` - (http://docs.ansible.com/ansible/user_module.html)[users management]
