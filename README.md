# Ansible Workshop

Ansible workshop for Centric DevOps &amp; Cloud Academy

## Information

You have 3 VM's provided:

- Ansible management node (Ubuntu 20.04)
- Linux target (Ubuntu 20.04)
- Windows target (Windows Server 2019)

### Environment details

xx - environment number

| | |
| --- | --- |
| Management node public DNS | centric-ansible-lab-mgmt-xx.westeurope.cloudapp.azure.com |
| Linux target public DNS | centric-ansible-lab-lin-xx.westeurope.cloudapp.azure.com |
| Windows target public DNS | centric-ansible-lab-win-xx.westeurope.cloudapp.azure.com |
| --- | --- |
| Management node hostname | lab-mgmt-xx |
| Linux target hostname | lab-lin-xx |
| Windows target hostname | lab-win-xx |

## Part 1 - Preparation

In this part we will be setting up Ansible on management node.

Connect to management node using SSH

```console
ssh user@centric-ansible-lab-mgmt-xx.westeurope.cloudapp.azure.com
```

Install Ansible

```console
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible python3-winrm
```

You can test if ansible was installed by running

```console
ansible --version
```

Generate SSH Key

```console
ssh-keygen
```

Copy SSH Public Key to Linux target

```console
ssh-copy-id user@lab-lin-xx
```

You can test if the public key was copied correctly by logging in without a password prompt

```console
ssh user@lab-lin-xx
```

## Part 2 - Linux target

### Step 1 - Create inventory and test connection

<https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html>

Create ansible inventory file with variables for Linux target.
Needed parameters for inventory:

- hostname
- admin username

You can test if the inventory and connections are working by running an ad-hoc ansible command

```console
ansible -i inventory_file.ini group_name -m ansible.builtin.ping
```

You should get an OK response before continuing

### Step 2 - Create Ansible playbook

<https://docs.ansible.com/ansible/latest/getting_started/get_started_playbook.html>

Create an Ansible playbook that will:

- Create groups from a variable
- Create users from a variable that are part of the previously created groups
- Install nginx via apt
- Enables and starts nginx service
- Creates an index.html using a template. Template must include a hostname from an ansible fact
- Creates a new nginx site
- Uses a handler to restart nginx service when configuration changes

### Step 3 - Create roles from playbook

<https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html>

From existing playbook create Ansible roles for

- User and group management
- Nginx installation
- Website deployment

### Help

<details>
<summary>Useful modules</summary>

- ansible.builtin.group
- ansible.builtin.user
- ansible.builtin.apt
- ansible.builtin.service
- ansible.builtin.file
- ansible.builtin.template

</details>

<details>
<summary>Inventory file</summary>

```ini
[linux]
lab-lin-xx

[linux:vars]
ansible_user=__REDACTED__
```

</details>

<details>
<summary>Launching ansible-playbook</summary>

```console
ansible-playbook -i inventory_file.ini playbook_file.yaml -v
```

</details>

<details>
<summary>NGINX site config</summary>

```nginx
server {
  listen 80;

  server_name {{ ansible_hostname }};
  location / {
    root {{ www_path }};
  }
}
```

</details>

<details>
<summary>Testing website</summary>
From management node run

```console
curl lab-lin-xx
```

You should get a response with templated text

</details>

## Part 3 - Windows target

### Step 1 - Configure WinRM and create Ansible inventory

<https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html>
<https://www.ansible.com/blog/connecting-to-a-windows-host>

Create ansible inventory file with variables for Windows target.
Can use the same inventory created previously, with a different group name.
Needed parameters for a Windows inventory:

- hostname
- admin username
- admin password
- connection type set to winrm
- winrm certificate validation disabled

Before we can connect to Windows machine, we need to enable WinRM.

- Connect to windows machine using RDP (Remote desktop connection)
- Start PowerShell ***as Administrator***
- Execute command:

```powershell
$url = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"

(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)

powershell.exe -ExecutionPolicy ByPass -File $file
```

You can test if the inventory and connections are working by running an ad-hoc ansible command

```console
ansible -i inventory_file.ini group_name -m ansible.windows.win_ping
```

You should get an OK response before continuing

### Step 2 - Create Ansible playbook

<https://docs.ansible.com/ansible/latest/getting_started/get_started_playbook.html>

Create an Ansible playbook that will:

- Creates groups from a variable
- Creates users from a variable that are part of the previously created groups
- Installs firefox via chocolatey
- Installs Web-Server (IIS) Windows feature with sub and management features
- Reboots if Web-Server feature was installed (changed)
- Removes Default Website (name: "Default Web Site")
- Creates an index.html using a template. Template must include a hostname from an ansible fact
- Create new website on port 80, all ips ("*"), all hostnames ("*") and DefaultAppPool application pool

### Step 3 - Create roles from playbook

<https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html>

From existing playbook create Ansible roles for

- User and group management
- Firefox installation
- IIS Installation
- Website deployment

### Help

<details>
<summary>Useful modules</summary>

- ansible.windows.win_group
- ansible.windows.win_user
- chocolatey.chocolatey.win_chocolatey
- ansible.windows.win_feature
- ansible.windows.win_reboot
- community.windows.win_iis_website
- ansible.windows.win_file
- ansible.windows.win_template

</details>

<details>
<summary>Inventory file</summary>

```ini
[windows]
lab-lin-xx

[windows:vars]
ansible_user=__REDACTED__
ansible_password=__REDACTED__
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
```

</details>


<details>
<summary>IIS Website task</summary>

```yaml
- name: create new website
  community.windows.win_iis_website:
    name: mywebsite
    state: started
    port: 80
    ip: "*"
    hostname: "*"
    application_pool: DefaultAppPool
    physical_path: "{{ www_path }}"
```

</details>

<details>
<summary>Testing website</summary>
From management node run

```console
curl lab-win-xx
```

You should get a response with templated text

</details>
