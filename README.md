<img src="https://res.cloudinary.com/dad08bn3f/image/upload/v1619329089/github/gcloud%20cli/gcloud_el0pxc.png" width="100"><br>
# Create Google Cloud HTTP(S) Load Balancing with session affinity
### Step 0 - prerequisite
0.1 - Installing Google Cloud SDK : https://cloud.google.com/sdk/docs/install
### Step 1 — Authorize gcloud to access the Cloud Platform with Google user credentials
```bash
gcloud auth login
```
### Step 2 — Set Default Project,Region,Zone
```bash
#Set,Get default project
$ gcloud config set project neo-loan
$ gcloud config get-value project
neo-loan

#List config
$ gcloud config list
[compute]
region = asia-east1
zone = asia-east1-b
[core]
account = kitti.s@blockfint.com
disable_usage_reporting = False
project = neo-loan

#Set dault region,sone
$ gcloud config set compute/region asia-southeast1
$ gcloud config set compute/zone asia-southeast1-b

#list config after set
gcloud config list                                                                                                               ok  Thinker gcloud  12:48:04 
[compute]
region = asia-southeast1
zone = asia-southeast1-b
[core]
account = kitti.s@blockfint.com
disable_usage_reporting = False
project = neo-loan

Your active configuration is: [default]
```
### Step 3 — List Image,Machine type that region available   
> this step require install Ubuntu 20.04 LTS on size e2-medium
```
$ gcloud compute images list|grep ubuntu-2004-lts
ubuntu-2004-focal-v20210927                           ubuntu-os-cloud      ubuntu-2004-lts                               READY
$ gcloud compute machine-types list|grep e2-medium|grep asia-southeast
...
e2-medium         asia-southeast1-b          2     4.00
...
```
### Step 4 — Create 2 VMs
```bash
$ (
#Create VM:lbtest01
gcloud compute instances create lbtest01 \
--image=ubuntu-2004-focal-v20210927 \
--image-project=ubuntu-os-cloud \
--machine-type=e2-medium
#Create VM:lbtest02
gcloud compute instances create lbtest02 \
--image=ubuntu-2004-focal-v20210927 \
--image-project=ubuntu-os-cloud \
--machine-type=e2-medium)
...
Created [https://www.googleapis.com/compute/v1/projects/neo-loan/zones/asia-southeast1-b/instances/lbtest01].
NAME      ZONE               MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
lbtest01  asia-southeast1-b  e2-medium                  10.148.0.63  34.126.88.251  RUNNING
Created [https://www.googleapis.com/compute/v1/projects/neo-loan/zones/asia-southeast1-b/instances/lbtest02].
NAME      ZONE               MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP    EXTERNAL_IP    STATUS
lbtest01  asia-southeast1-b  e2-medium                  10.148.0.63  34.126.88.251  RUNNING
...
#List VMs
$ gcloud compute instances list
NAME                                         ZONE               MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP    EXTERNAL_IP     STATUS
lbtest01                                     asia-southeast1-b  e2-medium                   10.148.0.63    34.126.88.251   RUNNING
lbtest02                                     asia-southeast1-b  e2-medium                   10.148.15.192  35.240.151.51   RUNNING
```
>>https://cloud.google.com/compute/docs/instances/create-start-instance#startinstancegcloud
### Step 5 — Install docker,docker-compose both lbtest01,lbtest02
```bash
#Install docker
$ sudo apt update
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
$ sudo apt update
$ apt-cache policy docker-ce
$ sudo apt install docker-ce -y
$ sudo systemctl status docker --no-pager
$ sudo usermod -aG docker ${USER}
$ sudo docker info
#Install docker-compose
$ sudo apt install jq -y
$ compose_version=$(curl https://api.github.com/repos/docker/compose/releases/latest | jq .name -r)
$ output='/usr/local/bin/docker-compose'
$ sudo curl -L https://github.com/docker/compose/releases/download/$compose_version/docker-compose-$(uname -s)-$(uname -m) -o $output
$ sudo chmod +x $output
$ docker-compose --version
```
### Step 6 — Runnig container application both lbtest01,lbtest02
>https://nodejs.org/de/docs/guides/nodejs-docker-webapp/
```
$ git clone https://github.com/kittisuw/gcp-load-balancing-session-affinity.git
$ cd testapp
#Run container and specific project name
$ docker-compose -p testapp up -d --build
[+] Running 3/3
...

#Check container run inside this project
$ docker-compose -p testapp ps
NAME                COMMAND                  SERVICE             STATUS              PORTS
web01               "docker-entrypoint.s…"   web01               running             0.0.0.0:3400->8080/tcp, :::3400->8080/tcp
web02               "docker-entrypoint.s…"   web02               running             0.0.0.0:3500->8080/tcp, :::3500->8080/tcp

#Testing call container using http
curl -i 0:3400
curl -i 0:3500
```
> Stop container of this project :$ docker-compose -p testapp stop   
> Rebuild with run container:$ docker-compose -p testapp up -d --build
### Step 7 —  Creat Loadbalance
7.1 - Create Unmanage Instace group for grouping 2 VMs name uig01   
![g1](img/group-01.png)     
   
![g2](img/group-02.png)   
> limitation of Unmanage Instance group is all VMs instance are locate only one zone   

7.2 - Create HTTP Load balance(L7)   





## Clean up
```bash
(gcloud compute instances delete --zone "asia-southeast1-b" lbtest01
gcloud compute instances delete --zone "asia-southeast1-b" lbtest02)
```
