# k8s-example
Setup Kubernetes cluster from scratch

## 1. Spinning Up Kubernetes Cluster

### Prerequisites
* An SSH key pair on your local machine<sup>[1]</sup>
* Server running CentOS 7 with at least 2GB RAM and 2 vCPUs each and you should be able to SSH into each server as the root user with your SSH key pair<sup>[2]</sup>
* Ansible installed on your local machine<sup>[3],[4]</sup>

### Setting Up the Workspace Directory and Ansible
* Setup a `./ansible/hosts.ini` file containing inventory information such as the IP addresses of your servers and the groups that each server belongs to.
* (Optional) Define Ingress NodePorts in `./ansible/vars/main.yml`<sup>[5]</sup>

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
curl http://64.227.146.19:31904
```
![image](https://user-images.githubusercontent.com/44951703/215590958-07459d3e-a309-4d58-90d2-39605bac89de.png)

# TODO: curl to LB

## 2. Deploying services to Kubernetes cluster

### Deploy the sample app to the cluster
* Deploy "Online Boutique" demo application:
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/main/release/kubernetes-manifests.yaml
    ```
* Wait for the Pods to be ready:
    ```bash
    watch kubectl get pods -o wide
    ```
    ![image](https://user-images.githubusercontent.com/44951703/216015460-aba56e89-8c48-41e9-9305-e36445bfc309.png)
* Deploy ingress resource for frontend service:
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/viastakhov/k8s-example/main/kubernetes-manifests/frontend-ingress.yaml
    ```
* Access the web frontend in a browser using public IP of worker nodes and ingress controller NodePort port:
# TODO
* Access the web frontend in a browser using external LoadBalancer's public IP:
# TODO

### Deploy loadgenerator service using Helm via CI/CD
* Update existing loadgenerator instance in order to control deployment via helm:
    ```bash
    kubectl -n default label deployment loadgenerator "app.kubernetes.io/managed-by=Helm"
    kubectl -n default annotate deployment loadgenerator "meta.helm.sh/release-name=loadgenerator" "meta.helm.sh/release-namespace=default"
    ```
* Prepare kube config:
    ```bash
    # backup kube config:
    cp $HOME/.kube/config $HOME/config
    # set public IP in $HOME/config (i.e. "server: https://64.227.132.241:6443")
    # encrypt kube config and then use it as kube config secret in CI/CD tool:
    cat $HOME/config | base64
    ```
    # TODO: add screen of secret
* Push a commit into master branch of loadgenerator's source code
* Verify GitHub Actions workflow runs:
# TODO: add link and snapshot

---
[1]: https://https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys#generating-and-working-with-ssh-keys
[2]: https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-centos7
[3]: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-the-control-machine
[4]: https://phoenixnap.com/kb/install-ansible-on-windows
[5]: https://kubernetes.github.io/ingress-nginx/deploy/baremetal/#using-a-self-provisioned-edge
[6]: https://github.com/viastakhov/microservices-demo
