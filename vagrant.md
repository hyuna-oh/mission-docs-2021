# vagrant (2021.01.04 ~ 2021.01.10)
* 베이그런트는 가상화 소프트웨어 (버추얼박스, 도커 컨테이너, AWS 등)의 생성 및 유지보수를 위한 오픈소스 소프트웨어 제품
* 루비 언어로 작성됨
* Provision : 게스트 머신을 반복적으로 생성하고 사용하기 위해 프로비져닝으로 만들어서 사용
* Vagrant.configure("2") 의 의미
  + 여기서, configure("2")는 버전을 의미함.
  + 최근에는 "1", "2" 두 가지 버전만을 지원하며, 버전 1의 경우 Vagrant 1.0.x 버전을, 버전 2의 경우엔 1.1+ ~ 2.0.x 버전을 나타낸다.
```
Vagrant.configure("2") do |config|
  # ...
end
```
 버전별로 지원하는 configuration이 다름. 그렇기 때문에, 부득이하게 버전 1과 2의 configuration을 사용해야 한다면 다음과 같이 사용해야 함.
```
Vagrant.configure("1") do |config|
  # v1 configs...
end

Vagrant.configure("2") do |config|
  # v2 configs...
end
```
Vagrant.configure("2")에 관한 자세한 정보 : https://www.vagrantup.com/docs/vagrantfile/version
***
# 작업 LIST
## 1. Box로 VM 생성하기 (CentOS, Ubuntu 등)
* 방법1) vagrant init bento/centos-7.4 명령어로 Vagrantfile을 생성 후 생성된 파일에 다음의 내용 작성 (Vagrantfile 위치는 처음 Vagrant를 설치한 위치)
```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.2"
  config.vm.box_version = "2.3.1"
end
```
(여기서 init 파일은 VM을 구동하기 위한 메타데이터 정보를 가지고 있음)
* 방법2) vagrant가 설치된 디렉토리 상위 폴더에 Vagrantfile을 직접 생성하여 작성
## 2. VM 삭제
```diff
! TODO
```
* 삭제 명령어 : ```vagrant box remove NAME``` 
## 3. VM의 설정 변경
### 3.1. CORE & MEMORY 
* 방법1)
1. 일단 box가 실행중이라면, ```vagrant halt``` 명령어를 통해 서버를 내린다.
2. 그리고 기존 Vagrantfile 에 다음의 내용을 작성한다. 
```
Vagrant.configure("2") do |config|

  config.vm.box = "bento/centos-7.2"
  config.vm.provider "virtualbox" do |vb|
     vb.cpus = "4"
     vb.memory = "4096"
  end

end
```
3. ```vagrant up``` 명령어로 서버를 다시 올린다.

**TIP** : 다음의 설정으로 gui 및 name등을 세팅할 수 있다. 
```diff
! TODO
```
```
config.vm.provider "virtualbox" do |v|
  v.gui = true 
  # 부팅되면서 gui 환경으로 부팅
  v.name = "my_vm"
  # VirtualBox GUI에서의 name을 세팅할 수 있음
end 
```
* 방법2) ```vagrant reload``` 명령어를 통해서도 Vagrantfile 수정 후 서버를 다시 올릴 수 있다.
(주의사항 : Provision의 경우엔 --provision flag 를 사용해 줘야 reload가 됨)

### 3.2. DISK (중요)
```diff
! TODO
```

* disk types
disk (symbol)
dvd (symbol)
floppy (symbol)

```
config.vm.disk :disk, name: "backup", size: "10GB"
config.vm.disk :dvd, name: "installer", path: "./installer.iso"
config.vm.disk :floppy, name: "cool_files"
```
* 참고 : https://www.vagrantup.com/docs/disks/configuration

## 4. 포트 포워딩
- 포트를 포워딩하여 Host(local PC) port와 Guest(VM) Port를 포워딩할 수 있다.
```
Vagrant.configure("2") do |config|
  config.vm.network "forwarded_port", guest: 80, host: 8080
end
```
- window에서 다음의 command ```netstat -a -b```를 실행했을 경우
```
  TCP    0.0.0.0:80             DESKTOP-G85CPR1:0      LISTENING
```  
**TODO** 포트포워딩을 사용하는 이유는?
* 참고 : https://www.vagrantup.com/docs/networking/forwarded_ports
## 5. SSH 접속
 * SSH 접속 Command : ```vagrant ssh``` 

