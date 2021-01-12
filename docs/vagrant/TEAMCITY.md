# TeamCity 
- 2021.01.11 ~ 2021.01.17 동안 작업한 내용
- TeamCity는 JetBrains의 빌드 관리 및 지속적 통합 서버를 위한 
- 깃랩은 깃랩 사가 개발한 깃 저장소 및 CI/CD, 이슈 추적, 보안성 테스트 등의 기능을 갖춘 웹 기반의 데브옵스 플랫폼

## Vagrant + CentOS 7에서 Gitlab 설치
1. ```vagrant init bento/centos-7``` 으로 새로운 CentOS 7 프로젝트 생성 (https://app.vagrantup.com/boxes/search 참고)
2. 생성 후 Gitlab 설치
- install_gitlab.sh 라는 쉘스크립트를 Vagrantfile이 있는 디렉토리에 작성 후 저장한 뒤 ``reload --provision`` 명령어로 실행시킨다.
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
- java를 먼저 설치한 후 진행 (Java 8)
```
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
sudo /usr/sbin/alternatives --config java
```
- ```vagrant ssh```로 쉘에 접속하여 다음의 코맨드들을 입력
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
- 설치가 완료 되면 postgres 구동  
(만약 ```su - postgres -c 'psql'``` 명령어를 입력했을 때 Authentication failure 에러가 난다면, ```sudo -u postgres psql -c "ALTER USER postgres PASSWORD 'postgres';"``` 명령어로 변경해 줌)
```
# postgresql 구동
systemctl start postgresql
# postgres 계정 비밀번호 수정
su - postgres -c 'psql'
# 데이터베이스 생성
postgres=# alter user postgres password 'postgres';
postgres=# create database teamcity;
```
- postgres 설치 완료 시 다음의 절차대로 설치
```
# tar xvfz TeamCity-2020.2.1.tar.gz
# cd TeamCity/bin
# sh catalina.sh
```
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
```
!TODO
```
- SSH KEY 추가해보기!
## https://start.spring.io/ 에서 REST Web 프로젝트를 구성해서 Gitlab 프로젝트에 소스코드 추가
## TeamCity에서 프로젝트 추가하고 빌드 설정
