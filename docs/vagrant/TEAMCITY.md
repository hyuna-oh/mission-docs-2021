# TeamCity 
- 2021.01.11 ~ 2021.01.17 동안 작업한 내용

## Vagrant + CentOS 7에서 Gitlab 설치
1. ```vagrant init bento/centos-7``` 으로 새로운 CentOS 7 프로젝트 생성 (https://app.vagrantup.com/boxes/search 참고)
2. 생성 후 Gitlab 설치
- install_gitlab.sh 라는 쉘스크립트를 Vagrantfile이 있는 디렉토리에 작성 후 저장한 뒤 ``reload --provision`` 명령어로 실행시킨다.
- install_gitlab.sh
```
echo ==== Installing GitLab CE =================================================
wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/7/gitlab-ce-13.4.7-ce.0.el7.x86_64.rpm/download.rpm
yum install -y epel-release
yum install -y curl policycoreutils-python openssh-server postfix
systemctl enable sshd
systemctl enable postfix
systemctl start sshd
systemctl start postfix
sudo yum localinstall gitlab-ce-13.4.7-ce.0.el7.x86_64.rpm
sudo gitlab-ctl reconfigure
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
- TeamCity의 기본 포트는 8111 이므로, local 브라우저에서 [VM IP]:8111로 접속
- 

### TeamCity Agent 설치

## Gitlab에 프로젝트 생성
## https://start.spring.io/ 에서 REST Web 프로젝트를 구성해서 Gitlab 프로젝트에 소스코드 추가
## TeamCity에서 프로젝트 추가하고 빌드 설정
