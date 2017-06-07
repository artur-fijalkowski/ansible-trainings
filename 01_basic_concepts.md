## Playbook's anatomy

Ansible can be used to write oneliners like in other languages, but 99% percent of usage is related to playbooks.
Playbook for Ansible is like script for drama theater - it describes persons involved (hosts) and what they do (tasks).

For ease of reading (and writing!) playbooks use YAML syntax which looks as follows:

```yaml
- list member 1
- list member 2
- list member 3: being a hash
  second key: and it's value
  third key:
  - is another list
  - and this is how nesting works
```

Above file show almost everything one need to know about YAML to use it with Ansible. The missing part is knowing that YAML
is extending JSON syntax, thus it can use also JSON way of writing documents:

```yaml
[ list member 1, list member 2, 
  {list member 3: being a hash, second key: and it's value, third key: 
      [ is another list, and this is how nesting works]
  }
]
```

Important to remember is that compared to JSON **quoting is fully optional**. Because of this fact following characters: **[:{** have 
special meaning and must be quoted accordingly.

Many programming functionalities are provided in Ansible using Jinja2 templating system. Using Jinja2 needs proper habit for quoting:

```yaml
- name: create user in system
  user:
    name: "{{ user_name }}"
    group: "{{ group_name }}"
    groups: games, video
```

In above example "{{ user_name }}" will evaluate into variable value without any additional whitespaces, etc. This means multiple in-place
evaluations can be used for concatenating variables - "{{ user_prefix }}{{ user_infix }}{{ user_suffix }}".

Above example also shows 2 good coding practice rules:
1. **Use quotes only when it's mandatory**
2. **Always use one-paramater-per-line for actions**

Above rules can be motivated by following:
1. If quoting is not abused " is immediately meaning there is some Jinja2 templating in (maybe long) line of code. This improves
readability greatly.
2. Above example can be rewritten in form below, which is maybe more compact, but not that good to compare when using diff. Please also
note that using form with equal sign doesn't need additional quoting of Jinja2. But this is **false simplification**

```yaml
- name: create user in system
  user: name={{ user_name }} group={{ group_name }} groups: "games, video"
```

Now we can analize simple playbook:

```yaml
- hosts: all
  vars:
    packages:
    - vim
    - curl
  tasks:
    apt:
      name: "{{ packages }}"
      state: present
    
- hosts: web
  vars:
    web_packages:
    - apache2
    - php
  tasks:
    apt:
      name: "{{ packages }}"
      state: present
```

From already mentioned information we can read above playbook as:
* list of plays (like acts in drama)
* every play is dictionary having following possible keys (not all are listed!):
  * **hosts**: list of hosts in scope of this play. Hosts specification can be list of hostnames, hostgroups, or mix of those two. There is always automatic *all* hostgroup populated with all hosts from inventory. (**mandatory**)
  * **name**: name of the play. (optional).
  * **tasks**:  list of tasks to be executed on hosts (optional)
  * **vars**: dictionary of variables defined in scope of this play. This is not best way of defining variables! (optional)
  * **roles**: list of roles to be executed on hosts. Role is kind of grouped together tasks in Ansible. (optional)
  * **pre_tasks**, **post_tasks**: lists of tasks to be executed before or after roles. (optional)
* every task is dictionary describing what should be done. There are dozens of possible keys for tasks:
  * **action**: name of the action to perform (module name). This is usually abbreviated by the name of the module itself. (**mandatory**)
  * **name**: human-readable name of the task. (optional)
  * **become**, **become_user**: control of sudo action for module. (optional)
  * **register**: save module output into variable. (optional)
  * **with\_\***: multiple ways of looping. (optional)
  * **when**: conditional execution. (optional)
  * **delegate_to**: execute task on different host then current actor. (optional)
  * many-many more
* every module has it's own list of parameters supported. Some parameters are mandatory and some are optional (in most cases they have default value which is most commonly used).

## Syntax limitations
There are multiple limitations in Ansible that come directly from YAML syntax. Most important thing is that **task is decribed by dictionary**. This implicates multiple things:
* only one module can be called in one task (module name is key of dictionary).
* only one set of parameters can be passed to module in task (can be mitigated by with_\* loops).
* modules like *set_fact* process variables in ''random'' order (one variable cannot depend on another variable defined in same set_fact).
* processing of registered variable from looped module can be challenging.
* Jinja2 code in when clause can be really complex.

## Inventory

Even more important than having a playbook is having an inventory. In simpliest approach inventory is INI file containing definitions of hosts we can connect to.

```ini
# contents of inventory.ini

[web]
web1 ansible_host=10.0.0.11
web2 ansible_host=10.0.0.12

[web:vars]
webport=80
```

In above example stanza *[web]* defines group of hosts, while *[web:vars]* assignes some variables to hosts from this group.
*web1* and *web2* are hostnames. If no *ansible_host* variable is assigned those hostnames are used for resolving address. There are some important variables that can be assigned to host that control how Ansible connects to this host:
* **ansible_host**: defines how to connect to a host (dns name or IP can be assigned)
* **ansible_user**: user to be used when connecting to host
* **ansible_port**: TCP port used when connecting to host
* **ansible_connection**: which protocol to use. Most frequent values are *ssh* and *local* - later onaecan be used when connecting to localhost.

All variables defined in inventory are available in host scope.
[documentation](http://docs.ansible.com/ansible/intro_inventory.html)

If there is a lot of (default) values for variables they can be put into separate files in special directories: *group_vars/groupname* and *host_vars/hostname*. Those two directories are expected to be at same path as inventory itself. Files with group/host variables are YAML without extension, top level variables are members of dict at top-level of YAML file:

```yaml
# content of group_vars/web

web_port: 80
web_port_ssl: 443
web_vhosts:
- www.example.com
- example.io
```

## Parallelism
Ansible code is executed parallel out of the box. Default number of hosts executed at the same time is equal to number of CPUs in controller. This can be tuned both on configuration and play level (**serial** play parameter). Play is executed parallel task-by-task which means one task must be completed on all hosts before entering next. Also all variables available are **local to host context**. This means if there are eg. two hosts in play and both hosts modify variable it's not visible directly to each other, there is special way for this kind of access *hostvars['hostname']['variable_name']*. This can lead to common mistakes:

```yaml

- hosts: localhost
  vars:
    important_var: true

- hosts: web
  tasks:
  - debug: msg="important_var has value of: {{ important_var }}"
  - debug: msg="improtant_var has value of: {{ hostvars.localhost.important_var }}"
```

As one can guess in first debug *important_var* is not defined, because it's defined **only** in scope of localhost. This may be misleading at the beginning of work with Ansible.

## Error handling
By default Ansible executes on particular host until first error. Then play continues skipping the rest for failed host.
There is no complex error / exception handling at all. There is also no complex branching of Ansible execution flow (it's hard to execute parts of Ansible code conditionaly).

This should be mitigated by the most important feature of Ansible - idempotency. Re-running properly written play should not modify anything if previous pass completed succesfully (in other words: every task or group of tasks should not change state of system if final
state was already achieved). Then if one host failed one can safely re-run playbook to fix problems caused by intermittent problems.
