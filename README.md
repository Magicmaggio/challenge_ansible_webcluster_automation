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
inventory = ./inventory.yml
host_key_checking = False
retry_files_enabled = False
```
## Step 5: Set Up Ansible Inventory

1. Create the inventory folder and file:
```
touch inventory.yml
```
2. Add your servers to ``inventory.yml``:
```
---
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.230
          ansible_user: root
        web2:
          ansible_host: 192.168.1.231
          ansible_user: root

    loadbalancer:
      hosts:
        lb1:
          ansible_host: 192.168.1.232
          ansible_user: root

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


