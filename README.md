Install VirtualBox and Vagrant on your Linux machine.

Create a new directory for your Vagrant project, e.g. ansible-windows-lab, and navigate into it.

Create a new Vagrantfile in the directory with the following contents:

```
Vagrant.configure("2") do |config|
  config.vm.box = "mwrock/Windows2019"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "windows.yml"
    ansible.extra_vars = {
      ansible_user: "vagrant",
      ansible_password: "vagrant",
      ansible_port: "5986",
      ansible_connection: "winrm",
      ansible_winrm_transport: "ntlm",
      ansible_winrm_server_cert_validation: "ignore"
    }
  end
end
```

This Vagrantfile will create a new Windows Server 2019 virtual machine and configure it for Ansible using the ansible_local provisioner.

Create a new Ansible playbook in the same directory called windows.yml, with the following contents:


```
- hosts: all
  vars:
    ansible_python_interpreter: /usr/bin/python3
  tasks:
    - name: Enable WinRM
      win_reboot:
        reboot_timeout: 3600
        test_command: dir C:\
      become: true
      become_method: runas

    - name: Install PowerShell 7
      win_chocolatey:
        name: powershell-core
        state: present
        version: 7.0.3
      become: true
      become_method: runas

    - name: Set PowerShell Execution Policy
      win_shell: Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine -Force
      become: true
      become_method: runas

    - name: Install WinRM HTTPS Certificate
      win_certificate_store:
        path: "{{ playbook_dir }}/winrm.crt"
        store_location: LocalMachine
        store_name: Root
      become: true
      become_method: runas

    - name: Import WinRM HTTPS Certificate
      win_shell: Import-Certificate -FilePath "{{ playbook_dir }}/winrm.crt" -CertStoreLocation Cert:\LocalMachine\Root -Verbose
      become: true
      become_method: runas
```
This playbook will enable WinRM, install PowerShell 7, set the PowerShell execution policy, and configure WinRM HTTPS with a self-signed certificate.

Create a new self-signed certificate for WinRM HTTPS by running the following commands on your Linux control node:

```
$ openssl req -newkey rsa:2048 -nodes -keyout winrm.key -x509 -days 365 -out winrm.crt 
$ cat winrm.key winrm.crt > winrm.pem
```

These commands will create a private key and a self-signed certificate for WinRM HTTPS, and combine them into a single PEM file.

Copy the winrm.pem file to the ansible-windows-lab directory on your Linux control node.

Start the Vagrant virtual machine by running vagrant up in the ansible-windows-lab directory.

Once the virtual machine is running, you can connect to it with Ansible by adding the following to your inventory file:

```
[windows]
192.168.33.10 ansible_user=vagrant ansible_password=vagrant ansible_port=5986 ansible_connection=winrm ansible_winrm
```
