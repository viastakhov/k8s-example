# k8s-example
Setup Kubernetes cluster from scratch

## 1. Spinning Up Kubernetes Cluster

### Prerequisites
* An SSH key pair on your local machine<sup>[1]</sup>
* Servers running CentOS 7 with at least 2GB RAM and 2 vCPUs each and you should be able to SSH into each server as the root user with your SSH key pair<sup>[2]</sup>
* Ansible installed on your local machine<sup>[3],[4]</sup>
* Extarnal load balancer is provisioned
* Domain name is provisioned (i.e. astakhoff.ru)
* Common names are assigned to the public IP (external load balancer):
  * k8s.astakhoff.ru
  * grafana.k8s.astakhoff.ru
  * prometheus.k8s.astakhoff.ru
  * store.k8s.astakhoff.ru

### Setting Up the Workspace Directory and Ansible
* Setup a `./ansible/hosts.ini` file containing inventory information such as the IP addresses of your servers and the groups that each server belongs to.
* Define common names in `./ansible/vars/cnames.yml`
* (Optional) Define Ingress NodePorts in `./ansible/vars/main.yml`<sup>[5]</sup>

### Installing Kubernetes cluster
* Execute the playbook:
    ```bash
    ansible-galaxy install kwoodson.yedit
    ansible-playbook -i hosts.ini main.yml
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

### Install cert-manager<sup>[cm1]</sup>
* Execute the playbook:
    ```bash
    ansible-playbook -i hosts.ini cert-manager.yml
    ```
* Verify the installation<sup>[cm2]</sup>
    ```bash
    kubectl get pods --namespace cert-manager
    ```
    # TODO: screen

### Deploy the sample app to the cluster
* Deploy "Online Boutique" demo application<sup>[goo]</sup>:
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
    ansible-playbook -i hosts.ini frontend-ingress-resource.yml
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
* Prepare kube config as kube config secret for CI/CD tool:
    ```bash
    cat $HOME/.kube/config | base64
    ```
    # TODO: add screen of secret
* Push a commit into master branch of loadgenerator's source code
* Verify GitHub Actions workflow runs:
# TODO: add link and snapshot


## 3. Monitoring setup
### Install OpenEBS local PV device storage engine<sup>[6]</sup>
```bash
ansible-playbook -i hosts.ini openebs.yml
```
# TODO: post installation verification + screens

### Install Prometheus stack<sup>[7]</sup>
* (Optional) Setup prometheus stack in `./ansible/vars/prom-stack.yml`
* Execute the playbook:
    ```bash
    ansible-playbook -i hosts.ini prometheus.yml
    ```
# TODO: post installation verification + screens

* Import Grafana dashboards from /dashboard folder
* There are following metrics being used:
  * Pod Resource Usage by Namespace:
    | Metriс | Purpose |
    | ------------- | ------------- |
    | CPU Usage | Detect CPU bottlenecks, setup CPU resource requests/limits and VerticalPodAutoscaler/HorizontalPodAutoscaler |
    | Memory Usage | Detect high memory presure and leakage, setup memory resource requests/limits and VerticalPodAutoscaler/HorizontalPodAutoscaler |
    # TODO: screen
  * Nodes Resourse Usage:
    | Metriс | Purpose |
    | ------------- | ------------- |
    | CPU Usage | Control memory usage on the node, add additional CPU cores on demand |
    | Load Average | Control system load on the node, add additional CPU cores on demand |
    | Memory Usage | Control memory utilization on the node, provide additional CPU cores on demand |
    | Disk I/O | Detect disk I/O bottleneck |
    | Disk Space Usage | Control disk space utilization |
    | Network Received | Inspect inbound network traffic, monitors for reception of data that exceeds the bandwidth of the network interface |
    | Network Transmitted | Inspect outbound network traffic on the interface | 
    # TODO: screen
  * Persistent Volume Usage:
    | Metriс | Purpose |
    | ------------- | ------------- |
    | Volume Space Usage | Control volume space utilization by PVC |
    | Volume Inode Usage | Control the number of inodes available on volume by PVC |
    # TODO: screen

   # TODO: url + creds

## 4. Logging setup
### Install Loki stack
* Execute the playbook:
    ```bash
    ansible-playbook -i hosts.ini loki.yml
    ```
* Wait for the Pods to be ready:
    ```bash
    kubectl get po -n monitoring -o wide --selector release=loki
    ```
    # TODO: screen

### Pod logs inspection
Open "Loki Logs" dashboard in Grafana in order to review pod logs:
# Screen

## 5. Pod Autoscaling
### Install metrics-server<sup>[ms]</sup>
* Execute the playbook:
    ```bash
    ansible-playbook -i hosts.ini metrics-server.yml
    ```
* Check whether the metrics-server is available and running
    ```bash
    kubectl get apiservices | grep metrics.k8s.io
     ```
    # TODO: screen

### Setup pod autoscalling
* Create HorizontalPodAutoscaler resource for frontend service:
    ```bash
    kubectl apply -f ...
    ```  
* Verify createdreaource:
    ```bash
    kubectl get hpa
    ```
* Increase workload on frontend service:
    ```bash
    kubectl set env deployment/loadgenerator USERS=500
    ```
* There are several frontend pods appeared:
    ```
    kubectl get pod --selector app=frontend
    ```
    ![image](https://user-images.githubusercontent.com/44951703/217029942-77945874-295c-46d9-ba8c-2d1eb0d9b435.png)


---
[1]: https://https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys#generating-and-working-with-ssh-keys
[2]: https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-centos7
[3]: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-the-control-machine
[4]: https://phoenixnap.com/kb/install-ansible-on-windows
[5]: https://kubernetes.github.io/ingress-nginx/deploy/baremetal/#using-a-self-provisioned-edge
[6]: https://openebs.io/docs/user-guides/installation
[7]: https://openebs.io/docs/stateful-applications/prometheus
[cm1]: https://cert-manager.io/docs/installation/helm/
[cm2]: https://cert-manager.io/docs/installation/verify/#manual-verification
[goo]: https://github.com/GoogleCloudPlatform/microservices-demo
[ms]: https://artifacthub.io/packages/helm/metrics-server/metrics-server
