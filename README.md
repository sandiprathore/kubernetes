 <div id="top"></div>

 #### Create k8s cluster and deploying Mysql in it and run MySQL quires using python
 *  ##### Table of Contents
 
1. <a href="#Prerequisite">Prerequisite</a>
2. <a href="#Launch-EC2-instances(Ubuntu)">Launch EC2 instances(Ubuntu)</a>
3. <a href="#Install-docker">Install docker on ubuntu WSL</a>
4. <a href="#Install-kind">Install kind</a>
5. <a href="#Install-kubectl">Install kubectl</a>
6. <a href="#Create-single-node-cluster-using-kind">Create single node cluster using kind</a>
7. <a href="#create-multi-node-note-cluster">Create multi node note cluster</a>
8. <a href="#Delete-cluster">Delete cluster</a>
9. <a href="#Install_helm">Install helm</a>
10. <a href="#Deploy-MYSQL-using-helm">Deploy MySQL using helm  </a>
11. <a href="#Run-mysql-inside-the-pod">Run MySQL inside the pod</a>
12. <a href="#Delete-mysql">Delete mysql</a>
13. <a href="#Run-mysql-outside-the-pod">Run MySQL outside the pod</a>
14. <a href="#Create-user-in-mysql">Create user in MySQL</a>
15. <a href="#Port-forwarding">Port forwarding</a>
16. <a href="#Connect-MYSQL-to-python">Connect MySQL to python</a>

<div id="Prerequisite"></div>

#### Pre requirements :
* Ec2 (ubuntu) or Windows Subsystem for Linux (WSL)
* docker (docker) - no sudo required
* Kind
* Kubectl
<div id="Launch-EC2-instances(Ubuntu)"></div>

<!--Launch EC2 -->
##### Step 1. Launch EC2 instances(Ubuntu)
you can follow my document to launch EC2 instance 
<a href="https://docs.google.com/document/d/1HMMw545C0GDP1mIhJS2xk9jjI1I7zEfXF9urkgitsNI/edit?usp=sharing ">launch EC2 instance</a>
<p align="right">(<a href="#top">back to top</a>)</p>


<!-- DOcker installation  -->
<div id="Install-docker"></div>

##### Step 2. Install docker on ubuntu WSL 
Follow the steps that given below to install docker  

```sh
sudo apt update
 ``` 
```sh
sudo apt install apt-transport-https ca-certificates curl software-properties-common
 ```
```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
 ```
```sh
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
 ```
```sh
apt-cache policy docker-ce
 ```
```sh
sudo apt install docker-ce
 ```
```sh
sudo usermod -aG docker ${USER}
 ```
 ```sh
su - ${USER}
 ```
 ```sh
groups
 ```
 ```sh
export DOCKER_HOST=localhost:2375
 ```
 ```sh
echo "export DOCKER_HOST=localhost:2375" >> ~/.bash_profile
 ```
IF this type of error comes in ubuntu WSL

```sh
Cannot connect to the Docker daemon at tcp://localhost:2375. Is the docker daemon running?
 ```
 <p align="right">(<a href="#top">back to top</a>)</p>


###### Install docker application 

Ref: https://docs.docker.com/desktop/windows/install/ 

