# Infrastructure Overview

**Control Machine**: Windows 11 with WSL installed // IP: 192.168.1.68

**Web Server 1** (web1): Debian 12 // IP: 192.168.1.230

**Web Server 2** (web2): Debian 12 // IP: 192.168.1.231

**Load Balancer** (lb1): Debian 12 // IP: 192.168.1.232


# SSH Setup: Connecting from Windows 11 (WSL) to Debian 12 Servers

This guide explains how to set up a secure SSH connection from your Windows 11 machine (with WSL) to your Debian 12 servers.

## Step 1: Install OpenSSH Client on WSL

1. Open WSL (Ubuntu or any other distribution).

2. Verify SSH installation:

``ssh -V``

3. If not installed, run:

``sudo apt update && sudo apt install -y openssh-client``

## Step 2: Generate SSH Keys

1. Generate an SSH key pair:

``ssh-keygen -t rsa -b 4096 -C "your_email@example.com"``

2. Press Enter to accept the default file location (/home/your_user/.ssh/id_rsa). Or name it, up to you.

3. Enter a passphrase (optional but recommended) or leave it empty.

4. Verify the generated keys:

``ls ~/.ssh``

You should see (or the name you gave):

 - id_rsa (private key)
 - id_rsa.pub (public key)

## Step 3: Configure SSH Access on Debian Servers

**3.1** Install OpenSSH Server (if not already installed)

Run this command on each Debian server (web1, web2, lb1):

``sudo apt update && sudo apt install -y openssh-server``

**3.2** Enable and Start SSH Service
```
sudo systemctl enable ssh
sudo systemctl start ssh
```
**3.3** Allow SSH in Firewall (if UFW is enabled)
```
sudo ufw allow ssh
sudo ufw enable
sudo ufw status
```
## Step 4: Copy SSH Public Key to Debian Servers

**4.1** Use ssh-copy-id

Run these commands from your WSL terminal to copy your public key to each server:
```
ssh-copy-id root@192.168.1.230  # web1
ssh-copy-id root@192.168.1.231  # web2
ssh-copy-id root@192.168.1.232  # lb1
```
Note: If ssh-copy-id is not available, install it:

``sudo apt install -y sshpass``

**4.2** Manual Alternative (if needed)

1. Display your public key:

``cat ~/.ssh/id_rsa.pub``

2. On each Debian server, append the public key:
```
echo "<your-public-key>" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

## Step 5: Test SSH Connection

1. Connect to web1:

``ssh root@192.168.1.230``

2. Connect to web2:

``ssh root@192.168.1.231``

3. Connect to lb1:

``ssh root@192.168.1.232``

If successful, you should be logged into each server without entering a password.

## Troubleshooting Tips

 - Permission Denied:

     - Verify file permissions: ``chmod 700 ~/.ssh and chmod 600 ~/.ssh/authorized_keys.``
     - Restart SSH: ``sudo systemctl restart ssh.``

 - SSH Service Not Running:

     - Check status: ``sudo systemctl status ssh``
     - Start if inactive: ``sudo systemctl start ssh``

 - Firewall Blocking SSH:

 - Check UFW rules: ``sudo ufw status``
 - Allow SSH: ``sudo ufw allow ssh``



# Ansible Installation Guide on WSL (with Virtual Environment)

This guide explains how to install Ansible on your Windows 11 machine using WSL, with a Python virtual environment for better package management.

## Step 1: Update WSL and Install Dependencies

1. Open your WSL terminal.

2. Update the package list and install dependencies:
```
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3 python3-pip python3-venv git sshpass
```

## Step 2: Set Up Python Virtual Environment

1. Create a directory for Ansible projects:
```
mkdir -p ~/ansible-webcluster
cd ~/ansible-webcluster
```
2. Create a Python virtual environment:

``python3 -m venv venv``

3. Activate the virtual environment:

``source venv/bin/activate``

Note: When the environment is active, (venv) appears at the start of your terminal prompt.

4. Upgrade pip:

``pip install --upgrade pip``

## Step 3: Install Ansible in Virtual Environment

1. Install Ansible:

``pip install ansible``

2. Verify the installation:

``ansible --version``

You should see output like:
```
ansible [core X.X.X]
  python version = 3.X.X
```
## Step 4: Configure Ansible

 - Create the Ansible configuration file:
``touch ansible.cfg``

 - Add the following configuration to ansible.cfg:
```
[defaults]
inventory = inventory/inventory.yml
roles_path = roles/
host_key_checking = False
retry_files_enabled = False
```
## Step 5: Set Up Ansible Inventory

1. Create the inventory folder and file:
```
mkdir -p inventory/ && touch inventory/inventory.yml
```
2. Add servers to ``inventory.yml``:
```
---
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.230
          ansible_user: root
          ansible_ssh_private_key_file: /home/maggio/.ssh/challenge_ansible_webcluster_ssh_key
        web2:
          ansible_host: 192.168.1.231
          ansible_user: root
          ansible_ssh_private_key_file: /home/maggio/.ssh/challenge_ansible_webcluster_ssh_key

    loadbalancer:
      hosts:
        lb1:
          ansible_host: 192.168.1.232
          ansible_user: root
          ansible_ssh_private_key_file: /home/maggio/.ssh/challenge_ansible_webcluster_ssh_key

  vars:
    ansible_python_interpreter: /usr/bin/python3
