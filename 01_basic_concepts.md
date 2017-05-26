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
  - and this how nesting works
```

Above file show almost everything one need to know about YAML to use it with Ansible. The missing part is knowing that YAML
is extending JSON syntax, thus it can use also JSON way of writing documents:

```yaml
[ list member 1, list member 2, 
  {list member 3: being a hash, second key: and it's value, third key: [ is another list, and this how nesting works]
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
* every play is dictionary having following possible keys:
  * **hosts**: list of hosts in scope of this play (**mandatory**)
  * **tasks**:  list of tasks to be executed on hosts (optional)
  * **vars**: dictionary of variables defined in scope of this play. This is not best way of defining variables! (optional)
  * **roles**: list of roles to be executed on hosts. Role is kind of grouped together tasks in Ansible. (optional)
  * **pre_tasks**, **post_tasks**: lists of tasks to be executed before or after roles. (optional)
* every task is dictionary describing what should be done.
  
  
