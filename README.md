# Create a VM instance from an image
## Authorize gcloud to access the Cloud Platform with Google user credentials
```bash
gcloud auth login
```
## Set Default Project,Region,Zone
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
## List Image,Machine type that region available
```
$ gcloud compute images list|grep ubuntu-2004-lts
ubuntu-2004-focal-v20210927                           ubuntu-os-cloud      ubuntu-2004-lts                               READY
$ gcloud compute machine-types list|grep e2-medium|grep asia-southeast
...
e2-medium         asia-southeast1-b          2     4.00
...
```
## Create 2 VMs for testing load balance
```bash
$ (
#Create vm01
gcloud compute instances create lbtest01 \
--image=ubuntu-2004-focal-v20210927 \
--image-project=ubuntu-os-cloud \
--machine-type=e2-medium
#Create vm02
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

## Install docker,docker-compose
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
# Runnig application
>https://nodejs.org/de/docs/guides/nodejs-docker-webapp/
```
mkdir testapp
cd testapp
#Run container and specific project name
$ docker-compose -p testapp up -d
CONTAINER ID   IMAGE                   COMMAND                  CREATED         STATUS         PORTS                                       NAMES
3541c3e71578   kittisuw/node-web-app   "docker-entrypoint.s…"   4 seconds ago   Up 3 seconds   0.0.0.0:3500->8080/tcp, :::3500->8080/tcp   node-web-app-01

#Check container run inside this project
$ docker-compose -p testapp ps
NAME                COMMAND                  SERVICE             STATUS              PORTS
node-web-app-01     "docker-entrypoint.s…"   node-web-app        running             0.0.0.0:3500->8080/tcp, :::3500->8080/tcp
#Stop container inside this project
docker-compose -p testapp stop
[+] Running 1/1
 ⠿ Container node-web-app-01  Stopped 
```
> rebuild with run container "docker-compose -p testapp up -d --build"
## Test call container
```bash
 $ curl -i localhost:3400
 $ curl -i localhost:3500
```
## Clean up
```bash
(gcloud compute instances delete --zone "asia-southeast1-b" lbtest01
gcloud compute instances delete --zone "asia-southeast1-b" lbtest02)
```
>https://cloud.google.com/compute/docs/instances/create-start-instance#startinstancegcloud