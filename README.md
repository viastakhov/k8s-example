# k8s-example
Setup Kubernetes cluser from scratch

# 1. Setup Kubernetes Cluster

### Prerequisites
* An SSH key pair on your local machine[1].
* Server running CentOS 7 with at least 2GB RAM and 2 vCPUs each and you should be able to SSH into each server as the root user with your SSH key pair[2].
* Ansible installed on your local machine[3],[4].

### Setting Up the Workspace Directory and Ansible Inventory File
* Setup a `./ansible/hosts.ini` file containing inventory information such as the IP addresses of your servers and the groups that each server belongs to.

### Installing Kubernetes cluster
* Execute the playbook:
```bash
ansible-playbook -i hosts main.yml
```





---
[1]: https://https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys#generating-and-working-with-ssh-keys
[2]: https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-centos7
[3]: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-the-control-machine
[4]: https://phoenixnap.com/kb/install-ansible-on-windows

