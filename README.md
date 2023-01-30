# k8s-example
Setup Kubernetes cluster from scratch

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
![image](https://user-images.githubusercontent.com/44951703/215588828-364b248f-26e6-4418-9e6a-c5a83593ce0e.png)

```bash
kubectl get po -A -o wide
```
![image](https://user-images.githubusercontent.com/44951703/215588982-e5fdd81b-04ed-4dc8-a496-ed84bb73a6bc.png)

### Checking Ingress controller
```bash
kubectl get po -n ingress-nginx -o wide
kubectl get svc -n ingress-nginx
```
![image](https://user-images.githubusercontent.com/44951703/215589953-065b92d5-4f40-4e88-b9b3-10a88376f423.png)

```bash
# Node IP
curl http://206.189.136.114:31135
```
![image](https://user-images.githubusercontent.com/44951703/215590958-07459d3e-a309-4d58-90d2-39605bac89de.png)




---
[1]: https://https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys#generating-and-working-with-ssh-keys
[2]: https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-centos7
[3]: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-the-control-machine
[4]: https://phoenixnap.com/kb/install-ansible-on-windows

