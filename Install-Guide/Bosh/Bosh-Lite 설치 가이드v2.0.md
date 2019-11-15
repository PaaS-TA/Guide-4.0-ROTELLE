# bosh-lite (CLI v2)

- [Pre-Reqs](#pre-reqs)   


## Pre-Reqs

실습환경 : Ubuntu 18.04


## Bosh dependency 설치

```
$ sudo apt-get update
```


Ubuntu 16.04
```
sudo apt-get install -y build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt-dev libxml2-dev libssl-dev libreadline6 libreadline6-dev libyaml-dev libsqlite3-dev sqlite3
```

Ubuntu 18.04
```
sudo apt-get install -y build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt1-dev libxml2-dev libssl-dev libreadline7 libreadline-dev libyaml-dev libsqlite3-dev sqlite3
```





<!--
$ wget --content-disposition https://download.virtualbox.org/virtualbox/5.2.22/virtualbox-5.2_5.2.22-126460~Ubuntu~bionic_amd64.deb
-->


Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
```
$ sudo apt update

$ wget --content-disposition https://download.virtualbox.org/virtualbox/5.2.20/virtualbox-5.2_5.2.20-125813~Ubuntu~bionic_amd64.deb

$ sudo dpkg -i virtualbox-5.2_5.2.20-125813~Ubuntu~bionic_amd64.deb

$ sudo apt --fix-broken install

$ sudo /sbin/vboxconfig

$ VBoxManage --version
5.2.22r126460

```


virtualbox 설치 실패시 실행

```
$ sudo /sbin/vboxconfig
```

<!--
UnInstall VirtualBox
```
sudo apt-get remove virtualbox* —purge


sudo rm -rf  ~/'VirtualBox VMs’/


sudo rm -rf ~/.config/VirtualBox/

```
-->

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






## Install  Bosh  Director VM

```

$ git clone https://github.com/cloudfoundry/bosh-deployment ~/workspace/bosh-deployment

$ cd ~/workspace/bosh-deployment/virtualbox

$ vi ~/workspace/bosh-deployment/virtualbox/cpi.yml
   
      memory: 8196 ==> 수정

$ cd ~/workspace/bosh-deployment

$ vi create-bosh.sh


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

$ chmod 755 ./create-bosh.sh

$ ./create-bosh.sh
Deployment manifest: '/home/ubuntu/workspace/bosh-deployment/bosh.yml'
Deployment state: './state.json'


Started validating
  Downloading release 'bosh'... Skipped [Found in local cache] (00:00:00)
  Validating release 'bosh'... Finished (00:00:00)
  Downloading release 'bpm'... Skipped [Found in local cache] (00:00:00)
  Validating release 'bpm'... Finished (00:00:00)
  Downloading release 'bosh-virtualbox-cpi'... Skipped [Found in local cache] (00:00:00)
  Validating release 'bosh-virtualbox-cpi'... Finished (00:00:02)
  Downloading release 'garden-runc'... Skipped [Found in local cache] (00:00:00)
  Validating release 'garden-runc'... Finished (00:00:01)
  Downloading release 'bosh-warden-cpi'... Skipped [Found in local cache] (00:00:00)
  Validating release 'bosh-warden-cpi'... Finished (00:00:00)
  Downloading release 'os-conf'... Skipped [Found in local cache] (00:00:00)
  Validating release 'os-conf'... Finished (00:00:00)
  Downloading release 'uaa'... Skipped [Found in local cache] (00:00:00)
  Validating release 'uaa'... Finished (00:00:01)
  Downloading release 'credhub'... Skipped [Found in local cache] (00:00:00)
  Validating release 'credhub'... Finished (00:00:00)
  Validating cpi release... Finished (00:00:00)
  Validating deployment manifest... Finished (00:00:00)
  Downloading stemcell... Skipped [Found in local cache] (00:00:00)
  Validating stemcell... Finished (00:00:03)
Finished validating (00:00:12)


Started installing CPI
  Compiling package 'golang-1.10-linux/48c842421b6f05acf88dc6ec17f7574dade28a86'... Finished (00:00:00)
  Compiling package 'golang-1.10-darwin/9eacb8fc6b1b693e4280be4e916e2000f375bfbf'... Finished (00:00:00)
  Compiling package 'virtualbox_cpi/491b894473950429800ed1ce9dbe1940aac37307'... Finished (00:00:00)
  Installing packages... Finished (00:00:05)
  Rendering job templates... Finished (00:00:00)
  Installing job 'virtualbox_cpi'... Finished (00:00:00)
Finished installing CPI (00:00:06)


Starting registry... Finished (00:00:00)
Uploading stemcell 'bosh-vsphere-esxi-ubuntu-xenial-go_agent/315.26'... Skipped [Stemcell already uploaded] (00:00:00)


Started deploying
  Creating VM for instance 'bosh/0' from stemcell 'sc-f0954928-01ad-4862-7c9d-b3f717f931ab'... Finished (00:00:02)
  Waiting for the agent on VM 'vm-d8bf054c-c682-434b-556d-2eaa523c067e' to be ready... Finished (00:01:48)
  Creating disk... Finished (00:00:00)
  Attaching disk 'disk-a05ee2f5-00c1-4bbf-46c6-f7bcde37e53d' to VM 'vm-d8bf054c-c682-434b-556d-2eaa523c067e'... Finished (00:00:05)
  Rendering job templates... Finished (00:00:11)
  Compiling package 'autoconf/4f8914a0ada02006da32066d2e374e2a065d3acafbceb60ffd45ea146df7af1f'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'golang/16d30b2f9b19b8c6b38bdab2445c763208f162a3acd9eae69efe2c70e17132c5'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'pkg-config/278dbe10dcc428102e39936da0c7ddd2254cb0029ec0c9b06f545b228484187a'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'libtool/3e211ee9e3aab09a9e8a9ff55ab9ce9ba81590d945640e2d29c078597db33a94'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'automake/f11b0c20f9b9a5de955bbe1cb877e836a8841ddf88407b25ebbe9d31e3992858'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'golang/b348840d6ee6de87fdd43e4792fdbe1af734039b'... Skipped [Package already compiled] (00:00:01)
  Compiling package 'ruby-2.4-r5/726cbb2214e138b576700db6a30698edb2b994e2'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'golang-1.10-linux/48c842421b6f05acf88dc6ec17f7574dade28a86'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'mysql/de7350b5d6d835f758a62c0b3a8f07d3f0150d67abfa0da3967bd1127fada4c6'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'libpq/b3fead55dd7c9fa83be2412b24db84d2b04dbdb5dd0583ee7e9afddbb200659a'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'apparmor/0133a2288b0f71d05d89bd117c957297ad19cd19c0253236ba0ce4dedeb27186'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'libseccomp/11f3a8e12b881d4d6f660d84df70d3fe99b12e1621f9e93ff66e120aec12ec96'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'grootfs/bbbccc3cdd867f7529137be089964676e449fdf9206ee30425ff1c12a82fb009'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'zlib/98e4e7f5eebfe3831cf312d32a44f71281fcbc2577770fffd7b9601991ab31de'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'util-linux/6310f68e8294c980de62481bced2fc915f449e3a2cc2d13a7a2a49c4a612d825'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'lzo/7ec718ef29da337d56f08e90534f561c138c2c88b576dc5da5185d85fdd42b20'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'bpm-runc/cb758414ed267820880c6bf1ca57d7467c2d70bf'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'openjdk_1.8.0/d2d85d44c6f0ee050ef1fb0149ce3da575234ad6'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'nginx/d9c13905876ac2bc3f62657048750257794ecc53121ad7cd92583d3ef292b8b8'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'health_monitor/8f3a84d055a292d385f4f2f0df42ba25eb424ccff6a3854e1209668fec263295'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'postgres-10/c4167f3ba3a46ff5fb147098d87b9d6001ea357459133a7cc0c21aad36d4a1bc'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'bosh-gcscli/a03b1fc29fd357b8e3023193c87ae9ee49db052965efe5b3d9bff3538ed2df4b'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'tar/f922f97b27619f8331332e4186c1f9c63fa5b4ddd213c4a02c7ed7cf68fced21'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'lunaclient/b922e045db5246ec742f0c4d1496844942d6167a'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'warden_cpi/5a07ecf85c489cd4c32ae26d3092513027e14b86'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'iptables/7a3108e45160133ba44434781ff364f395c197ba2c92db6b31cced950cf7a37a'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'director/35121e0b2f3354dcfccbed5ea52ea2ccb14dd0d0b90cdd5c4be0787cbdd96ca6'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'postgres-9.4/ee3022bfae2cb7f7454aed9e28fe7715765b9a0086ef2045219f7a4e16baca81'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'gonats/a32f561770ebdc422c2f9955c7614c23900b9edc62aec51a6b48adc636516802'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'garden-idmapper/4a1079066953806366544d9e1558bff456ac3b07f07993c9288619197a5579c0'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'busybox/02b86e9e891e78294e366cfc3361514b752308952f56b954f7a6ae31961f859c'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'runc/16e92ae589da1dc24b072fa7e56ab50659139f330d45368cc361e451cac33c63'... Skipped [Package already compiled] (00:00:00)
  Compiling package 's3cli/4b6a4e65f277e1e4cdd9bfd8330de14ea28e9e5762a304f2d3a35f47958d7376'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'uaa_utils/90097ea98715a560867052a2ff0916ec3460aabb'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'thresholder/c580ec5d1719bc24c2354c6ef15b92c370e7654d086ebb23558b6e9d891b1426'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'netplugin-shim/cbdab762cda43c1ac92f2b4937d3a4d5d9f14912048c8db97f8abc6047ad5896'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'xfs-progs/6ce014f88e05f358621a25ce48ee036d78bdb1a935a8eb556974d3de26c8614d'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'uaa/6e13a5edd2a3cceb79d181bcac64771ea95fcccd'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'greenskeeper/d97ffa6f9c80773df6eb625784ce32522c5105d52016f4a3d691a9ac5344999f'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'guardian/c2d4d7fe13dce0558c6f2f8a6fbcb682e02bdb8a32b391a3f2a06660550762a4'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'bpm/d7c8de76cea8201f6cdf37d4f848b0a52c9c5798'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'davcli/100c52066493ef0d766c464c9cdd811c902a555013543bd16b62cf59ba5c72eb'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'verify_multidigest/2405293bf44933ac48362d7e109fc38ea4469b962cf62715533ecca53c28bf7b'... Skipped [Package already compiled] (00:00:00)
  Compiling package 'credhub/0481a83e36b6179f4fc9f28088c4cd2ac3aa870a'... Skipped [Package already compiled] (00:00:00)
  Updating instance 'bosh/0'... Finished (00:01:03)
  Waiting for instance 'bosh/0' to be running... Finished (00:00:23)
  Running the post-start scripts 'bosh/0'... Finished (00:00:41)
Finished deploying (00:04:24)


Stopping registry... Finished (00:00:00)
Cleaning up rendered CPI jobs... Finished (00:00:00)


Succeeded

```

## Alias and log into the Director


```
$ cd ~/workspace/bosh-deployment

$ bosh alias-env vbox -e 192.168.50.6 --ca-cert <(bosh int ./creds.yml --path /director_ssl/ca)

Using environment '192.168.50.6' as anonymous user


Name      bosh-lite
UUID      c62febc7-14c9-4505-9447-fc666cda354d
Version   269.0.1 (00000000)
CPI       warden_cpi
Features  compiled_package_cache: disabled
          config_server: enabled
          local_dns: enabled
          power_dns: disabled
          snapshots: disabled
User      (not logged in)

$ export BOSH_CLIENT=admin
$ export BOSH_CLIENT_SECRET=`bosh int ./creds.yml --path /admin_password`


$ bosh -e vbox env

Using environment '192.168.50.6' as client 'admin'


Name      bosh-lite
UUID      c62febc7-14c9-4505-9447-fc666cda354d
Version   269.0.1 (00000000)
CPI       warden_cpi
Features  compiled_package_cache: disabled
          config_server: enabled
          local_dns: enabled
          power_dns: disabled
          snapshots: disabled
User      admin


Succeeded
```

## credhub
BOSH설치시 operation file에 credhub.yml을 추가 하였다. Credhub은 인증정보를 저장소이다. 
Bosh 설치 시 credhub.yml을 적용하면 paasta 설치시 인증정보를 credhub에 저장하게 된다. 
Credhub에 로그인 하기 위해서는 credhub cli를 통해 인증정보를 조회 수정 삭제 할 수 있다

```
$ cd ~/workspace/
$ wget https://github.com/cloudfoundry-incubator/credhub-cli/releases/download/2.0.0/credhub-linux-2.0.0.tgz
--2019-05-27 15:57:54--  https://github.com/cloudfoundry-incubator/credhub-cli/releases/download/2.0.0/credhub-linux-2.0.0.tgz
Resolving github.com (github.com)... 52.78.231.108
접속 github.com (github.com)|52.78.231.108|:443... 접속됨.
HTTP request sent, awaiting response... 302 Found
Location: https://github-production-release-asset-2e65be.s3.amazonaws.com/57993310/aacbac0e-9662-11e8-83d6-8f4e66c9988b?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20190527%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20190527T065754Z&X-Amz-Expires=300&X-Amz-Signature=048cc5479dcf353cbfdbb54ad5a35d471c7ca59562c7330dec860f3aac8a0785&X-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3Dcredhub-linux-2.0.0.tgz&response-content-type=application%2Foctet-stream [following]
--2019-05-27 15:57:54--  https://github-production-release-asset-2e65be.s3.amazonaws.com/57993310/aacbac0e-9662-11e8-83d6-8f4e66c9988b?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20190527%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20190527T065754Z&X-Amz-Expires=300&X-Amz-Signature=048cc5479dcf353cbfdbb54ad5a35d471c7ca59562c7330dec860f3aac8a0785&X-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3Dcredhub-linux-2.0.0.tgz&response-content-type=application%2Foctet-stream
Resolving github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)... 52.216.102.3
접속 github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)|52.216.102.3|:443... 접속됨.
HTTP request sent, awaiting response... 200 OK
Length: 3267079 (3.1M) [application/octet-stream]
Saving to: ‘credhub-linux-2.0.0.tgz’


credhub-linux-2.0.0.tgz                          100%[=========================================================================================================>]   3.12M  1.26MB/s    in 2.5s


2019-05-27 15:57:58 (1.26 MB/s) - ‘credhub-linux-2.0.0.tgz’ saved [3267079/3267079]


$ tar -xvf credhub-linux-2.0.0.tgz
./
./credhub

$ chmod +x credhub

$ sudo mv credhub /usr/local/bin/credhub
[sudo] ubuntu의 암호:

$ credhub --version
CLI Version: 2.0.0
Server Version: Not Found. Have you targeted and authenticated against a CredHub server?


```

credhub login

```
$ cd ~/workspace/bosh-deployment


$ export CREDHUB_CLIENT=credhub-admin
$ export CREDHUB_SECRET=$(bosh int --path /credhub_admin_client_secret ~/workspace/bosh-deployment/creds.yml)
$ export CREDHUB_CA_CERT=$(bosh int --path /credhub_tls/ca ~/workspace/bosh-deployment/creds.yml)
$ credhub login -s https://192.168.50.6:8844 --skip-tls-validation 
Warning: The targeted TLS certificate has not been verified for this connection.
Warning: The --skip-tls-validation flag is deprecated. Please use --ca-cert instead.
Setting the target url: https://192.168.50.6:8844
Login Successful

$ credhub find
credentials: []
```


## jumpbox
BOSH설치시 operation file에 jumpbox-user.yml을 추가 하였다. 
Jumpbox는 BOSH VM에 접근하기 위한 인증을 적용하게 된다. 
인증 key는 Bosh 자체적으로 생성하며 인증키를 통해 BOSH VM에 접근 할 수 있다. 
bosh vm에 이상이 이 있거나 상태를 체크 할때 jumpbox를 활용하여 bosh vm에 접근할 수 있다. 
만약 Bosh에 문제가 있어 Bosh VM에 접근할 필요가 있을때 사용한다.

```
$ cd ~/workspace/bosh-deployment

$ bosh int ~/workspace/bosh-deployment/creds.yml --path /jumpbox_ssh/private_key > jumpbox.key 

$ chmod 600 jumpbox.key

$ ssh jumpbox@192.168.50.6 -i jumpbox.key
Unauthorized use is strictly prohibited. All access and activity
is subject to logging and monitoring.
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-50-generic x86_64)


 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.


Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


Last failed login: Mon May 27 07:08:49 UTC 2019 from 192.168.50.1 on ssh:notty
There was 1 failed login attempt since the last successful login.
Last login: Mon May 27 07:09:04 2019 from 192.168.50.1
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.


bosh/0:~$

```


## VirtualBox 저장 및 시작 
```
# To save state of vm
vboxmanage controlvm $(bosh int ~/workspace/bosh-deployment/state.json --path /current_vm_cid) savestate

# To start vm
vboxmanage startvm $(bosh int ~/workspace/bosh-deployment/state.json --path /current_vm_cid) --type headless
```


