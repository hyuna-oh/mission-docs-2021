# mission2021
2021년에 공부한 내용들을 문서로 저장하는 곳.
# 1. vagrant (2021.01.04 ~ 2021.01.10)
- 베이그런트는 가상화 소프트웨어 (버추얼박스, 도커 컨테이너, AWS 등)의 생성 및 유지보수를 위한 오픈소스 소프트웨어 제품
- 루비 언어로 작성됨

## 작업해볼 LIST
### Box로 VM 생성하기 (CentOS, Ubuntu 등)
 1. Vagrantfile 을 로컬의 user 디렉토리 밑에 생성
 - 방법1) vagrant init bento/centos-7.4 명령어로 Vagrantfile을 생성 후 생성된 파일에 다음의 내용 작성
===============================================================================================================
 # vagrant init bento/centos-7.4
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.

# cat Vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "bento/centos-7.4"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  config.vm.network "forwarded_port", guest: 8080, host: 80

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder ".", "/data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
     vb.cpus = "4"
     vb.memory = "4096"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
===============================================================================================================
 - 방법2) vagrant가 설치된 디렉토리 상위 폴더에 Vagrantfile을 직접 생성하여 작성 (디렉토리 한번 확인해보기***)
### VM 삭제
### VM의 설정 변경
#### CORE
#### MEMORY
#### DISK (중요)
### 포트 포워딩

### SSH 접속
SSH 접속 Command : vagrant ssh 

### GITLAB 설치
### VM과 로컬 PC의 디렉토리 공유
### 네트워크 설정 변경
#### Private IP 설정
#### DHCP 설정
### Plugin 설치 및 사용
### 기존 Box로 자체 신규 Box 만들기
### 신규 Box 파일을 웹 서버에 올려놓고 자체 Box 서버를 통해 Box 파일 내려받기