![docker_1](https://user-images.githubusercontent.com/93520937/154838962-2a070260-0010-4e4b-acea-0f2619590f65.PNG)

###### Open the downloaded "docker_installer"

![docker_file](https://user-images.githubusercontent.com/93520937/154839098-e16d8a1f-4fd2-4cb6-8d20-251477cb326c.PNG)

###### After docker installation goto Setting

![DOCEKR_SETTING](https://user-images.githubusercontent.com/93520937/154838838-a24c4714-f240-4cf9-a160-8307c166390d.PNG)

###### tick on "Expose daemon on tcp://localhost:2375 without TLS" and click on "Apply&Restart"

![DOCKER_2](https://user-images.githubusercontent.com/93520937/154839219-14becd0f-fb05-4c83-83d0-24b2d94c51a0.PNG)

Check docker version and print “hello word”

<p align="right">(<a href="#top">back to top</a>)</p>
```sh
docker --version
 ```
```sh
docker run hello-world
 ```
Result:

```sh
$ docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:97a379f4f88575512824f3b352bc03cd75e239179eea0fecc38e597b2209f49a
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
  ```

<!-- kind installation -->
<div id="Install-kind"></div>

##### Step 3. Install kind 

```sh
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64
 ```
```sh
chmod +x ./kind
 ```
```sh
sudo mv kind /usr/local/bin
 ```

Check kind version
``` 
kind --version
 ```
<p align="right">(<a href="#top">back to top</a>)</p>

<div id="Install-kubectl"></div>
 

<!-- kubectl installation  -->
##### Step 4. Install kubectl
 
```sh
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
 ```
 
```sh
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
 ```
 
```sh
echo "$(<kubectl.sha256) kubectl" | sha256sum --check
 ```
 
```sh
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
 ```
 
Check with kubectl version
```sh
kubectl version --client
 ```

 <p align="right">(<a href="#top">back to top</a>)</p>


<!-- create single node cluster -->
<div id="Create-single-node-cluster-using-kind"></div>

##### Step 5. Create K8S Cluster with kind
###### Create single node  cluster using kind (Optional)
###### Default cluster context name is `kind`.
```sh
kind create cluster
 ```
###### create cluster with name  
```sh
kind create cluster --name
 ````
Verify  with
```sh
kind get clusters
 ```
<p align="right">(<a href="#top">back to top</a>)</p>
 

<!-- creating multi node cluster -->
<div id="create-multi-node-note-cluster"></div>

###### create multi node note cluster 
```sh 
kind create cluster --name cluster_name --config kind-cluster-config.yaml
 ```
the example of cluuster config file is given below 
###### Yaml file
File: <a href="https://raw.githubusercontent.com/sandiprathore/kubernetes/MySQL-deployment/MySQL-deployment/kind-cluster-config.yaml">kind-cluster-config.yaml</a> 
```yaml

kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
containerdConfigPatches:
  - |-
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry:5000"]
      endpoint = ["http://registry:5000"]
nodes:
- role: control-plane
- role: worker
  kubeadmConfigPatches:
  - |
    kind: JoinConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "has-cpu=true"
 ```

 <p align="right">(<a href="#top">back to top</a>)</p>
<div id="Delete-cluster"></div>

<!-- Delete cluster -->
###### Delete cluster

```sh 
kind delete cluster  --<cluster_name>
 ```
 

 <!--helm intallation-->
 <div id="Install_helm"></div>

##### Step 6. Install helm   
```sh
 wget https://get.helm.sh/helm-v3.2.0-linux-amd64.tar.gz
 ```
```sh
tar -zxvf helm-v3.2.0-linux-amd64.tar.gz
 ```
 
```sh
sudo mv linux-amd64/helm /usr/local/bin/helm
 ```
 
```sh
helm repo add stable https://charts.helm.sh/stable
 ```
check with helm version
```sh 
helm version
 ```
 <p align="right">(<a href="#top">back to top</a>)</p>
 
 
 <!-- MySQL Deployment -->
 <div id="Deploy-MYSQL-using-helm"></div>

##### Step 7. Deploy MySQL using helm  
 
```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
 ```
```sh
helm install my-release bitnami/mysql 
 ```
```sh
MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default my-release-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode)
 ```
check kubectl pods

```sh
kubectl get pods
 ```
wait for a moment to get pod 'my-release-mysql-0' in ready comdition 
 <p align="right">(<a href="#top">back to top</a>)</p>


<!-- Run mysql inside the pod -->
<div id="Run-mysql-inside-the-pod"></div>

###### Run MySQL inside the pod
```sh
kubectl exec --stdin --tty my-release-mysql-0 -- /bin/bash
 ```

output you will be:
```sh
I have no name!@my-release-mysql-0:/$
 ```
After that run command that given below:

```sh
 mysql -h my-release-mysql.default.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"
 ```

Output will be:
```
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1217
Server version: 8.0.27 Source distribution
 
Copyright (c) 2000, 2021, Oracle and/or its affiliates.
 
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
 
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
 
mysql>
 
 ```
###### Now you can run your sql queries inside the port
Check with sql queries
<p align="right">(<a href="#top">back to top</a>)</p>


<!-- Delete MySQL -->
<div id="Delete-mysql"></div>

###### Delete MySQL:
```
helm delete my-release
 ```
<div id="Run-mysql-outside-the-pod"></div>

###### Steps to Run MySQL outside the pod with python 
 
<div id="Create-user-in-mysql"></div>

##### Step 8. Create user in MySQL 
Create user  in MySQL
```sql
CREATE USER 'User_name'@'host' IDENTIFIED BY 'Password';
 ```
Example:
```sql
CREATE USER 'new_user'@'%' IDENTIFIED BY 'new_user@123';
 ```
 
 
Grant all permissions to this user
```sql
GRANT ALL PRIVILEGES ON *.* TO 'user_name'@'host';
 ```
Example:

```sql
GRANT ALL PRIVILEGES ON *.* TO 'new_user'@'%';
 ```
 
 
Save and active your user
 
```sql
FLUSH PRIVILEGES;
 ``` 
Delete user:

```sql
DROP USER 'username'@'host';
 ```
<p align="right">(<a href="#top">back to top</a>)</p>


<!-- Port forwarding -->
<div id="Port-forwarding"></div>

##### Step 9. Port forwarding
```sh
kubectl port-forward my-release-mysql-0 3306:3306 --address host
 ```
 
Output will be: now can connect with you mysql with other ubuntu terminal
```sh 
Forwarding from 127.0.0.1:3306 -> 3306
Forwarding from [::1]:3306 -> 3306
 ```
Note: don’t close port-forwarding terminal
<p align="right">(<a href="#top">back to top</a>)</p>

<!-- Connect MYSQL to python -->
<div id="Connect-MYSQL-to-python"></div>

##### Step 10. Connect MySQL to python
 
Install MySQL connector
```
pip install mysql-connector-python
 ```
 
###### <a href="https://github.com/sandiprathore/SQL#mysql-queries-run-with-python">Run MySQL queries with python</a>

Connect to MySQL from local 
```
mysql -h host -uuser_name -p"password"
 ```
Example:
```
mysql -h 0.0.0.0 -uroot -p"gjQe0Stiin"
 ```
 <p align="right">(<a href="#top">back to top</a>)</p>
 
 <div id="Read-csv-file-using-python-pandas"></div> 