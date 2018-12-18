## Table of Contents

1. [문서개요](#1)
  * [목적](#2)
  * [범위](#3)
  * [참고자료](#4)
2. [IaaS/PaaS 통합  Monitoring ](#5)
    * [IaaS/PaaS 통합  Monitoring Architecture](#6)
    * [IaaS Monitoring Architecture](#7)
    * [PaaS Monitoring Architecture](#8)
3. [IaaS-PaaS 통합 모니터링 설치](#9)
    * [Pre-requsite](#10)
    * [IaaS/PaaS Monitoring 설치환경](#11)
      * [Monasca 설치](#12)
        * [Monasca Server 설치](#13)
        * [Monasca Client(agent) 설치](#14)
      * [Logsearch 설치](#15)
        * [logsearch-deployment.yml](#16)
        * [logsearch deploy.sh](#17)
      * [Redis/InfluxDB 설치](#18)
        *  [redis-influxdb.yml](#19)
        *  [redis-influxdb deploy.sh](#20)
      * [IaaS/PaaS 통합 Monitoring 설치](#21)
        *  [PaaS Database 생성](#22)
        *  [소스 다운로드](#23)
        *  [paasta-monitoring-batch 설치 및 실행](#24)
        *  [iaas-paas-monitoring-web 설치 및 실행](#25)

# <div id='1'/>1.  문서 개요 

클라우드 서비스(IaaS/PaaS) 통합 운영관리 기술 개발 프로젝트의 IaaS-PaaS-Monitoring 시스템에서 IaaS(Openstack)시스템의 상태와 PaaS-Ta 서비스(Bosh/CF/Diego/App)들의 상태를 조회하여 사전에 설정한 임계치 값과 비교 후, 초과된 시스템 자원을 사용중인 서비스들의 목록을 관리자에게 통보하기 위한 애플리케이션 개발하고, 배포하는 방법을 설명한다.

## <div id='2'/>1.1.  목적
본 문서(설치가이드)는 파스타를 4.0 IaaS/PaaS 통합 모니터링을 설치하는데 있다.

## <div id='3'/>1.2.  범위
본 문서(설치가이드)는 파스타를 4.0 IaaS/PaaS 통합 모니터링을 설치하는데 있다. 

## <div id='4'/>1.3.  참고자료

본 문서는 Cloud Foundry의 BOSH Document와 Cloud Foundry Document를 참고로 작성하였다.

BOSH Document: [http://bosh.io](http://bosh.io)

Cloud Foundry Document: [https://docs.cloudfoundry.org/](https://docs.cloudfoundry.org/)

BOSH DEPLOYMENT: [https://github.com/cloudfoundry/bosh-deployment](https://github.com/cloudfoundry/bosh-deployment)

CF DEPLOYMENT: [https://github.com/cloudfoundry/cf-deployment](https://github.com/cloudfoundry/cf-deployment)



# <div id='5'/>2. IaaS/PaaS 통합  Monitoring 
IaaS-PaaS Monitoring은 openstack/PaaS-TA를 통합 모니터링 할 수 있는 모니터링 도구이다. IaaS모니터링은 Openstack Newton version을 기준으로 개발 되었다. IaaS-PaaS Monitoring 통합모니터링을 설치 하려면 아래 설치 가이드에 따라 설치 한다.

## <div id='6'/>2.1. IaaS/PaaS 통합  Monitoring Architecture
IaaS-PaaS Monitoring Application의 IaaS는 Openstack과 Monasca를 기반으로 구성되어 있다. IaaS는 Openstack Node에 monasca Agent가 설치되어 Metric Data를 Monasca에 전송하여 InfluxDB에 저장한다. PaaS는 PaaS-TA에 모니터링 Agent가 설치되어 InfluxDB에 전송 저장한다. 
Log Agent도 IaaS/PaaS에 설치되어 Log Data를 각각의 Log Repository에 전송한다.

![iaas_paas_architecure]

## <div id='7'/>2.2. IaaS Monitoring Architecture
IaaS 모니터링은 Openstack Compute/Controller Node에 Monasca agent(IaaS Monitoring Agent)와 Log Agent를 설치하여 Metric/Log Data를 모니터링 시스템에 전송한다. Metric Data는 Monasca를 통하여 MetricDB(InfluxDB)에 저장되며 Log Data는 ElasticSearch에 저장된다.

![iaas_architecure]

## <div id='8'/>2.3. PaaS Monitoring Architecture
PaaS 모니터링은 PaaS-TA모니터링을 말한다.
PaaS-TA 서비스 모니터링 운영환경에서는 크게 Backend 환경에서 실행되는 Batch 프로세스 영역과 Frontend 환경에서 실행되는 Monitoring 시스템 영역으로 나누어진다.
Batch 프로세스는 PaaS-TA Portal 서비스에서 등록한 임계치 정보와 AutoScale 대상 정보를 기준으로 주기적으로 시스템 metrics 정보를 조회 및 분석하여, 임계치를 초과한 서비스 발견시 관리자에게 Alarm을 전송하며, 임계치를 초과한 컨테이너 리스트 중에서 AutoScale 대상의 컨테이너 존재시 AutoScale Server 서비스에 관련 정보를 전송하여, 자동으로 AutoScaling 기능이 수행되도록 처리한다.
Monitoring 시스템 은 TSDB(InfluxDB)로부터 시스템 환경 정보 데이터를 조회하고, Lucene(Elasticsearch)을 통해 로그 정보를 조회한다. 조회된 정보는 PaaS-TA Monitoring 시스템의 현재 자원 사용 현황을 조회하고, Kibana를 통해 로그 정보를 조회할 수 있도록 한다. Monitoring Portal은 관리자 화면으로 알람이 발생된 이벤트 현황 정보를 조회하고, 컨테이너 배치 현황과 장애 발생 서비스에 대한 통계 정보를 조회할 수 있으며, 이벤트 관련 처리정보를 이력관리할 수 있는 화면을 제공한다

![paas_architecure]


# <div id='9'/>3. IaaS-PaaS 통합 모니터링 설치

> **[Monitoring Source Git](https://github.com/PaaS-TA/PaaS-TA-Monitoring)**

## <div id='10'/>3.1.	Pre-requsite

 1. Openstack newton version
 2. PaaS-TA가 Openstack에 설치 되어 있어야 한다.
 3. 설치된 Openstack위에 PaaS-TA에 설치 되어 있어야 한다.(PaaS-TA Agent설치 되어 있어야 한다)
 4. IaaS-PaaS-Monitoring 시스템에는 선행작업(Prerequisites)으로 Monasca Server가 설치 되어 있어야 한다. Monasca Client(agent)는 openstack controller, compute node에 설치되어 있어야 한다. 아래 Monasca Server/Client를 먼저 설치 후 IaaS-PaaS-Monitoring을 설치 해야 한다.
 5. IaaS-PaaS-Monitoring이 설치되는 서버에 golang 1.9.x 버전 이상이 설치 되어 있어야 한다.

## <div id='11'/>3.2. IaaS/PaaS Monitoring 설치환경

IaaS/PaaS 통합 모니터링 환경을 구성하기 위해서는 IaaS에서는 Monasca-Server/Client를 설치해야 한다.
PaaS에서 사용하는 logsearch(paasta Log repository)와 Redis/InfluxDB는 PaaS-TA를 설치한 Inceptoin(설치환경)에서 먼저 설치 해야 한다. 



### <div id='12'/>3.2.1.	Monasca 설치
Monasca는 Server와 Client로 구성되어 있다. Openstack controller/compute Node에 Monasca-Client(Agent)를 설치 하여 Monasca 상태정보를 Monasca-Server에 전송한다. 수집된 Data를 기반으로 IaaS 모니터링을 수행한다.
Monasca-Server는 Openstack에서 VM을 수동 생성하여 설치를 진행한다.

#### <div id='13'/>3.2.1.1.	Monasca Server 설치

> **[Monasca - Server](./monasca-server.md)**

#### <div id='14'/>3.2.1.2.	Monasca Client(agent) 설치

> **[Monasca - Client](./monasca-client.md)**

### <div id='15'/>3.2.2.	Logsearch 설치
Paasta-4.0 설치한 Inception(설치환경)에서 PaaS-TA VM Log수집을 하는 logsearch를 Bosh를 사용하여 설치 한다.

```
$ cd ~/workspace/paasta-4.0/deployment/service-deployment/logsearch
```

#### <div id='16'/>3.2.2.1.	logsearch-deployment.yml
logsearch-deployment.yml에는 ls-router, cluster-monitor, elasticsearch_data, elastic_master, kibana, mainternance 의 명세가 정의되어 있다. 

```
---
name: logsearch
update:
  canaries: 3
  canary_watch_time: 30000-1200000
  max_in_flight: 1
  serial: false
  update_watch_time: 5000-1200000
instance_groups:

- name: elasticsearch_master
  azs:
  - z5
  instances: 1
  persistent_disk_type: 10GB
  vm_type: medium
  stemcell: default
  update:
    max_in_flight: 1
    serial: true
  networks:
  - name: default
  jobs:
  - name: elasticsearch
    release: logsearch
    consumes:
      elasticsearch: {from: elasticsearch_master}
    provides:
      elasticsearch: {as: elasticsearch_master}
    properties:
      elasticsearch:
        node:
          allow_master: true
  - name: syslog_forwarder
    release: logsearch
    consumes:
      syslog_forwarder: {from: cluster_monitor}
    properties:
      syslog_forwarder:
        config:
        - file: /var/vcap/sys/log/elasticsearch/elasticsearch.stdout.log
          service: elasticsearch
        - file: /var/vcap/sys/log/elasticsearch/elasticsearch.stderr.log
          service: elasticsearch
        - file: /var/vcap/sys/log/cerebro/cerebro.stdout.log
          service: cerebro
        - file: /var/vcap/sys/log/cerebro/cerebro.stderr.log
          service: cerebro
- name: cluster_monitor
  azs:
  - z6
  instances: 1
  persistent_disk_type: 10GB
  vm_type: medium
  stemcell: default
  update:
    max_in_flight: 1
    serial: true
  networks:
  - name: default
  jobs:
  - name: elasticsearch
    release: logsearch
    consumes:
      elasticsearch: {from: elasticsearch_cluster_monitor}
    provides:
      elasticsearch: {as: elasticsearch_cluster_monitor}
    properties:
      elasticsearch:
        cluster_name: monitor
        node:
          allow_data: true
          allow_master: true
  - name: elasticsearch_config
    release: logsearch
    consumes:
      elasticsearch: {from: elasticsearch_cluster_monitor}
    properties:
      elasticsearch_config:
        templates:
        - shards-and-replicas: '{ "template" : "logstash-*", "order" : 100, "settings"
            : { "number_of_shards" : 1, "number_of_replicas" : 0 } }'
        - index-settings: /var/vcap/jobs/elasticsearch_config/index-templates/index-settings.json
        - index-mappings: /var/vcap/jobs/elasticsearch_config/index-templates/index-mappings.json
  - name: ingestor_syslog
    release: logsearch
    provides:
      syslog_forwarder: {as: cluster_monitor}
    properties:
      logstash_parser:
        filters:
        - monitor: /var/vcap/packages/logsearch-config/logstash-filters-monitor.conf
  - name: curator
    release: logsearch
    consumes:
      elasticsearch: {from: elasticsearch_cluster_monitor}
    properties:
      curator:
        purge_logs:
          retention_period: 7
  - name: kibana
    release: logsearch
    consumes:
      elasticsearch: {from: elasticsearch_cluster_monitor}
    properties:
      kibana:
        memory_limit: 30
        wait_for_templates: [shards-and-replicas]
- name: maintenance
  azs:
  - z5
  - z6
  instances: 1
  vm_type: medium 
  stemcell: default
  update:
    serial: true
  networks:
  - name: default
  jobs:
  - name: elasticsearch_config
    release: logsearch
    consumes:
      elasticsearch: {from: elasticsearch_master}
    properties:
      elasticsearch_config:
        index_prefix: logs-
        templates:
          - shards-and-replicas: /var/vcap/jobs/elasticsearch_config/index-templates/shards-and-replicas.json
          - index-settings: /var/vcap/jobs/elasticsearch_config/index-templates/index-settings.json
          - index-mappings: /var/vcap/jobs/elasticsearch_config/index-templates/index-mappings.json
          - index-mappings-lfc: /var/vcap/jobs/elasticsearch-config-lfc/index-mappings.json
          - index-mappings-app-lfc: /var/vcap/jobs/elasticsearch-config-lfc/index-mappings-app.json
          - index-mappings-platform-lfc: /var/vcap/jobs/elasticsearch-config-lfc/index-mappings-platform.json
  - name: curator
    release: logsearch
    consumes:
      elasticsearch: {from: elasticsearch_master}
  - name: elasticsearch-config-lfc
    release: logsearch-for-cloudfoundry
  - name: syslog_forwarder
    release: logsearch
    consumes:
      syslog_forwarder: {from: cluster_monitor}
    properties:
      syslog_forwarder:
        config:
        - file: /var/vcap/sys/log/curator/curator.log
          service: curator
- name: elasticsearch_data
  azs:
  - z5
  - z6
  instances: 2
  persistent_disk_type: 30GB
  vm_type: medium 
  stemcell: default
  update:
    max_in_flight: 1
    serial: true
  networks:
  - name: default
  jobs:
  - name: elasticsearch
    release: logsearch
    consumes:
      elasticsearch: {from: elasticsearch_master}
    properties:
      elasticsearch:
        node:
          allow_data: true
  - name: syslog_forwarder
    release: logsearch
    consumes:
      syslog_forwarder: {from: cluster_monitor}
    properties:
      syslog_forwarder:
        config:
        - file: /var/vcap/sys/log/elasticsearch/elasticsearch.stdout.log
          service: elasticsearch
        - file: /var/vcap/sys/log/elasticsearch/elasticsearch.stderr.log
          service: elasticsearch
        - file: /var/vcap/sys/log/cerebro/cerebro.stdout.log
          service: cerebro
        - file: /var/vcap/sys/log/cerebro/cerebro.stderr.log
          service: cerebro
- name: kibana
  azs:
  - z5
  instances: 1
  persistent_disk_type: 5GB
  vm_type: medium 
  stemcell: default
  networks:
  - name: default

  jobs:
  - name: elasticsearch
    release: logsearch
    consumes:
      elasticsearch: {from: elasticsearch_master}
  - name: redis
    release: logsearch-for-cloudfoundry
    provides:
      redis: {as: redis_link}
  - name: kibana
    release: logsearch
    provides:
      kibana: {as: kibana_link}
    consumes:
      elasticsearch: {from: elasticsearch_master}
    properties:
      kibana:
        health:
          timeout: 300
        env:
          - NODE_ENV: production
  - name: syslog_forwarder
    release: logsearch
    consumes:
      syslog_forwarder: {from: cluster_monitor}
    properties:
      syslog_forwarder:
        config:
        - file: /var/vcap/sys/log/elasticsearch/elasticsearch.stdout.log
          service: elasticsearch
        - file: /var/vcap/sys/log/elasticsearch/elasticsearch.stderr.log
          service: elasticsearch
        - file: /var/vcap/sys/log/cerebro/cerebro.stdout.log
          service: cerebro
        - file: /var/vcap/sys/log/cerebro/cerebro.stderr.log
          service: cerebro
- name: ingestor
  azs:
  - z4
  - z6
  instances: 2
  persistent_disk_type: 10GB
  vm_type: medium 
  stemcell: default
  networks:
  - name: default
  jobs:
  - name: elasticsearch
    release: logsearch
    consumes:
      elasticsearch: {from: elasticsearch_master}
  - name: parser-config-lfc
    release: logsearch-for-cloudfoundry
  - name: ingestor_syslog
    release: logsearch
    provides:
      ingestor: {as: ingestor_link}
    properties:
      logstash_parser:
        filters:
          - logsearch-for-cf: /var/vcap/packages/logsearch-config-logstash-filters/logstash-filters-default.conf
        elasticsearch:
          index: logs-%{[@metadata][index]}-%{+YYYY.MM.dd}
        deployment_dictionary:
          - /var/vcap/packages/logsearch-config/deployment_lookup.yml
          - /var/vcap/jobs/parser-config-lfc/config/deployment_lookup.yml
  - name: syslog_forwarder
    release: logsearch
    consumes:
      syslog_forwarder: {from: cluster_monitor}
    properties:
      syslog_forwarder:
        config:
        - file: /var/vcap/sys/log/elasticsearch/elasticsearch.stdout.log
          service: elasticsearch
        - file: /var/vcap/sys/log/elasticsearch/elasticsearch.stderr.log
          service: elasticsearch
        - file: /var/vcap/sys/log/ingestor_syslog/ingestor_syslog.stdout.log
          service: ingestor
        - file: /var/vcap/sys/log/ingestor_syslog/ingestor_syslog.stderr.log
          service: ingestor
- name: ls-router
  azs:
  - z4
  instances: 1
  vm_type: small
  stemcell: default
  networks:
  - name: default
    static_ips: 
    - ((router_ip)) 
#    default: [dns, gateway]
#  - name: vip
#    static_ips: [34.196.20.46]
  jobs:
  - name: haproxy
    release: logsearch
    consumes:
      elasticsearch: {from: elasticsearch_master}
      ingestor: {from: ingestor_link}
      kibana: {from: kibana_link}
      syslog_forwarder: {from: cluster_monitor}
    properties:
      inbound_port:
        https: 4443
  - name: route_registrar
    release: logsearch-for-cloudfoundry
    consumes:
      nats: {from: nats, deployment: paasta}
    properties:
      route_registrar:
        routes:
        - name: kibana
          port: 80
          registration_interval: 60s
          uris:
          - "logs.((system_domain))"
        - name: elastic 
          port: 9200
          registration_interval: 60s
          uris:
          - "elastic.((system_domain))"


variables:
- name: kibana_oauth2_client_secret
  type: password
- name: firehose_client_secret
  type: password

releases:
- name: logsearch
  url: file:///home/((inception_os_user_name))/workspace/paasta-4.0/release/monitoring/logsearch-boshrelease-209.0.1.tgz 
  version: "latest"
- name: logsearch-for-cloudfoundry
  url: file:///home/((inception_os_user_name))/workspace/paasta-4.0/release/monitoring/logsearch-for-cloudfoundry-207.0.1.tgz
  version: "latest"
stemcells:
- alias: default
  os: ubuntu-xenial
  version: "latest"
```

#### <div id='17'/>3.2.2.2. logsearch deploy.sh

deploy.sh의 –v 의 inception_os_user_name, router_ip, system_domain 및 director_name을 시스템 상황에 맞게 설정한다.
system_domain은 paasta 설치시 설정했던 system_domain을 입력하면 된다.
router_ip는 ls-router가 설치된 cloud-config azs에서 정의한 cider값 내의 적당한 IP를 지정한다.

```
Bosh -e {director_name} -d logsearch deploy logsearch-deployment.yml \
  -v inception_os_user_name=ubuntu \  # home user명 (release file path와 연관성 있음. /home/ubuntu/paasta-4.0 이하 release 파일들의 경로 설정)
  -v router_ip=10.20.50.34 \   # 배포한 ls-router VM의 private ip
  -v system_domain={system_domain}  #PaaS-TA 설치시 설정한 System Domain
```

deploy.sh을 실행하여 logsearch를 설치 한다.

```
$ cd ~/workspace/paasta-4.0/deployment/service-deployment/logsearch
$ deploy.sh 
```

설치 완료후 logsearch가 설치 완료 되었음을 확인한다.
```
$ bosh -e {director_name} -d logsearch vms
```
![PaaSTa_logsearch_vms]


### <div id='18'/>3.2.3.	Redis/InfluxDB 설치

PaaS-TA VM 에서 발생되는 MetricData 저장소인 InfluxDB와 통합 Loging시 사용되는 Redis를 설치 한다.

```
$ cd ~/workspace/paasta-4.0/deployment/service-deployment/paasta-influxdb-redis
```

#### <div id='19'/>3.2.3.1.	redis-influxdb.yml
redis-influxdb.yml에는 influxdb와 redis vm의 명세가 정의되어 있다. 

```
---
name: redis-influxdb                      # 서비스 배포이름(필수) bosh deployments 로 확인 가능한 이름

addons:
- name: bpm
  jobs:
  - name: bpm
    release: bpm

stemcells:
- alias: default
  os: ubuntu-xenial
  version: latest

releases:
- name: bpm
  sha1: 0845cccca348c6988debba3084b5d65fa7ca7fa9
  url: file:///home/((inception_os_user_name))/workspace/paasta-4.0/release/paasta/bpm-0.13.0-ubuntu-xenial-97.28-20181023-211102-981313842.tgz
  version: 0.13.0
- name: redis
  version: 14.0.1
  url: file:///home/((inception_os_user_name))/workspace/paasta-4.0/release/service/redis-14.0.1.tgz
  sha1: fd4a6107e1fb8aca1d551054d8bc07e4f53ddf05
- name: influxdb
  version: latest
  url: file:///home/((inception_os_user_name))/workspace/paasta-4.0/release/service/influxdb.tgz
  sha1: 2337d1f26f46100b8d438b50b71e300941da74a2


instance_groups:
- name: redis
  azs: [z4]
  instances: 1
  vm_type: small
  stemcell: default
  persistent_disk: 10240
  networks:
  - name: default
#    default: [dns, gateway]
    static_ips:
    - ((redis_ip))
#  - name: vip
#    static_ips: [xxx.xxx.xxx.xxx]
  jobs:
  - name: redis
    release: redis
    properties:
      password: ((redis_password))
- name: sanity-tests
  azs: [z4]
  instances: 1
  lifecycle: errand
  vm_type: small
  stemcell: default
  networks: [{name: default}]
  jobs:
  - name: sanity-tests
    release: redis

- name: influxdb
  azs: [z5]
  instances: 1
  vm_type: large
  stemcell: default
  persistent_disk_type: 50GB
  networks:
  - name: default
#    default: [dns, gateway]
    static_ips:
    - ((influxdb_ip)) 
#  - name: vip                         
#    static_ips: [xxx.xxx.xxx.xxx]     

  jobs:
  - name: influxdb
    release: influxdb
    properties:
      influxdb:
        database: cf_metric_db                                        #InfluxDB default database
        user: root                                                                                              #admin account
        password: root                                                                                  #admin password
        replication: 1
        ips: 127.0.0.1                                                                                  #local I2
  - name: chronograf
    release: influxdb

variables:
- name: redis_password
  type: password


update:
  canaries: 1
  canary_watch_time: 1000-180000
  max_in_flight: 1
  serial: true
  update_watch_time: 1000-180000

```


#### <div id='20'/>3.2.3.2. redis-influxdb deploy.sh

deploy.sh의 –v 의 inception_os_user_name, router_ip, system_domain 및 director_name을 시스템 상황에 맞게 설정한다.
system_domain은 paasta 설치시 설정했던 system_domain을 입력하면 된다.
router_ip는 ls-router가 설치된 azs에서 정의한 cider값의 적당한 IP를 지정한다.

```
bosh -e {director_name} -d redis-influxdb deploy redis-influxdb.yml \
  -v inception_os_user_name=ubuntu \  # home user명 (release file path와 연관성 있음. /home/ubuntu/paasta-4.0 이하 release 파일들의 경로 설정)
  -v redis_ip=10.20.40.11 \   # 배포할 redis VM의 private ip
  -v influxdb_ip=10.20.50.11  \ # 배포할 influxdb VM의 private ip
  -v redis_password=password    # Redis password
```

deploy.sh을 실행하여 redis-influxdb를 설치 한다.

```
$ cd ~/workspace/paasta-4.0/deployment/service-deployment/paasta-influxdb-redis
$ deploy.sh 
```

설치 완료후 redis-influxdb가 설치 완료 되었음을 확인한다.
```
$ bosh -e {director_name} -d redis-influxdb vms
```
![redus_influxdb_vms]

### <div id='21'/>3.2.4.	IaaS/PaaS 통합 Monitoring 설치

> **git 설치** <div id='2.3.1.1' />

- monasca-server에서 아래 URL에서 자신에 OS에 맞는 Git client를 다운로드 받아 설치 한다.
    + https://git-scm.com/downloads


> **golang 설치** 
- monasca-server에서 아래 URL에서 자신에 OS에 맞는 go SDK를 다운로드 받아 설치 한다. (1.9.x 이상)
  + https://golang.org/dl

- GOROOT, 및 PATH를 설정한다.
> **[golang 설치 참고 URL](https://medium.com/@patdhlk/how-to-install-go-1-9-1-on-ubuntu-16-04-ee64c073cd79)**


#### <div id='22'/>3.2.4.1	PaaS Database 생성

위에서 구성한 Monasca-Server가 정상적으로 설치 되어 있다면 설치된 Monasca-Server에 접속하여 PaastaMonitoring Database를 생성한다.

```
$ mysql –u root –p    # mysql 로그인
$ CREATE DATABASE IF NOT EXISTS PaastaMonitoring CHARACTER SET utf8 COLLATE utf8_general_ci;
```

#### <div id='23'/>3.2.4.2	소스 다운로드

위에서 구성한 Monasca Server에서 IaaS-PaaS-Monitoring 소스를 다운 받는다.

```
$ cd ~/workspace/
$ git clone https://github.com/PaaS-TA/PaaS-TA-Monitoring
```
paasta-monitoring golang 외부 Library를 설치 한다.
```
$ cd ~/workspace/PaaS-TA-Monitoring/paasta-monitoring-batch
$ ./install.sh
```

#### <div id='24'/>3.2.4.3.	paasta-monitoring-batch 설치 및 실행
paasta-monitoring batch는 Bosh, PaaS-TA VM, container 모니터링 및 PaaS-TA Application Container Autoscaling을 수행한다.
paasta-monitoring-batch config정보를 시스템 상황에 맞게 수정한다.

```
$ cd ~/workspace/ PaaS-TA-Monitoring
$ cd ~/workspace/paasta-monitoring-batch/src/kr/paasta/monitoring-batch
$ vi config.ini
```
config.ini파일은 paasta-monitoring-batch가 실행되는 환경 정보이다.

config.ini
```
#Web Server Port
server.port = 9999                       # Batch 서버 Port

#InfluxDB
influx.user =
influx.pass =
influx.cf_measurement = cf_metrics                   # paasta metricData measurement(수정 필요없음)
influx.cf_process_measurement = cf_process_metrics   # paasta process metricData measurement(수정 필요없음)
influx.url =http://xxx.xxx.xxx.xxx:8086              # PaaS metricDB URL(앞에서 설치한 influxdb IP)

#Metric DB Name
influx.paasta.db_name=cf_metric_db                   # paasta metricDB(influxDB) database (수정 필요없음)
influx.bosh.db_name=bosh_metric_db                   # bosh metricDB(influxDB) database (수정 필요없음)
influx.container.db_name=container_metric_db         # container metricDB(influxDB) database (수정 필요없음)

influx.defaultTimeRange =130s

#MonitoringDB 접속 정보
monitoring.db.type=mysql                             # PaaS-TA Monitoring Database Type
#PaastaMonitoring 먼저 생성 해야함.
monitoring.db.dbname=PaastaMonitoring                # PaaS-TA Monitoring Database Name  
monitoring.db.username=root                          # PaaS-TA Monitoring Database user(root필요)
monitoring.db.password=xxxx                          # PaaS-TA Monitoring Database user password
monitoring.db.host=xxx.xxx.xxx.xxx                   # PaaS-TA Monitoring Database IP
monitoring.db.port=3306                              # PaaS-TA Monitoring Database port

# bosh
bosh.api.url=115.68.151.189:25555                    # Bosh Url (PaaS-TA를 설치한 Bosh IP/port)
bosh.ip=115.68.151.189                               # Bosh IP (PaaS-TA를 설치한 Bosh IP)
bosh.admin=admin                                     # bosh admin ID
bosh.password=m89jsip6ld1zkb0uwyuo                   # bosh admin Password  (bosh 설치 생성된 creds.yml의 admin_password 참조)
bosh.cf.deployment.name=paasta                       # PaaS-TA Deployment Name (수정 필요없음)
bosh.cell.name.prefix=cell                           # Paas-TA Cell 명  (수정 필요없음)
bosh.service.name=bosh                               # Bosh Service 명 (수정 필요없음)

mail.smtp.host=localhost                             #smtp ip(monasca-server 설치시 같이 설치됨)
mail.smtp.port=25                                    #smtp port
mail.sender=xxxx.com@gmail.com                       # smtp server ID
mail.sender.password=nuxokzkviwfpjwqg                # smtp server Password
mail.resource.url=http://xxx.xxx.xxx.xxx:8080        # monitoring-web URL(IaaS-PaaS 모니터링이 설치된 VM public IP)
mail.alarm.send=true                                 # Alarm 발생시 메일 전송유무
mail.tls.send=false                                  # smtp 세 서버 인증 Mode가 TlS

batch.interval.second=60                             #Monitoring Batch 실행 주기
gmt.time.hour.gap=0                                  #utc time zone과 Monitoring Server local time zone과의 시간 차이

#redis
redis.addr=xxx.xxx.xxx.xxx:6379                      # redis IP(앞에서 설치한 redis IP)
redis.password=password                              # redis Password
redis.db=0

user.portal.alarm.interval=60                        #사용자포털 알람 재발송 주기(분)

#cf-client
cf.client.api_address=http://api.{system_domain}     # api.{system_domain} PaaS-TA설치된 System domain
cf.client.username=admin                             # PaaS-TA admin ID
cf.client.password=admin                             # PaaS-TA admin password
```


config.ini가 모두 수정 되었으면 MonitoringBatch를 실행한다.
```
$ cd ~/workspace/PaaS-TA-Monitoring/paasta-monitoring-batch
$ nohup ./batch_run.sh &
```

#### <div id='25'/>3.2.4.4.	iaas-paas-monitoring-web 설치 및 실행

iaas-paas-monitoring-management는 IaaS/PaaS 통함 모니터링 Dashboard 및 알람정책 설정, 알람 관리등의 기능을 제공한다. 

```
$ cd ~/workspace/ PaaS-TA-Monitoring
$ cd ~/workspace/PaaS-TA-Monitoring/iaas-paasta-monitoring-management/src/kr/paasta/monitoring
$ vi config.ini
```
config.ini파일은 paasta-monitoring-batch가 실행되는 환경 정보이다.
iaas-paas-monitoring-web config정보를 시스템 상황에 맞게 수정한다.

config.ini
```


server.url = http://127.0.0.1:8080
server.port = 8080            #monitoring-web port

#모니터링 시스템 사용 옵션 정보
#( IaaS : IaaS 만 사용 , PaaS : PaaS 만 사용, ALL : IaaS, PaaS 모두 사용)
system.monitoring.type=ALL

#redis
redis.addr=10.20.40.11:6379                              #Redis endpoint URL
redis.password=password                                  #Redis password
redis.db=0

################## IaaS Config 
# Monasca RDB 접속 정보(IaaS 설정)
iaas.monitoring.db.type=mysql       #IaaS Monitoring config DB Type
iaas.monitoring.db.dbname=mon       #IaaS Monitoring database name
iaas.monitoring.db.username=root    #IaaS Monitoring database user_name
iaas.monitoring.db.password=password   #IaaS Monitoring database user password
iaas.monitoring.db.host=115.68.151.184 #IaaS Monitoring database IP
iaas.monitoring.db.port=3306           #IaaS Monitoring database port

# InfluxDB
iaas.metric.db.username =
iaas.metric.db.password =
iaas.metric.db.url=http://xxx.xxx.xxx.xxx:8086  # IaaS metricDB URL(앞에서 설치한 influxdb IP)
iaas.metric.db.name=mon                         # IaaS metricDB(influxDB) database (수정 필요없음)

# Openstack Admin
default.region=RegionOne                             #Openstack Region Name
default.domain=default
default.username=admin                               #default domain admin user
default.password=cfmonit                             #default domain admin password
default.tenant_name=admin                            #default domain admin user
default.tenant_id=61e66f7d847e4951aa38452fe74c93eb   #default domain admin project id(Horizon>identity>project 에서 확인)
identity.endpoint=http://115.68.151.175:5000/v3      #openstack keystone api endpoint
keystone.url=http://115.68.151.175:35357/v3          #openstack keystone api endpoint

# Monasca Api
monasca.url=http://115.68.151.184:8020/v2.0          #Monasca-server api IP:port
monasca.connect.timeout=60
monasca.secure.tls=false

# Openstack Nova
nova.target.url=http://115.68.151.175:8774           #Openstack(Controller Node) nova ip:port
nova.target.version=v2.1                             #Openstack nova api Version
nova.target.tenant_id=61e66f7d847e4951aa38452fe74c93eb  #default domain admin project id(Horizon>identity>project 에서 확인)

# Openstack Keystone
keystone.target.url=http://115.68.151.175:35357  #openstack keystone api endpoint
keystone.target.version=v3                       #openstack keystone api version

# Openstack Neutron
neutron.target.url=http://115.68.151.175:9696    #openstack neutron api endpoint
neutron.target.version=v2.0                      #openstack neutron api version

# Openstack Cinder
cinder.target.url=http://115.68.151.175:8776     #openstack cinder api endpoint
cinder.target.version=v2                         #openstack cinder api version

# Openstack Glance
glance.target.url=http://115.68.151.175:9191     #openstack glance api endpoint
glance.target.version=v2                         #openstack glance api version

# RabbitMQ
rabbitmq.user=openstack                          #openstack rabbitmq user name
rabbitmq.pass=cfmonit                            #openstack rabbitmq user password
rabbitmq.ip=115.68.151.175                       #openstack(controller Node) rabbitmq ip
rabbitmq.port=15672                              #openstack(controller Node) rabbitmq port
rabbitmq.target.node=rabbit@controller           #openstack(controller Node) rabbitmq target Node

# Elasticsearch URL
iaas.elastic.url=115.68.151.184:9200             #IaaS Log Repository(elasticSearch) url

################## PaaS Config 

paas.elastic.url=elastic.115.68.151.185.xip.io   #PaaS Log Repository(elasticSearch) url


# PaaS RDB 접속 정보
paas.monitoring.db.type=mysql                   # PaaS-TA Monitoring Database Type
paas.monitoring.db.dbname=PaastaMonitoring      # PaaS-TA Monitoring Database Name
paas.monitoring.db.username=root                # PaaS-TA Monitoring Database user(root필요)
paas.monitoring.db.password=password            # PaaS-TA Monitoring Database user password
paas.monitoring.db.host=115.68.151.184          # PaaS-TA Monitoring Database IP
paas.monitoring.db.port=3306                    # PaaS-TA Monitoring Database port

paas.metric.db.username =
paas.metric.db.password =
paas.metric.db.url = http://xxx.xxx.xxx.xxx:8086   # PaaS metricDB URL(앞에서 설치한 influxdb IP)
paas.metric.db.name.paasta=cf_metric_db            # paasta metricDB(influxDB) database (수정 필요없음)
paas.metric.db.name.bosh=bosh_metric_db            # bosh metricDB(influxDB) database (수정 필요없음)
paas.metric.db.name.container=container_metric_db  # container metricDB(influxDB) database (수정 필요없음)


# Bosh Info
bosh.count=1                                     #PaaS Bosh Count(수정 필요없음)
bosh.0.name=micro-bosh                           #PaaS Bosh Director name
bosh.0.ip=10.20.0.7                              #PaaS Bosh VM IP
bosh.0.deployname=bosh                           #PaaS Bosh Deployment name(수정 필요없음)

# BOSH client

bosh.client.api.address=https://10.20.0.7:25555  #PaaS Bosh Director endpoint
bosh.client.api.username=admin                   #PaaS Bosh admin name
bosh.client.api.password=j2kpbpmzfwgi6gg6pbqc    #PaaS Bosh admin password

#disk mount point
disk.mount.point=/,/var/vcap/data
disk./.resp.json.name=/
disk./var/vcap/data.resp.json.name=data

#disk io mount point
disk.io.mount.point=/,/var/vcap/data
disk.io./.read.json.name=/-read
disk.io./.write.json.name=/-write
disk.io./var/vcap/data.read.json.name=data-read
disk.io./var/vcap/data.write.json.name=data-write

#network monitor item
network.monitor.item=eth0

# Time difference(hour)
gmt.time.gap=0                                  #utc time zone과 Monitoring Server local time zone과의 시간 차이

#cfProvider
paas.cf.client.apiaddress=http://api.{system_domain}     # api.{system_domain} PaaS-TA설치된 System domain
paas.cf.client.skipsslvalidation=true 
```

config.ini가 모두 수정 되었으면 iaas-paas-monitoring-web을 실행한다.
```
$ cd ~/workspace/PaaS-TA-Monitoring/iaas-paasta-monitoring-managemen
$ nohup ./run.sh &
```

### <div id='23'/>3.3.1.5. 통합 monitoring dashboard접속
 
 http://{monitoring-web-ip}:8080 에 접속하여 회원 가입 후 Main Dashboard에 접속한다.

 Login 화면에서 회원 가입 버튼을 클릭한다.

 ![PaaSTa_monitoring_login]


member_info에는 사용자가 사용할 ID/PWD를 입력한다. iaas-info에는 Openstack admin 권한의 계정을 입력한다. paas-info에는 PaaS-TA admin 권한의 계정을 입력한다. paasta deploy시 입력한 admin/pwd를 입력해야 한다. 입력후 [인증수행]를 실행후 Joing버튼을 클릭하면 회원가입이 완료된다.

 ![PaaSTa_monitoring_join]

모니터링 main dashboard 화면

 ![PaaSTa_monitoring_main_dashboard]


[iaas_paas_architecure]:./images/iaas-paas-archi.png
[iaas_architecure]:./images/iaas-archi.png
[paas_architecure]:./images/monit_architecture.png
[redus_influxdb_vms]:./images/redis-influxdb-vm.png

[PaaSTa_release_dir]:./images/paasta-release.png
[PaaSTa_logsearch_vms]:./images/logsearch.png
[PaaSTa_monitoring_vms]:./images/paasta-monitoring.png
[PaaSTa_monitoring_login]:./images/monit_login.png
[PaaSTa_monitoring_join]:./images/iaas-paas-member-join.png


