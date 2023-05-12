## Configuring Windows Host to use (SSH)

1. Install OpenSSH for Windows: Download and install the OpenSSH for Windows package from the official website. Make sure to choose the correct architecture (32-bit or 64-bit) and version (latest stable release).

2. Generate SSH keys: Generate an SSH key pair on the control node (e.g. Linux) using the ssh-keygen command. This will create a public and private key. Copy the public key to the Windows host using a secure method (e.g. SCP).

3. Install the public key: Create a directory on the Windows host to store the authorized keys (e.g. C:\Users\ansibleuser\\.ssh). Then, copy the public key to this directory and rename it to authorized_keys. Make sure to set the correct permissions on the directory and file (read-only for the ansibleuser account).

4. Configure OpenSSH for Windows: Edit the sshd_config file (located in C:\ProgramData\ssh) to allow SSH connections from the control node. Set the following options:

```
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile  %h\.ssh\authorized_keys
```

Note: make sure to uncomment the AuthorizedKeysFile option if it is commented out.

5. Start the OpenSSH service: Start the OpenSSH service on the Windows host using the Services management console or the net start sshd command.

6. Test the connection: From the control node, use the ssh command to connect to the Windows host using the ansibleuser account and the private key. If everything is configured correctly, you should be able to log in without entering a password.

Once the Windows host is configured for SSH authentication, you can use it as a target in your Ansible playbooks by specifying the SSH connection parameters in the inventory file (e.g. ansible_host, ansible_user, private_key).
