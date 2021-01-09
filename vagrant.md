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
* 버전별로 지원하는 configuration이 다름. 그렇기 때문에, 부득이하게 버전 1과 2의 configuration을 사용해야 한다면 다음과 같이 사용해야 함.
```
Vagrant.configure("1") do |config|
  # v1 configs...
end

Vagrant.configure("2") do |config|
  # v2 configs...
end
```
* Vagrant.configure("2")에 관한 자세한 정보 : https://www.vagrantup.com/docs/vagrantfile/version  
* What is Box? package 포맷의 Vagrant 환경설정을 말한다. 이 박스는 누구든지 어떤 플랫폼에서든 동일한 환경을 생성할 수 있게 만들어준다. 그래서, 주로 해당 VM의 BOX를 package 포맷으로 생성한 뒤 이를 ```vagrant init 박스명``` 과 같이 사용한다.
# 작업 LIST
## 1. Box로 VM 생성하기 (CentOS, Ubuntu 등)
- ```vagrant init bento/centos-7.4``` 명령어로 VM 생성하면, bento/centos-7.4 box가 자동으로 생성됨. (```vagrant box list```를 통해 확인이 가능함)
- 또한 다음의 Vagrantfile이 생성됨.
```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.4"
end
```
(여기서 Vagrantfile 파일은 VM을 구동하기 위한 메타데이터 정보를 가지고 있음)

## 2. VM 삭제
* VM 삭제 명령어 : ```vagrant destroy VM명/ID``` 명령어를 통해 삭제할 수 있다.  
**NOTE** The ```vagrant destroy``` 명령어는 ```vagrant up``` 명령어가 진행되는 동안 설치된 박스를 제거하지 않는다. 그러므로 만약 삭제하려면 vagrant up 명령어를 수행하기 전에 box를 삭제해야 한다.
**Note** vagrant를 통해 생성된 VM만 삭제가 가능한것으로 보인다.
* box 삭제 명령어 : ```vagrant box remove NAME``` 

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
* 방법1) 메인 디스크 크기를 변경하려면 다음의 config를 Vagrantfile에 넣어주면 된다.  
**Note** : ```the primary: true``` 라는 옵션은 vm의 메인 드라이브 크기를 확장하여준다. 만약 이 옵션이 없다면, Vagrant는 새로운 디스크를 vm 드라이브에 첨부하여 줄 것이다.  
**Note** : 여기서, h.vm.box에 이름은 vm 명칭을 뜻하는데, 이름을 다르게 하여 만들면 새로운 BOX가 생성된다. 때문에 default로 하려면,  
```config.vm.disk :disk, size: "50GB", primary: true```만 작성해야한다.  
**Note** : 디스크 크기를 줄일 수는 없다.  
**Note** : iso 형태의 디스크를 추가할 수 있다.  
- 다음의 코드를 Vagrantfile에 추가한다.
- 이 기능은 현재 실험용 플래그를 사용해야한다. 실험용 플래그는 다음과 같이 설정할 수 있다.
```
ENV['VAGRANT_EXPERIMENTAL'] = 'disks'
Vagrant.configure("2") do |config|
... 생략 ...
```
- 추가 한 뒤 다음의 코드도 추가한다.
```
Vagrant.configure("2") do |config|
  config.vm.define "hashicorp" do |h|
    # box명을 입력해야 함. (이 때, 다르게 입력하면 새로운 디스크를 가진 서버가 생성됨.)
    h.vm.box = "hashicorp/bionic64"
    # provider는 필수값은 아님.
    h.vm.provider "virtualbox"
    h.vm.disk :disk, size: "100GB", primary: true
  end
end
```
- 만약, 메인 디스크 크기를 변경하는 것이 아닌 디스크를 새로 생성하는 것이라면, 다음의 설정을 넣으면 된다.
```
Vagrant.configure("2") do |config|
  config.vm.define "hashicorp" do |h|
    h.vm.box = "hashicorp/bionic64"
    # h.vm.provider :virtualbox
    h.vm.provider "virtualbox"
    h.vm.disk :disk, size: "10GB", name: "extra_storage"
  end
end
```
* 참고 : https://www.vagrantup.com/docs/disks/configuration

