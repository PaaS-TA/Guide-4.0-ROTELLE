## Table of Contents

1. [문서개요](#1)
  * [목적](#2)
  * [범위](#3)
  * [참고자료](#4)
2. [PaaS-TA  Monitoring Architecture](#5)
    * [PaaS-TA  Monitoring Architecture](#6)
    * [PaaS-TA 자원정보 수집 Architecture](#7)
3. [PaaS-TA Monitoring 설치](#8)
    * [Pre-requsite](#9)
    * [PaaS-TA 4.0 모니터링 설치 파일 다운로드](#10)
    * [PaaS-Ta Monitoring 설치환경](#11)
    * [Logsearch 설치](#12)
        *  [logsearch-deployment.yml](#13)
        *  [deploy.sh](#14)
    * [PaaS-TA Monitoring 설치](#15)
        *  [paasta-monitoring.yml](#16)
        *  [monit-deploy.sh](#17)
    * [monitoring dashboard접속](#18)

# <div id='1'/>1.  문서 개요 

## <div id='2'/>1.1.  목적
본 문서(설치가이드)는 파스타를 4.0 PaaS-TA 모니터링을 설치하는데 있다.

## <div id='3'/>1.2.  범위
본 문서(설치가이드)는 파스타를 4.0 PaaS-TA 모니터링을 설치하는데 있다. 모니터링중 IaaS-PaaS 통합 모니터링은 별도 통합 모니터링문서를 제공하며 본문서는 PaaS 모니터링 설치 가이드를 제공함에 있다.

## <div id='4'/>1.3.  참고자료

본 문서는 Cloud Foundry의 BOSH Document와 Cloud Foundry Document를 참고로 작성하였다.

BOSH Document: [http://bosh.io](http://bosh.io)

Cloud Foundry Document: [https://docs.cloudfoundry.org/](https://docs.cloudfoundry.org/)

BOSH DEPLOYMENT: [https://github.com/cloudfoundry/bosh-deployment](https://github.com/cloudfoundry/bosh-deployment)

CF DEPLOYMENT: [https://github.com/cloudfoundry/cf-deployment](https://github.com/cloudfoundry/cf-deployment)



# <div id='5'/>2. PaaS-TA  Monitoring Architecture

## <div id='6'/>2.1. PaaS-TA  Monitoring Architecture
PaaS-TA 서비스 모니터링 운영환경에서는 크게 Backend 환경에서 실행되는 Batch 프로세스 영역과 Frontend 환경에서 실행되는 Monitoring 시스템 영역으로 나누어진다.
Batch 프로세스는 PaaS-TA Portal 서비스에서 등록한 임계치 정보와 AutoScale 대상 정보를 기준으로 주기적으로 시스템 metrics 정보를 조회 및 분석하여, 임계치를 초과한 서비스 발견시 관리자에게 Alarm을 전송하며, 임계치를 초과한 컨테이너 리스트 중에서 AutoScale 대상의 컨테이너 존재시 AutoScale Server 서비스에 관련 정보를 전송하여, 자동으로 AutoScaling 기능이 수행되도록 처리한다.
Monitoring 시스템 은 TSDB(InfluxDB)로부터 시스템 환경 정보 데이터를 조회하고, Lucene(Elasticsearch)을 통해 로그 정보를 조회한다. 조회된 정보는 PaaS-TA Monitoring 시스템의 현재 자원 사용 현황을 조회하고, Kibana를 통해 로그 정보를 조회할 수 있도록 한다. Monitoring Portal은 관리자 화면으로 알람이 발생된 이벤트 현황 정보를 조회하고, 컨테이너 배치 현황과 장애 발생 서비스에 대한 통계 정보를 조회할 수 있으며, 이벤트 관련 처리정보를 이력관리할 수 있는 화면을 제공한다

![PaaSTa_Monit_architecure_Image]

## <div id='7'/>2.2. PaaS-TA 자원정보 수집 Architecture
PaaS-TA 서비스는 내부적으로 메트릭스 정보를 수집 및 전달하는 Metric Agent와 로그 정보를 수집 및 전달하는 syslog 모듈을 제공한다. Metric Agent는 시스템 관련 메트릭스를 수집하여 Influx DB에 정보를 저장한다. syslog는 PaaS-TA 서비스를 Deploy 하기 위한 manfiest 파일의 설정으로도 로그 정보를 ELK 서비스에 전달할 수 있으며, 로그 정보를 전달하기 위해서는 relp 프로토콜(reliable event logging protocol)을 사용한다.

![PaaSTa_Monit_collect_architecure_Image]

# <div id='8'/>3.	PaaS-TA Monitoring 설치

## <div id='9'/>3.1. Pre-requsite

1. PaaS-Ta 4.0 Monitoring을 설치 하기 위해서는 bosh 설치과정에서 언급한 것 처럼 관련 deployment, release , stemcell을 파스타 사이트에서 다운로드 받아 정해진 경로에 복사 해야 한다.
2. PaaS-TA 4.0이 설치되어 있어야 하며 monitoring Agent가 설치되어 있어야 한다.
3. bosh login이 되어 있어야 한다.

## <div id='10'/>3.2.	PaaS-TA 4.0 모니터링 설치 파일 다운로드

> **[설치 파일 다운로드 받기](https://paas-ta.kr/download/package)**

> **[Monitoring Source Git](https://github.com/PaaS-TA/PaaS-TA-Monitoring)**

파스타 다운로드 URL에서 [PaaS-TA 설치 릴리즈] 파일을 다운로드 받아 ~/workspace/paasta-4.0/release 이하 디렉토리에 압축을 푼다. 압출을 풀면 아래 그림과 같이 ~/workspace/paasta-4.0/release/monitoring 이하 디렉토리가 생성되며 이하에 릴리즈 파일(tgz)이 존재한다.

![PaaSTa_release_dir]

## <div id='11'/>3.3. PaaS-Ta Monitoring 설치환경

~/workspace/paasta-4.0/deployment/service-deployment 이하 디렉토리에는 logsearch, pasta-monitoring 디렉토리가 존재한다. Logsearch는 logAgent에서 발생한 Log정보를 수집하여 저장하는 Deployment이다. Pasta-monitoring은 PaaS-TA VM에서 발생한 Metric정보를 수집하여 모니터링을 실행한다.

```
$ cd ~/workspace/paasta-4.0/deployment/service-deployment
```

## <div id='12'/>3.4.	Logsearch 설치

PaaS-TA VM Log수집을 위해서는 logsearch가 설치되어야 한다. 

```
$ cd ~/workspace/paasta-4.0/deployment/service-deployment/logsearch
```

### <div id='13'/>3.4.1.	logsearch-deployment.yml
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

### <div id='14'/>3.4.2. deploy.sh

deploy.sh의 –v 의 inception_os_user_name, router_ip, system_domain 및 director_name을 시스템 상황에 맞게 설정한다.
system_domain은 paasta 설치시 설정했던 system_domain을 입력하면 된다.
router_ip는 ls-router가 설치된 azs에서 정의한 cider값의 적당한 IP를 지정한다.

```
Bosh –e {director_name} -d logsearch deploy logsearch-deployment.yml \
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
$ bosh –e {director_name} vms
```
![PaaSTa_logsearch_vms]


## <div id='15'/>3.5.	PaaS-TA Monitoring 설치

PaaS 모니터링을 위해서 paasta-monitoring가 설치되어야 한다. 

```
$ cd ~/workspace/paasta-4.0/deployment/service-deployment/paasta-monitoring
```

### <div id='16'/>3.5.1.	paasta-monitoring.yml
paasta-monitoring.yml에는 redis, influxdb(metric_db), mariadb, monitoring-web, monitoring-batch에 대한 명세가 있다.

```
---
name: paasta-monitoring                      # 서비스 배포이름(필수) bosh deployments 로 확인 가능한 이름

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
- name: paasta-monitoring  # 서비스 릴리즈 이름(필수) bosh releases로 확인 가능
  version: latest                                              # 서비스 릴리즈 버전(필수):latest 시 업로드된 서비스 릴리즈 최신버전
  url: file:///home/((inception_os_user_name))/workspace/paasta-4.0/release/monitoring/monitoring-release.tgz 
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
    default: [dns, gateway]
    static_ips:
    - ((redis_ip))
  - name: vip
    static_ips:
    - 115.68.151.177 
 
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
  persistent_disk_type: 10GB
  networks:
  - name: default
    default: [dns, gateway]
    static_ips:
    - ((influxdb_ip)) 
  - name: vip
    static_ips: 
    - 115.68.151.187

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

- name: mariadb
  azs: [z5]
  instances: 1
  vm_type: medium 
  stemcell: default
  persistent_disk_type: 5GB
  networks:
  - name: default
    default: [dns, gateway]
    static_ips: ((mariadb_ip))
  - name: vip
    static_ips:
    - 115.68.151.188
  jobs:
  - name: mariadb
    release: paasta-monitoring
    properties:
      mariadb:
        port: ((mariadb_port))                                        #InfluxDB default database
        admin_user:
          password: '((mariadb_password))'                             # MARIA DB ROOT 계정 비밀번호

- name: monitoring-batch
  azs: [z6]
  instances: 1
  vm_type: small 
  stemcell: default
  networks:
  - name: default
  jobs:
  - name: monitoring-batch
    release: paasta-monitoring
    consumes:
      influxdb: {from: influxdb}
    properties:
      monitoring-batch:
        influxdb:
          url: ((influxdb_ip)):8086
        db:
          ip: ((mariadb_ip))
          port: ((mariadb_port))
          username: ((mariadb_username))
          password: ((mariadb_password))
        paasta:
          cell_prefix: ((paasta_cell_prefix))
        bosh:
          url: ((bosh_url))
          password: ((bosh_password))
          director_name: ((director_name))
          paasta:
            deployments: ((paasta_deploy_name))
        mail:
          smtp:
            url: ((smtp_url))
            port: ((smtp_port))
          sender:
            name: ((mail_sender))
            password: ((mail_password))
          resource:
            url: ((resource_url))
          send: ((mail_enable))
          tls: ((mail_tls_enable))
        redis:
          url: ((redis_ip)):6379
          password: ((redis_password))
        paasta:
          url: http://api.((system_domain))
          username: ((paasta_username))
          password: ((paasta_password))

- name: monitoring-web
  azs: [z6]
  instances: 1
  vm_type: small 
  stemcell: default
  networks:
  - name: default
    default: [dns, gateway]
  - name: vip
    static_ips: [((monit_public_ip))]

  jobs:
  - name: monitoring-web
    release: paasta-monitoring
    properties:
      monitoring-web:
        db:
          ip: ((mariadb_ip))
          port: ((mariadb_port))
          username: ((mariadb_username))
          password: ((mariadb_password))
        influxdb:
          url: http://((influxdb_ip)):8086
        paasta:
          system_domain: ((system_domain))
        bosh:
          ip: ((bosh_url))
          password: ((bosh_password))
          director_name: ((director_name))
        redis:
          url: ((redis_ip)):6379
          password: ((redis_password))
        time:
          gap: ((utc_time_gap))

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

### <div id='17'/>3.5.2.	monit-deploy.sh
monit-deploy.sh의 –v 의 inception_os_user_name, system_domain 및 director_name을 시스템 상황에 맞게 설정한다.

```
bosh –e {director_name} -d paasta-monitoring deploy paasta-monitoring.yml  \
     -v inception_os_user_name=ubuntu \
     -v mariadb_ip=10.20.50.11 \  # mariadb vm private IP
     -v mariadb_port=3306 \      # mariadb port
     -v mariadb_username=root \  # mariadb root 계정
     -v mariadb_password=password \  # mariadb root 계정 password
     -v influxdb_ip=10.20.50.15 \   # influxdb vm private IP
     -v bosh_url=10.20.0.7 \        # bosh private IP
     -v bosh_password=2w87no4mgc9mtpc0zyus \  # bosh admin password
     -v director_name=micro-bosh \       # bosh director 명
     -v paasta_deploy_name=paasta \      # paasta deployment 명
     -v paasta_cell_prefix=cell \        # paasta cell 명
     -v paasta_username=admin \          # paasta admin 계정
     -v paasta_password=admin \          # paasta admin password
     -v smtp_url=127.0.0.1 \             # smtp server url
     -v smtp_port=25 \                   # smtp server port
     -v mail_sender=csupshin\            # smtp server admin id
     -v mail_password=xxxx\              # smtp server admin password
     -v mail_enable=flase \              # alarm 발생시 mail전송 여부
     -v mail_tls_enable=false \          # smtp서버 인증시 tls모드인경우 true
     -v redis_ip=10.20.40.11 \           # redis private ip
     -v redis_password=password \        # redis 인증 password
     -v utc_time_gap=9 \                 # utc time zone과 Client time zone과의 시간 차이
     -v monit_public_ip=xxx.xxx.xxx.xxx \ # 설치시 monitoring-web VM의 public ip
     -v system_domain={System_domain}    #PaaS-TA 설치시 설정한 System Domain

```


Note: 
1)	mariadb, influxdb, redis vm은 사용자가 직접 ip를 지정한다. Ip 지정시 paasta-monitoring.yml의 az와 cloud-config의 subnet이 일치하는 ip대역내에 ip를 지정한다.
2)	bosh_url: bosh 설치시 설정한 bosh private ip
3)	bosh_password: bosh admin Password로 bosh deploy시 생성되는 bosh admin password를 입력해야 한다. 
~/workspace/paasta-4.0/deployment/bosh-deployment/{iaas}/creds.yml
creds.yml
admin_password: xxxxxxxxx 
4)	smtp_url: smtp Server ip (PaaS-TA를 설치한 시스템에서 사용가능한 smtp 서버 IP
5)	monit_public_ip: monitoring web의 public ip로 외부에서 모니터링 화면에 접속하기 위해 필요한 외부 ip(public ip 필요)
6)	system_domain: paasta를 설치 할때 설정한 system_domain을 입력한다.


Monit-deploy.sh을 실행하여 paasta-monitoring을 설치 한다
```
$ cd ~/workspace/paasta-4.0/deployment/service-deployment/paasta-monitoring
$ monit-deploy.sh
```

설치 완료후 PaaS-TA 모니터링이 설치 완료 되었음을 확인한다.
```
$ bosh –e {director_name} vms
```
![PaaSTa_monitoring_vms]


### <div id='18'/>3.5.3. monitoring dashboard접속
 
 http://{monitoring-web-ip}:8080 에 접속하여 회원 가입 후 Main Dashboard에 접속한다.

 Login 화면에서 회원 가입 버튼을 클릭한다.

 ![PaaSTa_monitoring_login]


member_info에는 사용자가 사용할 ID/PWD를 입력하고 하단 paas-info에는 PaaS-TA admin 권한의 계정을 입력한다. paasta deploy시 입력한 admin/pwd를 입력해야 한다. 입력후 [인증수행]를 실행후 Joing버튼을 클릭하면 회원가입이 완료된다.

 ![PaaSTa_monitoring_join]

모니터링 main dashboard 화면

 ![PaaSTa_monitoring_main_dashboard]


[PaaSTa_Monit_architecure_Image]:./images/monit_architecture.png
[PaaSTa_Monit_collect_architecure_Image]:./images/collect_architecture.png
[PaaSTa_release_dir]:./images/paasta-release.png
[PaaSTa_logsearch_vms]:./images/logsearch.png
[PaaSTa_monitoring_vms]:./images/paasta-monitoring.png

[PaaSTa_monitoring_login]:./images/monit_login.png
[PaaSTa_monitoring_join]:./images/member_join.png
[PaaSTa_monitoring_main_dashboard]:./images/monit_main.png

