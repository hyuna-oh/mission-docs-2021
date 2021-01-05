# mission2021
2021년에 공부한 내용들을 문서로 저장하는 곳.
# 1. vagrant (2021.01.04 ~ 2021.01.10)
- 베이그런트는 가상화 소프트웨어 (버추얼박스, 도커 컨테이너, AWS 등)의 생성 및 유지보수를 위한 오픈소스 소프트웨어 제품
- 루비 언어로 작성됨

## 작업 LIST
### Box로 VM 생성하기 (CentOS, Ubuntu 등)
 * Vagrantfile 을 로컬의 user 디렉토리 밑에 생성
 1. 방법1) vagrant init bento/centos-7.4 명령어로 Vagrantfile을 생성 후 생성된 파일에 다음의 내용 작성 
```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.2"
  config.vm.box_version = "2.3.1"
end
```
 2. 방법2) vagrant가 설치된 디렉토리 상위 폴더에 Vagrantfile을 직접 생성하여 작성 (***디렉토리 한번 확인해보기)
### VM 삭제
### VM의 설정 변경
#### CORE
#### MEMORY
#### DISK (중요)
### 포트 포워딩

### SSH 접속
 * SSH 접속 Command
 ```vagrant ssh``` 

### GITLAB 설치
### VM과 로컬 PC의 디렉토리 공유
### 네트워크 설정 변경
#### Private IP 설정
#### DHCP 설정
### Plugin 설치 및 사용
### 기존 Box로 자체 신규 Box 만들기
### 신규 Box 파일을 웹 서버에 올려놓고 자체 Box 서버를 통해 Box 파일 내려받기
