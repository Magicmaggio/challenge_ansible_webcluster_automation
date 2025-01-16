# SSH Key Configuration with WSL and Debian Servers

## Objective

Establish secure SSH connections from a Windows 11 machine using WSL (Windows Subsystem for Linux) to three Debian 12 servers:

 - Machine de contrôle (WSL): Windows 11, IP: 192.168.1.68
 - Web Server 1: Debian 12, IP: 192.168.1.230
 - Web Server 2: Debian 12, IP: 192.168.1.231
 - Load Balancer: Debian 12, IP: 192.168.1.232

## Problem Encountered with WSL

When generating and using SSH keys stored in the Windows file system (/mnt/c/...), SSH reported permission errors:
```
WARNING: UNPROTECTED PRIVATE KEY FILE!
Permissions 0777 for '...ssh_key' are too open.
This private key will be ignored.
```

**Root Cause**

 - WSL mounts the Windows file system with permissions that are too permissive for SSH (rwxrwxrwx).
 - SSH requires the private key to have strict permissions (read/write only for the owner).

## Correct Procedure

1. Generate an SSH Key Pair

``ssh-keygen -t rsa -b 4096 -f ~/.ssh/challenge_ansible_webcluster_ssh_key``
 - -t rsa: RSA encryption.
 - -b 4096: 4096-bit key for better security.
 - -f: Specifies the file name for the key pair.

2. Set Correct Permissions
```
chmod 600 ~/.ssh/challenge_ansible_webcluster_ssh_key
chmod 644 ~/.ssh/challenge_ansible_webcluster_ssh_key.pub
```
 - 600: Read/write for the user only (private key).
 - 644: Readable by all, writable only by the user (public key).

3. Copy the Public Key to Debian Servers
```
ssh-copy-id -i ~/.ssh/challenge_ansible_webcluster_ssh_key.pub root@192.168.1.230
ssh-copy-id -i ~/.ssh/challenge_ansible_webcluster_ssh_key.pub root@192.168.1.231
ssh-copy-id -i ~/.ssh/challenge_ansible_webcluster_ssh_key.pub root@192.168.1.232
```

Sometimes :
```
# We have to put it manually on the remote server
root@web2:~# mkdir -p ~/.ssh/
root@web2:~# echo "<public_key>" >> ~/.ssh/authorized_keys
root@web2:~# chmod 600 ~/.ssh/authorized_keys
root@web2:~# chmod 700 ~/.ssh
```
1. Connect via SSH

``ssh -i ~/.ssh/challenge_ansible_webcluster_ssh_key root@192.168.1.230``

5. Alternative: Use SSH Agent (Optional)

Start the SSH agent and add the key:
```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/challenge_ansible_webcluster_ssh_key
```

## Key Takeaways

 - Always store SSH keys in the WSL native Linux file system (~/.ssh) to avoid permission issues.
 - Never use keys from /mnt/c/... because Windows' file system permissions conflict with SSH security requirements.
 - Set the right permissions using chmod to prevent SSH from rejecting the keys.

## Troubleshooting

 - If SSH still complains about permissions, verify with:
``ls -l ~/.ssh/``

 - Check SSH agent status:
``ssh-add -l``

## Conclusion

Following these steps ensures secure and seamless SSH connections from WSL to Debian servers, avoiding common pitfalls related to cross-platform file permissions.