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