```
3. Test the connection to all servers:
``ansible -i inventory.yml all -m ping``
You should see SUCCESS messages from each server.

## Step 6: Managing the Virtual Environment

Activate the virtual environment:
``source ~/ansible-webcluster/venv/bin/activate``

Deactivate the virtual environment:
``deactivate``

## Troubleshooting

 - Issue: ansible: command not found
     - Fix: Ensure the virtual environment is activated: source venv/bin/activate

 - Issue: SSH permissions error
     - Fix: Set proper permissions on your SSH keys:
```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```


# What is what ?

📂 inventory/

Contains Ansible inventory files that list target servers and machine groups.

    hosts.ini → Defines host groups (webservers, loadbalancer) and IP addresses.
    📌 Purpose: Informs Ansible which machines to configure.

📂 group_vars/

Stores global and group-specific variables for servers.

    all.yml → Variables common to all machines.
    webservers.yml → Variables specific to Nginx servers.
    loadbalancer.yml → Variables specific to the HAProxy server.

📌 Purpose: Centralize configuration with variables to simplify management.

📂 roles/

Organizes Ansible roles to structure tasks in a reusable way.

    nginx/ → Role for installing, configuring Nginx, and deploying the test web page.
    haproxy/ → Role for installing and configuring HAProxy.

📌 Purpose: Organize code into reusable blocks based on functionality.

📁 Role Structure

Each role follows a standard structure:

    tasks/ → Actions to execute (installation, configuration).
    templates/ → Template files (.j2) customized with variables.
    handlers/ → Actions triggered upon changes (e.g., restarting a service).

📂 playbooks/

Contains playbooks that orchestrate roles and tasks.

    webservers.yml → Deploys and configures Nginx on web servers.
    loadbalancer.yml → Configures HAProxy on the load balancer server.
    site.yml → Global playbook that runs both of the above.

📌 Purpose: Automate the full deployment of the infrastructure.


## Documentation: Self-Signed SSL Certificate Implementation  

**Objective**
The goal is to enable HTTPS on the Nginx servers by generating and configuring a self-signed SSL certificate. This ensures encrypted communication between clients and the web server.  

Steps Performed

1. Create SSL Directory
 - Task: Created a directory at /etc/nginx/ssl to store the SSL certificate and key.
 - Command Used: ansible.builtin.file
 - Details:
   - Owner: root
   - Permissions: 0755 (readable and executable by all, writable by owner).
 - Purpose: Organizes SSL files in a dedicated directory.  

```
- name: Create SSL directory
  ansible.builtin.file:
    path: /etc/nginx/ssl
    state: directory
    owner: root
    group: root
    mode: '0755'
```

1. Generate SSL Key and Certificate
 - Task: Created a self-signed SSL certificate valid for 1 year (365 days).
 - Command Used: openssl via ansible.builtin.command
 - Key Details:
   - Algorithm: RSA 2048-bit
   - Certificate Subject: Common Name (CN) set to maggio.com.
   - Files Created:
     - Private Key: /etc/nginx/ssl/nginx-selfsigned.key
     - Certificate: /etc/nginx/ssl/nginx-selfsigned.crt
 - Purpose: Provides encryption for HTTPS without requiring a third-party certificate authority.
 - Idempotence: The task is guarded using the creates argument to avoid regenerating the certificate if it already exists.

```
- name: Generate SSL key and certificate
  ansible.builtin.command: >
    openssl req -new -newkey rsa:2048 -days 365 -nodes -x509
    -keyout /etc/nginx/ssl/nginx-selfsigned.key
    -out /etc/nginx/ssl/nginx-selfsigned.crt
    -subj "/CN=maggio.com"
  args:
    creates: /etc/nginx/ssl/nginx-selfsigned.crt
```

3. Configure Nginx for SSL

 - Task: Updated the Nginx configuration to serve HTTPS traffic using the self-signed certificate.
 - Command Used: ansible.builtin.template
 - Template:
   - Path: /etc/nginx/conf.d/ssl.conf
   - Owner: root
   - Permissions: 0644 (readable by all, writable by owner).
 - Key Configuration:
   - Listen on port 443 for HTTPS traffic.
   - Specify the paths to the certificate (nginx-selfsigned.crt) and key (nginx-selfsigned.key).
 - Purpose: Allows secure traffic on port 443 using the self-signed certificate.

```
- name: Configure SSL in Nginx
  ansible.builtin.template:
    src: nginx-ssl.conf.j2
    dest: /etc/nginx/conf.d/ssl.conf
    owner: root
    group: root
    mode: 0644
  notify:
    - Reload Nginx
