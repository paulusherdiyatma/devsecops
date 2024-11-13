# Ansible

## Table of Contents
- [Prerequisite](#prerequisite)
- [Requirements](#requirements)
- [Installation](#installation)
- [SSH Configuration](#ssh-configuration)
- [Key Concepts](#key-concepts)

## Prerequisite
-  Python 3.12

## Requirements
- `Control Node`: A machine where Ansible is installed (can be your local machine).
- `Managed Nodes`: Remote hosts that Ansible manages; they should be accessible over SSH.

## Installation
- execute below commands to install `Python 3.12`:
    ```
    sudo apt update
    sudo apt install -y software-properties-common
    sudo add-apt-repository ppa:deadsnakes/ppa
    sudo apt update
    sudo apt install -y python3.12 python3.12-venv python3.12-distutils 
    ```
- Verify the `Python 3.12` installation:
    ```
    python3.12 --version
    ```
- Create a Virtual Environment with `Python 3.12`
    ```
    python3.12 -m venv ansible-env
    source ansible-env/bin/activate
    ```
- Upgrade Pip and Install `Ansible`
    ```
    pip install --upgrade pip
    pip install ansible
    ```
- To confirm that `Ansible` is using `Python 3.12`, check Ansible’s version
    ```
    ansible --version
    ```

## SSH Configuration
Ansible communicates with remote hosts over SSH. For seamless automation, it’s best to configure passwordless SSH access between your control node (the machine where Ansible is installed) and the managed nodes (the remote machines you want to configure).
- Generate SSH Keys on the Control Node  
  If you don’t already have an SSH key pair on your control node, you can generate one using the following command:
  ```
  ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa
  ```
- View the SSH Public Key  
  Once the key pair is generated, the public key will be saved in `~/.ssh/id_rsa.pub`. To view the contents of the public key, run:
  ```
  cat ~/.ssh/id_rsa.pub
  ```
- Copy the Public Key to the Remote Host Manually  
  You need to add this public key to the `~/.ssh/authorized_keys` file on the remote host.
  - SSH into the Remote Host
    ```
    ssh username@remote_host_ip
    ```
  - Create `.ssh` Directory (if it doesn't exist)  
    Once logged in, check if the `.ssh` directory exists in the home directory. If not, create it:
    ```
    mkdir -p ~/.ssh
    ```
  - Set the correct permissions for the `.ssh` directory  
    ```
    chmod 700 ~/.ssh
    ```
  - Add the Public Key to authorized_keys  
    Now, add the public key to the `~/.ssh/authorized_keys` file. You can do this by copying the public key from your control node and appending it to the authorized_keys file on the remote host.  
    On the remote host, open the `authorized_keys` file (it may not exist, in which case it will be created)
    ```
    nano ~/.ssh/authorized_keys
    ```
    On your local control node, copy the public key that you viewed earlier (it should be the entire line starting with `ssh-rsa`).  
    Paste the public key into the `authorized_keys` file.
  - Set Permissions for `authorized_keys` 
    Ensure the correct permissions for the `authorized_keys` file:
    ```
    chmod 600 ~/.ssh/authorized_keys
    ```
  - Test Passwordless SSH Access  
    Now you should be able to SSH into the remote host without being prompted for a password:
    ```
    ssh username@remote_host_ip
    ```

## Key Concepts

### Control Node  
The Control Node is the machine where Ansible is installed. This node is used to run Ansible commands and playbooks, and it manages other nodes (or hosts) in the environment. It does not need any agent installed on the managed nodes.

### Managed Nodes
Managed Nodes are the remote machines that Ansible configures. These machines are typically accessed over SSH (for Linux/Unix systems) or WinRM (for Windows systems). Ansible uses SSH for Linux/Unix hosts to execute tasks without requiring an agent.

### Inventory
An Inventory is a list of managed nodes (remote machines) in Ansible. It can be a simple text file (like `inventory.ini`) or dynamic (via scripts or cloud APIs). The inventory file defines which hosts or groups of hosts will be managed and provides connection details such as IP addresses and SSH user credentials.   

Example of an inventory file:
```
[my_vms]
vm1 ansible_host=192.168.1.100 ansible_user=ubuntu
vm2 ansible_host=192.168.1.101 ansible_user=ubuntu
```

### Playbooks
An `Ansible Playbook` is a file (written in YAML) that defines a series of tasks to be executed on the managed nodes. Playbooks are used to automate configuration management, application deployment, and task automation.  

A basic playbook example:
```
- name: Install Nginx on web servers
  hosts: webservers
  tasks:
    - name: Install nginx package
      apt:
        name: nginx
        state: present
```
In this example:
- `name`: Describes the purpose of the playbook or task.
- `hosts`: Specifies which hosts or group of hosts the playbook will run on.
- `tasks`: A list of tasks (commands or modules) that will be executed on the managed nodes.

### Tasks
A Task is a single unit of work in an Ansible playbook. Each task calls a module to perform an action on the managed node. Common tasks include installing packages, managing files, or starting services.  

Example of a task using the apt module to install a package:
```
- name: Install nginx
  apt:
    name: nginx
    state: present
```

### Modules
An Ansible Module is a small piece of code that is designed to perform a specific task, like installing software, copying files, or managing users. Ansible has hundreds of built-in modules, and you can also create your own custom modules.

For example, the `apt` module is used to manage packages on Debian-based systems (like Ubuntu), and the `service` module is used to manage system services.

### Roles
Roles are a way to organize playbook content into reusable components. A role may include tasks, variables, templates, files, and handlers. Roles allow you to reuse code across multiple playbooks and share configuration in a modular way.  

Example structure of a role:
```
my_role/
  tasks/
    main.yml
  templates/
    config.j2
  files/
    script.sh
  defaults/
    main.yml
```

### Handlers
A Handler is a special type of task in Ansible. Handlers are tasks that are only run when notified by another task. For example, if a file is changed, you can notify a handler to restart a service.  

Example of using a handler:
```
- name: Install nginx
  apt:
    name: nginx
    state: present
  notify:
    - restart nginx

handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
```

### Variables
Variables allow you to store values that can be reused in tasks and playbooks. They can be defined in the inventory, in the playbook itself, or in separate files (like `vars.yml`). Variables provide flexibility and make playbooks more dynamic.  

Example of defining variables in a playbook:
```
- name: Install Nginx on web servers
  hosts: webservers
  vars:
    nginx_package: nginx
  tasks:
    - name: Install nginx
      apt:
        name: "{{ nginx_package }}"
        state: present
```

### Facts
Facts are pieces of information about a managed node that are gathered automatically by Ansible at the start of a playbook run. These facts include details about the system, such as the operating system, IP addresses, and available memory. Facts are useful for making decisions during playbook execution.  

You can gather facts using the setup module:
```
- name: Gather system facts
  hosts: all
  tasks:
    - name: Gather facts
      setup:
```

### Ansible Galaxy
Ansible Galaxy is a repository for sharing roles and collections. It contains community-contributed Ansible roles and collections that can be used to automate common tasks, such as setting up web servers, databases, or security configurations.  
To install a role from Ansible Galaxy, you can use the following command:
```
ansible-galaxy install username.role_name
```

### Ansible Vault
Ansible Vault is a feature that allows you to encrypt sensitive data, such as passwords or API keys, within Ansible files. This helps protect sensitive information while still allowing it to be included in playbooks or inventory files.

To create an encrypted file:
```
ansible-vault create secrets.yml
```

To edit an encrypted file:
```
ansible-vault edit secrets.yml
```

### Ansible Tower
```
Ansible Tower is a web-based user interface (UI) and management tool for Ansible. It provides a graphical interface for managing playbooks, inventory, and schedules, making it easier to scale Ansible automation in larger environments.
```