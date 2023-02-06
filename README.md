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
![image](https://user-images.githubusercontent.com/44951703/217035876-4690f8ac-f752-4733-aeed-bb019a426d33.png)

```bash
kubectl get po -A -o wide
```
![image](https://user-images.githubusercontent.com/44951703/217036950-b154ecfe-0d2c-466d-a0f7-bce1a94fdcd7.png)

### Checking Ingress controller
```bash
kubectl get po -n ingress-nginx -o wide
```
![image](https://user-images.githubusercontent.com/44951703/217037320-3a6a9eaf-6717-4657-bdb4-837de1bb6cda.png)
```bash
kubectl get svc -n ingress-nginx
```
![image](https://user-images.githubusercontent.com/44951703/217037459-b6c8a0dc-e62f-4956-a273-2c989c88169e.png)

```bash
curl http://64.227.132.241:30080
```
![image](https://user-images.githubusercontent.com/44951703/217039487-280962bf-f9ce-49d9-9204-bcfc9d80dccd.png)

```bash
curl http://k8s.astakhoff.ru
```
![image](https://user-images.githubusercontent.com/44951703/217039785-461f0294-2dc9-4ec3-9178-5609311cd904.png)


## 2. Deploying services to Kubernetes cluster

### Install cert-manager<sup>[6]</sup>
* Execute the playbook:
    ```bash
    ansible-playbook -i hosts.ini cert-manager.yml
    ```
* Verify the installation<sup>[7]</sup>
    ```bash
    kubectl get pods --namespace cert-manager
    ```
    ![image](https://user-images.githubusercontent.com/44951703/217049002-79b168b0-4610-4742-a42a-4e67c6b9c10e.png)

### Deploy the sample app to the cluster
* Deploy "Online Boutique" demo application<sup>[8]</sup>:
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/main/release/kubernetes-manifests.yaml
    ```
* Wait for the Pods to be ready:
    ```bash
    watch kubectl get pods -o wide
    ```
    ![image](https://user-images.githubusercontent.com/44951703/217049836-aec046a2-22a4-42ec-a272-de02c49dad7b.png)
* Deploy ingress resource for frontend service:
    ```bash
    ansible-playbook -i hosts.ini frontend-ingress-resource.yml
    ```
* Access the web frontend using public IP of the worker node and ingress controller NodePort port:
  ```bash
  curl -kI -H 'Host: store.k8s.astakhoff.ru' https://64.227.136.238:30443
  ```
  ![image](https://user-images.githubusercontent.com/44951703/217051804-bd930905-b871-4ead-9455-caaaf40a8d44.png)
* Access the web frontend in a browser using external LoadBalancer's public IP:
![image](https://user-images.githubusercontent.com/44951703/217050986-be0a4018-ea25-4f87-8fbf-25c860ad811c.png)

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
    ![image](https://user-images.githubusercontent.com/44951703/217053762-dc67be0f-ff8b-424e-85a5-076e9401b48a.png)

* Push a commit into master branch of loadgenerator's source code
* Verify GitHub Actions workflow runs:
# TODO: add link and snapshot


## 3. Monitoring setup
### Install OpenEBS local PV device storage engine<sup>[9]</sup>
```bash
ansible-playbook -i hosts.ini openebs.yml
```
# TODO: post installation verification + screens

### Install Prometheus stack<sup>[10]</sup>
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
### Install metrics-server<sup>[11]</sup>
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
[6]: https://cert-manager.io/docs/installation/helm/
[7]: https://cert-manager.io/docs/installation/verify/#manual-verification
[8]: https://github.com/GoogleCloudPlatform/microservices-demo
[9]: https://openebs.io/docs/user-guides/installation
[10]: https://openebs.io/docs/stateful-applications/prometheus
[11]: https://artifacthub.io/packages/helm/metrics-server/metrics-server
