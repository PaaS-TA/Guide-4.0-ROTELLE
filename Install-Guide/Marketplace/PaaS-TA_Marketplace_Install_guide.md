## Table of Contents

[1. 문서 개요](#1)

  -  [1.1. 목적](#11)
  -  [1.2. 범위](#12)
  -  [1.3. 시스템 구성도](#13)

[2. 마켓플레이스 배포](#2)

  -  [2.1. 설치 전 준비사항](#21)
  -  [2.1.1. App 파일 및 Manifest 파일 다운로드](#211)
  -  [2.2. 마켓플레이스 Manifest 파일 수정 및 App 배포](#22)


# <div id='1'/> 1. 문서 개요

### <div id='11'/> 1.1. 목적
본 문서(마켓플레이스 설치 가이드)는 개방형 PaaS 플랫폼 고도화 및 개발자 지원 환경 기반의 Open PaaS에서 마켓플레이스를 설치하는 방법을 기술하였다.


### <div id='12'/> 1.2. 범위
설치 범위는 마켓플레이스 기본 설치를 기준으로 작성하였다.


### <div id='13'/> 1.3. 시스템 구성도
본 문서의 설치된 시스템 구성도이다. 마켓플레이스 Server, DB, Object Storage 로 최소사항을 구성하였다.

![Architecture]

<br>
<br>

# <div id='2'/> 2. 마켓플레이스 배포

### <div id='21'/> 2.1. 설치 전 준비사항
본 설치 가이드는 Linux 환경에서 설치하는 것을 기준으로 하였다.
마켓플레이스를 설치하기 위해서는 먼저 BOSH CLI v2 및 PaaS-TA, PaaS-TA 사용자 포털이 설치되어 있어야 한다.<br>
위에 해당하는 사항들이 설치 되어 있지 않을 경우, 먼저 BOSH 2.0 설치 가이드 문서를 참고하여 BOSH CLI v2를 설치를 하고 사용법을 숙지해야한다.<br>

- 설치 및 사용자 가이드
  >BOSH 2 사용자 가이드 : <https://github.com/PaaS-TA/Guide-4.0-ROTELLE/blob/master/PaaS-TA_BOSH2_사용자_가이드v1.0.md>
  >
  >BOSH CLI V2 사용자 가이드 : <https://github.com/PaaS-TA/Guide-4.0-ROTELLE/blob/master/Use-Guide/Bosh/PaaS-TA_BOSH_CLI_V2_사용자_가이드v1.0.md>
  >
  >CF CLI V2 사용자 가이드 : <https://github.com/PaaS-TA/Guide-1.0-Spaghetti-/blob/master/Use-Guide/OpenPaas%20CLi%20%EA%B0%80%EC%9D%B4%EB%93%9C.md>
  >
  >PaaS-TA 설치 가이드 : <https://github.com/PaaS-TA/Guide-4.0-ROTELLE/blob/master/PaaS_TA_PaaS-TA_Install_Guide-v4.6.md>
  >
  >PaaS-TA 사용자 포털 설치 가이드 : <https://github.com/PaaS-TA/Guide-4.0-ROTELLE/blob/master/Install-Guide/Portal/PaaS-TA_Portal_install.md>


#### <div id='211'/> 2.1.1. App 파일 및 Manifest 파일 다운로드

마켓플레이스 설치에 필요한 App 파일 및 Manifest 파일을 다운로드 받아 서비스 설치 작업 경로로 위치시킨다.

-	설치 파일 다운로드 위치 : https://paas-ta.kr/download/package
-	App, Manifest 파일은 /home/{user_name}/workspace/paasta-4.6 이하에 다운로드 받아야 한다.
-	설치 작업 경로 생성 및 파일 다운로드

>	Manifest 파일
 >> paasta-marketplace

> Application 파일
 >> marketplace-api.jar<br>
 >> marketplace-web-user.war<br>
 >> marketplace-web-seller.war<br>
 >> marketplace-web-admin.war<br>


```
- Deployment 다운로드 파일 위치 경로 생성
$ mkdir -p ~/workspace/paasta-4.6/paasta-marketplace

- Application 파일 다운로드 파일 위치 경로 생성
$ mkdir -p ~/workspace/paasta-4.6/paasta-marketplace
```

-	Manifest 파일을 다운로드 받아 ~/workspace/paasta-4.6/paasta-marketplace 이하 디렉토리에 이동한다.
-	Application 파일을 다운로드 받아 ~/workspace/paasta-4.6/paasta-marketplace 이하 디렉토리에 이동한다.


### <div id='22'/> 2.2. 마켓플레이스 Manifest 파일 수정 및 App 배포

- 마켓플레이스를 설치할 CF 공간을 확인한다.
  - PaaS-TA 어드민 계정으로 포탈을 배포할 조직 및 공간을 생성하거나 배포할 공간으로 target 설정을 한다.
  ```
  $ cf create-quota <쿼타명> -m 100G -i -1 -s -1 -r -1 --reserved-route-ports -1 --allow-paid-service-plans
  $ cf create-org <조직명> -q <생성한 쿼타명>
  $ cf create-space <공간명> -o <생성한 조직명>
  $ cf target –o <조직명> -s <공간명>
  ```

- 생성한 조직과 공간, 쿼타에 대한 GUID 를 확인한다.
  ```
  - 조직 GUID : $ cf org <생성한 조직명> --guid
  - 공간 GUID : $ cf space <생성한 공간명> --guid
  - 쿼타 GUID : $ cf curl "/v2/quota_definitions" => 생성한 쿼타명에 해당하는 resources.metadata.guid
  - 도메인 GUID : $ cf curl "/v2/domains" => resources.metadata.guid
  ```

- 마켓플레이스에 필요한 DBMS 를 신청한다.
  ```
  $ cf service-brokers

  $ cf service-access -b mysql-service-broker

  $ cf create-service <서비스명> <서비스플랜> <내 서비스명>
  예시 : $ cf create-service Mysql-DB Mysql-Plan1-10con marketplace-mysql
  ```
  > MySQL 서비스 신청 참고 : https://github.com/PaaS-TA/Guide-4.0-ROTELLE/blob/master/Service-Guide/DBMS/PaaS-TA%20MySQL%20%EC%84%9C%EB%B9%84%EC%8A%A4%ED%8C%A9%20%EC%84%A4%EC%B9%98%20%EA%B0%80%EC%9D%B4%EB%93%9C.md#32

  또는

  > PaaS-TA 사용자 포탈 참고 : https://github.com/PaaS-TA/Guide-4.0-ROTELLE/blob/master/Use-Guide/portal/PaaS-TA%20%EC%82%AC%EC%9A%A9%EC%9E%90%20%ED%8F%AC%ED%83%88%20%EA%B0%80%EC%9D%B4%EB%93%9C_v1.1.md#36


- 마켓플레이스에 필요한 Object Storage(Swift) 정보를 확인한다.
  ```
  1) 아래 URL 들을 참고하여 Swift 를 설치한다.
  > https://docs.openstack.org/swift/latest/development_saio.html


  2) 설치를 모두 마친 뒤 아래와 같이 Marketplace Api 와 Marketplace Seller 프로젝트의 manifest.yml 에 Swfit 정보를 수정해 준다.

    *******************************< manifest.yml >*******************************
    objectStorage.swift.tenantName: <생성한 Object Storage tenant 이름>
    objectStorage.swift.username: <생성한 Object Storage account 사용자 이름>
    objectStorage.swift.password: <생성한 Object Storage account password>
    objectStorage.swift.authUrl: <생성한 Object Storage API 엔드포인트>
    objectStorage.swift.authMethod: <Object Storage 인증 메서드>
    objectStorage.swift.preferredRegion: <설정해준 Region>
    objectStorage.swift.container: <생성한 Object Storage Container 이름>
    ******************************************************************************

  ```


- 마켓플레이스 manifest 파일
  - 마켓플레이스 manifest는 Components 요소 및 배포의 속성을 정의한 YAML 파일이다. manifest 파일에는 어떤 name, memory, instance, host, path, buildpack, env 등을 사용 할 것인지 정의가 되어 있다.

  > 1) 마켓플레이스의 각 프로젝트 별 manifest 파일을 확인한다.
  ```
  $ vi manifest.yml
  ```

  > 2) 마켓플레이스 App 을 배포하기 위해 필요한 아래의 manifest 파일에 앞서 확인하였던 조직과 공간, 쿼타에 대한 GUID, Object Storage 정보, 마켓플레이스 DBMS 신청 서비스 정보 들을 수정하여 넣는다.

  ```
  [ marketplace-api ]
  ---
  applications:
  - name: marketplace-api
    memory: 1G
    instances: 1
    buildpacks:
    - java_buildpack
    path: ./marketplace-api.jar
    env:
      server_port: 8777
      spring_application_name: marketplace-api
      spring_security_username: admin
      spring_security_password: openpaasta
      spring_datasource_driver-class-name: com.mysql.cj.jdbc.Driver
      spring_datasource_url: jdbc:${vcap.services.Mysql-DB.credentials.uri}/marketplace?characterEncoding=utf8&autoReconnect=true
      spring_datasource_username: ${vcap.services.Mysql-DB.credentials.username}
      spring_datasource_password: ${vcap.services.Mysql-DB.credentials.password}
      spring_jpa_database: mysql
      spring_jpa_hibernate_ddl-auto: update
      spring_jpa_hibernate_use-new-id-generator-mappings: false
      spring_jpa_show-sql: true
      spring_jpa_database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
      spring_jackson_serialization_fail-on-empty-beans: false
      spring_jackson_default-property-inclusion: NON_NULL
      spring_servlet_multipart_max-file-size: 100MB

      ### 수정 필요 ###
      cloudfoundry.cc.api.url: https://api.15.164.20.58.xip.io
      cloudfoundry.cc.api.uaaUrl: https://uaa.15.164.20.58.xip.io
      cloudfoundry.cc.api.sslSkipValidation: true
      cloudfoundry.cc.api.proxyUrl: ""
      cloudfoundry.cc.api.host: ".15.164.20.58.xip.io"
      cloudfoundry.user.admin.username: admin
      cloudfoundry.user.admin.password: 'admin'
      cloudfoundry.user.uaaClient.clientId: login
      cloudfoundry.user.uaaClient.clientSecret: login-secret
      cloudfoundry.user.uaaClient.adminClientId: admin
      cloudfoundry.user.uaaClient.adminClientSecret: admin-secret
      cloudfoundry.user.uaaClient.loginClientId: login
      cloudfoundry.user.uaaClient.loginClientSecret: login-secret
      cloudfoundry.user.uaaClient.skipSSLValidation: true
      cloudfoundry.authorization: cf-Authorization

      ### 수정 필요 ###
      market.org.name: <조직 이름>
      market.org.guid: <조직 GUID>
      market.space.name: <공간 이름>
      market.space.guid: <공간 GUID>
      market.quota_guid: <쿼타 GUID>
      market.domain_guid: <도메인 GUID>
      market.naming-type: "Auto"

      ### 수정 필요 ###
      objectStorage.swift.tenantName: <생성한 Object Storage tenant 이름>
      objectStorage.swift.username: <생성한 Object Storage account 사용자 이름>
      objectStorage.swift.password: <생성한 Object Storage account password>
      objectStorage.swift.authUrl: <생성한 Object Storage API 엔드포인트>
      objectStorage.swift.authMethod: keystone
      objectStorage.swift.preferredRegion: Public
      objectStorage.swift.container: <생성한 Object Storage Container 이름>

      provisioning.pool-size: 3
      provisioning.try-count: 3
      provisioning.timeout: 3600000
      provisioning.ready-fixed-rate: 10000
      provisioning.ready-initial-delay: 3000
      provisioning.progress-fixed-rate: 10000
      provisioning.progress-initial-delay: 5000
      provisioning.timeout-fixed-rate: 30000
      provisioning.timeout-initial-delay: 1700

      deprovisioning.pool-size: 3
      deprovisioning.try-count: 3
      deprovisioning.timeout: 3600000
      deprovisioning.ready-fixed-rate: 10000
      deprovisioning.ready-initial-delay: 7000
      deprovisioning.progress-fixed-rate: 10000
      deprovisioning.progress-initial-delay: 13000
      deprovisioning.timeout-fixed-rate: 30000
      deprovisioning.timeout-initial-delay: 1700

      task.execution.restrict-to-same-host: false

  ```

  <br>

  - 앞서 확인하였던 Object Storage 정보, 마켓플레이스 DBMS 신청 서비스 정보 들을 아래의 manifest 파일에 수정하여 넣는다. 다만 marketplace api 와는 달리 marketplace web-user/ web-seller/ web-admin 의 경우에는 'marketplace_api_url' 에 미리 배포하였던 marketplace-api 의 url 을 넣어 수정한다.

  ```
  [ marketplace-web-admin ]
  ---
  applications:
  - name: marketplace-webadmin
    memory: 1G
    instances: 1
    buildpacks:
    - java_buildpack
    path: ./marketplace-web-admin.war
    env:
      server_port: 8778
      spring_application_name: marketplace-webadmin
      spring_servlet_multipart_max-file-size: 1024MB
      spring_servlet_multipart_max-request-size: 1024MB
      spring_session_store-type: jdbc
      spring_session_jdbc_initialize-schema: always
      spring_session_jdbc_schema: classpath:org/springframework/session/jdbc/schema-mysql.sql
      spring_mvc_static-path-pattern: /static/**
      spring_datasource_driver-class-name: com.mysql.cj.jdbc.Driver
      spring_datasource_url: jdbc:${vcap.services.Mysql-DB.credentials.uri}/marketplace_admin?characterEncoding=utf8&autoReconnect=true
      spring_datasource_username: ${vcap.services.Mysql-DB.credentials.username}
      spring_datasource_password: ${vcap.services.Mysql-DB.credentials.password}

      ### 수정 필요 ###
      marketplace_api_url: http://marketplace-api.15.164.20.58.xip.io   # 먼저 배포한 'marketplace-api' App 의 urls
      marketplace_registration: cf
      marketplace_client-id: marketclient
      marketplace_client-secret: clientsecret
      marketplace_redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"
      ### AWS
      marketplace_authorization-uri: https://uaa.15.164.20.58.xip.io/oauth/authorize
      marketplace_token-uri: https://uaa.15.164.20.58.xip.io/oauth/token
      marketplace_user-info-uri: https://uaa.15.164.20.58.xip.io/userinfo
      marketplace_jwk-set-uri: https://uaa.15.164.20.58.xip.io/token_keys

  ```
<br>
<br>

- 마켓플레이스 App 배포 (처음 배포 시에 --no-start 옵션을 넣어준다.)

  - 마켓플레이스 API App 배포
  ```
  $ cf push marketplace-api -f manifest.yml --no-start
  Pushing from manifest to org market-org / space dev as admin...
  Using manifest file manifest-test.yml
  Getting app info...
  Creating app with these attributes...
  + name:         marketplace-api
    path:         /home/ubuntu/workspace/user/hrjin/marketplace/api/marketplace-api.jar
    buildpacks:
  +   java_buildpack
  + instances:    1
  + memory:       1G
    env:
  +   cloudfoundry.authorization
  +   cloudfoundry.cc.api.host
  +   cloudfoundry.cc.api.proxyUrl
  +   cloudfoundry.cc.api.sslSkipValidation
  +   cloudfoundry.cc.api.uaaUrl
  +   cloudfoundry.cc.api.url
  +   cloudfoundry.user.admin.password
  +   cloudfoundry.user.admin.username
  +   cloudfoundry.user.uaaClient.adminClientId
  +   cloudfoundry.user.uaaClient.adminClientSecret
  +   cloudfoundry.user.uaaClient.clientId
  +   cloudfoundry.user.uaaClient.clientSecret
  +   cloudfoundry.user.uaaClient.loginClientId
  +   cloudfoundry.user.uaaClient.loginClientSecret
  +   cloudfoundry.user.uaaClient.skipSSLValidation
  +   deprovisioning.pool-size
  +   deprovisioning.progress-fixed-rate
  +   deprovisioning.progress-initial-delay
  +   deprovisioning.ready-fixed-rate
  +   deprovisioning.ready-initial-delay
  +   deprovisioning.timeout
  +   deprovisioning.timeout-fixed-rate
  +   deprovisioning.timeout-initial-delay
  +   deprovisioning.try-count
  +   market.domain_guid
  +   market.naming-type
  +   market.org.guid
  +   market.org.name
  +   market.quota_guid
  +   market.space.guid
  +   market.space.name
  +   objectStorage.swift.authMethod
  +   objectStorage.swift.authUrl
  +   objectStorage.swift.container
  +   objectStorage.swift.password
  +   objectStorage.swift.preferredRegion
  +   objectStorage.swift.tenantName
  +   objectStorage.swift.username
  +   provisioning.pool-size
  +   provisioning.progress-fixed-rate
  +   provisioning.progress-initial-delay
  +   provisioning.ready-fixed-rate
  +   provisioning.ready-initial-delay
  +   provisioning.timeout
  +   provisioning.timeout-fixed-rate
  +   provisioning.timeout-initial-delay
  +   provisioning.try-count
  +   server_port
  +   spring_application_name
  +   spring_datasource_driver-class-name
  +   spring_datasource_password
  +   spring_datasource_url
  +   spring_datasource_username
  +   spring_jackson_default-property-inclusion
  +   spring_jackson_serialization_fail-on-empty-beans
  +   spring_jpa_database
  +   spring_jpa_database-platform
  +   spring_jpa_hibernate_ddl-auto
  +   spring_jpa_hibernate_use-new-id-generator-mappings
  +   spring_jpa_show-sql
  +   spring_security_password
  +   spring_security_username
  +   spring_servlet_multipart_max-file-size
  +   spring_servlet_multipart_max-request-size
  +   task.execution.restrict-to-same-host
    routes:
  +   marketplace-api.15.164.20.58.xip.io

  Creating app marketplace-api...
  Mapping routes...
  Comparing local files to remote cache...
  Packaging files to upload...
  Uploading files...
   938.31 KiB / 938.31 KiB [=================================================================================================================================================================================] 100.00% 1s

  Waiting for API to complete processing files...

  name:              marketplace-api
  requested state:   stopped
  routes:            marketplace-api.15.164.20.58.xip.io
  last uploaded:     
  stack:             
  buildpacks:        

  type:           web
  instances:      0/1
  memory usage:   1024M
       state   since                  cpu    memory   disk     details
  #0   down    2019-09-25T01:40:24Z   0.0%   0 of 0   0 of 0     


  ```

  - 마켓플레이스 Web Admin App 배포 (marketplace-webseller 와 marketplace-webuser 도 같은 방식)
  ```
  $ cf push marketplace-webadmin -f manifest.yml --no-start
  Pushing from manifest to org market-org / space dev as admin...
  Using manifest file manifest-test.yml
  Getting app info...
  Creating app with these attributes...
  + name:         marketplace-webadmin
    path:         /home/ubuntu/workspace/user/hrjin/marketplace/admin/marketplace-web-admin.war
    buildpacks:
  +   java_buildpack
  + instances:    1
  + memory:       1G
    env:
  +   marketplace_api_url
  +   marketplace_authorization-uri
  +   marketplace_client-id
  +   marketplace_client-secret
  +   marketplace_jwk-set-uri
  +   marketplace_redirect-uri
  +   marketplace_registration
  +   marketplace_token-uri
  +   marketplace_user-info-uri
  +   server_port
  +   spring_application_name
  +   spring_datasource_driver-class-name
  +   spring_datasource_password
  +   spring_datasource_url
  +   spring_datasource_username
  +   spring_mvc_static-path-pattern
  +   spring_servlet_multipart_max-file-size
  +   spring_servlet_multipart_max-request-size
  +   spring_session_jdbc_initialize-schema
  +   spring_session_jdbc_schema
  +   spring_session_store-type
    routes:
  +   marketplace-webadmin.15.164.20.58.xip.io

  Creating app marketplace-webadmin...
  Mapping routes...
  Comparing local files to remote cache...
  Packaging files to upload...
  Uploading files...
   1.66 MiB / 1.66 MiB [=====================================================================================================================================================================================] 100.00% 1s

  Waiting for API to complete processing files...

  name:              marketplace-webadmin
  requested state:   stopped
  routes:            marketplace-webadmin.15.164.20.58.xip.io
  last uploaded:     
  stack:             
  buildpacks:        

  type:           web
  instances:      0/1
  memory usage:   1024M
       state   since                  cpu    memory   disk     details
  #0   down    2019-09-25T06:25:07Z   0.0%   0 of 0   0 of 0   

  ```

<br>

- 생성된 Marketplace Api App을 확인한다.
```
$ cf apps
Getting apps in org market-org / space dev as admin...
OK

name                   requested state   instances   memory   disk   urls
marketplace-api        stopped           0/1         1G       1G     marketplace-api.15.164.20.58.xip.io
marketplace-webadmin   stopped           0/1         1G       1G     marketplace-webadmin.15.164.20.58.xip.io
marketplace-webuser    stopped           0/1         1G       1G     marketplace-webuser.15.164.20.58.xip.io
marketplace-webseller  stopped           0/1         1G       1G     marketplace-webseller.15.164.20.58.xip.io
```


<br>

- 신청한 백엔드 서비스를 생성한 4개의 App 과 하나씩 각각 바인딩한다.
```
$ cf services
Getting services in org market-org / space dev as admin...

name                service    plan                bound apps        last operation     broker                 upgrade available
marketplace-mysql   Mysql-DB   Mysql-Plan1-10con   marketplace-api   create succeeded   mysql-service-broker   


$ cf bind-service <생성한 App 이름> <신청한 서비스>
예시 : $ cf bind-service marketplace-api marketplace-mysql

```

<br>

- App 을 marketplace-api 를 가장 먼저 시작한다.
  - [marketplace-api]
  ```
  $ cf start marketplace-api
  Starting app marketplace-api in org market-org / space dev as admin...

  Staging app and tracing logs...
     Downloading java_buildpack...
     Downloaded java_buildpack
     Cell 81d75576-549f-4b9e-8ddc-eb65410d877d creating container for instance cca4ceca-1ad2-4c00-b930-3af1ed28d2ed
     Cell 81d75576-549f-4b9e-8ddc-eb65410d877d successfully created container for instance cca4ceca-1ad2-4c00-b930-3af1ed28d2ed
     Downloading app package...
     Downloaded app package (53M)
     -----> Java Buildpack v4.19 | https://github.com/cloudfoundry/java-buildpack.git#3f4eee2
     -----> Downloading Jvmkill Agent 1.16.0_RELEASE from https://java-buildpack.cloudfoundry.org/jvmkill/bionic/x86_64/jvmkill-1.16.0-RELEASE.so (0.1s)
     -----> Downloading Open Jdk JRE 1.8.0_222 from https://java-buildpack.cloudfoundry.org/openjdk/bionic/x86_64/openjdk-jre-1.8.0_222-bionic.tar.gz (5.5s)
            Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (0.9s)
            JVM DNS caching disabled in lieu of BOSH DNS caching
     -----> Downloading Open JDK Like Memory Calculator 3.13.0_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/bionic/x86_64/memory-calculator-3.13.0-RELEASE.tar.gz (0.0s)
            Loaded Classes: 21392, Threads: 250
     -----> Downloading Client Certificate Mapper 1.11.0_RELEASE from https://java-buildpack.cloudfoundry.org/client-certificate-mapper/client-certificate-mapper-1.11.0-RELEASE.jar (0.0s)
     -----> Downloading Container Security Provider 1.16.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-security-provider/container-security-provider-1.16.0-RELEASE.jar (0.0s)
     -----> Downloading Spring Auto Reconfiguration 2.9.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-2.9.0-RELEASE.jar (5.1s)
     Exit status 0
     Uploading droplet, build artifacts cache...
     Uploading droplet...
     Uploading build artifacts cache...
     Uploaded build artifacts cache (43.4M)
     Uploaded droplet (96.5M)
     Uploading complete
     Cell 81d75576-549f-4b9e-8ddc-eb65410d877d stopping instance cca4ceca-1ad2-4c00-b930-3af1ed28d2ed
     Cell 81d75576-549f-4b9e-8ddc-eb65410d877d destroying container for instance cca4ceca-1ad2-4c00-b930-3af1ed28d2ed
     Cell 81d75576-549f-4b9e-8ddc-eb65410d877d successfully destroyed container for instance cca4ceca-1ad2-4c00-b930-3af1ed28d2ed

  Waiting for app to start...

  name:              marketplace-api
  requested state:   started
  routes:            marketplace-api.15.164.20.58.xip.io
  last uploaded:     Wed 25 Sep 10:43:50 KST 2019
  stack:             cflinuxfs3
  buildpacks:        java_buildpack

  type:            web
  instances:       1/1
  memory usage:    1024M
  start command:   JAVA_OPTS="-agentpath:$PWD/.java-buildpack/open_jdk_jre/bin/jvmkill-1.16.0_RELEASE=printHeapHistogram=1 -Djava.io.tmpdir=$TMPDIR -XX:ActiveProcessorCount=$(nproc)
                   -Djava.ext.dirs=$PWD/.java-buildpack/container_security_provider:$PWD/.java-buildpack/open_jdk_jre/lib/ext -Djava.security.properties=$PWD/.java-buildpack/java_security/java.security $JAVA_OPTS" &&
                   CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-3.13.0_RELEASE -totMemory=$MEMORY_LIMIT -loadedClasses=22692 -poolType=metaspace -stackThreads=250
                   -vmOptions="$JAVA_OPTS") && echo JVM Memory Configuration: $CALCULATED_MEMORY && JAVA_OPTS="$JAVA_OPTS $CALCULATED_MEMORY" && MALLOC_ARENA_MAX=2 SERVER_PORT=$PORT eval exec
                   $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.JarLauncher
       state     since                  cpu    memory         disk           details
  #0   running   2019-09-25T01:44:15Z   0.0%   115.6M of 1G   170.7M of 1G   

  ```

  - [marketplace-webadmin] (Web Seller 와 Web User 도 같은 방식)
  ```
  $ cf start marketplace-webadmin
  Starting app marketplace-webadmin in org market-org / space dev as admin...

  Staging app and tracing logs...
     Downloading java_buildpack...
     Downloaded java_buildpack
     Cell 81d75576-549f-4b9e-8ddc-eb65410d877d creating container for instance cfd9a5b6-d011-4d99-bcaf-cca33beca89a
     Cell 81d75576-549f-4b9e-8ddc-eb65410d877d successfully created container for instance cfd9a5b6-d011-4d99-bcaf-cca33beca89a
     Downloading app package...
     Downloaded app package (72.2M)
     -----> Java Buildpack v4.19 | https://github.com/cloudfoundry/java-buildpack.git#3f4eee2
     -----> Downloading Jvmkill Agent 1.16.0_RELEASE from https://java-buildpack.cloudfoundry.org/jvmkill/bionic/x86_64/jvmkill-1.16.0-RELEASE.so (5.1s)
     -----> Downloading Open Jdk JRE 1.8.0_222 from https://java-buildpack.cloudfoundry.org/openjdk/bionic/x86_64/openjdk-jre-1.8.0_222-bionic.tar.gz (0.5s)
            Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (0.9s)
            JVM DNS caching disabled in lieu of BOSH DNS caching
     -----> Downloading Open JDK Like Memory Calculator 3.13.0_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/bionic/x86_64/memory-calculator-3.13.0-RELEASE.tar.gz (0.0s)
            Loaded Classes: 20682, Threads: 250
     -----> Downloading Client Certificate Mapper 1.11.0_RELEASE from https://java-buildpack.cloudfoundry.org/client-certificate-mapper/client-certificate-mapper-1.11.0-RELEASE.jar (0.0s)
     -----> Downloading Container Customizer 2.6.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-customizer/container-customizer-2.6.0-RELEASE.jar (0.0s)
     -----> Downloading Container Security Provider 1.16.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-security-provider/container-security-provider-1.16.0-RELEASE.jar (5.0s)
     -----> Downloading Spring Auto Reconfiguration 2.9.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-2.9.0-RELEASE.jar (0.1s)
     Exit status 0
     Uploading droplet, build artifacts cache...
     Uploading build artifacts cache...
     Uploading droplet...
     Uploaded build artifacts cache (43.4M)
     Uploaded droplet (115.6M)
     Uploading complete
     Cell 81d75576-549f-4b9e-8ddc-eb65410d877d stopping instance cfd9a5b6-d011-4d99-bcaf-cca33beca89a
     Cell 81d75576-549f-4b9e-8ddc-eb65410d877d destroying container for instance cfd9a5b6-d011-4d99-bcaf-cca33beca89a
     Cell 81d75576-549f-4b9e-8ddc-eb65410d877d successfully destroyed container for instance cfd9a5b6-d011-4d99-bcaf-cca33beca89a

  Waiting for app to start...

  name:              marketplace-webadmin
  requested state:   started
  routes:            marketplace-webadmin.15.164.20.58.xip.io
  last uploaded:     Wed 25 Sep 15:50:02 KST 2019
  stack:             cflinuxfs3
  buildpacks:        java_buildpack

  type:            web
  instances:       1/1
  memory usage:    1024M
  start command:   JAVA_OPTS="-agentpath:$PWD/.java-buildpack/open_jdk_jre/bin/jvmkill-1.16.0_RELEASE=printHeapHistogram=1 -Djava.io.tmpdir=$TMPDIR -XX:ActiveProcessorCount=$(nproc)
                   -Djava.ext.dirs=$PWD/.java-buildpack/container_security_provider:$PWD/.java-buildpack/open_jdk_jre/lib/ext -Djava.security.properties=$PWD/.java-buildpack/java_security/java.security $JAVA_OPTS" &&
                   CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-3.13.0_RELEASE -totMemory=$MEMORY_LIMIT -loadedClasses=21985 -poolType=metaspace -stackThreads=250
                   -vmOptions="$JAVA_OPTS") && echo JVM Memory Configuration: $CALCULATED_MEMORY && JAVA_OPTS="$JAVA_OPTS $CALCULATED_MEMORY" && MALLOC_ARENA_MAX=2 SERVER_PORT=$PORT eval exec
                   $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.WarLauncher
       state     since                  cpu    memory         disk           details
  #0   running   2019-09-25T06:50:20Z   0.0%   153.7M of 1G   222.5M of 1G   


  ```


<br>

- 배포된 마켓플레이스 관련 App 들을 확인한다.


```
$ cf apps
Getting apps in org market-org / space dev as admin...
OK

name                    requested state   instances   memory   disk   urls
marketplace-webseller   started           1/1         1G       1G     marketplace-webseller.15.164.20.58.xip.io
marketplace-webadmin    started           1/1         1G       1G     marketplace-webadmin.15.164.20.58.xip.io
marketplace-webuser     started           1/1         1G       1G     marketplace-webuser.15.164.20.58.xip.io
marketplace-api         started           1/1         1G       1G     marketplace-api.15.164.20.58.xip.io

```



[Architecture]:./Market_Place_Architecture.png
