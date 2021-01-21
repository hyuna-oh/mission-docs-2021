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
- multi machine 참고 [너무 느려져서 실패함ㅠㅠ 동작은 잘 됨. 다만 share folder는 지정못하는듯 함.]
```
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

### Ubuntu에 Ubuntu 16.04 image 다운받기
```
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable"
```

```
sudo apt update
apt-cache search docker-ce
```
- 결과
```
docker-ce - Docker: the open-source application container engine
```
- docker-ce 설치
```
sudo apt install docker-ce
```
- docker image 다운로드
```
docker pull ubuntu 
docker run -i -t ubuntu       
```
- docker 목록 확인
```
docker images
```

### Centos 7 에서 Ubuntu 16.04 image 다운받기
- Centos7에 docker install 문서 : https://docs.docker.com/engine/install/centos/

#### yum으로 이전 docker 삭제
```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

#### yum으로 설치
```
# yum 패키지 업데이트
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum -y update

# docker registry 설정
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# docker 목록 조회 및 설치
sudo yum -y list docker-ce --showduplicates | sort -r
sudo yum -y install docker docker-registry
sudo yum -y install docker-ce docker-ce-cli containerd.io
````
- 여기서 도커를 특정 버전으로 다운받고 싶다면, ```$ sudo yum -y install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io``` 라는 명령어를 수행하면 됨.

#### docker 실행
```
# 부팅 시 실행하도록 등록
sudo systemctl enable docker.service

# 도커 실행
sudo systemctl start docker.service

# 도커 status 확인
sudo systemctl status docker.service

# 도커 설치 확인
sudo docker run hello-world
```

#### 도커 이미지 pull 
```
# 도커 컨테이너 다운로드
sudo docker pull ubuntu
```
- 다음과 같은 에러메시지 발생
```
Error response from daemon: Get [해당 url]: 45 net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```


## 2. 1월 둘째주에 만든 Spring Boot 프로젝트로 이미지 구성하고 컨테이너로 실행하기
- !TODO yml파일 먼저 생성한 뒤 수행
