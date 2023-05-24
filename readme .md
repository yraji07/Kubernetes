### Kubernetes installation steps:
 ----------------------------------
## These above steps all are used in Master , Node1, Node2
* docker 
  ------
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
sudo usermod -aG docker ubuntu
exit
re-login
>> This url used for refernce steps[RefereHere](https://github.com/Mirantis/cri-dockerd)

# Run these commands as root user
* ### Install GO###

sudo -i
wget https://storage.googleapis.com/golang/getgo/installer_linux
chmod +x ./installer_linux
./installer_linux
source ~/.bash_profile
git clone https://github.com/Mirantis/cri-dockerd.git
cd cri-dockerd
mkdir bin
go build -o bin/cri-dockerd
mkdir -p /usr/local/bin
install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
cp -a packaging/systemd/* /etc/systemd/system
sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
systemctl daemon-reload
systemctl enable cri-docker.service
systemctl enable --now cri-docker.socket
cd ~
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
  
-------------------------   till here   ---------------------------------    

### Note use these command only in master

 This is oly for master 
 * ` kubeadm init --pod-network-cidr "10.244.0.0/16" --cri-socket "unix:///var/run/cri-dockerd.sock" `
  
 * ` exit `

>> [ReferHere](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
 in master
 as a normal user (Ubuntu)

* mkdir -p $HOME/.kube
* sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
* sudo chown $(id -u):$(id -g) $HOME/.kube/config
* kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
 

### use these commands in Node1, Node2 ( as a root user) afetr installattion we will get kubeadm output take that command and fix the cri socket network  in node 

* kubeadm join 172.31.36.73:6443 --token paqndf.xvzsuv4f7mctr3xs \
       --cri-socket "unix:///var/run/cri-dockerd.sock" \ 
        --discovery-token-ca-cert-hash sha256:b29c8db59af2357007431c22bc3c776a908618d7fb64cbafd533db70023f06de 

`above steps taken in master node successfully installation part`

### Then go to master and check the nodes 

` kubectl get nodes ` it shows the ip of nodes 


### Kubernetes tasks:
-----------------
### Task 1st day 26/4/23
1) Write a Pod Spec for Spring PetClinic and nopCommerce Applications
In vs.code write a pod spec and apply it on k8s master 
In master terminal 
` vi spc.yaml `     
Check the docker hub registry to get our image example: raji07/rajispringpetclinic:spc  username/reponame:tagname 
` kubectl apply -f spc.yaml `        (it will apply for particular image)
check the pods status 
` kubectl get pods `

```yaml
Spring PetClinic
-----------------

---
apiVersion: v1
kind: Pod
metadata:
  name: spc
spec:
  containers:
    - name: spc
      image: raji07/rajispringpetclinic:spc
      ports:
        - containerPort: 8080
		
2) Nopcommerce
--------------
 vi spc.yaml 
 kubectl apply -f spc.yaml 
 kubectl get pods 

---
apiVersion: v1
kind: Pod
metadata:
   name: nopcommerce
spec:
   containers:
     - name: nopcommerce
      image: raji07/rajeshwari-nopcommerce:latest
      ports:
        - containerPort: 5000 
```
![preview](images/k81.png) 
![preview](images/k82.png)
![preview](images/k3.png) 
![preview](images/k4.png)
![preview](images/k5.png)

### 2. kubectl get pods and describe pods 
used commands:
--------------
` vi spc.yaml `
` kubectl apply -f spc.yaml `
` kubectl get pods `
In Master If we create in master we can see our ip adress of nodes 
form the above task continuation 
1.	` kubectl describe pods springpetclinic `
       we can see the status of application in the 

2.	` kubectl describe pods nopcommerce `
        we can see the status of application

If we create in master we can see our ip adress of nodes

Kubernetes (k8s) Activities (DAY02-27/APR/2023)
---------------------------------------------------------
### Task 2nd day 27/4/23

1) Explain Kubernetes architecture

A. Kubernetes is an architecture that offers a loosely coupled mechanism for service discovery across a cluster.
   A Kubernetes cluster has one or more control planes, and one or more compute nodes


### 2) Setup k8s on single node using minikube and kind  
   https://minikube.sigs.k8s.io/docs/start/ refer this document
# Minikube installation and run spc
* Create a liniux mancine i.e ec2 t2.medium 
* Then connect to the terminal and install docker commands 
* Install docker in vm 
* Next install kubectl refer this link  url:  https://minikube.sigs.k8s.io/docs/start/ in this follow the liniux commands 
use these commands: 
` curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 `
` sudo install minikube-linux-amd64 /usr/local/bin/minikub `
` minikube start `
` minikube kubectl -- get po -A `
` minikube dashboard `
` alias kubectl="minikube kubectl --"  `
` kubectl create deployment hellospc-minikube --image=raji07/rajispringpetclinic:spc ` 
` kubectl expose deployment hellospc-minikube --type=NodePort --port=8080 `
` minikube service hellospc-minikube `
` kubectl port-forward --address "0.0.0.0" service/hellospc-minikube 7080:8080  `

# Kind
------
url: https://kind.sigs.k8s.io/docs/user/quick-start/ 

* Create a liniux mancine i.e ec2 t2.medium 
* Then connect to the terminal and install docker commands 
* Install docker in vm 

` curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.18.0/kind-linux-amd64 `
` chmod +x ./kind `
` sudo mv ./kind /usr/local/bin/kind `

![preview](images/kind1.png)
![preview](images/kind2.png)
![preview](images/kind3.png)

### 3) Run the Spring Pet Clinic

A. Its running in port http://54.176.85.248:7080 

### All activities in class given task on 3 day 3

https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.27/#pod-v1-core 

```yaml 
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cronjob
spec:
  schedule: "* * * * *"
  jobTemplate:
    metadata:
      name: cronjobdata
    spec:
      backoffLimit: 2
      template:
        metadata:
          name: cronjobpod
          labels: 
            purpose: execute
        spec:
          restartPolicy: OnFailure
          containers:
            - name: cronjob
              image: alpine:3
              command: 
                - sleep
                - 5s 
```

* To execute the file use these cpmmands
```
vi demo.yaml 
kubectl apply -f demo.yaml
kubectl get jobs.batch
kubectl get po
kubectl delete -f . ( . deletes all the present files)
ls
rm demo.yaml
```
![preview](images/cronjob1.png)


### Creating Replicasets:
-------------------------
vi replicas.yaml 
kubectl apply -f replicas.yaml
kubectl get jobs.batch
kubectl get po
kubectl delete -f . ( . deletes all the present files)
ls
rm replicas.yaml

```yaml
---
apiVersion: apps/v1 
kind: ReplicaSet
metadata: 
  name: replicas
spec:
  minReadySeconds: 5
  replicas: 5
  selector:
    matchLabels: 
      app: nginx
  template:
    metadata:
      name: nginx pod
      labels: 
        app: nginx
    spec: 
      containers:
        - name: nginxpod
          image: nginx
          ports: 
            - containerPort: 80 
```

![preview](images/rs1.png) 
![preview](images/rs2.png)


### task day 4 4th may23
* Pods / Containers
* Jobs / CronJobs
* ReplicaSets
* Deployment
* Service / Headless Service
* Volumes / Persistent Volumes / Persistent Volume Claims
* Stateful Sets and 
* Namespaces etc.

* Pods / Containers
A pod is the smallest execution unit in Kubernetes. A pod encapsulates one or more applications. Pods are ephemeral by nature, if a pod (or the node it executes on) fails, Kubernetes can automatically create a new replica of that pod to continue operations.



### day 9 task 5
------------------
# 1.	Create a MySQL pod with Stateful Set with 1 replica 
```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-svc
spec:
  selector:
    app: mysql
  ports:
    - name: mysql
      port: 3306
      targetPort: 3306
      protocol: TCP

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-svc
  labels:
    app: 'mysql'
spec:
  minReadySeconds: 2
  replicas: 1
  serviceName: mysql-svc
  selector:
    matchLabels:
      app: 'mysql'
  template:
    metadata:
      name: mysql-pod
      labels:
        app: 'mysql'
    spec:
      containers:
        - name: mysql
          image: mysql:8
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: raji
            - name: MYSQL_USER
              value: raji
            - name: MYSQL_PASSWORD
              value: raji
            - name: MYSQL_DATABASE
              value: employees
```

# 2.	Create a nopCommerce deployment with 1 replica
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nopcommerce
spec:
  minReadySeconds: 10
  replicas: 1
  selector:
    matchLabels:
      app: nopcommerce
  template:
    metadata:
      labels:
        app: nopcommerce
    spec:
      containers:
      - name: nop
        image: raji07/rajeshwari-nopcommerce
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 5000
``` 

# 3.	Create a Headless Service to interact with nopCommerce with MySQL 
```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: headless
spec:
  type: ClusterIP
  clusterIP: None
  ports:
    - port: 80
      protocol: TCP
      targetPort: 5000
  selector:
    app: nop
    db: mysql

```

# 4.	Create a Load Balancer to expose the nopCommerce to External World 




### Kubernetes (k8s) Today Activities for Practice
----------------------------------------------
 Task: 9th may 23
----------------

Create a Kubernetes cluster using kubeadm
master and node 
### Use below steps to create AKS cluster
```yaml
curl -L https://aka.ms/InstallAzureCli | bash
source ~/.bashrc
az login
az login --tenant 00e5206b-f549-447d-8eaf-917573339f60
az group create --name myResourceGroup --location eastus
az aks create -g myResourceGroup -n myAKSCluster --enable-managed-identity --node-count 2 --enable-addons monitoring --enable-msi-auth-for-monitoring  --generate-ssh-keys
sudo -i
az aks install-cli
exit
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
kubectl get nodes
```

## Deploy any application using kubectl
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: nginx-deployment
  labels:
    app: nginx-deploy
spec: 
  selector:
    matchLabels:
      app: nginx-deploy
  minReadySeconds: 2
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
        - name: nginx
          image: nginx:1.23 
          ports:
            - containerPort: 8080
```
![preview](images/cls3.png) 
![preview](images/cls2.png)
![preview](images/cls1.png) 
![preview](images/cls6.png)
![preview](images/cls5.png) 

### Backup Kubernetes I.e backup etcd

### List out all the pod’s running in kube system namespace 
 
` kubectl get pods --namespace=kube-system ` shows all the namespaces list

![preview](images/cls4.png)  
![preview](images/cls9.png)

### Write down all the steps required to make Kubernetes highly available


### Do a rolling update and roll back 

` kubectl rollout history deployment/nginx-deployment `
update the rolling update in file shown below 
` kubectl rollout status deployment/nginx-deployment `
for  the same nginx deployment.yaml file we added startegy for rollout and we also changed the image version for it in ` kubectl get svc `  you will get the ip address so check the image in `it Works `  
` vi deployment.yaml ` 
` kubectl apply -f deployment.yaml ` 

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: nginx-deployment
  labels:
    app: nginx-deploy
spec: 
  selector:
    matchLabels:
      app: nginx-deploy
  minReadySeconds: 2
  replicas: 3
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25% 
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
        - name: nginx
          image: nginx:1
          ports:
            - containerPort: 80

# service file 
---
apiVersion: v1
kind: Service 
metadata:
  name: nginx-service
spec:
  selector: 
    app: nginx-deploy   
  ports: 
    type: LoadBalancer 
     - port: 80
       protocol: TCP
       targetPort: 80 
```
![preview](images/cls8.png) 
![preview](images/cls7.png)

or

` kubectl rollout history deployment/nginx-deployment `
update the rolling update in file shown below 
` kubectl rollout status deployment/nginx-deployment `
for  the same nginx deployment.yaml file we added startegy for rollout and we also changed the image version for it in ` kubectl get svc `  you will get the ip address so check the image in `it Works `  
` vi deployment.yaml ` 
` kubectl apply -f deployment.yaml ` 

```yaml 
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spc-deploy
  labels:
    app: spc
spec:
  minReadySeconds: 3
  replicas: 1
  selector:
    matchLabels:
      app: spc
  template:
    metadata:
      labels:
        app: spc
    spec:
      containers:
        - name: spc-cont
          image: raji07/rajispringpetclinic
          ports :
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: spc-lb
spec:
  selector:
    app: spc
  ports:
    - name: spc
      port: 32000
      targetPort: 8080
  type: LoadBalancer
```
![preview](images/spc2.png)

![preview](images/spc3.png)
![preview](images/spc1.png)


### Ensure usage of secret in MySQL and configmaps
Added configmaps and secret usage

```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-configmap
data: 
  MYSQL_DATABASE: Mysql
  MYSQL_ROOT_PASSWORD: cm9vdHJvb3Q=   # rootroot
  MYSQL_USER: cmFqaWxva2k=     #rajiloki
  MYSQL_PASSWORD: cm9vdHJvb3Q=   # rootroot

--- 
apiVersion: v1
kind: Pod
metadata:
  name: mysqlcm
spec:
  containers:
    - name: mysql
      image: mysql:8
      envFrom:
        - configMapRef:
            name: mysql-configmap
            optional: false
      ports:
        - containerPort: 3306
```
![preview](images/cm1.png) 

### Create a nop commerce deployment with MySQL statefulset and nop deployment 

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nopcommerce
spec:
  minReadySeconds: 10
  replicas: 1
  selector:
    matchLabels:
      app: nopcommerce
  template:
    metadata:
      labels:
        app: nopcommerce
    spec:
      containers:
      - name: nop
        image: raji07/rajeshwari-nopcommerce
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 5000

---
apiVersion: v1
kind: Service
metadata:
  name: mysql-svc
spec:
  selector:
    app: mysql
  ports:
    - name: mysql
      port: 3306
      targetPort: 3306
      protocol: TCP

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-svc
  labels:
    app: 'mysql'
spec:
  minReadySeconds: 2
  replicas: 1
  serviceName: mysql-svc
  selector:
    matchLabels:
      app: 'mysql'
  template:
    metadata:
      name: mysql-pod
      labels:
        app: 'mysql'
    spec:
      containers:
        - name: mysql
          image: mysql:8
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: raji
            - name: MYSQL_USER
              value: raji
            - name: MYSQL_PASSWORD
              value: raji
            - name: MYSQL_DATABASE
              value: employees
```


## a. Node selector b.	Affinity  c.	Taints and tolerances 
a)
* First we need to se the label for the node:
  ``kubectl label nodes <your-node-name> disktype=ssd``
  ``kubectl label nodes  ip-172-31-47-231 app=nginx``
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  minReadySeconds: 1
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      nodeSelector:
        app: nginx
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```    
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: mysql
spec:
  nodeSelector:
    app: mysql
  containers:
    - name: mysql
      image: mysql:8
      env:
        - name: MYSQL_ROOT_PASSWORD
          value: rajiloki
        - name: MYSQL_DATABASE
          value: employees
        - name: MYSQL_USER
          value: rajiloki
        - name: MYSQL_PASSWORD
          value: rajiloki
      ports:
        - containerPort: 3306
```
### b)affinity
------------------
• NopCommerce
### NopCommerce Deployment File with Affinity
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: nop-deploy
  labels:
    app: nopcommerce
spec: 
  minReadySeconds: 1
  replicas: 2
  selector: 
    matchLabels: 
      app: nopcommerce
  strategy:
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
    type: RollingUpdate  
  template:
    metadata:
      name: nop-pod
      labels:
        app: nopcommerce
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: nodename
                    operator: In
                    values: 
                      - node1
      containers:
        - name: nopcommerce
          image: prakashreddy2525/nopcommerce
          ports:
            - containerPort: 5000
              protocol: TCP
          readinessProbe:
            httpGet:
              path: /
              port: 5000
          livenessProbe:
            tcpSocket:
              port: 5000
```
• Mysql
### Mysql Deployment File with Affinity
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deploy
spec:
  minReadySeconds: 3
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
  template:
    metadata:
      name: mysql-pod
      labels:
        app: mysql
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: nodename
                    operator: In
                    values: 
                      - node2
      containers:
        - name: mysql
          image: mysql:8
          ports:
            - containerPort: 3306
          envFrom:
            - configMapRef:
                name: mysql-configmap
                optional: false 
```
### C. Taints and tolerences
------------------------------
• We Should have to attach taints to nodes
• kubectl taint nodes key=value:effect
kubectl taint nodes ip-172-31-44-59 app=web:NoSchedule
kubectl taint nodes ip-172-31-44-149 app=db:NoSchedule
### NopCommerce Deployment File with Tolerances
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: nop-deploy
  labels:
    app: nopcommerce
spec: 
  minReadySeconds: 1
  replicas: 1
  selector: 
    matchLabels: 
      app: nopcommerce
  strategy:
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
    type: RollingUpdate  
  template:
    metadata:
      name: nop-pod
      labels:
        app: nopcommerce
    spec:
      tolerations:
        - key: app
          operator: Equal
          value: db
          effect: NoSchedule
      containers:
        - name: nopcommerce
          image: prakashreddy2525/nopcommerce
          ports:
            - containerPort: 5000
              protocol: TCP
          readinessProbe:
            httpGet:
              path: /
              port: 5000
          livenessProbe:
            tcpSocket:
              port: 5000
```
• Mysql
### Mysql Deployment File with Tolerances
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deploy
spec:
  minReadySeconds: 1
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
  template:
    metadata:
      name: mysql-pod
      labels:
        app: mysql
    spec:
      tolerations:
        - key: app
          operator: Equal
          value: nop
          effect: NoSchedule
      containers:
        - name: mysql
          image: mysql:8
          ports:
            - containerPort: 3306
          envFrom:
            - configMapRef:
                name: mysql-configmap
                optional: false
```
### 2.	Create k8s cluster with version 1.25 and run any deployment(nginx/any) and then upgrade cluster to version 1.27 

done check pictures


# Eksctl Creation
* Lets create a EC2 instance to install aws cli, kubectl and eksctl
### Install AWS cli
```
sudo apt update
sudo apt install unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
* Craete a IAM user and give the administration permissions to that user.
* And then do the `aws configure` by using the aws credentials of `AWS Access Key and AWS Secret Key`.
### Install kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version
```
* Lets create key-gen `ssh-keygen`
### create eksctl cluster
```bash
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
# (Optional) Verify checksum
curl -sL "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin
```
* Create a file cluster.yaml
```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: basic-cluster
  region: ap-south-1
nodeGroups:
  - name: basic-eksctl
    instanceType: t2.large take   # mediuum are  small this is autoscaled
    desiredCapacity: 2
    volumeSize: 20
    ssh:
      allow: true # will use ~/.ssh/id_rsa.pub as the default ssh key
```
* Lets run command `eksctl create cluster -f cluster.yaml`
* And then run `kubectl get nodes`
* For the autocompletion run the below command
```bash
source <(kubectl completion bash) # set up autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.
```
# Create a Module in terraform to create ubuntu 22.04 vm 
```yaml
resource "aws_instance" "firstinstence" {
  for_each                    = toset(["one", "two"])
  ami                         = "ami-02eb7a4783e7e9317" # change urs
  instance_type               = "t2.micro"
  key_name                    = "id_rsa"
  associate_public_ip_address = true
  vpc_security_group_ids      = ["sg-0968c4e2044456b8f"] # change urs
  subnet_id                   = data.aws_subnet.first.id
  tags = {
    Name = "firstinstence" 
  }
}

datasource.tf
data "aws_vpc" "default" {
  default = true
}
data "aws_subnet" "first" {
  vpc_id            = data.aws_vpc.default.id
  availability_zone = "${var.region}a"
}
}

inputs.tf
variable "region" {
  type    = string
  default = "ap-west-1"
}
provider.tf
2:53
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "4.66.1"
    }
  }
}
provider "aws" {
  region = "ap-west-1"
}


