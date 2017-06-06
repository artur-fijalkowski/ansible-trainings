Provided box file has already following features on board:
* installed Apache 2 and PHP 7.0
* installed Ansible in virtualenv (on controller VM)
* provisioned /etc/hosts file for all hosts

## Enabling some PHP welcome page

This excercise is to practice basic usage of ansible. Running some tasks using following modules:
* `command` (running remote command)
* `file` (manipulating paths)
* `debug` (reviewing variables / registered variables)
* `template` (using Jinja2 templates)
* `service` (managing services)

Some basic techniques are showed:
* using `become` to run module as root
* using `register` to register module output
* using `changed_when` to mark command module as `changed state` based on command output
* using `when` for conditional execution
* passing variable from command line of ansible

Result of running following command:
* `ansible-playbook -i inventory.ini playbooks/set_php_script.yml`

Should be replaced homepage on all 3 servers (check on your web browser).
You can also provide custom message from commandline:

* `ansible-playbook -i inventory.ini playbooks/set_php_script.yml -e 'custom_message=Hello'`

Also check if anything chages when alternative `inventory_simple.ini` is used.

## Configuring Apache proxy with load-balancer

This excercise shows some additional modules:
* `set_fact` (define variable)
* `apache2_module` (managing modules enabled in Apache2)
* `lineinfile` (adding / removing lines in config files)
* `uri` (interacting with URLs)

Techniques introduced in this excercise:
* using `with_items` loop
* using loop inside template (`balancer.conf.j2`)

Following command can be used:

* `ansible-playbook -i inventory.ini playbooks/set_proxy_balancer.yml`

If this playbook completes successhully following URL should be load-balancer: [test] (http://localhost/test)

## re-writing proxy playbook into role

Roles allow better organization of roles.

This excercise is to convert playbook into role using following information:

* `roles/apache_proxy/handlers/main.yml` should be handler file (we want to restart apache **only** when needed)
* `roles/apache_proxy/defaults/main.yml` should contain default values for variables
* `roles/apache_proxy/tasks/main.yml` should be file with tasks
* `roles/apache_proxy/templates/` is directory for templates to use
