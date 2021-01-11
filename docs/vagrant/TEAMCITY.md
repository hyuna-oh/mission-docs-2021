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
* TeamCity Server
* TeamCity Agent

## Gitlab에 프로젝트 생성
## https://start.spring.io/ 에서 REST Web 프로젝트를 구성해서 Gitlab 프로젝트에 소스코드 추가
## TeamCity에서 프로젝트 추가하고 빌드 설정
