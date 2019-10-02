# ansible-101-starter
Use this as a potential starter to help you experiment with Ansible


## Review the base folder structure

```sh
ansible-101-starter wanlessc$ tree
.
├── README.md
├── Vagrantfile
└── inventory
    ├── group_vars
    │   └── all.yaml
    └── hosts
```

* `./Vagrantfile` contains our vagrant code to build our local sandbox
* `./inventory/hosts` is your inventory file.  It also inherently pulls in everything from the `./inventory/group_vars` and `./inventory/host_vars` folder respectively when used.

## Start your sandbox

```sh
ansible-101-starter wanlessc$ vagrant up
```

## Test access to your sandbox

```sh
ansible-101-starter wanlessc$ ansible all -m ping -i inventory
db01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
db02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
web01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
web02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
``` 

## Play around with Ansible CLI

* Check out the fstab on all the boxes

```sh
ansible all -m shell -i inventory -a 'cat /etc/fstab | grep -v ^#'
```

## Build your first playbook

* Create a file `./pb_build_env.yml`
* Build the playbook to install and start apache

```yaml
---
- hosts: web
  gather_facts: false
  become: true
  tasks:
    - name: Install Apache
      package:
        name: httpd
        state: latest
    
    - name: Start Apache
      service:
        name: httpd
        state: started
```

* Execute the playbook with `ansible-playbook -i inventory pb_build_env.yml`

* Now let's extend the playbook by copying in our HTTP home page to the web server
    * Create the file `./templates/index.html.j2`
        ```j2
        <!DOCTYPE html>
        <html>
        <head>
            <title>Hello World!</title>
        </head>
        <body>
            <h1>Hello World!</h1>
            <p>This is running from server: {{ inventory_hostname }} on {{ ansible_host }}</p>
        </body>
        </html>
        ```
    * Now Add the following task to your playbook
        ```yaml
            - name: Add index file to www directory
              template:
                  src: index.html.j2
                  dest: /var/www/html/index.html
        ```


## Tips and Tricks

* `ansible.cfg` can be used to set a number of defaults. go check the docs for all, but here are a couple useful ones
  * inventory:
  ```
  [defaults]
  inventory = ./inventory
  ```