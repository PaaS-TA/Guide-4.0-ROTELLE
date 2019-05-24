# bosh-lite (CLI v2)

- [Pre-Reqs](#pre-reqs)   


## Pre-Reqs

실습환경 : Ubuntu 18.04


Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
```
$ sudo apt update

$ wget --content-disposition https://download.virtualbox.org/virtualbox/5.2.22/virtualbox-5.2_5.2.22-126460~Ubuntu~bionic_amd64.deb

$ wget --content-disposition https://download.virtualbox.org/virtualbox/5.2.20/virtualbox-5.2_5.2.20-125813~Ubuntu~bionic_amd64.deb

$ sudo dpkg -i virtualbox-5.2_5.2.22-126460~Ubuntu~bionic_amd64.deb

$ sudo apt --fix-broken install

$ sudo /sbin/vboxconfig

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
$ cd ~/workspace/bosh-deployment/virtualbox

$ git clone https://github.com/cloudfoundry/bosh-deployment ~/workspace/bosh-deployment

$ vi ~/workspace/bosh-deployment/virtualbox/cpi.yml
   
      memory: 8196 ==> 수정


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

~/workspace/bosh-deployment$ chmod 755 ./create-bosh.sh

~/workspace/bosh-deployment$ ./create-bosh.sh
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

## PaaS-TA 설치파일 업로드

```

$ scp ./paasta-4.0.tar  ubuntu@192.168.1.122:~/workspace/


```


## PaaS-TA 설치파일 압축풀기

```
$ cd ~/workspace/

$ tar -xvf ./paasta-4.0.tar 


```


## Local Route
```
$ sudo route add -net 10.244.0.0/16     192.168.50.6 # Mac OS X

$ sudo ip route add   10.244.0.0/16 via 192.168.50.6 # Linux (using iproute2 suite)

$ route add           10.244.0.0/16     192.168.50.6 # Windows
```

## Update cloud config
```

$ cd ~/workspace/paasta-4.0/deployment/cloud-config


$ bosh -e vbox update-cloud-config bosh-lite-cloud-config.yml
```


### Upload a runtime-config
```
~/workspace/paasta-4.0/deployment/bosh-deployment$ chmod 755 ./update-runtime-config.sh

~/workspace/paasta-4.0/deployment/bosh-deployment$ ./update-runtime-config.sh
```
>https://github.com/cloudfoundry/cf-deployment/issues/648  
>https://github.com/cloudfoundry/cf-deployment/blob/master/deployment-guide.md#upload-a-runtime-config

## Upload stemcell


```
cd ~/workspace/paasta-4.0/stemcell

wget --content-disposition https://bosh.io/d/stemcells/bosh-warden-boshlite-ubuntu-xenial-go_agent?v=97.28

~/workspace/paasta-4.0/stemcell$ bosh -e vbox upload-stemcell bosh-warden-boshlite-ubuntu-xenial-go_agent-97.28.tgz
Using environment '192.168.50.6' as client 'admin'


######################################################## 100.00% 221.00 MiB/s 2s
Task 3


Task 3 | 16:18:04 | Update stemcell: Extracting stemcell archive (00:00:03)
Task 3 | 16:18:07 | Update stemcell: Verifying stemcell manifest (00:00:00)
Task 3 | 16:18:08 | Update stemcell: Checking if this stemcell already exists (00:00:00)
Task 3 | 16:18:08 | Update stemcell: Uploading stemcell bosh-warden-boshlite-ubuntu-xenial-go_agent/97.28 to the cloud (00:00:11)
Task 3 | 16:18:19 | Update stemcell: Save stemcell bosh-warden-boshlite-ubuntu-xenial-go_agent/97.28 (b302aed0-ee21-4c80-474b-308c68129bb9) (00:00:00)


Task 3 Started  Mon May 20 16:18:04 UTC 2019
Task 3 Finished Mon May 20 16:18:19 UTC 2019
Task 3 Duration 00:00:15
Task 3 done


Succeeded


```

## Deploy PaaS-TA

stemcell 버전은 paasta-deployment.yml 파일의 "stemcells" 버전을 입력한다.
```
~/workspace/paasta-4.0/deployment/paasta-deployment$ vi paasta-deployment.yml
…..
stemcells:
- alias: default
  os: ubuntu-xenial
  #version: "97.28"
```


```
$ cd ~/workspace/paasta-4.0/deployment/paasta-deployment

$ vi ./paasta-deployment.yml

bosh -e vbox -d paasta deploy paasta-deployment.yml \
   -o operations/bosh-lite.yml \
   -o operations/use-compiled-releases.yml \
   -v inception_os_user_name=ubuntu \
   -v cf_admin_password=admin \
   -v cc_db_encryption_key=db-encryption-key \
   -v uaa_database_password=uaa_admin \
   -v cc_database_password=cc_admin \
   -v cert_days=3650 \
   -v system_domain=bosh-lite.com \
   -v uaa_login_logout_redirect_parameter_disable=false \
   -v uaa_login_logout_redirect_parameter_whitelist=["http://portal-web-user.115.68.151.187.xip.io","http://portal-web-user.115.68.151.187.xip.io/callback","http://portal-web-user.115.68.151.187.xip.io/login"] \
   -v uaa_login_branding_company_name="PaaS-TA R&D" \
   -v uaa_login_branding_footer_legal_text="Copyright © PaaS-TA R&D Foundation, Inc. 2017. All Rights Reserved."
```


```
~/workspace/paasta-4.0/deployment/paasta-deployment$ ./deploy-bosh-lite.sh

Task 32 | 07:43:05 | Preparing deployment: Preparing deployment (00:00:29)
Task 32 | 07:43:34 | Preparing deployment: Rendering templates (00:00:14)
Task 32 | 07:43:49 | Preparing package compilation: Finding packages to compile (00:00:01)
Task 32 | 07:43:50 | Compiling packages: proxy/74970cceed3c4c838ebc13eaee8aafd7593839f9
Task 32 | 07:43:50 | Compiling packages: golang-1-linux/864e21e6d4f474b33b5d810004e2382cd5c64972
Task 32 | 07:43:50 | Compiling packages: pid_utils/37ad75a08069799778151b31e124e28112be659f
Task 32 | 07:43:50 | Compiling packages: golang-1-linux/8fb48ae1b653b7d0b49d0cbcea856bb8da8a5700
Task 32 | 07:44:24 | Compiling packages: proxy/74970cceed3c4c838ebc13eaee8aafd7593839f9 (00:00:34)
Task 32 | 07:44:24 | Compiling packages: pid_utils/37ad75a08069799778151b31e124e28112be659f (00:00:34)
Task 32 | 07:45:02 | Compiling packages: golang-1-linux/8fb48ae1b653b7d0b49d0cbcea856bb8da8a5700 (00:01:12)
Task 32 | 07:45:02 | Compiling packages: bosh-dns/138f3bd2440ba97f0a7d8912facb5d4a2b320850
Task 32 | 07:45:02 | Compiling packages: golang-1-linux/864e21e6d4f474b33b5d810004e2382cd5c64972 (00:01:12)
Task 32 | 07:45:02 | Compiling packages: route_emitter/7f099e2f0bf0ead10ff7f30fa985a9b9fb77c8e8
Task 32 | 07:45:02 | Compiling packages: certsplitter/f2893b563467137aa3aaadab456aac3213a278c3
Task 32 | 07:45:02 | Compiling packages: rep/859598a60ffcc643362184e288ca97e69f5c6a60
Task 32 | 07:45:02 | Compiling packages: auctioneer/213840407bc21831517b231398be804a2e3108c5
Task 32 | 07:45:02 | Compiling packages: ssh_proxy/6f6c4057e8e1658bc38700502cc36be1fd898ef3
Task 32 | 07:45:44 | Compiling packages: certsplitter/f2893b563467137aa3aaadab456aac3213a278c3 (00:00:42)
Task 32 | 07:45:44 | Compiling packages: diego-sshd/33a8975cfbed3c3ae23d4802201ca51e4a047d99
Task 32 | 07:46:43 | Compiling packages: route_emitter/7f099e2f0bf0ead10ff7f30fa985a9b9fb77c8e8 (00:01:41)
Task 32 | 07:46:43 | Compiling packages: healthcheck/57004bfd97bd8fa59e2920744bc1c4a39af381a8
Task 32 | 07:47:01 | Compiling packages: ssh_proxy/6f6c4057e8e1658bc38700502cc36be1fd898ef3 (00:01:59)
Task 32 | 07:47:01 | Compiling packages: file_server/13d5384c0ba93016bbb93b2268aa6061031cb99c
Task 32 | 07:47:01 | Compiling packages: auctioneer/213840407bc21831517b231398be804a2e3108c5 (00:01:59)
Task 32 | 07:47:01 | Compiling packages: locket/bc8258b7b357857cd075006889f57721b25e5dbf
Task 32 | 07:47:28 | Compiling packages: rep/859598a60ffcc643362184e288ca97e69f5c6a60 (00:02:26)
Task 32 | 07:47:28 | Compiling packages: bbs/c3b8027641f733035fd12f02e49e2002e9c4d766
Task 32 | 07:48:04 | Compiling packages: bosh-dns/138f3bd2440ba97f0a7d8912facb5d4a2b320850 (00:03:02)
Task 32 | 07:48:04 | Compiling packages: cfdot/691a258380ddd3d9914bcce5eaa03a8aa842ed62
Task 32 | 07:48:13 | Compiling packages: file_server/13d5384c0ba93016bbb93b2268aa6061031cb99c (00:01:12)
Task 32 | 07:48:18 | Compiling packages: locket/bc8258b7b357857cd075006889f57721b25e5dbf (00:01:17)
Task 32 | 07:48:50 | Compiling packages: bbs/c3b8027641f733035fd12f02e49e2002e9c4d766 (00:01:22)
Task 32 | 07:48:59 | Compiling packages: cfdot/691a258380ddd3d9914bcce5eaa03a8aa842ed62 (00:00:55)
Task 32 | 07:49:22 | Compiling packages: healthcheck/57004bfd97bd8fa59e2920744bc1c4a39af381a8 (00:02:39)
Task 32 | 07:49:47 | Compiling packages: diego-sshd/33a8975cfbed3c3ae23d4802201ca51e4a047d99 (00:04:03)
Task 32 | 07:49:47 | Compiling packages: docker_app_lifecycle/73147e8496ce8ec0b9dfcce161a7201007a4cbd0
Task 32 | 07:49:47 | Compiling packages: windows_app_lifecycle/68286ac0c53a7fd6a5ab1656e071af2cd0362e93
Task 32 | 07:49:47 | Compiling packages: buildpack_app_lifecycle/9d742fa46579b97cc7ed73c6308dc0a8b7e15644
Task 32 | 07:51:36 | Compiling packages: docker_app_lifecycle/73147e8496ce8ec0b9dfcce161a7201007a4cbd0 (00:01:49)
Task 32 | 07:51:42 | Compiling packages: windows_app_lifecycle/68286ac0c53a7fd6a5ab1656e071af2cd0362e93 (00:01:55)
Task 32 | 07:52:24 | Compiling packages: buildpack_app_lifecycle/9d742fa46579b97cc7ed73c6308dc0a8b7e15644 (00:02:37)
Task 32 | 07:52:25 | Creating missing vms: adapter/6eae8258-103d-4197-ad98-bd33c2d88aaa (0)
Task 32 | 07:52:25 | Creating missing vms: diego-api/c8ddde37-0813-42e1-9eec-954803605a86 (0)
Task 32 | 07:52:25 | Creating missing vms: uaa/55406075-0455-434b-8a44-8ac91d23634c (0)
Task 32 | 07:52:25 | Creating missing vms: nats/039d66bf-212c-4e95-ac58-b47fe8b8c631 (0)
Task 32 | 07:52:25 | Creating missing vms: database/55b742a6-11bf-4c0f-b74b-ba4640f201fb (0)
Task 32 | 07:52:25 | Creating missing vms: api/8a7f5660-ef6b-4f78-a5df-c9244b3a817c (0)
Task 32 | 07:52:25 | Creating missing vms: router/d20f4994-4979-49f9-8c28-e5e188832bad (0)
Task 32 | 07:52:25 | Creating missing vms: diego-cell/2abca670-ff56-443e-a2a3-336ff6a34b3c (0)
Task 32 | 07:52:25 | Creating missing vms: scheduler/cd308352-8bbe-4373-b0da-43dde4dacbbf (0)
Task 32 | 07:52:25 | Creating missing vms: log-api/a90cd222-4839-40b4-b151-aedfde3793b8 (0)
Task 32 | 07:52:25 | Creating missing vms: tcp-router/6538788b-884c-40f6-81ee-59a2640b9f90 (0)
Task 32 | 07:52:25 | Creating missing vms: singleton-blobstore/0a3064fa-6b6e-4b66-afbf-5fb3280eb886 (0)
Task 32 | 07:52:25 | Creating missing vms: cc-worker/528eed4d-df16-4b8f-8c07-3615a47d6ac2 (0)
Task 32 | 07:52:25 | Creating missing vms: doppler/fab64435-74ed-4e29-a862-310c31a1d240 (0)
Task 32 | 07:52:25 | Creating missing vms: credhub/c2827328-019b-44e7-beb6-b1695ea76e33 (0)
Task 32 | 07:52:55 | Creating missing vms: tcp-router/6538788b-884c-40f6-81ee-59a2640b9f90 (0) (00:00:30)
Task 32 | 07:52:56 | Creating missing vms: diego-api/c8ddde37-0813-42e1-9eec-954803605a86 (0) (00:00:31)
Task 32 | 07:52:57 | Creating missing vms: cc-worker/528eed4d-df16-4b8f-8c07-3615a47d6ac2 (0) (00:00:32)
Task 32 | 07:52:57 | Creating missing vms: credhub/c2827328-019b-44e7-beb6-b1695ea76e33 (0) (00:00:32)
Task 32 | 07:52:58 | Creating missing vms: diego-cell/2abca670-ff56-443e-a2a3-336ff6a34b3c (0) (00:00:33)
Task 32 | 07:52:58 | Creating missing vms: router/d20f4994-4979-49f9-8c28-e5e188832bad (0) (00:00:33)
Task 32 | 07:52:58 | Creating missing vms: log-api/a90cd222-4839-40b4-b151-aedfde3793b8 (0) (00:00:33)
Task 32 | 07:52:59 | Creating missing vms: uaa/55406075-0455-434b-8a44-8ac91d23634c (0) (00:00:34)
Task 32 | 07:52:59 | Creating missing vms: doppler/fab64435-74ed-4e29-a862-310c31a1d240 (0) (00:00:34)
Task 32 | 07:52:59 | Creating missing vms: nats/039d66bf-212c-4e95-ac58-b47fe8b8c631 (0) (00:00:34)
Task 32 | 07:53:00 | Creating missing vms: adapter/6eae8258-103d-4197-ad98-bd33c2d88aaa (0) (00:00:35)
Task 32 | 07:53:00 | Creating missing vms: scheduler/cd308352-8bbe-4373-b0da-43dde4dacbbf (0) (00:00:35)
Task 32 | 07:53:01 | Creating missing vms: api/8a7f5660-ef6b-4f78-a5df-c9244b3a817c (0) (00:00:36)
Task 32 | 07:53:01 | Creating missing vms: database/55b742a6-11bf-4c0f-b74b-ba4640f201fb (0) (00:00:36)
Task 32 | 07:53:01 | Creating missing vms: singleton-blobstore/0a3064fa-6b6e-4b66-afbf-5fb3280eb886 (0) (00:00:36)
Task 32 | 07:53:02 | Updating instance nats: nats/039d66bf-212c-4e95-ac58-b47fe8b8c631 (0) (canary)
Task 32 | 07:53:02 | Updating instance adapter: adapter/6eae8258-103d-4197-ad98-bd33c2d88aaa (0) (canary)
Task 32 | 07:53:47 | Updating instance nats: nats/039d66bf-212c-4e95-ac58-b47fe8b8c631 (0) (canary) (00:00:45)
Task 32 | 07:53:48 | Updating instance adapter: adapter/6eae8258-103d-4197-ad98-bd33c2d88aaa (0) (canary) (00:00:46)
Task 32 | 07:53:48 | Updating instance database: database/55b742a6-11bf-4c0f-b74b-ba4640f201fb (0) (canary) (00:01:37)
Task 32 | 07:55:25 | Updating instance diego-api: diego-api/c8ddde37-0813-42e1-9eec-954803605a86 (0) (canary) (00:00:43)
Task 32 | 07:56:08 | Updating instance uaa: uaa/55406075-0455-434b-8a44-8ac91d23634c (0) (canary) (00:01:53)
Task 32 | 07:58:01 | Updating instance singleton-blobstore: singleton-blobstore/0a3064fa-6b6e-4b66-afbf-5fb3280eb886 (0) (canary) (00:00:46)
Task 32 | 07:58:47 | Updating instance api: api/8a7f5660-ef6b-4f78-a5df-c9244b3a817c (0) (canary)
Task 32 | 07:58:47 | Updating instance cc-worker: cc-worker/528eed4d-df16-4b8f-8c07-3615a47d6ac2 (0) (canary) (00:01:43)
Task 32 | 08:01:09 | Updating instance api: api/8a7f5660-ef6b-4f78-a5df-c9244b3a817c (0) (canary) (00:02:22)
Task 32 | 08:01:09 | Updating instance router: router/d20f4994-4979-49f9-8c28-e5e188832bad (0) (canary) (00:01:52)
Task 32 | 08:03:01 | Updating instance tcp-router: tcp-router/6538788b-884c-40f6-81ee-59a2640b9f90 (0) (canary) (00:01:01)
Task 32 | 08:04:02 | Updating instance scheduler: scheduler/cd308352-8bbe-4373-b0da-43dde4dacbbf (0) (canary) (00:01:02)
Task 32 | 08:05:04 | Updating instance doppler: doppler/fab64435-74ed-4e29-a862-310c31a1d240 (0) (canary)
Task 32 | 08:05:04 | Updating instance diego-cell: diego-cell/2abca670-ff56-443e-a2a3-336ff6a34b3c (0) (canary)
Task 32 | 08:06:05 | Updating instance doppler: doppler/fab64435-74ed-4e29-a862-310c31a1d240 (0) (canary) (00:01:01)
Task 32 | 08:07:00 | Updating instance diego-cell: diego-cell/2abca670-ff56-443e-a2a3-336ff6a34b3c (0) (canary) (00:01:56)
Task 32 | 08:07:00 | Updating instance log-api: log-api/a90cd222-4839-40b4-b151-aedfde3793b8 (0) (canary) (00:00:48)
Task 32 | 08:07:48 | Updating instance credhub: credhub/c2827328-019b-44e7-beb6-b1695ea76e33 (0) (canary) (00:01:17)


Task 32 Started  Tue May 21 07:43:05 UTC 2019
Task 32 Finished Tue May 21 08:09:05 UTC 2019
Task 32 Duration 00:26:00
Task 32 done


Succeeded
```


```
$ bosh -e vbox -d paasta vms
Using environment '192.168.50.6' as client 'admin'


Task 33. Done


Deployment 'paasta'


Instance                                                  Process State  AZ  IPs           VM CID                                VM Type  Active
adapter/6eae8258-103d-4197-ad98-bd33c2d88aaa              running        z2  10.244.0.129  8c03470d-20d0-4330-73df-07cd3d263293  minimal  true
api/8a7f5660-ef6b-4f78-a5df-c9244b3a817c                  running        z2  10.244.0.134  a17178fc-95bb-40ef-7978-14a98c68cf7c  small    true
cc-worker/528eed4d-df16-4b8f-8c07-3615a47d6ac2            running        z2  10.244.0.135  68417295-390d-4453-447b-3b677374fb86  minimal  true
credhub/c2827328-019b-44e7-beb6-b1695ea76e33              running        z2  10.244.0.141  594e253d-bf99-4eb6-76f8-e4b01c833eca  default  true
database/55b742a6-11bf-4c0f-b74b-ba4640f201fb             running        z3  10.244.0.130  2f7a5ea0-31d7-4832-737f-b7d5f0126fa0  default  true
diego-api/c8ddde37-0813-42e1-9eec-954803605a86            running        z2  10.244.0.131  412dd27b-3ea5-44fa-4a7b-8efd09023620  small    true
diego-cell/2abca670-ff56-443e-a2a3-336ff6a34b3c           running        z1  10.244.0.139  059b805b-5c98-4818-68b3-5feed0e1ffec  default  true
doppler/fab64435-74ed-4e29-a862-310c31a1d240              running        z1  10.244.0.138  d4e04a0c-6a3f-414a-5b62-f4489be3a807  minimal  true
log-api/a90cd222-4839-40b4-b151-aedfde3793b8              running        z2  10.244.0.140  bbe77860-6062-4de6-64e8-b7a71e04df66  minimal  true
nats/039d66bf-212c-4e95-ac58-b47fe8b8c631                 running        z1  10.244.0.128  98ac8a8f-37b0-4cef-4f41-46d6357f1406  minimal  true
router/d20f4994-4979-49f9-8c28-e5e188832bad               running        z1  10.244.0.34   b64d2bcd-f729-410b-6627-859f52e6d467  minimal  true
scheduler/cd308352-8bbe-4373-b0da-43dde4dacbbf            running        z2  10.244.0.137  a18e5712-0441-4b58-4f70-cfba9804c1f8  small    true
singleton-blobstore/0a3064fa-6b6e-4b66-afbf-5fb3280eb886  running        z3  10.244.0.133  d1aa6083-7249-4e9c-641e-6a3099b14413  small    true
tcp-router/6538788b-884c-40f6-81ee-59a2640b9f90           running        z2  10.244.0.136  d5277d13-6f3b-4d07-61e8-07abfde2415a  minimal  true
uaa/55406075-0455-434b-8a44-8ac91d23634c                  running        z2  10.244.0.132  871edec4-a849-4252-5deb-51e7324ed511  default  true


15 vms


Succeeded

```

https://github.com/cloudfoundry/cf-deployment/blob/master/deployment-guide.md

```
cf api https://api.bosh-lite.com --skip-ssl-validation
export CF_ADMIN_PASSWORD=$(bosh int <(credhub get -n /bosh-lite/cf/cf_admin_password --output-json) --path /value)
cf auth admin $CF_ADMIN_PASSWORD
```

## Commands

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


## cf cli 설치 (Linux)
```
#1. Add the Cloud Foundry Foundation public key and package repository to your system:
$ wget -q -O - https://packages.cloudfoundry.org/debian/cli.cloudfoundry.org.key | sudo apt-key add -

#2. 
$ echo "deb https://packages.cloudfoundry.org/debian stable main" | sudo tee /etc/apt/sources.list.d/cloudfoundry-cli.list

#3. Update your local package index:
$ sudo apt-get update

#4. Install the cf CLI:
$ sudo apt-get install cf-cli

$ cf -v
cf 버전 6.44.1+c3b20bfbe.2019-05-08
```

## cf login
```

$ cf api api.bosh-lite.com --skip-ssl-validation

$ cf login
API 엔드포인트: https://api.bosh-lite.com

Email> admin

Password>
인증 중...
확인

대상 지정된 조직 system



API 엔드포인트:   https://api.bosh-lite.com(API 버전: 2.125.0)
사용자:           admin
조직:             system
영역:             대상 지정된 영역이 없습니다. 'cf target -s SPACE' 명령을 사용하십시오.
```


##  cf create-org 
```
$ cf create-org test-org

$ cf create-space test-space -o test-org

$ cf create-user test test1234

$ cf set-org-role test test-org OrgManager

$ cf set-space-role test test-org test-space SpaceManager

$ cf set-space-role test test-org test-space SpaceDeveloper

```

## cf login test user
```
$ cf login 
API 엔드포인트: https://api.bosh-lite.com

Email> test

Password>
인증 중...
확인

대상 지정된 조직 test-org

대상 지정된 영역 test-space



API 엔드포인트:   https://api.bosh-lite.com(API 버전: 2.125.0)
사용자:           test
조직:             test-org
```

## Java 8 install
```

$ sudo apt-get update

$ sudo apt install openjdk-8-jdk

$ java -version
openjdk version "1.8.0_212"
OpenJDK Runtime Environment (build 1.8.0_212-8u212-b03-0ubuntu1.18.04.1-b03)
OpenJDK 64-Bit Server VM (build 25.212-b03, mixed mode)
```


## Sample app test
```
$ mkdir ~/workspace/sample

& cd ~/workspace/sample

$ git clone https://github.com/cloudfoundry-samples/spring-music.git

$ cd spring-music/

$ ./gradlew clean assemble
```


## cf push
```
Manifest에서 test(으)로 test-org 조직/test-space 영역에 푸시 중...
Manifest 파일 /home/ubuntu/workspace/sample/spring-music/manifest.yml 사용
앱 정보를 가져오는 중...
이러한 속성의 앱 작성 중...
+ 이름:     spring-music
  경로:     /home/ubuntu/workspace/sample/spring-music/build/libs/spring-music-1.0.jar
+ 메모리:   1G
  라우트:
+   spring-music-reflective-nyala.bosh-lite.com


spring-music 앱 작성 중...
라우트 맵핑 중...
로컬 파일을 원격 캐시와 비교 중...
Packaging files to upload...
파일 업로드 중...
 40.80 MiB / 40.80 MiB [============================================================================================================================================================] 100.00% 2s


API의 파일 처리가 완료되기를 기다리는 중...


앱 스테이징 및 로그 추적 중...
   Downloading binary_buildpack...
   Downloading dotnet_core_buildpack...
   Downloading staticfile_buildpack...
   Downloading java_buildpack...
   Downloaded java_buildpack (207.1K)
   Downloading ruby_buildpack...
   Downloaded staticfile_buildpack (4.1M)
   Downloading go_buildpack...
   Downloaded binary_buildpack (3.5M)
   Downloading nodejs_buildpack...
   Downloaded dotnet_core_buildpack (3.9M)
   Downloading python_buildpack...
   Downloaded ruby_buildpack (4.5M)
   Downloading php_buildpack...
   Downloaded go_buildpack (4.5M)
   Downloaded php_buildpack (592.8K)
   Downloaded nodejs_buildpack (3.8M)
   Downloaded python_buildpack (3.7M)
   Cell 2abca670-ff56-443e-a2a3-336ff6a34b3c creating container for instance 966ab8a7-e8a3-4c32-846f-bbf263b0d76d
   Cell 2abca670-ff56-443e-a2a3-336ff6a34b3c successfully created container for instance 966ab8a7-e8a3-4c32-846f-bbf263b0d76d
   Downloading app package...
   Downloaded app package (40.8M)
   -----> Java Buildpack v4.16.1 | https://github.com/cloudfoundry/java-buildpack.git#41b8ff8
   -----> Downloading Jvmkill Agent 1.16.0_RELEASE from https://java-buildpack.cloudfoundry.org/jvmkill/trusty/x86_64/jvmkill-1.16.0-RELEASE.so (0.5s)
   -----> Downloading Open Jdk JRE 1.8.0_212 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-jre-1.8.0_212-trusty.tar.gz (3.8s)
          Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.5s)
          JVM DNS caching disabled in lieu of BOSH DNS caching
   -----> Downloading Open JDK Like Memory Calculator 3.13.0_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-3.13.0-RELEASE.tar.gz (0.2s)
          Loaded Classes: 17933, Threads: 250
   -----> Downloading Client Certificate Mapper 1.8.0_RELEASE from https://java-buildpack.cloudfoundry.org/client-certificate-mapper/client-certificate-mapper-1.8.0-RELEASE.jar (0.1s)
   -----> Downloading Container Security Provider 1.16.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-security-provider/container-security-provider-1.16.0-RELEASE.jar (0.3s)
   -----> Downloading Spring Auto Reconfiguration 2.7.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-2.7.0-RELEASE.jar (0.5s)
   Exit status 0
   Uploading droplet, build artifacts cache...
   Uploading droplet...
   Uploading build artifacts cache...
   Uploaded build artifacts cache (42.9M)
   Uploaded droplet (83.8M)
   Uploading complete
   Cell 2abca670-ff56-443e-a2a3-336ff6a34b3c stopping instance 966ab8a7-e8a3-4c32-846f-bbf263b0d76d
   Cell 2abca670-ff56-443e-a2a3-336ff6a34b3c destroying container for instance 966ab8a7-e8a3-4c32-846f-bbf263b0d76d
   Cell 2abca670-ff56-443e-a2a3-336ff6a34b3c successfully destroyed container for instance 966ab8a7-e8a3-4c32-846f-bbf263b0d76d


