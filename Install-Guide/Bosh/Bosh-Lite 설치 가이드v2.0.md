# bosh-lite (CLI v2)

- [Pre-Reqs](#pre-reqs)   


## Pre-Reqs

실습환경 : Ubuntu 18.04


Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
```
$ sudo apt update

$ wget --content-disposition https://download.virtualbox.org/virtualbox/5.2.22/virtualbox-5.2_5.2.22-126460~Ubuntu~bionic_amd64.deb

$ sudo dpkg -i virtualbox-5.2_5.2.22-126460~Ubuntu~bionic_amd64.deb

$ VBoxManage --version
5.2.22r126460

```

UnInstall VirtualBox
```
sudo apt-get remove virtualbox* —purge


sudo rm -rf  ~/'VirtualBox VMs’/


sudo rm -rf ~/.config/VirtualBox/

```

Install Git & Curl 
```
$ sudo apt-get install git-core

$ sudo apt install curl

```



## Install CLI v2
Install [Bosh CLI](https://github.com/cloudfoundry/bosh-cli/releases)
```
$ mkdir -p ~/bosh-env/virtualbox

$ cd ~/bosh-env/virtualbox

$ curl -Lo ./bosh https://s3.amazonaws.com/bosh-cli-artifacts/bosh-cli-5.3.1-linux-amd64

$ chmod +x ./bosh

$ sudo mv ./bosh /usr/local/bin/bosh

$ bosh -v
version 5.3.1-8366c6fd-2018-09-25T18:25:51Z


Succeeded
```


## Bosh dependency 설치

Ubuntu 16.04
```
sudo apt-get install -y build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt-dev libxml2-dev libssl-dev libreadline6 libreadline6-dev libyaml-dev libsqlite3-dev sqlite3
```

Ubuntu 18.04
```
sudo apt-get install -y build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt1-dev libxml2-dev libssl-dev libreadline7 libreadline-dev libyaml-dev libsqlite3-dev sqlite3
```



## Install  Bosh  Director VM

```
git clone https://github.com/cloudfoundry/bosh-deployment ~/workspace/bosh-deployment
vi ~/workspace/bosh-deployment/virtualbox/cpi.yml
    memory: 8196

mkdir -p ~/deployments/vbox
cd ~/deployments/vbox
bosh create-env ~/workspace/bosh-deployment/bosh.yml \
  --state ./state.json \
  -o ~/workspace/bosh-deployment/virtualbox/cpi.yml \
  -o ~/workspace/bosh-deployment/virtualbox/outbound-network.yml \
  -o ~/workspace/bosh-deployment/bosh-lite.yml \
  -o ~/workspace/bosh-deployment/bosh-lite-runc.yml \
  -o ~/workspace/bosh-deployment/uaa.yml \
  -o ~/workspace/bosh-deployment/credhub.yml \
  -o ~/workspace/bosh-deployment/jumpbox-user.yml \
  --vars-store ./creds.yml \
  -v director_name=bosh-lite \
  -v internal_ip=192.168.50.6 \
  -v internal_gw=192.168.50.1 \
  -v internal_cidr=192.168.50.0/24 \
  -v outbound_network_name=NatNetwork
```

## Alias and log into the Director
```
bosh alias-env vbox -e 192.168.50.6 --ca-cert <(bosh int ./creds.yml --path /director_ssl/ca)
export BOSH_CLIENT=admin
export BOSH_CLIENT_SECRET=`bosh int ./creds.yml --path /admin_password`

$ bosh -e vbox env
Using environment '192.168.50.6' as '?'

Name: ...
User: admin

Succeeded
```

## Local Route
```
sudo route add -net 10.244.0.0/16     192.168.50.6 # Mac OS X
sudo ip route add   10.244.0.0/16 via 192.168.50.6 # Linux (using iproute2 suite)
route add           10.244.0.0/16     192.168.50.6 # Windows
```

## Update cloud config
```
git clone https://github.com/cloudfoundry/cf-deployment ~/workspace/cf-deployment
cd ~/workspace/cf-deployment
bosh -e vbox update-cloud-config iaas-support/bosh-lite/cloud-config.yml
```
### Upload a runtime-config
```
bosh -e vbox update-runtime-config ~/workspace/bosh-deployment/runtime-configs/dns.yml --name dns
```
>https://github.com/cloudfoundry/cf-deployment/issues/648  
>https://github.com/cloudfoundry/cf-deployment/blob/master/deployment-guide.md#upload-a-runtime-config

## Upload stemcell
stemcell 버전은 cf-deployment.yml 파일의 "stemcells" 버전을 입력한다.
```
export IAAS_INFO=warden-boshlite
export STEMCELL_VERSION=$(bosh interpolate cf-deployment.yml --path=/stemcells/alias=default/version)
bosh -e vbox upload-stemcell https://bosh.io/d/stemcells/bosh-${IAAS_INFO}-ubuntu-xenial-go_agent?v=${STEMCELL_VERSION}
```
## Deploy CF

```
bosh -e vbox -d cf deploy cf-deployment.yml \
   -o operations/bosh-lite.yml \
   -v system_domain=bosh-lite.com
```
https://github.com/cloudfoundry/cf-deployment/blob/master/deployment-guide.md
```
cf api https://api.bosh-lite.com --skip-ssl-validation
export CF_ADMIN_PASSWORD=$(bosh int <(credhub get -n /bosh-lite/cf/cf_admin_password --output-json) --path /value)
cf auth admin $CF_ADMIN_PASSWORD
```
## Set CredHub
```
export BOSH_CA_CERT="$(bosh interpolate ~/deployments/vbox/creds.yml --path /director_ssl/ca)"

export CREDHUB_SERVER=https://192.168.50.6:8844
export CREDHUB_CLIENT=credhub-admin
export CREDHUB_SECRET=$(bosh interpolate ~/deployments/vbox/creds.yml --path=/credhub_admin_client_secret)
export CREDHUB_CA_CERT="$(bosh interpolate ~/deployments/vbox/creds.yml --path=/credhub_tls/ca )"$'\n'"$( bosh interpolate ~/deployments/vbox/creds.yml --path=/uaa_ssl/ca)"
```
### CredHub CLI
Download release file
> https://github.com/cloudfoundry-incubator/credhub-cli/releases

and install
```
tar -xvf credhub-linux-2.2.0.tgz
mv credhub /usr/local/bin
```

and retrieve credentials in CredHub
```
credhub login -s https://192.168.50.6:8844 --skip-tls-validation
credhub find
credhub find -n cf_admin_password
credhub get -n <FULL_CREDENTIAL_NAME>
credhub get -n /bosh-lite/cf/cf_admin_password
```

## Release Modify
```
git clone https://github.com/cloudfoundry/diego-release ~/workspace/diego-release
cf ~/workspace/diego-release
git checkout tags/v2.22.0
bosh -e vbox releases
bosh -e vbox create-release --force
bosh -e vbox upload-release
```
modify cf-deployment.yml
```
releases:
- name: diego
#  url: https://bosh.io/d/github.com/cloudfoundry/diego-release?v=2.22.0
  version: 2.22.0+dev.3
  sha1: 0cd26089729d4e526dd27c47191cc3b9cb3189c6
```
deploy
```
cd ~/workspace/cf-deployment
bosh -e vbox -d cf deploy cf-deployment.yml \
   -o operations/bosh-lite.yml \
   -v system_domain=bosh-lite.com
```

## Commands
- 자동복구모드 off
```
bosh -e vbox -d cf update-resurrection off
```

- releases
```
bosh -e vbox releases
bosh -e vbox create-release --force --tarball=diego-2.22.0+dev.4.tgz
bosh -e vbox upload-release ./diego-2.22.0+dev.4.tgz
bosh -e vbox delete-release diego/2.22.0+dev.4
```
>https://bosh.io/docs/uploading-releases/

- deployments
```
bosh -e vbox deployments

#delete
bosh -e vbox -d zookeeper delete-deployment --force
```
- vms & instances
```
bosh -e vbox vms
bosh -e vbox instancesreleases:
```
- ssh
```
bosh –e vbox ssh –d {deployment-name} {job-name}
bosh –e vbox ssh –d cf nats
```
- tasks
```
bosh –e vbox tasks
```
- locks
```
bosh -e vbox locks
```


### 참고 BBS 테스트코드 생성
1. Install Protobuf
```
curl -OL https://github.com/protocolbuffers/protobuf/releases/download/v3.6.1/protoc-3.6.1-linux-x86_64.zip
# unzip protoc-3.6.1-linux-x86_64.zip -d /usr/local/ -x readme.txt include/*
# chmod a+x /usr/local/bin/protoc

unzip protoc-3.6.1-linux-x86_64.zip -d protoc3
sudo mv protoc3/bin/* /usr/local/bin/
sudo mv protoc3/include/* /usr/local/include/

sudo chown [user] /usr/local/bin/protoc
sudo chown -R [user] /usr/local/include/google

chmod a+x /usr/local/bin/protoc
```
2. Run Protoc
```
cd ~/workspace/diego-release
source .envrc
src/code.cloudfoundry.org/bbs/scripts/generate_protos.sh
cd src/code.cloudfoundry.org/bbs
```