* 방법2) plugin을 사용한다.
1. 다음의 명령어로 플러그인을 설치한다.
```
# vagrant plugin install vagrant-disksize
Installing the 'vagrant-disksize' plugin. This can take a few minutes...
Fetching vagrant-disksize-0.1.3.gem
Installed the plugin 'vagrant-disksize (0.1.3)'!
```
2. Vagrantfile에 다음과 같은 내용을 입력한다.
```
Vagrant.configure('2') do |config|
  config.vm.box = 'ubuntu/xenial64'
  config.disksize.size = '50GB'
end
```
## 4. 포트 포워딩
- 포트를 포워딩하여 Host(local PC) port와 Guest(VM) Port를 포워딩할 수 있다. 설정 후엔 `vagrant reload`를 한다.
```
Vagrant.configure("2") do |config|
  config.vm.network "forwarded_port", guest: 8080, host: 80
end
```
- localhost에서 웹 브라우저에 다음의 url을 입력하면 접속이 가능하다.
`http://127.0.0.1:80`
- ```vagrant up```으로 실행하면 다음과 같이 터널링하여 사용할 수 있다.
```
vagrant ssh -- -L 80:localhost:8080
```
* 포트포워딩을 사용하는 이유는? https://lamanus.kr/59 => 쉽게 말해 특정 포트를 통해 공유기를 통해 원하는 서버에 접속할 때 필요함.
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
* filezila로 접속 시
  + 호스트 : sftp://192.168.xx.yy
  + 사용자명/비밀번호 입력
* 참고 : https://www.vagrantup.com/docs/networking/private_network
### 8.2. DHCP 설정
* private ip를 DHCP 방식으로 구성할 경우
```
Vagrant.configure("2") do |config|
  config.vm.network "private_network", type: "dhcp"
end
```
* public IP를 DHCP 방식으로 구현할 경우
```
Vagrant.configure("2") do |config|
  config.vm.network "public_network"
end
```
* ifconfig 명령어로 조회한 결과 (private network를 사용했을 때)
```
enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.28.xx.yy  netmask 255.255.255.0  broadcast 172.28.128.255
        inet6 fe80::a00:27ff:fee4:d7b2  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:e4:d7:b2  txqueuelen 1000  (Ethernet)
        RX packets 132  bytes 13102 (12.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 65  bytes 10198 (9.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
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
1. vagrant 로 기존의 BOX를 pacakge화 하여 저장한다.
```
vagrant package --output mynew.box
```
2. 신규 box를 생성한다.
```
vagrant box add mynewbox mynew.box
```
3. 신규 box를 사용하기 위해서 기존의 박스를 제거하고 init 명령어로 신규 box를 설정한다.
```
vagrant destroy
rm Vagrantfile
vagrant init mynewbox
```
- add box 명령어 참고 : https://www.vagrantup.com/docs/cli/box#box-add
- add box 전에 box를 경량화 하고자 한다면 참고 : https://scotch.io/tutorials/how-to-create-a-vagrant-base-box-from-an-existing-one
## 11. 신규 Box 파일을 웹 서버에 올려놓고 자체 Box 서버를 통해 Box 파일 내려받기
1. Host(localhost)에 웹 서버를 설치
- Windows 10 에서 웹 서버 구축하기
- http://apachelounge.com/download/에 들어가서 64비트로 설치
- C:/Server/Apache24/conf/httpd.conf 를 수정한다.
```
ServerRoot "c:/Server/Apache24"
Listen 80
ServerAdmin 000@gmail.com
ServerName 127.0.0.1:80
DocumentRoot "c:/Server/Apache24/htdocs"
ErrorLog "logs/error.log" 
```
- 환경변수를 설정한다. (시스템변수 - path - C:\Apache24\bin)
- 권리자 권한으로 cmd창을 열고 ```httpd.exe -k install``` 명령어를 입력한다.
- 그리고 ```httpd.exe -k start``` 명령어를 입력한다.
2. C:/Server/Apache24/conf/htdocs/box 디렉토리에 해당 box를 넣는다.
3. cmd에서 ```vagrant box add mynew http://[Host IP 입력]:[port Number 입력]/box/mynew.box``` 명령어로 박스를 다운받는다.  
**NOTE** : 만약 centos에 있는 웹 서버에 올려놓은 BOX를 다운받는다면, nginx 서버를 설치한 뒤 /etc/share/nginx 폴더에 해당 파일을 올린 뒤 3번의 내용을 진행하면 된다.

*참고 : 코드 색상표
```diff
- text in red
+ text in green
! text in orange
# text in gray
```
* vagrant 관련 설명이 잘 되어 있는 article : https://tech.osteel.me/posts/how-to-use-vagrant-for-local-web-development#port-forwarding