앱이 시작되기를 기다리는 중...


이름:                  spring-music
요청된 상태:           started
라우트:                spring-music-reflective-nyala.bosh-lite.com
마지막으로 업로드함:   Tue 21 May 18:14:23 KST 2019
스택:                  cflinuxfs2
빌드팩:                client-certificate-mapper=1.8.0_RELEASE container-security-provider=1.16.0_RELEASE
                       java-buildpack=v4.16.1-https://github.com/cloudfoundry/java-buildpack.git#41b8ff8 java-main java-opts java-security jvmkill-agent=1.16.0_RELEASE
                       open-jdk-like-j...


유형:          web
인스턴스:      1/1
메모리 사용:   1024M
시작 명령:     JAVA_OPTS="-agentpath:$PWD/.java-buildpack/open_jdk_jre/bin/jvmkill-1.16.0_RELEASE=printHeapHistogram=1 -Djava.io.tmpdir=$TMPDIR
               -Djava.ext.dirs=$PWD/.java-buildpack/container_security_provider:$PWD/.java-buildpack/open_jdk_jre/lib/ext
               -Djava.security.properties=$PWD/.java-buildpack/java_security/java.security $JAVA_OPTS" &&
               CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-3.13.0_RELEASE -totMemory=$MEMORY_LIMIT -loadedClasses=19233 -poolType=metaspace
               -stackThreads=250 -vmOptions="$JAVA_OPTS") && echo JVM Memory Configuration: $CALCULATED_MEMORY && JAVA_OPTS="$JAVA_OPTS $CALCULATED_MEMORY" && MALLOC_ARENA_MAX=2
               SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.JarLauncher
     상태      이후                   CPU     메모리        디스크      세부사항
#0   실행 중   2019-05-21T09:15:02Z   73.8%   149.9M / 1G   155M / 1G


$ curl spring-music-reflective-nyala.bosh-lite.com

```

https://github.com/cloudfoundry-incubator/service-fabrik-broker/wiki/Bootstrap-BOSH-2.0-with-local-VirtualBox


## VirtualBox 저장 및 시작 
```
# To save state of vm
vboxmanage controlvm $(bosh int ~/deployments/vbox/state.json --path /current_vm_cid) savestate

# To start vm
vboxmanage startvm $(bosh int ~/deployments/vbox/state.json --path /current_vm_cid) --type headless
```