```

4. Nginx SSL Server Block

 - Configuration:
   - Port 443 is configured to use SSL.
   - The server_name is set to _, allowing it to handle requests for all domains or IPs.
   - The SSL certificate and private key paths are specified.
   - The default root directory (/var/www/html) and index file (index.html) are retained for serving content.  

```
server {
    listen 443 ssl;
    server_name _;

    ssl_certificate /etc/nginx/ssl/nginx-selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx-selfsigned.key;

    location / {
        root /var/www/html;
        index index.html;
    }
}
```

5. Service Notification

 - The Reload Nginx handler ensures that changes in the SSL configuration are applied without downtime.  

**Verification**

1. Ensure the playbook completes successfully.
2. Access the server via HTTPS: ``https://192.168.1.230``
3. Accept the browser's warning (due to the self-signed certificate).


## Documentation: Nginx Status Configuration

The following Nginx configuration block enables the Nginx stub status module, which provides basic statistics about the Nginx server, such as active connections, requests, and other performance metrics. This is useful for monitoring and debugging.  

**Configuration**:
```
server {
    listen 127.0.0.1:8080; # The IP and port where the status page is accessible
    server_name localhost; # The server name, typically for internal use

    location /nginx_status {
        stub_status on; # Activates the stub status module to provide metrics
        # Optional access control (currently commented out):
        # allow 127.0.0.1; # Allow only localhost to access the status
        # deny all; # Deny access to all other IPs
    }
}
```

**Explanation**:
```
listen 127.0.0.1:8080;
The server listens on the localhost IP address (127.0.0.1) and port 8080. This ensures the status page is nopublicly accessible and is limited to the local machine by default.

server_name localhost;
Specifies the server name for this configuration. This can be customized but is typically set as localhost fointernal access.

location /nginx_status {}
Defines the path /nginx_status to serve the Nginx status page.

stub_status on;
Activates the stub status module to expose basic metrics, such as:
    Active connections
    Total handled requests
    Current reading, writing, and waiting states of connections

Access Control (optional):
    The allow directive restricts access to specific IPs (e.g., 127.0.0.1 for localhost).
    The deny all; directive blocks all other IP addresses.
```
**Accessing the Status Page**:

    The status page can be accessed from the local machine using:
``curl http://127.0.0.1:80/nginx_status``  

**Use Case**:
This configuration is primarily used for monitoring and diagnosing Nginx performance and activity. It is recommended to restrict access using allow and deny directives for security.


## Documentation: Configuration Backup Implementation
**Objective**

The aim is to ensure that the Nginx configuration files are regularly backed up to a dedicated directory for recovery purposes in case of misconfiguration or data loss.  

**Steps Performed**

1. Create Backup Directory
 - Task: Created a directory at /backup/nginx to store Nginx configuration backups.
 - Command Used: ansible.builtin.file
 - Details:
   - Owner: root
   - Permissions: 0755 (readable and executable by all, writable by the owner).
 - Purpose: Provides a secure and centralized location for configuration backups.  

```
- name: Create backup directory
  ansible.builtin.file:
    path: /backup/nginx
    state: directory
    owner: root
    group: root
    mode: 0755
```

2. Backup Nginx Configuration

 - Task: Copied the contents of the /etc/nginx/ directory to /backup/nginx/.
 - Command Used: ansible.builtin.copy
 - Details:
   - src: Specifies the source directory (/etc/nginx/) on the target machine.
   - dest: Defines the backup directory (/backup/nginx/).
   - remote_src: yes: Ensures the src path is on the remote host.
 - Purpose: Retains a copy of the current Nginx configuration for rollback or analysis.  

```
- name: Backup Nginx configuration
  ansible.builtin.copy:
    src: /etc/nginx/
    dest: /backup/nginx/
    remote_src: yes
```

3. Role Task File: roles/nginx/tasks/backup.yml

 - The above tasks are grouped into a dedicated task file named backup.yml in the nginx role. This allows modular and reusable backup management specific to the Nginx role.

Content of ``backup.yml``:
```
- name: Create backup directory
  ansible.builtin.file:
    path: /backup/nginx
    state: directory
    owner: root
    group: root
    mode: 0755

- name: Backup Nginx configuration
  ansible.builtin.copy:
    src: /etc/nginx/
    dest: /backup/nginx/
    remote_src: yes
```

4. Integrate the Backup Role

 - Playbook: Updated playbooks/webservers.yml to include the backup.yml tasks using the include_role directive.
 - Details:
   - Hosts: webservers group is targeted.
   - Role: nginx role is invoked with the backup.yml task file to execute the backup-related tasks.

Content of ``webservers.yml``:
```
---
- name: Set up Web Servers
  hosts: webservers
  become: true

  roles:
    - nginx

  tasks:
    - include_role:
        name: nginx
        tasks_from: backup.yml
```
