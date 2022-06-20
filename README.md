**Overview:**

Home k3s HA environment consisting of the following servers:
  - 2 x Intel NUCS (running Proxmox for virtualization)
  - 1 x MySQL server
  - 1 x Nginx load balancer
  - 2 x k3s master nodes
  - 3 x k3s worker nodes

The installation of all software on the operating system is done via Ansible playbooks.

**Hardware:**

Intel NUCS (Proxmox Virtualiztion Servers)
  - CPU: i5-3427U CPU @ 1.80GHz (1 Socket):
  - RAM: 16 GB
  - SSD: 256 GB 

All virtual machines (Ubuntu 20.04 LTS):
  - CPU: 1 core
  - RAM: 2 GB
  - SSD: 20 GB

**Prerequisites:**

  - Virtual machines provisioned on Proxmox (or any other virtualization platform)
  - Static IP addresses assigned to each virtual machine
  - SSH key access configured on each virtual machine
  - Kubectl installed on your dev machine unless managing this cluster from one of the master nodes
 
**Sameple Files:**

./samples/inventory-sample
  - Rename file to inventory and move to the root of project-home-k3s-ha folder
  - Configure hostnames and IP addresses for each server within the inventory file

./samples/ansible-sample.cfg
  - Rename file to ansible.cfg and move to the root of project-home-k3s-ha folder
  - Edit file setting the values for the following:
    - inventory = path to the inventory file
    - remote_user = user configured on each virtual machine
    - private_key_file = path to ssh key configured on each virtual machine

./samples/secrets-sample.yml
  - Create a folder called secrets in the root of project-home-k3s-ha folder
  - Rename file to secrets.yml and move to ../secrets/
  - Edit file setting the values for the following:
    - mysql_password = password to set for the root MySQL user
    - k3s_mysql_password = password to set for the k3sadmin user
    - mysql ip address = IP address of the MySQL server

./samples/install-k3sworker-sample.yml
  - Rename file to install-k3sworker.yml and move to ../playbooks/
  - Edit file setting the values for the following:
    - lb_server_hostname = hostname of the load balancer in the inventory file

**Secrets / Ansible Vault**

Once ./secrets/secrets.yml is configured you need to encrypt the file with ansible-vault

Run the following command:
  - ansible-vault encrypt ./secrets/secrets.yml

Set the vault password when prompted

**Execution:**

When all of the above is completed, execute the following command to test connectivity:
  - ansible-playbook ./playbooks/ping.yml -K

If all servers ping successfully, execute the following command to run all the playbooks:
  - ansible-playbook ./playbooks/all-playbooks.yml --ask-vault-pass -K

**Completion:**

Once the playbooks complete, k3s should be up and running.  You will need to get the config file from one of the master nodes:
  - SSH into one of your master nodes
  - cat out the file /etc/rancher/k3s/k3s.yaml and copy
  - Paste the contents of the file into the ~/.kube/config file on your local dev machine
  - Edit the ~/.kube/config file replacing https://127.0.0.1:6443 with the IP:Port of the load balancer