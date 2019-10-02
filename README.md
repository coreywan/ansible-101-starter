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

* As you can see, this is very easy to do. However, this type of task is something that people have been doing across all enterprises for years. Is there a platform to help get this done quicker?! yes, infact there is.  There is a concept called Ansible Roles. Roles are a combination of tasks roled into a nice package. You can also publicly make these accessible through the Ansible Galaxy Platform.  Let's go check it out for our DB servers.  https://galaxy.ansible.com/geerlingguy/mysql

  ```
  ansible-galaxy install robertdebock.mysql
  ```

  ```yaml
  - hosts: db
    become: true
    vars:
      mysql_bind_address: 0.0.0.0
      mysql_databases:
        - name: mydb
          encoding: utf8
          collation: utf8_bin
      mysql_users:
        - name: myuser
          password: password
          priv: "*.*:ALL"
          host: 192.168.10.1
    pre_tasks:
      - name: install python requirements
        package:
          name:
            - libselinux-python
            - MySQL-python 
    roles:
      - robertdebock.mysql
  ```

## Tips and Tricks

* verbosity - add -vv.. to add verbosity. the more `v` you add the more verbose it gets
* Check Mode - The ability to execute your playbook and safely check to see if any changes may happen, but not actually execute on the change.
* limit - You can limit the playbook run to a group or individual hosts. This is useful when you are just wanting to test a section of your playbook.
* start-at-task - you can skip to a specific task to start from.

* `ansible.cfg` can be used to set a number of defaults. go check the docs for all, but here are a couple useful ones
  * Get rid of the need to type in the inventory path every time
    ```
    [defaults]
    inventory = ./inventory
    ```
  * Don't like the output? Think YAML response is easier to read in the stdout?
    ```
    # Use the YAML callback plugin.
    stdout_callback = yaml
    # Use the stdout_callback when running ad-hoc commands.
    bin_ansible_callbacks = True
    ```
* ansible vault - Do you have secrets you want to keep, and want a secure way to save those in your inventory/code. Ansible Vault is your friend.  You will need to think of a good master password. That master password is used to decrypt / encrypt the password using aes256.
  * There are two ways to use ansible-vault. You can encrypt a whole file, or just an individual variable. There are pro's and cons to each. When you encrypt a single variable, you can see what changed in git messages. When, you encrypt a whole variable file, you can't tell what changed in git because the whole file has been encrypted. So it makes troubleshooting harder down the line.  With that said, encrypting files comes with an easy button when you need to rotate the master password. 
    ```sh
    ansible-101-starter wanlessc$ ansible-vault encrypt_string
    New Vault password: 
    Confirm New Vault password: 
    Reading plaintext input from stdin. (ctrl-d to end input)
    supersecret!vault |
            $ANSIBLE_VAULT;1.1;AES256
            66666637636662396639366537643334626330373136666564643736313461663163336335376337
            3133396437356534393637333537393531353162303930360a636638356637653663323433663637
            34663563363636633366323130366639653134376365666263346661626138656635346363393536
            3362626335376265650a386138393439393432356364343630383439363032353531393161363265
            3136
    Encryption successful
    ```


