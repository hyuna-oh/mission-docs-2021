# DOCKER
- 21.01.18 ~ 21.01.24 작업한 내용
- 컨테이너 기반의 오픈소스 가상화 플랫폼
- 여기서 컨테이너란? 다양한 OS에 여러 application이 올려져 있는 것을 의미
- 각 컨테이너는 격리된 공간이기 때문에 하나의 컨테이너에 문제가 생겨도 다른 컨테이너에 영향을 끼치지 않음
​
## 1. Docker 시작하기 전체 실습하기
- Windows 10 Pro에 설치 가능하지만 현재 Virtual Box가 설치되어 있어서 충돌이 우려됨.  
-> 그러므로 VB Linux에서 Docker를 설치해야함..
-> Vagrantfile을 통해 multi VM을 생성하자.
​
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
​
Vagrant.configure(2) do |config|
# config.vm.box = "nrel/CentOS-6.5-x86_64"
​
 boxes.each do |box|
   config.vm.define box[:name] do |srv|
     srv.vm.box = box[:os]
     srv.vm.hostname = box[:name]
     srv.vm.network :private_network, ip: box[:eth1]
​
     srv.vm.provider :virtualbox do |vb|
       vb.memory = box[:mem]
       vb.cpus = box[:cpu]
     end
   end
 end
end
```
​
### Ubuntu에 Ubuntu 16.04 image 다운받기
```
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable"
sudo apt update
# docker-ce 설치
sudo apt install docker-ce
# docker image 다운로드
docker pull ubuntu
docker run -i -t ubuntu      
# docker 목록 확인
docker images
```
​
### Centos 7 에서 Ubuntu 16.04 image 다운받기
- Centos 7 docker install 문서 : https://docs.docker.com/engine/install/centos/
​
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
​
#### yum으로 설치
```
# yum 패키지 업데이트
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum -y update
​
# docker registry 설정
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
​
# docker 목록 조회 및 설치
sudo yum -y list docker-ce --showduplicates | sort -r
sudo yum -y install docker docker-registry
sudo yum -y install docker-ce docker-ce-cli containerd.io
````
- 여기서 도커를 특정 버전으로 다운받고 싶다면, ```$ sudo yum -y install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io``` 라는 명령어를 수행하면 됨.
​
#### docker 실행
```
# 부팅 시 실행하도록 등록
sudo systemctl enable docker.service
​
# 도커 실행
sudo systemctl start docker.service
​
# 도커 status 확인
sudo systemctl status docker.service
​
# 도커 설치 확인
sudo docker run hello-world
```
​
#### 도커 이미지 pull
```
# 도커 컨테이너 다운로드
sudo docker pull ubuntu
```
- 다음과 같은 에러메시지 발생
```
Error response from daemon: Get [해당 url]: 45 net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```
- 해결책 1) https://docs.docker.com/config/daemon/systemd/#httphttps-proxy 을 참고하여 프록시 설정
1. 디렉토리 생성  
```
sudo mkdir -p /etc/systemd/system/docker.service.d
```
​
2. 파일 생성
- Create a file named /etc/systemd/system/docker.service.d/http-proxy.conf that adds the HTTP_PROXY environment variable:
```
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80"
```  
​
### pull 이나 search 할 때 에러 날 경우
-  그냥 proxy 파일에서 [Service] 구문 부분을 제외하고 모든 항목을 지운다. 그러면 됨...
### docker container의 CPU 및 MEMORY 제한
-  주의 : 반드시 stop 및 rm으로 현재 container가 없는 상태에서 진행해야 함.
### build시 contextPath를 지정할 경우
- contextPath란, Docker 이미지 생성시(docker build  커맨드) Dockerfile을 로딩하는 현재 디렉토리 (-f 옵션으로 위치는 변경이 가능)  
- 주의 ㅣ -f 옵션을 사용할 때는 꼭 ``` docker build --no-cache -t helloapp:v2 -f dockerfiles/Dockerfile dockerfiles ``` 명령어와 같이 Dockerfile이 있는 곳의 경로가 같아야 함.  
(안그럼 에러가 발생함)
​
## 2. 1월 둘째주에 만든 Spring Boot 프로젝트로 이미지 구성하고 컨테이너로 실행하기
### 1. TeamCity를 통해 빌드
### 2. 이미지 생성
- 자세한 Spring boot와 docker에 관한 내용은 https://spring.io/guides/gs/spring-boot-docker/ 를 참고
- Dockerfile 파일 작성
```
# FROM으로 기반이 되는 이미지를 생성
# Multi Stage Build로 FROM을 두번 수행
FROM centos:7
​
# centos7 와 별개로 수행됨
FROM openjdk:8-jdk-alpine
# 작성자 정보
LABEL maintainer="hyuna oh"
# FROM에서 설정한 이미지 최 상단에서 스크립트 혹은 커맨드를 실행
# 그러므로 jdk 설정시 다음의 커맨드들이 먼저 작성됨
RUN addgroup -S spring && adduser -S spring -G spring
# 명령을 실행할 사용자
USER spring:spring
# CMD, ENTRYPOINT에서 사용할 경로
WORKDIR /opt/app
# 변수 설정
ARG WAR_FILE=demo-0.0.1-SNAPSHOT.war
# 파일을 복사
COPY ${WAR_FILE} spring-build/demo-0.0.1-SNAPSHOT.war
# 포트 번호 명시 (실제로는 run 명령어와 수행해야 함)
EXPOSE 8083
# 컨테이너가 시작되었을 때 다음의 명령어를 실행
ENTRYPOINT ["java","-jar","spring-build/demo-0.0.1-SNAPSHOT.war"]
```
​
### 3. 이미지 적용 후 빌드 및 진행
​
- docker build 명령어 : ``` docker build -t demo-spring-boot-docker .```
​
- docker run 명령어 : ``` docker run -p 8083:8083 demo-spring-boot-docker```
