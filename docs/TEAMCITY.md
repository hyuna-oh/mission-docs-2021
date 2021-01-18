# TeamCity & Gitlab
- 2021.01.11 ~ 2021.01.17 동안 작업한 내용
- TeamCity는 JetBrains의 빌드 관리 및 지속적 통합 서버를 위한 빌드 관리 
- Gitlab은 깃랩 사가 개발한 깃 저장소 및 CI/CD, 이슈 추적, 보안성 테스트 등의 기능을 갖춘 웹 기반의 데브옵스 플랫폼

## 1. Vagrant + CentOS 7에서 Gitlab 설치
1. ```vagrant init bento/centos-7``` 으로 새로운 CentOS 7 프로젝트 생성 (https://app.vagrantup.com/boxes/search 참고)
2. 생성 후 Gitlab 설치
- install_gitlab.sh 라는 쉘스크립트를 Vagrantfile이 있는 디렉토리에 작성 후 저장한 뒤 ``reload --provision`` 명령어로 실행시킨다.  
  (provision 적용이 안되는 경우가 있는데, 이럴 땐 그냥 ```vagrant halt```와 ```vagrant up``` 명령어로 재시작을 하면 적용된다.)
- 나머지는 gitlab 설치 페이지를 보고 설치하면 됨.
- install_gitlab.sh  
**주의** wget 을 제외한 모든 줄은 ```vagrant ssh``` 쉘에서 진행하는 게 좋음. 그래야 문제가 생겨도 해결하기가 수월함.
```
echo ==== Installing GitLab CE =================================================
sudo yum install wget
wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/7/gitlab-ce-13.4.7-ce.0.el7.x86_64.rpm/download.rpm
yum install -y epel-release
yum install -y curl policycoreutils-python openssh-server postfix
systemctl enable sshd
systemctl enable postfix
systemctl start sshd
systemctl start postfix
sudo yum localinstall gitlab-ce-13.4.7-ce.0.el7.x86_64.rpm
sudo yum update
```
- Vagrantfile 
```
Vagrant.configure("2") do |config|
    config.vm.provision "shell", path: "install_gitlab.sh"
end
```
- 결과 로그 메시지
```
Running handlers:
Running handlers complete
Chef Infra Client finished, 567/1530 resources updated in 03 minutes 44 seconds
gitlab Reconfigured!
```

## 2. Vagrant + CentOS 7에서 TeamCity Server 및 TeamCity Agent 설치
### 2.1 TeamCity Server 설치
- **- 웬만하면 sudo나 root 계정으로 접속하기**  
1. java를 먼저 설치한 후 진행
```
# 설치 된 자바버전을 확인 후
rpm -qa | grep jdk  
# 자바 버전 삭제
yum remove [해당 자바 버전]  
# 설치할 수 있는 자바 버전을 확인 후
# (여기서, java-버전-openjdk 패키지가 JRE, java-버전-openjdk-devel 패키지가 JDK라고 생각하면 된다.)
yum list java*jdk
yum list java*jdk-devel
# 해당 자바 버전을 설치
sudo yum install -y [원하는 java*jdk 버전]
sudo yum install -y [원하는 java*jdk-devel 버전]
# 설치 확인
rpm -qa java*jdk-devel
javac -version
# 환경변수 설정 (vi /etc/profile 제일 아래 추가)
   export JAVA_HOME=/usr/local/jdk버전
   export PATH=$PATH:$JAVA_HOME/bin
   export CLASSPATH=.:$JAVA_HOME/lib/tools.jar
```
2. ```vagrant ssh```로 쉘에 접속하여 다음의 코맨드들을 입력
 ```
 # 팀시티 서버 설치
 wget https://download.jetbrains.com/teamcity/TeamCity-2020.2.1.tar.gz
 # postgresql 도 필요하기 때문에 설치
 yum install postgresql postgresql-server
 # OS 부팅시 postgres 자동 실행
 systemctl enable postgresql
 # postgres 시작 전 데이터베이스 초기화
 systemctl enable postgresql
 # Postgres 설정 파일인 /var/lib/pgsql/data/pg_hba.conf 파일을 vi로 오픈하여 local connection에 대해서 METHOD를 md5로 수정
 vi /var/lib/pgsql/data/pg_hba.conf
 ```
 - pg_hba.conf file 
 ```
 # TYPE  DATABASE        USER            ADDRESS                 METHOD 
                                                                       
# "local" is for Unix domain socket connections only                   
local   all             all                                     peer   
# IPv4 local connections:                                              
host    all             all             127.0.0.1/32            md5    
# IPv6 local connections:                                              
host    all             all             ::1/128                 md5    
# Allow replication connections from localhost, by a user with the     
# replication privilege.                                               
#local   replication     postgres                                peer  
#host    replication     postgres        127.0.0.1/32            ident 
#host    replication     postgres        ::1/128                 ident 
```
3. 설치가 완료 되면 postgres 구동  
(만약 ```su - postgres -c 'psql'``` 명령어를 입력했을 때 Authentication failure 에러가 난다면, ```sudo su - postgres -c 'psql'``` 명령어로 변경해 주거나 root로 접속하여 해당 명령어 수행)
```
# postgresql 구동
systemctl start postgresql
# postgres 계정 비밀번호 수정
su - postgres -c 'psql'
# 데이터베이스 생성
postgres=# alter user postgres password 'postgres';
postgres=# create database teamcity;
```
4. postgres 설치 완료 시 다음의 순서대로 TeamCity 설치
```
# tar xvfz TeamCity-2020.2.1.tar.gz
# cd TeamCity/bin
# sh catalina.sh
```
※ HOST 브라우저에서 http://[해당 IP]:8111 접속이 안 될 경우, 다음의 커맨드들을 실행 후 GUEST VM의 GUI로 해당 포트 접속 (참고로 오래걸림)
```
# sudo yum update
## group list를 확인한 후
# yum group list
## 해당 group들을 설치
# sudo yum groupinstall "GNOME Desktop" "Graphical Administration Tools"
## 설치한 파일들의 run level을 변경한 후
# sudo ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target
## 재시작
# sudo reboot
```
※ 만약 위의 방법으로도 접속이 안된다면 80 포트가 이미 사용중이라서 그럴 수 있음  
- gitlab에 설치된 nginx 포트가 80이라 생기는 문제일수도 있음. 그러므로 TeamCity의 port를 변경해줌.
그럴 땐 ```TeamCity/conf/server.xml``` 과 ```TeamCity\buildAgent\conf\buildAgent.properties```에서 80포트를 81포트로 바꿔 줌 
```
<Connector port="80" protocol="HTTP/1.1"
           connectionTimeout="60000"
           redirectPort="8543"
           useBodyEncodingForURI="true" />
```
- ```https://serverfault.com/questions/585528/set-gitlab-external-web-port-number``` 를 참고하여 gitlab의 nginx 포트를 변경해줘도 됨.

※ JDK 버전 충돌일 수 있음
- TeamCity 버전이 2019.2.1 이상부터는 JDK 8u242+ 이하의 버전을 사용할 때 ```java.lang.NoClassDefFoundError: Could not initialize class XXX errors, ```와 같은 에러가 발생할 수 있음.
- 현재 2020.2.1 버전이므로 jdk 최신버전으로 돌려보기  

* 결국 재설치하니까 됨..ㅠㅠ
5. TeamCity 실행
- ```sudo sh bin/catalina.sh run``` 명령어로 실행  (혹은 ```sh bin/runAll.sh start```)
(만약 TeamCity 실행 실패시 해당 log 참조)
### 2.2 TeamCity Server 설정
1. TeamCity의 기본 포트는 8111 이므로, 로컬 브라우저에서 [VM IP]:8111로 접속
2. database type : postgresql / database host : localhost / name : teamcity / username : postgres / passwd : postgres 로 설정
3. admin 계정 생성

### 2.3 TeamCity Agent 설치
- TeamCity는 빌드를 전담하는 Agent가 필요. 이 Agent는 TeamCity가 설치되어 있는 서버에 설치를 해도 되고, 다른 서버에 설치를 해도 됨. 
- TeamCity Build Agent는 소스코드를 다운로드하고 빌드를 수행하기 때문에 Disk 소비가 많으므로 디스크 공간이 충분한 파티션을 선택
1. "메뉴 > Agents > Install Build Agents"를 클릭
2. Full zip file distribution의 link를 복사
3. Build Agent를 설치
```
# mkdir <BUILD_AGENT_HOME>
# cd <BUILD_AGENT_HOME>
# wget http://<TEAMCITY_SERVER_IP>:8111/update/buildAgentFull.zip
# unzip buildAgentFull.zip
# ls
bin  BUILD_85633  buildAgentFull.zip  conf  contrib  launcher  lib  plugins  service.properties  system
```
4. Build Agent가 TeamCity Server와 통신할 수 있도록 환경설정 파일을 수정
```
# cp <BUILD_AGENT_HOME>/conf/buildAgent.dist.properties <BUILD_AGENT_HOME>/conf/buildAgent.properties
# vi <BUILD_AGENT_HOME>/conf/buildAgent.properties
## TeamCity build agent configuration file

######################################
#   Required Agent Properties        #
######################################

## The address of the TeamCity server. The same as is used to open TeamCity web interface in the browser.
## Example:  serverUrl=https://buildserver.mydomain.com:8111
serverUrl=http://localhost:8111/

## The unique name of the agent used to identify this agent on the TeamCity server
## Use blank name to let server generate it.
## By default, this name would be created from the build agent's host name
name=TeamCity Build Agent
... 생략
```
- 자세한 사항은 wiki의 TeamCity Agent 설치 항목을 참조
5. 커맨드 실행
```
# cd <BUILD_AGENT_HOME>/bin
# ./agent.sh start
Starting TeamCity build agent...
Java executable is found: '/usr/lib/jvm/jre/bin/java'
Starting TeamCity Build Agent Launcher...
Agent home directory is /root/build-agent
Agent Launcher Java runtime version is 1.8
Lock file: /root/build-agent/logs/buildAgent.properties.lock
Using no lock
Done [6532], see log at /root/build-agent/logs/teamcity-agent.log
```
## 3. Gitlab에 프로젝트 생성
- HOST PC에서 브라우저로 접속 후 ```localhost:80```으로 접속한 뒤 (nginx 서버가 gitlab에 설치되어 있음) UI에서 프로젝트를 생성
- *** 중요 *** : 꼭 시작전에 project path를 정해놓자!
1. 다음의 명령어를 실행하여 vi로 gitlab.rb 편집창을 연다.
```sudo vi /etc/gitlab/gitlab.rb```
2. git_data_dirs 문자를 찾아 다음의 소스코드를 작성한다. (밑에는 여러 경로에 저장할 경우)
```
git_data_dirs({ 
  "default" => { "path" => "/data/git-data" }, 
  "nfs_1" => { "path" => "/mnt/nfs1/git-data" }, 
  "nfs_2" => { "path" => "/mnt/nfs2/git-data" } 
})
```
```
!TODO - SSH KEY 추가해보기!
```

```
!TODO https://stackoverflow.com/questions/31813080/windows-10-ssh-keys : 참고
```
## 4. https://start.spring.io/ 에서 REST Web 프로젝트를 구성해서 Gitlab 프로젝트에 소스코드 추가
- 현재 예시 소스로 작업한 내용은 GUEST VM의 gitlab에 등록되어 있음
- maven repository를 이용
- 트리구조 [예시]
```
[Project명]
ㄴ.idea
ㄴlog
ㄴsrc
  ㄴpackage 명(ex) com.example.employee
  ㄴmodel
  ㄴrepository
  ㄴservice
  ㄴshared
  ㄴutil
  package-info.java
  Starter
ㄴresource
  ㄴmybatis
    ㄴmapper.xml
  ㄴapplication.properties
  ㄴapplication.yml
  ㄴlog4jdbc-log4j2.properties
  ㄴlogback.xml
  ㄴmybatis-config.xml
ㄴ.gitignore
ㄴpom.xml
ㄴREADME.md
```
- gitignore 파일
```
HELP.md
target/
*.log

.idea
*.iws
*.iml
*.ipr

/dist
build/
```
- 참고 : ```https://spring.io/guides/tutorials/rest/```

## 5. TeamCity에서 프로젝트 추가하고 빌드 설정
### 5.1 빌드
- Create project를 클릭 후 Manually로 선택한 후 Name에 프로젝트명을 입력하고 Project ID에 적당한 식별자를 입력
- 나머지는 TeamCity 빌드 부분 위키 참고 (빌드까지 가능)

### 5.2 배포 방법1 (Container Deploy를 통한 배포)  
#### 5.2.1 Tomcat 8.0 설치
1. Tomcat 8 버전을 GUEST VM에 설치한다.  
- **NOTE** 8080 포트를 사용하고 있는지 꼭 확인한다.
- 다음의 명령어로 Tomcat 8을 설치한다.
```
# wget http://archive.apache.org/dist/tomcat/tomcat-8/v8.5.27/bin/apache-tomcat-8.5.27.tar.gz

// 압축 해체
# tar zxvf apache-tomcat-8.5.27.tar.gz

// 톰캣을 /usr/local/로 이동시키고 디렉토리 이름을 tomcat8로 변경
# mv apache-tomcat-8.5.27 /usr/local/tomcat8
```
- URL 인코딩 설정
```
// vi /usr/local/tomcat8/conf/server.xml
// 아래 설정을 찾아서 URIEncoding="UTF-8"을 추가한다.

...
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               URIEncoding="UTF-8" />
...
```
- 환경변수 설정 (CATALINA_HOME)
```
...

JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64
CATALINA_HOME=/usr/local/tomcat8
CLASSPATH=$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar:$CATALINA_HOME/lib-jsp-api.jar:$CATALINA_HOME/lib/servlet-api.jar
PATH=$PATH:$JAVA_HOME/bin:/bin:/sbin
export JAVA_HOME PATH CLASSPATH CATALINA_HOME
```


2. 설치 후 TeamCity에서 Tomcat의 manager 디렉토리 수정 및 등록 권한을 위해 다음의 파일들을 설정한다. (안하면 403 에러가 발생함)
- conf 디렉터리 tomcat-users.xml 설정
```
# vi [tomcat 디렉토리]/conf/tomcat-users.xml
```
- 편집기에 다음과 같이 내용을 입력한다.
```
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">
 
<role rolename="admin"/>
<role rolename="admin-gui"/>
<role rolename="admin-script"/>
<role rolename="manager"/>
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>
<user username="유저명" password="비밀번호" roles="admin,manager,admin-gui,admin-script,manager-gui,manager-script,manager-jmx,manager-status" />
 
</tomcat-users>
```
- manager 디렉터리 context.xml 설정
```
# vi [tomcat 디렉토리]/webapps/manager/META-INF/context.xml
```
- 편집기가 열리면 다음 내용을 입력한다.
```
<Context antiResourceLocking="false" privileged="true" >

  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow=".*" />

  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>
```
3. conf/server.xml 파일을 다음과 같이 수정한다. (port 및 address 설정을 바꾸는 게 중요)
```
<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" address="0.0.0.0" /> 
```

4. Tomcat 실행
```
# /usr/local/tomcat8/bin/startup.sh

//톰캣 프로세스 확인
# ps -ef|grep tomcat8

// 8080 포트가 열려있는지 확인 
# netstat -tln
# wget http://localhost:8080/
```

5. 서비스 등록 
- vi /etc/systemd/system/tomcat8.service
```
# Systemd unit file for tomcat
[Unit]
Description=Apache Tomcat Web Application Container
After=syslog.target network.target

[Service]
Type=forking

Environment="JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64/"
Environment="CATALINA_HOME=/usr/local/tomcat8"
Environment="CATALINA_BASE=/usr/local/tomcat8"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"
Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom"

ExecStart=/usr/local/tomcat8/bin/startup.sh
ExecStop=/usr/local/tomcat8/bin/shutdown.sh

User=root
Group=root
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```
- systemctl 설정
```
# systemctl daemon-reload
# systemctl enable tomcat8

// tomcat8 실행
# systemctl start tomcat8
```
- 부팅 시 자동 실행
```
//부팅 시 자동 실행 서비스 등록
# systemctl enable tomcat8.service
//등록된 서비스 조회
# systemctl list-unit-files --type service |grep tomcat8
```
#### 5.2.2 Spring boot project에 다음의 내용들을 추가해서 war file을 target에 생성
1. main 클래스에 SpringBootServletInitializer 상속받는 것으로 변경
- DemoApplication.java
```
@SpringBootApplication
public class DemoApplication extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(DemoApplication.class);
    }

}
```
2. pom.xml에 tomcat scope를 provided로 변경하고 packaging을 war로 변경
- pom.xml
```
	<packaging>war</packaging>
  ... 생략
  
  <!-- 패키지를 jar로 배포하기 위해 필요 -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
  </dependency>
```

#### 5.2.3 TeamCity Build Configuration에 설정을 추가
1. Edit Configuration - add Build Steps로 먼저 Maven을 등록
![image](https://user-images.githubusercontent.com/57924258/104836885-4ce0d680-58f4-11eb-90de-9edc1e46a552.png)

2. Container deploy 등록
- Target : Tomcat이 설치된 IP와 PORT번호 입력 (ex)localhost:8080
- UserName / Password : 위에서 Tomcat에 등록된 name/password를 등록 
- Path to Archive : war file이 있는 경로
![image](https://user-images.githubusercontent.com/57924258/104834146-ebfbd300-58e0-11eb-8b9b-7949f2f55a45.png)

3. 다음과 같이 Build Step들이 생성 됨
![image](https://user-images.githubusercontent.com/57924258/104836858-2c188100-58f4-11eb-9ccf-6b290594edfd.png)

4. 해당 프로젝트에 Run을 클릭
![image](https://user-images.githubusercontent.com/57924258/104836948-b5c84e80-58f4-11eb-9078-2034ed7cec70.png)

#### 5.2.4 Centos Service에 TeamCity 등록
```
TODO
```

#### 5.2.5 Centos Service에 Spring Project 등록
```
TODO : yml 파일로 바꾸기 (시간이 없어서 일단 properties 파일로 함) --> 보안에 유리. 
application.xml은 외부에 노출되기 때문에 안 쓰는게 좋음.
war에 yml 파일이 포함되는지 한번 확인해보기.
```
- vi /etc/systemd/system/demo.service 
```
[Unit]
Description=Demo Java Service

[Service]
WorkingDirectory=/opt/app
ExecStart=/usr/lib/jvm/java-11-openjdk-11.0.9.11-2.el7_9.x86_64/bin/java -Dspring.config.location=/opt/TeamCityAgent/work/567965ce87d56df1/target/classes/application.properties  -jar /opt/app/demo-0.0.1-SNAPSHOT.war --server.port=8083
User=root
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```
- 나머지는 위키를 참조
- 이후에는 다음과 같이 start 한 뒤 status로 상태를 확인
```
# systemctl start demo.service
# systemctl status demo.service
```

### 5.3 배포 방법2 (java 명령어를 통한 배포 - 패키지에 tomcat이 이미 설치되어 있는 경우)
#### 5.3.1 system 파일 생성
- vi /etc/systemd/system/[프로젝트 명]
- 위 내용들을 참고하여 작성하면 됨 (아래는 예시)
```
[Unit]
Description=[프로젝트 명]
After=syslog.target

[Service]
User=[Linux 사용자 admin 명]
Group=[Linux 그룹 admin 명]
WorkingDirectory=[프로젝트 경로]
ExecStart=java -Djava.net.preferIPv4Stack=true -Dspring.config.location=application.yml -Dsun.misc.URLClassPath.disableJarChecking=true -jar [프로젝트 경로 및 명칭].jar
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

#### 5.3.2 yml 파일 작성
- /opt/[프로젝트명]/conf/[yml 파일명].yml 
```
spring:
  application:
    name: 프로젝트명

  ###################
  ## Auto Configuration
  ###################

  autoconfigure:
    exclude:
      - org.springframework.boot.actuate.autoconfigure.solr.SolrHealthContributorAutoConfiguration

  ###################
  ## JDBC
  ###################

  datasource:
    hikari:
      jdbc-url: jdbc:log4jdbc:postgresql://[ip]:[port]/[프로젝트명]
      username: 사용자 명
      password: 사용자 패스워드
      driver-class-name: net.sf.log4jdbc.sql.jdbcapi.DriverSpy
      connection-test-query: SELECT 1
      maximum-pool-size: 10
      minimum-idle: 3
    sql-script-encoding: UTF-8
    continue-on-error: true
    initialization-mode: always

  ###################
  ## Hibernate
  ###################

  jpa:
    generate-ddl: false
    database: postgresql
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        jdbc:
          lob:
            non_contextual_creation: true
    open-in-view: false

  ###################
  ## MVC
  ###################

  mvc:
    view:
      prefix: /WEB-INF/jsp/
      suffix: .jsp
    locale: ko_KR

  ###################
  ## Data REST
  ###################

  data:
    rest:
      base-path: /api

  ###################
  ## Data Redis
  ###################

  redis:
    host: [ip]
    port: [port]

###################
## Web Container
###################

server:
  address: 0.0.0.0
  port: [port]
  compression:
    enabled: true
    mime-types: application/json,application/xml,text/html,text/xml,text/plain,application/javascript
  http2:
    enabled: true
  error:
    include-exception: true
    include-stacktrace: always
    whitelabel:
      enabled: false
  tomcat:
    threads:
      max: 120
    max-http-form-post-size: -1
  servlet:
    session:
      timeout: 1440m

###################
## Application
###################

app:
  dev-mode: true
  cors-allow-origin-hosts: http://[ip]:[port], http://[ip]:[port]
  ranger:
    url: http://[ip]:[port]
    username: [id]
    password: [password]
    impala-service-name: [service name]
  ldap:
    prefix:
      base: [DNS 명칭]
      admin: [Admin 정보]
    ldap-server-url: ldap://[ip]:[port]
    ldap-server-admin-password: [password]

###################
## Eureka
###################

eureka:
  server:
    eviction-interval-timer-in-ms: 1000
    registry-sync-retry-wait-ms: 0
  instance:
    appname: [appName]
  client:
    service-url:
      defaultZone: http://[ip]:[port]/[zone 명칭]
    register-with-eureka: false
    fetch-registry: false

###################
## Logging
###################

logging:
  config: [log 경로 및 log명].xml

###################
## Management
###################

management:
  health:
    mail:
      enabled: false
  endpoint:
    env:
      enabled: true
    beans:
      enabled: true
    threaddump:
      enabled: true
    heapdump:
      enabled: true
    info:
      enabled: true
    metrics:
      enabled: true
```


#### 5.3.3 TeamCity에서 Maven Build -> jar file copy -> systemctl restart 를 순서로 진행
1. Maven build 시 Goal을 clean package로 설정
2. Command Line으로 jar file을 복사할 때는 다음과 같이 설정
- 예시 
```
ls -lsa [이전 디렉토리]
cp [이전 파일 경로 및 파일명]  [복사 파일 경로 및 파일명]
chown admin:admin [복사 파일 경로 및 파일명]
ls -lsa [복사 디렉토리]
```
3. Command Line으로 systemctl 명령어로 해당 파일 restart
```
systemctl restart [프로젝트 명]
```
