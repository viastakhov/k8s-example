# k8s-example
Setup Kubernetes cluser from scratch

# 1. Setup Kubernetes Cluster

### Prerequisites
* An SSH key pair on your local machine<sup>[1]</sup>
* Server running CentOS 7 with at least 2GB RAM and 2 vCPUs each and you should be able to SSH into each server as the root user with your SSH key pair<sup>[2]</sup>
* Ansible installed on your local machine<sup>[3],[4]</sup>

### Setting Up the Workspace Directory and Ansible Inventory File
* Setup a `./ansible/hosts.ini` file containing inventory information such as the IP addresses of your servers and the groups that each server belongs to.

### Installing Kubernetes cluster
* Execute the playbook:
```bash
ansible-playbook -i hosts main.yml
```

### Checking Kubernetes cluster
```bash
kubectl get no -o wide
```
![image](https://user-images.githubusercontent.com/44951703/215072456-dc4178b1-f9c3-4664-941f-7b8f1464924d.png)
```bash
kubectl get po -A -o wide
```
![image](https://user-images.githubusercontent.com/44951703/215072746-b6281bbf-6e3c-4fc3-8960-865043d920a3.png)




---
[1]: https://https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys#generating-and-working-with-ssh-keys
[2]: https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-centos7
[3]: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-the-control-machine
[4]: https://phoenixnap.com/kb/install-ansible-on-windows