## 6. GITLAB 설치
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
- 로그 메시지
```
Running handlers:
Running handlers complete
Chef Infra Client finished, 567/1530 resources updated in 03 minutes 44 seconds
gitlab Reconfigured!
```
* gitlab-ctl reconfigure ? 설정이 초기에 제공된 것과 동일한 형태로 설정이 되어있는지 확인 후 초기에 제공된 것과 동일하게 설정함.
* 여기서 el7 의 의미는? Red Hat 7.x, CentOS 7.x, and CloudLinux 7.x  
**TODO** 그리고 Gitlab runner란?  
**NOTE** rpm 파일이 있다면 'rpm -Uvh' 대신 'yum localinstall' 을 실행해 패키지를 설치할 수 있다. 좋은 점은 현재 디렉터리의 rpm 파일에 의존성 문제가 있을 때, 문제를 해결할 수 있는 파일을 인터넷에서 다운로드해서 설치해준다는 점이다. 'rpm -Uvh rpm파일이름.rpm' 대신에 사용하면 된다.
(출처: https://linuxstory1.tistory.com/entry/편리하게-패키지를-설치하는-YUM [Linux 세상속으로])

## 7. VM과 로컬 PC의 디렉토리 공유
```
Vagrant.configure("2") do |config|
  # other config here

  config.vm.synced_folder "data-vm/", "/data"
  # host 파일경로, guest 파일경로
end
```
![image](https://user-images.githubusercontent.com/57924258/103769489-b85eb480-5067-11eb-96eb-0af7477478cc.png)

* 여기서 host는 local 디렉토리, guest는 VM 디렉토리를 의미함.
* 참고 : https://www.vagrantup.com/docs/synced-folders/basic_usage
## 8. 네트워크 설정 변경
### 8.1. Private IP 설정
* static한 ip로 구성할 경우
```
Vagrant.configure("2") do |config|
  config.vm.network "private_network", ip: "192.168.xx.yy"
end
```
![image](https://user-images.githubusercontent.com/57924258/103778324-6f156180-5075-11eb-8c88-49555bef93fd.png)
* filezila로 접속 시
  + 호스트 : sftp://192.168.xx.yy
  + 사용자명/비밀번호 입력
* 참고 : https://www.vagrantup.com/docs/networking/private_network
### 8.2. DHCP 설정
```diff
! TODO
```
* private ip를 DHCP 방식으로 구성할 경우
```
Vagrant.configure("2") do |config|
  config.vm.network "private_network", type: "dhcp"
end
```
* public IP를 DHCP 방식으로 구현할 경우
```
Vagrant.configure("2") do |config|
  config.vm.network "public_network",
    use_dhcp_assigned_default_route: true
end
```
## 9. Plugin 설치 및 사용
- Plugin 설치
```
C:\...\hyunaoh>vagrant plugin install vagrant-global-status
Installing the 'vagrant-global-status' plugin. This can take a few minutes...
Fetching vagrant-vbguest-0.29.0.gem
Fetching vagrant-global-status-0.1.4.gem
Successfully uninstalled vagrant-vbguest-0.29.0
Installed the plugin 'vagrant-global-status (0.1.4)'!
```
- Plugin 사용 
```
C:\...\hyunaoh>vagrant global-status --timestamp

C:/.../hyunaoh
  default      running      (virtualbox)   2021-01-07 09:46:01 +0900

```
- Plugin 삭제
```
# Removing a plugin from a known gem source
$ vagrant plugin uninstall my-plugin
```
- Plugin 업데이트
```
C:\...\hyunaoh>vagrant plugin update vagrant-global-status
Updating plugins: vagrant-global-status. This may take a few minutes...
Fetching vagrant-vbguest-0.29.0.gem
Successfully uninstalled vagrant-vbguest-0.28.0
Updated 'vagrant-vbguest' to version '0.29.0'!
```
- Plugin 리스트
```
C:\...\hyunaoh>vagrant plugin list
vagrant-disksize (0.1.3, global)
vagrant-global-status (0.1.4, global)
vagrant-vbguest (0.28.0, global)
```

**Note** : 향후에는 ```vagrant plugin``` 명령어가 각 플러그인을 설치할 때 서브명령어에 포함될 예정이다.

## 10. 기존 Box로 자체 신규 Box 만들기
```diff
! TODO 이게 맞는지 잘 모르겠는데.. 다른 방법도 찾아보자
```
* Linked clones는 master VM을 기반으로 box를 처음 한번 import할 때 필요하다. linked clones는 master VM에 속한 상위 디스크 이미지와 다를 때 생성한다.
Linked clones are based on a master VM, which is generated by importing the base box only once the first time it is required. For the linked clones only differencing disk images are created where the parent disk image belongs to the master VM.

```
config.vm.provider "virtualbox" do |v|
  v.linked_clone = true
end
```

## 11. 신규 Box 파일을 웹 서버에 올려놓고 자체 Box 서버를 통해 Box 파일 내려받기
```diff
! TODO
```

*참고 : 코드 색상표
```diff
- text in red
+ text in green
! text in orange
# text in gray
```
