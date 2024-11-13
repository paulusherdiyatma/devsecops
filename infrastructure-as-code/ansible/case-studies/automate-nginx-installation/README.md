# Automating Nginx Installation and Configuration

## Problem
A company needs to deploy Nginx as a web server across multiple virtual machines (VMs) in different environments (development, staging, production). The goal is to automate the installation and configuration of Nginx, including setting up a custom configuration file for each environment.

## Outcome
- `Patch Management`: The playbook ensures that all systems in the inventory are patched automatically without any manual intervention.
- `Reusability`: The playbook is reusable for different environments and can be customized to install or upgrade specific packages.

## Solution
- `Inventory`: We create an inventory file that defines the hosts for different environments.  
  **inventory.ini**:
  ```
  [dev]
  dev1 ansible_host=192.168.1.101 ansible_user=ubuntu
  dev2 ansible_host=192.168.1.102 ansible_user=ubuntu

  [prod]
  prod1 ansible_host=192.168.1.201 ansible_user=ubuntu
  prod2 ansible_host=192.168.1.202 ansible_user=ubuntu
  ```
- `Playbook`: The playbook will install Nginx and configure it with different settings for development and production environments.  
  **nginx_install.yml**:
  ```
  - name: Install and configure Nginx
    hosts: all
    become: yes
    vars:
      nginx_config_file: "/etc/nginx/sites-available/default"

    tasks:
      - name: Install Nginx
        apt:
          name: nginx
          state: present
        notify:
          - restart nginx

      - name: Copy custom Nginx configuration
        template:
          src: "nginx_{{ ansible_hostname }}.conf.j2"
          dest: "{{ nginx_config_file }}"

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
  ```
- `Templates`: We use a Jinja2 template to customize the Nginx configuration based on the host.  
  **nginx_dev.conf.j2** (Development Environment):
    ```
    server {
        listen 80;
        server_name {{ ansible_hostname }};
        root /var/www/html;
    }
    ```
  **nginx_prod.conf.j2** (Production Environment):
  ```
  server {
    listen 80;
    server_name {{ ansible_hostname }};
    root /var/www/html;
    access_log /var/log/nginx/{{ ansible_hostname }}_access.log;
    error_log /var/log/nginx/{{ ansible_hostname }}_error.log;
  }
  ```
- Running the Playbook
  - **Ensure SSH Access**: Ensure that your private key (`~/.ssh/id_rsa`) is accessible and the public key is added to the authorized_keys on each of the remote VMs.
  - **Run the Playbook**: Use the following command to run the playbook with the inventory file:
    ```
    ansible-playbook -i inventory.ini nginx_install.yml
    ```
    This command will:
    - Install  `Nginx` on all the servers listed in the `inventory.ini` file.
    - Apply the respective `Nginx` configuration template based on the hostname of each machine.
    - Restart the `Nginx` service to apply the new configuration.
- Verifying the Setup  
  After the playbook completes, you can verify the following:
  - **Nginx Installation**: On each VM, check that Nginx is installed and running:
    ```
    systemctl status nginx
    ```
  - **Configuration Applied**: Ensure that the correct Nginx configuration file is applied by checking the file:
    ```
    cat /etc/nginx/sites-available/default
    ```
  - **Nginx Logs**: If you're in the production environment, you should also see logs being generated based on the configured access_log and error_log in `/var/log/nginx/`.

  - **Test in Browser**: Visit the IP address of the remote VMs in a browser (e.g., `http://192.168.1.101` for dev1). You should see the default Nginx page or your custom content based on the root configuration.