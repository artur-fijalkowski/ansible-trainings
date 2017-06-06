Provided box file has already following features on board:
* installed Apache 2 and PHP 7.0
* installed Ansible in virtualenv (on controller VM)
* provisioned /etc/hosts file for all hosts

## Enabling some PHP welcome page

This excercise is to practice basic usage of ansible. Running some tasks using following modules:
* command (running remote command)
* file (manipulating paths)
* debug (reviewing variables / registered variables)
* template (using Jinja2 templates)
* service (managing services)

Result of running following command:
* `ansible-playbook -i inventory.ini playbooks/set_php_script.yml`

Should be replaced homepage on all 3 servers (check on your web browser).
You can also provide custom message from commandline:

* `ansible-playbook -i inventory.ini playbooks/set_php_script.yml -e 'custom_message=Hello'`
