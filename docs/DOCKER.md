# DOCKER
- 21.01.18 ~ 21.01.24 작업한 내용
- 컨테이너 기반의 오픈소스 가상화 플랫폼
- 여기서 컨테이너란? 다양한 OS에 여러 application이 올려져 있는 것을 의미
- 각 컨테이너는 격리된 공간이기 때문에 하나의 컨테이너에 문제가 생겨도 다른 컨테이너에 영향을 끼치지 않음

## 1. Docker 시작하기 전체 실습하기
- Windows 10 Pro에 설치 가능하지만 현재 Virtual Box가 설치되어 있어서 충돌이 우려됨.  
-> 그러므로 VB Linux에서 Docker를 설치해야함..
-> Vagrantfile을 통해 multi VM을 생성하자.

### 1.1 VM을 package 로 export 해둔 다음 multi VM을 통해 생성  
공식사이트 (multi-machine) : https://www.vagrantup.com/docs/multi-machine  
참고 (multi-machine) : https://askme.tistory.com/335
- multi machine 참고
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

boxes = [
  {
    os: "",
    name: "server1.local",
    eth1: "192.168.0.101",
    mem: 1024,
    cpu: 1
  },
  {
    os: "",
    name: "server2.local",
    eth1: "192.168.0.102",
    mem: 1024,
    cpu: 1
  }
]

Vagrant.configure(2) do |config|
# config.vm.box = "nrel/CentOS-6.5-x86_64"

  boxes.each do |box|
    config.vm.define box[:name] do |srv|
      srv.vm.box = box[:os]
      srv.vm.hostname = box[:name]
      srv.vm.network :private_network, ip: box[:eth1]

      srv.vm.provider :virtualbox do |vb|
        vb.memory = box[:mem]
        vb.cpus = box[:cpu]
      end
    end
  end
end
```

```diff
! TODO
혹시 모르니 Vagrantfile 을 git에 configuration 폴더에 넣어두기 (provision 관련 파일도 올려두기)  
Vagrantfile을 package 한 명령어 적어 놓기
```
- package 명령어 참고

1. Install virtual box and vagrant on the remote machine  
2. Wrap up your vagrant machine
```
vagrant package --base [machine name as it shows in virtual box] --output /Users/myuser/Documents/Workspace/my.box
```
3. copy the box to your remote
4. init the box on your remote machine by running
```vagrant init [machine name as it shows in virtual box] /Users/myuser/Documents/Workspace/my.box```
5. Run ```vagrant up```

- wiki 참고
- 참고사항
- 이슈

## 2. 1월 둘째주에 만든 Spring Boot 프로젝트로 이미지 구성하고 컨테이너로 실행하기
- !TODO yml파일 먼저 생성한 뒤 수행
