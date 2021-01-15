# TeamCity & Gitlab
- 2021.01.11 ~ 2021.01.17 동안 작업한 내용
- TeamCity는 JetBrains의 빌드 관리 및 지속적 통합 서버를 위한 빌드 관리 
- Gitlab은 깃랩 사가 개발한 깃 저장소 및 CI/CD, 이슈 추적, 보안성 테스트 등의 기능을 갖춘 웹 기반의 데브옵스 플랫폼

## Vagrant + CentOS 7에서 Gitlab 설치
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

## Vagrant + CentOS 7에서 TeamCity Server 및 TeamCity Agent 설치
### TeamCity Server 설치
#### **- 웬만하면 sudo나 root 계정으로 접속하기**
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
- ```sudo sh bin/catalina.sh run``` 명령어로 실행  
(만약 TeamCity 실행 실패시 해당 log 참조)
### TeamCity Server 설정
1. TeamCity의 기본 포트는 8111 이므로, 로컬 브라우저에서 [VM IP]:8111로 접속
2. database type : postgresql / database host : localhost / name : teamcity / username : postgres / passwd : postgres 로 설정
3. admin 계정 생성

### TeamCity Agent 설치
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
## Gitlab에 프로젝트 생성
- HOST PC에서 브라우저로 접속 후 ```localhost:80```으로 접속한 뒤 (nginx 서버가 gitlab에 설치되어 있음) UI에서 프로젝트를 생성
```
!TODO - SSH KEY 추가해보기!
```

```
!TODO https://stackoverflow.com/questions/31813080/windows-10-ssh-keys : 참고
```
## https://start.spring.io/ 에서 REST Web 프로젝트를 구성해서 Gitlab 프로젝트에 소스코드 추가
```
!TODO 인텔리제이 설치하기 
```
- maven repository를 이용
- ```https://spring.io/guides/tutorials/rest/``` 참고
- 트리구조
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

.idea
*.iws
*.iml
*.ipr

/dist
build/
```
 
## TeamCity에서 프로젝트 추가하고 빌드 설정
- ```!TODO``` 프로젝트 2개이상 추가해보기
