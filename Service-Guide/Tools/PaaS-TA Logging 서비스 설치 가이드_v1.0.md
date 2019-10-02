## Table of Contents

[1. 문서 개요](#1)

  - [1.1. 목적](#1.1)
  - [1.2. 범위](#1.2)
  - [1.3. 시스템 구성](#1.3)
  - [1.4. 참고자료](#1.4)

[2. Logging 서비스 설치](#2)

  - [2.1. 설치 전 준비 사항](#2.1)
  - [2.1.1. Logging 서비스 설치 파일 다운로드](#2.1.1)
  - [2.1.2. Stemcell 다운로드](#2.1.2)
  - [2.2. Logging 서비스 릴리즈 업로드](#2.2)
  - [2.3. Logging 서비스 Deployment 파일 수정 및 배포](#2.3)

[3. Logging 서비스 관리](#3)

  - [3.1. Logging 서비스 UAA Client 등록](#3.1)
  - [3.2. Logging 서비스 사용 활성화](#3.2)



## <div id="1"/> 1. 문서 개요

### <div id="1.1"/> 1.1. 목적

본 문서는 Logging 서비스 Release를 Bosh2.0을 이용하여 설치 하는 방법을 기술하였다.

### <div id="1.2"/> 1.2. 범위

설치 범위는 Logging 서비스 Release를 검증하기 위한 기본 설치를 기준으로 작성하였다.

### <div id="1.3"/> 1.3. 시스템 구성

본 장에서는 Logging 서비스의 시스템 구성에 대해 기술하였다. Logging 서비스 시스템은 Router, Collector, Queue, Parser, Elasticsearch, Visualization의 최소사항을 구성하였다.  
![001]

VM명 | 인스턴스 수 | vCPU수 | 메모리(GB) | 디스크(GB)
:--- | :---: | :---: | :---:| :---
Router | 1 | 1 |1 | Root 8G
Collector | 1 | 1 | 2 | Root 10G
Queue | 1 | 1 | 2 |  Root 10G + Persistent disk 10G
Parser | N | 1 | 2 | Root 10G
Elasticsearch Master | 1 | 1 | 2 | Root 10G + Persistent disk 10G
Elasticsearch Data | N | 2 | 4 | Root 20G + Persistent disk 30G
Visualization | 1 |  1 | 2 | Root 10G
maintenance | 1 | 1 | 1 | Root 8G

### <div id="1.4"/> 1.4. 참고자료
> http://bosh.io/docs  
> http://docs.cloudfoundry.org/  

## <div id="2"/> 2. Logging 서비스 설치  

### <div id="2.1"/> 2.1. 설치 전 준비 사항  

본 설치 가이드는 Linux 환경에서 설치하는 것을 기준으로 하였다. Logging 서비스 설치를 위해서는 먼저 BOSH CLI v2 가 설치 되어 있어야 하고 BOSH 에 로그인 되어 있어야 한다.
BOSH CLI v2 가 설치 되어 있지 않을 경우 먼저 BOSH2.0 설치 가이드 문서를 참고 하여 BOSH CLI v2를 설치해야 한다.

>BOSH2 사용자 가이드 : https://github.com/PaaS-TA/Guide-4.0-ROTELLE/blob/master/PaaS-TA_BOSH2_사용자_가이드v1.0.md  
>
>BOSH CLI V2 사용자 가이드 : https://github.com/PaaS-TA/Guide-4.0-ROTELLE/blob/master/Use-Guide/Bosh/PaaS-TA_BOSH_CLI_V2_사용자_가이드v1.0.md  

※ 설치 전 확인 사항

uaac client에 "firehose-to-syslog"가 등록되어 있는지 확인 하여, 등록되어 있는 경우에는 "authorities"를 확인하여 "cloud_controller.admin" 권한을 부여한다.
```
# endpoint 설정
$ uaac target https://uaa.<DOMAIN> --skip-ssl-validation

# target 확인
$ uaac target
Target: https://uaa.<DOMAIN>
Context: uaa_admin, from client uaa_admin

# uaac 로그인
$ uaac token client get <UAA_ADMIN_CLIENT_ID> -s <UAA_ADMIN_CLIENT_SECRET>

# "firehose-to-syslog" uaac client 확인
$ uaac client get firehose-to-syslog
scope: cloud_controller.admin_read_only cloud_controller.global_auditor openid routing.router_groups.write network.write scim.read cloud_controller.admin uaa.user cloud_controller.read
    password.write routing.router_groups.read cloud_controller.write network.admin doppler.firehose scim.write
client_id: firehose-to-syslog
resource_ids: none
authorized_grant_types: client_credentials
autoapprove: 
authorities: uaa.none doppler.firehose                   >>>>>>>>  cloud_controller.admin 권한 여부 확인
lastmodified: 1552530293656

# "firehose-to-syslog" uaac client 변경
$ uaac client update firehose-to-syslog --authorities "doppler.firehose, uaa.none, cloud_controller.admin"

# "firehose-to-syslog" uaac client 확인
$ uaac client get firehose-to-syslog
scope: cloud_controller.admin_read_only cloud_controller.global_auditor openid routing.router_groups.write network.write scim.read cloud_controller.admin uaa.user cloud_controller.read
    password.write routing.router_groups.read cloud_controller.write network.admin doppler.firehose scim.write
client_id: firehose-to-syslog
resource_ids: none
authorized_grant_types: client_credentials
autoapprove: 
authorities: uaa.none doppler.firehose cloud_controller.admin
lastmodified: 1552530293656
```

### <div id="2.1.1"/> 2.1.1 Logging 서비스 설치 파일 다운로드

Logging 서비스 설치에 필요한 Deployment 및 릴리즈 파일을 다운로드 받아 서비스 설치 작업 경로로 위치시킨다.

-	설치 파일 다운로드 위치 : https://paas-ta.kr/download/package  
  = Deployment : paasta-logging-service-2.0  
  = 릴리즈 파일 : paasta-logging-service-release.tgz

-	설치 작업 경로 생성 및 파일 다운로드

```
# Deployment 다운로드 파일 위치 경로 생성
$ mkdir -p ~/workspace/paasta-4.6/deployment/service-deployment

# Deployment 다운로드(paasta-logging-service-2.0) 및 파일 경로 확인
$ ls ~/workspace/paasta-4.6/deployment/service-deployment/paasta-logging-service-2.0
logging-service-deploy.sh  remove-logging-service-deployment.sh  manifests

# 릴리즈 다운로드 파일 위치 경로 생성
$ mkdir -p ~/workspace/paasta-4.6/release/service

# 릴리즈 파일 다운로드(paasta-logging-service-release.tgz) 및 파일 경로 확인
$ ls ~/workspace/paasta-4.6/release/service
paasta-logging-service-release.tgz

```

### <div id="2.1.2"/> 2.2.2 Stemcell 다운로드

Logging 서비스 설치에 필요한 을 확인하여 존재하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell을 업로드 한다. 

-	설치 파일 다운로드 위치 : https://paas-ta.kr/download/package

```
# Stemcell 목록 확인
- **사용 예시**

		$ bosh -e micro-bosh stemcells
		Name                                       Version  OS             CPI  CID  
		bosh-openstack-kvm-ubuntu-xenial-go_agent  315.41*  ubuntu-xenial  -    fb08e389-2350-4091-9b29-41743495e62c  
		~                                          315.36*  ubuntu-xenial  -    7076cf5d-a473-4c46-b6c1-4a7813911f76  

		(*) Currently deployed

		2 stemcells

		Succeeded
```

### <div id="2.2"/> 2.2. Logging 서비스 릴리즈 업로드

- 릴리즈 목록을 확인하여 Logging 서비스 릴리즈(paasta-logging-service-release)가 업로드 되어 있지 않은 것을 확인한다.

```
# 릴리즈 목록 확인
$ bosh -e micro-bosh releases

Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

Name                                  Version   Commit Hash  
paas-ta-portal-release                2.1*      3a5494a9+  

(*) Currently deployed
(+) Uncommitted changes

1 releases

Succeeded
```

- Logging 서비스 릴리즈 파일을 업로드한다.

```
# 릴리즈 파일 업로드
$ bosh -e micro-bosh upload-release ~/workspace/paasta-4.6/release/service/paasta-logging-service-release.tgz
Using environment '10.30.40.111' as client 'admin'

######################################################### 100.00% 92.74 MiB/s 0s
Task 60178

Task 60178 | 09:21:15 | Extracting release: Extracting release (00:00:01)
Task 60178 | 09:21:16 | Verifying manifest: Verifying manifest (00:00:00)
Task 60178 | 09:21:17 | Resolving package dependencies: Resolving package dependencies (00:00:00)
Task 60178 | 09:21:17 | Creating new packages: cerebro/278996853981243e3b921d18e255d55d998dcc7f (00:00:05)
Task 60178 | 09:21:22 | Creating new packages: curator/27ee549a38ebcef009bb0e63ac716d544dc8caa2 (00:00:01)
Task 60178 | 09:21:23 | Creating new packages: elasticsearch/cb154811849156850a88023c3dd1fd46698a09da (00:00:02)
Task 60178 | 09:21:25 | Creating new packages: firehose-to-syslog/57480e93d19cefa2f26dfa787419d378b2104b13 (00:00:00)
Task 60178 | 09:21:25 | Creating new packages: golang/fb3379fa5b3afb2e6e515657a33f2145ba4bd076 (00:00:08)
Task 60178 | 09:21:33 | Creating new packages: haproxy/1d59aa8ae61da532ade4e9f5129428bbc101be64 (00:00:00)
Task 60178 | 09:21:33 | Creating new packages: java8/ae41eecf1175fb16c678940d4d6e31af10405b6e (00:00:05)
Task 60178 | 09:21:38 | Creating new packages: kibana/0761dbc496f63511e66971283aca8ce6a329184d (00:00:00)
Task 60178 | 09:21:38 | Creating new packages: logsearch-config/4cd3b3efcb7ea642badd77ff3b5189ae1d324734 (00:00:00)
Task 60178 | 09:21:38 | Creating new packages: logstash/a5bc4ac7fb0f561db9b9e03833510b866236af8e (00:00:09)
Task 60178 | 09:21:47 | Creating new packages: nats_to_syslog/f3267716fa78721e65fc1b804684dd6c4bd3de9d (00:00:00)
Task 60178 | 09:21:47 | Creating new packages: python3/2a00051b1a0f02d70453c75e9036d41cb960ebfc (00:00:00)
Task 60178 | 09:21:47 | Creating new packages: redis/ff3e314387f91116dd8906cd656ecd0476b4c7b5 (00:00:01)
Task 60178 | 09:21:48 | Creating new packages: ruby2.3/613589acad7ddafc74fd12316e3d9cf5346f612e (00:00:00)
Task 60178 | 09:21:48 | Creating new jobs: archiver/5a7d857eb183097dcc7ea30625833ef3954f3752 (00:00:00)
Task 60178 | 09:21:48 | Creating new jobs: cerebro/299ff8638f7a21bc91724c0178a794304b54677e (00:00:00)
Task 60178 | 09:21:48 | Creating new jobs: curator/5c1a54bf651dbe118cd5b30699ae632539f60f5c (00:00:00)
Task 60178 | 09:21:48 | Creating new jobs: elasticsearch/7ad0042f72431c91b792dd14ca73825000f19e5c (00:00:00)
Task 60178 | 09:21:48 | Creating new jobs: elasticsearch_config/26db3d1920ec89bc31abb89213ec5b22bf0d7d53 (00:00:01)
Task 60178 | 09:21:49 | Creating new jobs: enable_shard_allocation/73b25c5ebb749d4599c2b7301d7955a4ba825ed5 (00:00:00
Task 60178 | 09:21:49 | Creating new jobs: haproxy/a4457814c0d587a54fcdd98b4a589971beb76dae (00:00:00)
Task 60178 | 09:21:49 | Creating new jobs: ingestor_archiver/8f02f568ef4642da6ea22288249e08c55623b42e (00:00:00)
Task 60178 | 09:21:49 | Creating new jobs: ingestor_cloudfoundry-firehose/5d99bea992d0116b0c746c3ddb96e6522e227492 (0
Task 60178 | 09:21:49 | Creating new jobs: ingestor_syslog/599509fad62f7a665d9403f6260bf7658b1d3271 (00:00:00)
Task 60178 | 09:21:49 | Creating new jobs: kibana/3b9fe3747baf507022c19c68b73c2ec65cf938d2 (00:00:00)
Task 60178 | 09:21:49 | Creating new jobs: nats_to_syslog/c7c59d33f131fb3c5f8e91d87512f7499d9ca176 (00:00:00)
Task 60178 | 09:21:49 | Creating new jobs: parser/c7c15e32ef7a09d68b76f836b7d3925ba3e7f8de (00:00:00)
Task 60178 | 09:21:49 | Creating new jobs: queue/fdad522d4253adbe4cb15c4739a9801ff2f69e17 (00:00:00)
Task 60178 | 09:21:49 | Creating new jobs: smoke_tests/71cc53e522099218784e2bb5b703e971417e8fc6 (00:00:01)
Task 60178 | 09:21:50 | Creating new jobs: syslog_forwarder/96d6bb20fbd03bcbe7b5019bc5dd3347d0ebd3e7 (00:00:00)
Task 60178 | 09:21:50 | Release has been created: paasta-logging-service-release/1.0 (00:00:00)

Task 60178 Started  Fri Nov  2 09:21:15 UTC 2018
Task 60178 Finished Fri Nov  2 09:21:50 UTC 2018
Task 60178 Duration 00:00:35
Task 60178 done

Succeeded

```

- 릴리즈 목록을 확인하여 Logging 서비스 릴리즈(paasta-logging-service-release)가 업로드 되어 있는 것을 확인한다.

```
# 릴리즈 목록 확인
$ bosh -e micro-bosh releases
Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

Name                                  Version   Commit Hash  
paas-ta-portal-release                2.1*      3a5494a9+
paasta-logging-service-release        1.0       ede7603+    

(*) Currently deployed
(+) Uncommitted changes

2 releases

Succeeded

```

### <div id="2.3"/> 2.3. Logging 서비스 Deployment 파일 수정 및 배포

BOSH Deployment manifest는 Components 요소 및 배포의 속성을 정의한 YAML 파일이다.
Deployment 파일에서 사용하는 network, vm_type, disk_type 등은 Cloud config를 활용하고, 활용 방법은 BOSH 2.0 가이드를 참고한다.

-	Cloud config 설정 내용을 확인한다.

```
# Cloud config 조회
$ bosh -e micro-bosh cloud-config
Using environment '10.30.40.111' as client 'admin'

azs:
- cloud_properties:
    datacenters:
    - clusters:
      - BD-HA:
          resource_pool: CF_BOSH2_Pool
      name: BD-HA
  name: z1
- cloud_properties:
    datacenters:
    - clusters:
      - BD-HA:
          resource_pool: CF_BOSH2_Pool
      name: BD-HA
  name: z2
- cloud_properties:
    datacenters:
    - clusters:
      - BD-HA:
          resource_pool: CF_BOSH2_Pool
      name: BD-HA
  name: z3
- cloud_properties:
    datacenters:
    - clusters:
      - BD-HA:
          resource_pool: CF_BOSH2_Pool
      name: BD-HA
  name: z4
- cloud_properties:
    datacenters:
    - clusters:
      - BD-HA:
          resource_pool: CF_BOSH2_Pool
      name: BD-HA
  name: z5
- cloud_properties:
    datacenters:
    - clusters:
      - BD-HA:
          resource_pool: CF_BOSH2_Pool
      name: BD-HA
  name: z6
compilation:
  az: z1
  network: default
  reuse_compilation_vms: true
  vm_type: large
  workers: 5
disk_types:
- disk_size: 1024
  name: default
- disk_size: 1024
  name: 1GB
- disk_size: 2048
  name: 2GB
- disk_size: 4096
  name: 4GB
- disk_size: 5120
  name: 5GB
- disk_size: 8192
  name: 8GB
- disk_size: 10240
  name: 10GB
- disk_size: 20480
  name: 20GB
- disk_size: 30720
  name: 30GB
- disk_size: 51200
  name: 50GB
- disk_size: 102400
  name: 100GB
- disk_size: 1048576
  name: 1TB
networks:
- name: default
  subnets:
  - azs:
    - z1
    - z2
    - z3
    - z4
    - z5
    - z6
    cloud_properties:
      name: Internal
    dns:
    - 8.8.8.8
    gateway: 10.30.20.23
    range: 10.30.0.0/16
    reserved:
    - 10.30.0.0 - 10.30.111.40
- name: public
  subnets:
  - azs:
    - z1
    - z2
    - z3
    - z4
    - z5
    - z6
    cloud_properties:
      name: External
    dns:
    - 8.8.8.8
    gateway: 115.68.46.177
    range: 115.68.46.176/28
    reserved:
    - 115.68.46.176 - 115.68.46.188
    static:
    - 115.68.46.189 - 115.68.46.190
  type: manual
- name: service_private
  subnets:
  - azs:
    - z1
    - z2
    - z3
    - z4
    - z5
    - z6
    cloud_properties:
      name: Internal
    dns:
    - 8.8.8.8
    gateway: 10.30.20.23
    range: 10.30.0.0/16
    reserved:
    - 10.30.0.0 - 10.30.106.255
    static:
    - 10.30.107.1 - 10.30.107.255
- name: service_public
  subnets:
  - azs:
    - z1
    - z2
    - z3
    - z4
    - z5
    - z6
    cloud_properties:
      name: External
    dns:
    - 8.8.8.8
    gateway: 115.68.47.161
    range: 115.68.47.160/24
    reserved:
    - 115.68.47.161 - 115.68.47.174
    static:
    - 115.68.47.175 - 115.68.47.185
  type: manual
- name: portal_service_public
  subnets:
  - azs:
    - z1
    - z2
    - z3
    - z4
    - z5
    - z6
    cloud_properties:
      name: External
    dns:
    - 8.8.8.8
    gateway: 115.68.46.209
    range: 115.68.46.208/28
    reserved:
    - 115.68.46.217 - 115.68.46.222
    static:
    - 115.68.46.214 - 115.68.46.216
  type: manual
vm_extensions:
- cloud_properties:
    ports:
    - host: 3306
  name: mysql-proxy-lb
- name: cf-router-network-properties
- name: cf-tcp-router-network-properties
- name: diego-ssh-proxy-network-properties
- name: cf-haproxy-network-properties
- cloud_properties:
    disk: 51200
  name: small-50GB
- cloud_properties:
    disk: 102400
  name: small-highmem-100GB
vm_types:
- cloud_properties:
    cpu: 1
    disk: 8192
    ram: 1024
  name: minimal
- cloud_properties:
    cpu: 1
    disk: 10240
    ram: 2048
  name: default
- cloud_properties:
    cpu: 1
    disk: 30720
    ram: 4096
  name: small
- cloud_properties:
    cpu: 2
    disk: 20480
    ram: 4096
  name: medium
- cloud_properties:
    cpu: 2
    disk: 20480
    ram: 8192
  name: medium-memory-8GB
- cloud_properties:
    cpu: 4
    disk: 20480
    ram: 8192
  name: large
- cloud_properties:
    cpu: 8
    disk: 20480
    ram: 16384
  name: xlarge
- cloud_properties:
    cpu: 2
    disk: 51200
    ram: 4096
  name: small-50GB
- cloud_properties:
    cpu: 2
    disk: 51200
    ram: 4096
  name: small-50GB-ephemeral-disk
- cloud_properties:
    cpu: 4
    disk: 102400
    ram: 8192
  name: small-100GB-ephemeral-disk
- cloud_properties:
    cpu: 4
    disk: 102400
    ram: 8192
  name: small-highmem-100GB-ephemeral-disk
- cloud_properties:
    cpu: 8
    disk: 20480
    ram: 16384
  name: small-highmem-16GB
- cloud_properties:
    cpu: 1
    disk: 4096
    ram: 2048
  name: caas_small
- cloud_properties:
    cpu: 1
    disk: 4096
    ram: 1024
  name: caas_small_api
- cloud_properties:
    cpu: 1
    disk: 4096
    ram: 4096
  name: caas_medium
- cloud_properties:
    cpu: 1
    disk: 4096
    ram: 256
  name: service_tiny
- cloud_properties:
    cpu: 1
    disk: 4096
    ram: 512
  name: service_small
- cloud_properties:
    cpu: 1
    disk: 4096
    ram: 1024
  name: service_medium
- cloud_properties:
    cpu: 1
    disk: 4096
    ram: 2048
  name: service_medium_1CPU_2G
- cloud_properties:
    cpu: 2
    disk: 8192
    ram: 4096
  name: service_medium_4G
- cloud_properties:
    cpu: 2
    disk: 10240
    ram: 2048
  name: service_medium_2G
- cloud_properties:
    cpu: 1
    disk: 4096
    ram: 256
  name: portal_tiny
- cloud_properties:
    cpu: 1
    disk: 4096
    ram: 512
  name: portal_small
- cloud_properties:
    cpu: 1
    disk: 4096
    ram: 1024
  name: portal_medium
- cloud_properties:
    cpu: 1
    disk: 4096
    ram: 2048
  name: portal_large

Succeeded

```

- Deployment YAML에서 사용하는 변수들을 서버 환경에 맞게 수정한다.

```
# 변수 설정
$ vi ~/workspace/paasta-4.6/deployment/service-deployment/paasta-logging-service-2.0/manifests/vars.yml
# RELEASE
logging_service_release_name: "paasta-logging-service-release"
logging_service_release_version: "1.0"

# STEMCELL
stemcell_os: "ubuntu-xenial"
stemcell_version: "315.41"

# VM_TYPE
vm_type_minimal: "minimal"
vm_type_default: "default"
vm_type_medium: "medium"

# NETWORK
private_network_name: "service_private"
public_network_name: "service_public"
private_nat_network_name: "default"                # AWS의 경우 nat network

# ELASTICSEARCH_MASTER
es_master_azs: [z5]
es_master_instances: 1
es_master_persistent_disk_type: "10GB"
es_master_private_ips: ["10.30.107.135"]
es_master_private_url: "10.30.107.135"

# QUEUE
queue_azs: [z5]
queue_instances: 1
queue_persistent_disk_type: "10GB"
queue_private_ips: ["10.30.107.139"]
queue_private_url: "10.30.107.139"

# MAINTENANCE
maintenance_azs: [z5]
maintenance_instances: 1
maintenance_private_ips: ["10.30.107.136"]

# ELASTICSEARCH_DATA
es_data_azs: [z5]
es_data_instances: 2
es_data_persistent_disk_type: "20GB"
es_data_private_ips: ["10.30.107.133", "10.30.107.134"]

# VISUALIZATION
visualization_azs: [z5]
visualization_instances: 1
visualization_private_ips: ["10.30.107.143"]
visualization_version:  "5.3.0"

# COLLECTOR
collector_azs: [z5]
collector_instances: 1
collector_private_ips: ["10.30.107.131"]

# PARSER
parser_azs: [z5]
parser_instances: 2
parser_private_ips: ["10.30.107.137", "10.30.107.138"]
parser_es_index: "%{[@metadata][index]}-%{+YYYY.MM.dd.HH}"
parser_es_index_type: '%{[@metadata][type]}'


# ROUTER
router_azs: [z5]
router_instances: 1
router_private_ips: ["10.30.107.140"]
router_public_ips: "115.68.47.181"
router_private_url: "10.30.107.140"

# CF
cf_client_id : "laasclient"
cf_client_secret : "clientsecret”
system_domain: "<DOMAIN>"

# LOGGING SERVICE
es_config_index_prefix: "laas-"
retention_period: 7
laas_logo: "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAALQAAABGCAYAAABll74gAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAA3FpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuNi1jMTQyIDc5LjE2MDkyNCwgMjAxNy8wNy8xMy0wMTowNjozOSAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0ieG1wLmRpZDpkYzhkNmI4YS1kZWNkLThkNGItOThiNC0zMjRjZjU1OTE0NmYiIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6MkRBNUQ0REMyQ0EwMTFFOEI2QkZBQUY2QUYwM0UzRjkiIHhtcE1NOkluc3RhbmNlSUQ9InhtcC5paWQ6MkRBNUQ0REIyQ0EwMTFFOEI2QkZBQUY2QUYwM0UzRjkiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENDIChXaW5kb3dzKSI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjQ5YzFjOTUyLWE2MDEtZTI0NC1iMzIzLTMzYTIyNzI1NDcxYiIgc3RSZWY6ZG9jdW1lbnRJRD0ieG1wLmRpZDpkYzhkNmI4YS1kZWNkLThkNGItOThiNC0zMjRjZjU1OTE0NmYiLz4gPC9y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"

```

-	Deploy 스크립트 파일을 서버 환경에 맞게 수정한다.
  - vSphere : -o manifests/ops-files/vsphere-network.yml
  - AWS : -o manifests/ops-files/aws-network.yml
  - OpenStack : -o manifests/ops-files/openstack-network.yml

```
# Deploy 스크립트 수정
$ vi ~/workspace/paasta-4.6/deployment/service-deployment/paasta-logging-service-2.0/logging-service-deploy.sh
#!/bin/bash

# SET VARIABLES
export LOGGING_SERVICE_DEPLOYMENT_NAME='paasta-logging-service'
export BOSH2_NAME='micro-bosh'

bosh -e ${BOSH2_NAME} -d ${LOGGING_SERVICE_DEPLOYMENT_NAME} deploy --no-redact manifests/paasta_logging_service.yml \
    -l manifests/vars.yml \
    -o manifests/ops-files/vsphere-network.yml \
    -v uaa_admin_client_id=<UAA_ADMIN_CLIENT_ID> \
    -v uaa_admin_client_secret=<UAA_ADMIN_CLIENT_SECRET>

```

- Logging 서비스를 배포한다.

```
# Logging 서비스 Deploy (e.g vSphere)
$ cd ~/workspace/paasta-4.6/deployment/service-deployment/paasta-logging-service-2.0
$ ./logging-service-deploy.sh
Using environment '10.30.40.111' as client 'admin'

Using deployment 'paasta-logging-service'

+ azs:
+ - cloud_properties:
+     datacenters:
+     - clusters:
+       - BD-HA:
+           resource_pool: CF_BOSH2_Pool
+       name: BD-HA
+   name: z1
+ - cloud_properties:
+     datacenters:
+     - clusters:
+       - BD-HA:
+           resource_pool: CF_BOSH2_Pool
+       name: BD-HA
+   name: z2
+ - cloud_properties:
+     datacenters:
+     - clusters:
+       - BD-HA:
+           resource_pool: CF_BOSH2_Pool
+       name: BD-HA
+   name: z3
+ - cloud_properties:
+     datacenters:
+     - clusters:
+       - BD-HA:
+           resource_pool: CF_BOSH2_Pool
+       name: BD-HA
+   name: z4
+ - cloud_properties:
+     datacenters:
+     - clusters:
+       - BD-HA:
+           resource_pool: CF_BOSH2_Pool
+       name: BD-HA
+   name: z5
+ - cloud_properties:
+     datacenters:
+     - clusters:
+       - BD-HA:
+           resource_pool: CF_BOSH2_Pool
+       name: BD-HA
+   name: z6

+ vm_types:
+ - cloud_properties:
+     cpu: 1
+     disk: 8192
+     ram: 1024
+   name: minimal
+ - cloud_properties:
+     cpu: 1
+     disk: 10240
+     ram: 2048
+   name: default
+ - cloud_properties:
+     cpu: 1
+     disk: 30720
+     ram: 4096
+   name: small
+ - cloud_properties:
+     cpu: 2
+     disk: 20480
+     ram: 4096
+   name: medium
+ - cloud_properties:
+     cpu: 2
+     disk: 20480
+     ram: 8192
+   name: medium-memory-8GB
+ - cloud_properties:
+     cpu: 4
+     disk: 20480
+     ram: 8192
+   name: large
+ - cloud_properties:
+     cpu: 8
+     disk: 20480
+     ram: 16384
+   name: xlarge
+ - cloud_properties:
+     cpu: 2
+     disk: 51200
+     ram: 4096
+   name: small-50GB
+ - cloud_properties:
+     cpu: 2
+     disk: 51200
+     ram: 4096
+   name: small-50GB-ephemeral-disk
+ - cloud_properties:
+     cpu: 4
+     disk: 102400
+     ram: 8192
+   name: small-100GB-ephemeral-disk
+ - cloud_properties:
+     cpu: 4
+     disk: 102400
+     ram: 8192
+   name: small-highmem-100GB-ephemeral-disk
+ - cloud_properties:
+     cpu: 8
+     disk: 20480
+     ram: 16384
+   name: small-highmem-16GB
+ - cloud_properties:
+     cpu: 1
+     disk: 4096
+     ram: 2048
+   name: caas_small
+ - cloud_properties:
+     cpu: 1
+     disk: 4096
+     ram: 1024
+   name: caas_small_api
+ - cloud_properties:
+     cpu: 1
+     disk: 4096
+     ram: 4096
+   name: caas_medium
+ - cloud_properties:
+     cpu: 1
+     disk: 4096
+     ram: 256
+   name: service_tiny
+ - cloud_properties:
+     cpu: 1
+     disk: 4096
+     ram: 512
+   name: service_small
+ - cloud_properties:
+     cpu: 1
+     disk: 4096
+     ram: 1024
+   name: service_medium
+ - cloud_properties:
+     cpu: 1
+     disk: 4096
+     ram: 2048
+   name: service_medium_1CPU_2G
+ - cloud_properties:
+     cpu: 2
+     disk: 8192
+     ram: 4096
+   name: service_medium_4G
+ - cloud_properties:
+     cpu: 2
+     disk: 10240
+     ram: 2048
+   name: service_medium_2G
+ - cloud_properties:
+     cpu: 1
+     disk: 4096
+     ram: 256
+   name: portal_tiny
+ - cloud_properties:
+     cpu: 1
+     disk: 4096
+     ram: 512
+   name: portal_small
+ - cloud_properties:
+     cpu: 1
+     disk: 4096
+     ram: 1024
+   name: portal_medium
+ - cloud_properties:
+     cpu: 1
+     disk: 4096
+     ram: 2048
+   name: portal_large

+ vm_extensions:
+ - cloud_properties:
+     ports:
+     - host: 3306
+   name: mysql-proxy-lb
+ - name: cf-router-network-properties
+ - name: cf-tcp-router-network-properties
+ - name: diego-ssh-proxy-network-properties
+ - name: cf-haproxy-network-properties
+ - cloud_properties:
+     disk: 51200
+   name: small-50GB
+ - cloud_properties:
+     disk: 102400
+   name: small-highmem-100GB

+ compilation:
+   az: z1
+   network: default
+   reuse_compilation_vms: true
+   vm_type: large
+   workers: 5

+ networks:
+ - name: default
+   subnets:
+   - azs:
+     - z1
+     - z2
+     - z3
+     - z4
+     - z5
+     - z6
+     cloud_properties:
+       name: Internal
+     dns:
+     - 8.8.8.8
+     gateway: 10.30.20.23
+     range: 10.30.0.0/16
+     reserved:
+     - 10.30.0.0 - 10.30.111.40
+ - name: public
+   subnets:
+   - azs:
+     - z1
+     - z2
+     - z3
+     - z4
+     - z5
+     - z6
+     cloud_properties:
+       name: External
+     dns:
+     - 8.8.8.8
+     gateway: 115.68.46.177
+     range: 115.68.46.176/28
+     reserved:
+     - 115.68.46.176 - 115.68.46.188
+     static:
+     - 115.68.46.189 - 115.68.46.190
+   type: manual
+ - name: service_private
+   subnets:
+   - azs:
+     - z1
+     - z2
+     - z3
+     - z4
+     - z5
+     - z6
+     cloud_properties:
+       name: Internal
+     dns:
+     - 8.8.8.8
+     gateway: 10.30.20.23
+     range: 10.30.0.0/16
+     reserved:
+     - 10.30.0.0 - 10.30.106.255
+     static:
+     - 10.30.107.1 - 10.30.107.255
+ - name: service_public
+   subnets:
+   - azs:
+     - z1
+     - z2
+     - z3
+     - z4
+     - z5
+     - z6
+     cloud_properties:
+       name: External
+     dns:
+     - 8.8.8.8
+     gateway: 115.68.47.161
+     range: 115.68.47.160/24
+     reserved:
+     - 115.68.47.161 - 115.68.47.174
+     static:
+     - 115.68.47.175 - 115.68.47.190
+   type: manual
+ - name: portal_service_public
+   subnets:
+   - azs:
+     - z1
+     - z2
+     - z3
+     - z4
+     - z5
+     - z6
+     cloud_properties:
+       name: External
+     dns:
+     - 8.8.8.8
+     gateway: 115.68.46.209
+     range: 115.68.46.208/28
+     reserved:
+     - 115.68.46.217 - 115.68.46.222
+     static:
+     - 115.68.46.214 - 115.68.46.216
+   type: manual

+ disk_types:
+ - disk_size: 1024
+   name: default
+ - disk_size: 1024
+   name: 1GB
+ - disk_size: 2048
+   name: 2GB
+ - disk_size: 4096
+   name: 4GB
+ - disk_size: 5120
+   name: 5GB
+ - disk_size: 8192
+   name: 8GB
+ - disk_size: 10240
+   name: 10GB
+ - disk_size: 20480
+   name: 20GB
+ - disk_size: 30720
+   name: 30GB
+ - disk_size: 51200
+   name: 50GB
+ - disk_size: 102400
+   name: 100GB
+ - disk_size: 1048576
+   name: 1TB

+ stemcells:
+ - alias: default
+   os: ubuntu-xenial
+   version: '325.41'

+ releases:
+ - name: paasta-logging-service-release
+   version: '1.0'

+ update:
+   canaries: 1
+   canary_watch_time: 30000-600000
+   max_in_flight: 1
+   serial: false
+   update_watch_time: 5000-600000

+ instance_groups:
+ - azs:
+   - z5
+   instances: 1
+   jobs:
+   - name: elasticsearch
+     properties:
+       elasticsearch:
+         cluster_name: logsearch
+         exec:
+         master_hosts:
+         - 10.30.107.135
+         node:
+           allow_data: false
+           allow_master: true
+     release: paasta-logging-service-release
+   - name: cerebro
+     properties:
+       elasticsearch:
+         cluster_name: logsearch
+     release: paasta-logging-service-release
+   name: elasticsearch_master
+   networks:
+   - name: service_private
+     static_ips:
+     - 10.30.107.135
+   persistent_disk_type: 10GB
+   stemcell: default
+   update:
+     max_in_flight: 1
+   vm_type: default
+ - azs:
+   - z5
+   instances: 1
+   jobs:
+   - name: queue
+     properties:
+       redis:
+         host: 10.30.107.139
+     release: paasta-logging-service-release
+   name: queue
+   networks:
+   - name: service_private
+     static_ips:
+     - 10.30.107.139
+   persistent_disk_type: 10GB
+   stemcell: default
+   vm_type: default
+ - azs:
+   - z5
+   instances: 1
+   jobs:
+   - name: elasticsearch_config
+     properties:
+       elasticsearch_config:
+         elasticsearch:
+           host: 10.30.107.135
+         index_prefix: laas-
+         templates:
+         - shards-and-replicas: "/var/vcap/jobs/elasticsearch_config/index-templates/shards-and-replicas.json"
+         - index-settings: "/var/vcap/jobs/elasticsearch_config/index-templates/index-settings.json"
+         - index-mappings: "/var/vcap/jobs/elasticsearch_config/index-templates/index-mappings.json"
+         - index-mappings-laas: "/var/vcap/jobs/elasticsearch_config/index-templates/index-mappings-laas.json"
+         - index-mappings-app: "/var/vcap/jobs/elasticsearch_config/index-templates/index-mappings-app.json"
+         - index-mappings-platform: "/var/vcap/jobs/elasticsearch_config/index-templates/index-mappings-platform.json"
+     release: paasta-logging-service-release
+   - name: curator
+     properties:
+       curator:
+         elasticsearch:
+           host: 10.30.107.135
+           port: 9200
+         purge_logs:
+           retention_period: 7
+           unit: days
+       elasticsearch_config:
+         index_prefix: laas-
+     release: paasta-logging-service-release
+   name: maintenance
+   networks:
+   - name: service_private
+     static_ips:
+     - 10.30.107.136
+   stemcell: default
+   update:
+     serial: true
+   vm_type: minimal
+ - azs:
+   - z5
+   instances: 2
+   jobs:
+   - name: elasticsearch
+     properties:
+       elasticsearch:
+         cluster_name: logsearch
+         exec:
+         master_hosts:
+         - 10.30.107.135
+         node:
+           allow_data: true
+           allow_master: false
+     release: paasta-logging-service-release
+   name: elasticsearch_data
+   networks:
+   - name: service_private
+     static_ips:
+     - 10.30.107.133
+     - 10.30.107.134
+   persistent_disk_type: 20GB
+   stemcell: default
+   update:
+     max_in_flight: 1
+     serial: true
+   vm_type: medium
+ - azs:
+   - z5
+   instances: 1
+   jobs:
+   - name: kibana
+     properties:
+       kibana:
+         elasticsearch:
+           host: 10.30.107.135
+           port: 9200
+         version: 5.3.0
+       laas:
+         cf_api_url: https://api.<DOMAIN>
+         cf_client_id: laasclient
+         cf_client_secret: clientsecret
+         cf_uaa_callback_url: http://115.68.47.181/app/laas/callback
+         cf_uaa_scope: cloud_controller.read&openid
+         cf_uaa_url: https://uaa.<DOMAIN>
+         elasticsearch_index_prefix: laas-
+         top_left_logo_image: data:image/png;base64,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
+     release: paasta-logging-service-release
+   name: visualization
+   networks:
+   - name: service_private
+     static_ips:
+     - 10.30.107.143
+   stemcell: default
+   vm_type: default
+ - azs:
+   - z5
+   instances: 1
+   jobs:
+   - name: ingestor_syslog
+     properties:
+       logstash_ingestor:
+         debug: false
+         relp:
+           port:
+       redis:
+         host: 10.30.107.139
+     release: paasta-logging-service-release
+   - name: ingestor_cloudfoundry-firehose
+     properties:
+       cloudfoundry:
+         api_endpoint: https://api.<DOMAIN>
+         firehose_client_id: firehose-to-syslog
+         firehose_client_secret: firehose-secret
+         firehose_events: LogMessage
+         skip_ssl_validation: true
+       create-uaa-client:
+         cloudfoundry:
+           system_domain: <DOMAIN>
+           uaa_admin_client_id: <UAA_ADMIN_CLIENT_ID>
+           uaa_admin_client_secret: <UAA_ADMIN_CLIENT_SECRET>
+       syslog:
+         host: 10.30.107.140
+         port: 5514
+     release: paasta-logging-service-release
+   name: collector
+   networks:
+   - name: service_private
+     static_ips:
+     - 10.30.107.131
+   stemcell: default
+   vm_type: default
+ - azs:
+   - z5
+   instances: 2
+   jobs:
+   - name: parser
+     properties:
+       logstash_parser:
+         debug: false
+         elasticsearch:
+           index: laas-%{[@metadata][index]}-%{+YYYY.MM.dd.HH}
+           index_type: "%{[@metadata][type]}"
+       redis:
+         host: 10.30.107.139
+     release: paasta-logging-service-release
+   - name: elasticsearch
+     properties:
+       elasticsearch:
+         cluster_name: logsearch
+         exec:
+         master_hosts:
+         - 10.30.107.135
+     release: paasta-logging-service-release
+   name: parser
+   networks:
+   - name: service_private
+     static_ips:
+     - 10.30.107.137
+     - 10.30.107.138
+   stemcell: default
+   update:
+     max_in_flight: 4
+     serial: false
+   vm_type: default
+ - azs:
+   - z5
+   instances: 1
+   jobs:
+   - name: haproxy
+     properties:
+       haproxy:
+         cluster_monitor:
+         ingestor:
+           backend_servers:
+           - 10.30.107.131
+         kibana:
+           backend_servers:
+           - 10.30.107.143
+     release: paasta-logging-service-release
+   name: router
+   networks:
+   - name: service_private
+     static_ips:
+     - 10.30.107.140
+   - default:
+     - dns
+     - gateway
+     name: service_public
+     static_ips: 115.68.47.181
+   stemcell: default
+   vm_type: minimal

+ name: paasta-logging-service

Continue? [yN]: y

Task 68414

Task 68414 | 02:06:49 | Preparing deployment: Preparing deployment (00:00:02)
Task 68414 | 02:06:54 | Preparing package compilation: Finding packages to compile (00:00:00)
Task 68414 | 02:06:54 | Compiling packages: haproxy/1d59aa8ae61da532ade4e9f5129428bbc101be64
Task 68414 | 02:06:54 | Compiling packages: kibana/4bf52faddc3486d0f81857ddba1d5a617e52cc5b
Task 68414 | 02:06:54 | Compiling packages: python3/2a00051b1a0f02d70453c75e9036d41cb960ebfc
Task 68414 | 02:06:54 | Compiling packages: golang/fb3379fa5b3afb2e6e515657a33f2145ba4bd076
Task 68414 | 02:06:54 | Compiling packages: ruby2.3/613589acad7ddafc74fd12316e3d9cf5346f612e
Task 68414 | 02:09:51 | Compiling packages: kibana/4bf52faddc3486d0f81857ddba1d5a617e52cc5b (00:02:57)
Task 68414 | 02:09:51 | Compiling packages: redis/ff3e314387f91116dd8906cd656ecd0476b4c7b5
Task 68414 | 02:10:00 | Compiling packages: golang/fb3379fa5b3afb2e6e515657a33f2145ba4bd076 (00:03:06)
Task 68414 | 02:10:00 | Compiling packages: cerebro/278996853981243e3b921d18e255d55d998dcc7f
Task 68414 | 02:10:11 | Compiling packages: haproxy/1d59aa8ae61da532ade4e9f5129428bbc101be64 (00:03:17)
Task 68414 | 02:10:11 | Compiling packages: java8/ae41eecf1175fb16c678940d4d6e31af10405b6e
Task 68414 | 02:10:30 | Compiling packages: cerebro/278996853981243e3b921d18e255d55d998dcc7f (00:00:30)
Task 68414 | 02:10:30 | Compiling packages: elasticsearch/cb154811849156850a88023c3dd1fd46698a09da
Task 68414 | 02:10:33 | Compiling packages: redis/ff3e314387f91116dd8906cd656ecd0476b4c7b5 (00:00:42)
Task 68414 | 02:10:33 | Compiling packages: firehose-to-syslog/57480e93d19cefa2f26dfa787419d378b2104b13
Task 68414 | 02:10:47 | Compiling packages: java8/ae41eecf1175fb16c678940d4d6e31af10405b6e (00:00:36)
Task 68414 | 02:10:47 | Compiling packages: logstash/a5bc4ac7fb0f561db9b9e03833510b866236af8e
Task 68414 | 02:10:57 | Compiling packages: elasticsearch/cb154811849156850a88023c3dd1fd46698a09da (00:00:27)
Task 68414 | 02:11:09 | Compiling packages: firehose-to-syslog/57480e93d19cefa2f26dfa787419d378b2104b13 (00:00:36)
Task 68414 | 02:12:09 | Compiling packages: python3/2a00051b1a0f02d70453c75e9036d41cb960ebfc (00:05:15)
Task 68414 | 02:12:09 | Compiling packages: curator/27ee549a38ebcef009bb0e63ac716d544dc8caa2
Task 68414 | 02:12:16 | Compiling packages: logstash/a5bc4ac7fb0f561db9b9e03833510b866236af8e (00:01:29)
Task 68414 | 02:12:48 | Compiling packages: curator/27ee549a38ebcef009bb0e63ac716d544dc8caa2 (00:00:39)
Task 68414 | 02:16:41 | Compiling packages: ruby2.3/613589acad7ddafc74fd12316e3d9cf5346f612e (00:09:47)
Task 68414 | 02:16:41 | Compiling packages: logsearch-config/4cd3b3efcb7ea642badd77ff3b5189ae1d324734 (00:00:21)
Task 68414 | 02:17:50 | Creating missing vms: elasticsearch_master/4698c36b-413d-4370-b671-44ee075a0cf0 (0)
Task 68414 | 02:17:50 | Creating missing vms: elasticsearch_data/d779c528-8f75-4b4c-b2d9-ac367c1e5ece (0)
Task 68414 | 02:17:50 | Creating missing vms: maintenance/dba09e1e-06c0-42bf-a30d-d97a62c536bc (0)
Task 68414 | 02:17:50 | Creating missing vms: queue/cc986003-b6c1-4570-b2d7-32ecfd40eedf (0)
Task 68414 | 02:17:50 | Creating missing vms: visualization/d1ac0c78-aa4c-465d-9193-64f2e2de269a (0)
Task 68414 | 02:17:50 | Creating missing vms: elasticsearch_data/fa38698e-913c-4296-aac8-c0b56c84a71e (1)
Task 68414 | 02:17:50 | Creating missing vms: collector/d2a1aed9-d10f-42df-91ec-e21f1baecfb8 (0)
Task 68414 | 02:17:50 | Creating missing vms: parser/7ef8ffd6-7d8b-4ae0-bd8c-17f5e7092ca2 (0)
Task 68414 | 02:17:50 | Creating missing vms: parser/3dfdc7bc-8dde-4ed1-95d0-eb638d4900fa (1)
Task 68414 | 02:17:50 | Creating missing vms: router/c64e9519-713c-4f24-9b04-4bbf2d0ac457 (0)
Task 68414 | 02:21:00 | Creating missing vms: visualization/d1ac0c78-aa4c-465d-9193-64f2e2de269a (0) (00:03:10)
Task 68414 | 02:21:07 | Creating missing vms: elasticsearch_master/4698c36b-413d-4370-b671-44ee075a0cf0 (0) (00:03:17)
Task 68414 | 02:21:20 | Creating missing vms: parser/7ef8ffd6-7d8b-4ae0-bd8c-17f5e7092ca2 (0) (00:03:30)
Task 68414 | 02:21:20 | Creating missing vms: elasticsearch_data/fa38698e-913c-4296-aac8-c0b56c84a71e (1) (00:03:30)
Task 68414 | 02:21:22 | Creating missing vms: queue/cc986003-b6c1-4570-b2d7-32ecfd40eedf (0) (00:03:32)
Task 68414 | 02:21:22 | Creating missing vms: elasticsearch_data/d779c528-8f75-4b4c-b2d9-ac367c1e5ece (0) (00:03:32)
Task 68414 | 02:21:24 | Creating missing vms: collector/d2a1aed9-d10f-42df-91ec-e21f1baecfb8 (0) (00:03:34)
Task 68414 | 02:21:24 | Creating missing vms: maintenance/dba09e1e-06c0-42bf-a30d-d97a62c536bc (0) (00:03:34)
Task 68414 | 02:21:24 | Creating missing vms: router/c64e9519-713c-4f24-9b04-4bbf2d0ac457 (0) (00:03:34)
Task 68414 | 02:21:25 | Creating missing vms: parser/3dfdc7bc-8dde-4ed1-95d0-eb638d4900fa (1) (00:03:35)
Task 68414 | 02:21:28 | Updating instance elasticsearch_master: elasticsearch_master/4698c36b-413d-4370-b671-44ee075a0cf0 (0) (canary)
Task 68414 | 02:21:28 | Updating instance queue: queue/cc986003-b6c1-4570-b2d7-32ecfd40eedf (0) (canary)
Task 68414 | 02:25:25 | Updating instance elasticsearch_master: elasticsearch_master/4698c36b-413d-4370-b671-44ee075a0cf0 (0) (canary) (00:03:57
Task 68414 | 02:25:43 | Updating instance queue: queue/cc986003-b6c1-4570-b2d7-32ecfd40eedf (0) (canary) (00:04:15)
Task 68414 | 02:25:43 | Updating instance maintenance: maintenance/dba09e1e-06c0-42bf-a30d-d97a62c536bc (0) (canary) (00:01:06)
Task 68414 | 02:26:49 | Updating instance elasticsearch_data: elasticsearch_data/d779c528-8f75-4b4c-b2d9-ac367c1e5ece (0) (canary) (00:02:10)
Task 68414 | 02:28:59 | Updating instance elasticsearch_data: elasticsearch_data/fa38698e-913c-4296-aac8-c0b56c84a71e (1) (00:01:54)
Task 68414 | 02:30:53 | Updating instance collector: collector/d2a1aed9-d10f-42df-91ec-e21f1baecfb8 (0) (canary)
Task 68414 | 02:30:53 | Updating instance visualization: visualization/d1ac0c78-aa4c-465d-9193-64f2e2de269a (0) (canary)
Task 68414 | 02:30:53 | Updating instance router: router/c64e9519-713c-4f24-9b04-4bbf2d0ac457 (0) (canary)
Task 68414 | 02:30:53 | Updating instance parser: parser/7ef8ffd6-7d8b-4ae0-bd8c-17f5e7092ca2 (0) (canary)
Task 68414 | 02:31:45 | Updating instance router: router/c64e9519-713c-4f24-9b04-4bbf2d0ac457 (0) (canary) (00:00:52)
Task 68414 | 02:31:55 | Updating instance visualization: visualization/d1ac0c78-aa4c-465d-9193-64f2e2de269a (0) (canary) (00:01:02)
Task 68414 | 02:32:10 | Updating instance parser: parser/7ef8ffd6-7d8b-4ae0-bd8c-17f5e7092ca2 (0) (canary) (00:01:17)
Task 68414 | 02:32:10 | Updating instance parser: parser/3dfdc7bc-8dde-4ed1-95d0-eb638d4900fa (1) (00:01:06)
Task 68414 | 02:33:54 | Updating instance collector: collector/d2a1aed9-d10f-42df-91ec-e21f1baecfb8 (0) (canary) (00:03:01)

Task 68414 Started  Wed Nov 28 02:06:49 UTC 2018
Task 68414 Finished Wed Nov 28 02:33:54 UTC 2018
Task 68414 Duration 00:27:05
Task 68414 done

Succeeded

```

- 배포된 Logging 서비스를 확인한다.

```
$ bosh -e micro-bosh -d paasta-logging-service vms
Using environment '10.30.40.111' as client 'admin'

Task 68432. Done

Deployment 'paasta-logging-service'

Instance                                                   Process State  AZ  IPs            VM CID                                   VM Type  Active  
collector/d2a1aed9-d10f-42df-91ec-e21f1baecfb8             running        z5  10.30.107.131  vm-e73085ec-e336-4c54-a842-37989dc4fe1d  default  true  
elasticsearch_data/d779c528-8f75-4b4c-b2d9-ac367c1e5ece    running        z5  10.30.107.133  vm-5b1fed2f-774f-47cf-9a14-edc015e790f1  medium   true  
elasticsearch_data/fa38698e-913c-4296-aac8-c0b56c84a71e    running        z5  10.30.107.134  vm-36a47dab-8d09-4daa-bb8c-0394f4d83fd7  medium   true  
elasticsearch_master/4698c36b-413d-4370-b671-44ee075a0cf0  running        z5  10.30.107.135  vm-46152b8f-d660-413c-9396-8b4068a4a454  default  true  
maintenance/dba09e1e-06c0-42bf-a30d-d97a62c536bc           running        z5  10.30.107.136  vm-780e1595-9aa9-445c-b056-27ff4e844017  minimal  true  
parser/3dfdc7bc-8dde-4ed1-95d0-eb638d4900fa                running        z5  10.30.107.138  vm-44210be7-0dab-46db-8cb6-d71a2c29d3c8  default  true  
parser/7ef8ffd6-7d8b-4ae0-bd8c-17f5e7092ca2                running        z5  10.30.107.137  vm-1eb78459-3050-4ab2-8f49-78f0ddb795b0  default  true  
queue/cc986003-b6c1-4570-b2d7-32ecfd40eedf                 running        z5  10.30.107.139  vm-f11ec996-5c1e-46a0-972a-8b1415267df0  default  true  
router/c64e9519-713c-4f24-9b04-4bbf2d0ac457                running        z5  10.30.107.140  vm-32ebc53c-6bef-48d7-854e-4b09a4dd9d01  minimal  true  
                                                                              115.68.47.181                                                      
visualization/d1ac0c78-aa4c-465d-9193-64f2e2de269a         running        z5  10.30.107.143  vm-75fdb6a6-e77f-4adb-8336-ec77254c82fa  default  true

```

## <div id="3"/>3.  Logging 서비스 관리

Logging 서비스 배포가 완료 되면, PaaS-TA 포탈에서 서비스를 사용하기 위해 Logging 서비스 UAA Client 등록 및 Logging 서비스 활성화 코드 등록을 해 주어야 한다.


### <div id="3.1"/>  3.1.	Logging 서비스 UAA Client 등록
-	Logging 서비스 접근이 가능한 IP를 확인한다.

```
$ bosh -e micro-bosh -d paasta-logging-service vms
Using environment '10.30.40.111' as client 'admin'

Task 68432. Done

Deployment 'paasta-logging-service'

Instance                                                   Process State  AZ  IPs            VM CID                                   VM Type  Active  
collector/d2a1aed9-d10f-42df-91ec-e21f1baecfb8             running        z5  10.30.107.131  vm-e73085ec-e336-4c54-a842-37989dc4fe1d  default  true  
elasticsearch_data/d779c528-8f75-4b4c-b2d9-ac367c1e5ece    running        z5  10.30.107.133  vm-5b1fed2f-774f-47cf-9a14-edc015e790f1  medium   true  
elasticsearch_data/fa38698e-913c-4296-aac8-c0b56c84a71e    running        z5  10.30.107.134  vm-36a47dab-8d09-4daa-bb8c-0394f4d83fd7  medium   true  
elasticsearch_master/4698c36b-413d-4370-b671-44ee075a0cf0  running        z5  10.30.107.135  vm-46152b8f-d660-413c-9396-8b4068a4a454  default  true  
maintenance/dba09e1e-06c0-42bf-a30d-d97a62c536bc           running        z5  10.30.107.136  vm-780e1595-9aa9-445c-b056-27ff4e844017  minimal  true  
parser/3dfdc7bc-8dde-4ed1-95d0-eb638d4900fa                running        z5  10.30.107.138  vm-44210be7-0dab-46db-8cb6-d71a2c29d3c8  default  true  
parser/7ef8ffd6-7d8b-4ae0-bd8c-17f5e7092ca2                running        z5  10.30.107.137  vm-1eb78459-3050-4ab2-8f49-78f0ddb795b0  default  true  
queue/cc986003-b6c1-4570-b2d7-32ecfd40eedf                 running        z5  10.30.107.139  vm-f11ec996-5c1e-46a0-972a-8b1415267df0  default  true  
router/c64e9519-713c-4f24-9b04-4bbf2d0ac457                running        z5  10.30.107.140  vm-32ebc53c-6bef-48d7-854e-4b09a4dd9d01  minimal  true  
                                                                              115.68.47.181                                                      
visualization/d1ac0c78-aa4c-465d-9193-64f2e2de269a         running        z5  10.30.107.143  vm-75fdb6a6-e77f-4adb-8336-ec77254c82fa  default  true

```

-	uaac server의 endpoint를 설정한다.

```
# endpoint 설정
$ uaac target https://uaa.<DOMAIN> --skip-ssl-validation

# target 확인
$ uaac target
Target: https://uaa.<DOMAIN>
Context: uaa_admin, from client uaa_admin

```

-	uaac 로그인을 한다.

```
$ uaac token client get <UAA_ADMIN_CLIENT_ID> -s <UAA_ADMIN_CLIENT_SECRET>
Successfully fetched token via client credentials grant.
Target: https://uaa.<DOMAIN>
Context: admin, from client admin

```

-	Logging 서비스 계정을 생성 한다.
$ uaac client add <CF_UAA_CLIENT_ID> -s <CF_UAA_CLIENT_SECRET> --redirect_uri <Logging 서비스 URI> --scope <퍼미션 범위> --authorized_grant_types <권한 타입> --authorities=<권한 퍼미션> --autoapprove=<자동승인권한>

  -	<CF_UAA_CLIENT_ID> : uaac 클라이언트 id
  -	<CF_UAA_CLIENT_SECRET> : uaac 클라이언트 secret
  -	<Logging 서비스 URI> : 성공적으로 리다이렉션 할 Logging 서비스 접근 URL (router public IP)
  -	<퍼미션 범위> : 클라이언트가 사용자를 대신하여 얻을 수있는 허용 범위 목록
  -	<권한 타입> : 서비스가 제공하는 API를 사용할 수 있는 권한 목록
  -	<권한 퍼미션> : 클라이언트에 부여 된 권한 목록
  -	<자동승인권한> : 사용자 승인이 필요하지 않은 권한 목록

```
# Logging 서비스 계정 생성
$ uaac client add laasclient -s clientsecret --redirect_uri " http://115.68.47.181" \
  --scope "cloud_controller_service_permissions.read , openid , cloud_controller.read , cloud_controller.write , cloud_controller.admin" \
  --authorized_grant_types "authorization_code , client_credentials , refresh_token" \
  --authorities="uaa.resource" \
--autoapprove="openid , cloud_controller_service_permissions.read"

# Logging 서비스 계정 생성 확인
$ uaac clients
laasclient
    scope: cloud_controller.read cloud_controller.write cloud_controller_service_permissions.read openid
        cloud_controller.admin
    resource_ids: none
    authorized_grant_types: refresh_token client_credentials authorization_code
    redirect_uri: http://115.68.47.181
    autoapprove: cloud_controller_service_permissions.read openid
    authorities: uaa.resource
    name: laasclient
    lastmodified: 1542894096080

```

## <div id="3.2"/>  3.2. Logging 서비스 활성화 코드 등록

-	PaaS-TA 운영자 포탈에 접속한다.
![002]

-	운영관리의 코드관리 메뉴로 이동하여 다음과 같이 코드를 등록한다.

> ※ Group Table<br>
> 코드 ID  : LAAS<br>
> 코드 이름 : Logging Service<br>
> ![003]
>
> ※ Detail Table<br>
> Key : laas_base_url<br>
> Value : http://<Logging Service 접근 IP>/app/laas<br>
> 요약 : Logging Service Base URL<br>
> 사용 : Y<br>
> ![004]

![005]

[001]:/Service-Guide/images/logging_service/image001.png
[002]:/Service-Guide/images/logging_service/image002.png
[003]:/Service-Guide/images/logging_service/image003.png
[004]:/Service-Guide/images/logging_service/image004.png
[005]:/Service-Guide/images/logging_service/image005.png
