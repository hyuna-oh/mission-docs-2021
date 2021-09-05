# Nexus Repository
- Nexus Repository 란 ? maven에서 사용할 수 있는 오픈소스 repository
​
## Vagrant + CentOS 7에서 Nexus Repository Server 설치
​
### 패키지 다운로드
- Windows command 창에 ``` vagrant up ``` 명령어 실행 후 ```vagrant ssh``` 명령어로 shell 실행
- 설치할 목록은 https://help.sonatype.com/repomanager3/download/download-archives---repository-manager-3 를 참고
- 설치 과정은 https://help.sonatype.com/repomanager3/installation 를 참고
- 설치 이전에 jdk 1.8 버전이 설치되어 있어야 합니다.
​
- 설치 및 디렉토리 변경
```
# nexus repository 패키지 설치
$ wget install https://download.sonatype.com/nexus/3/latest-unix.tar.gz   --no-check-certificate
# 설치 후 압축 해제
$ tar -xvf latest-unix.tar.gz
# 압축 해제한 파일을 /opt 디렉토리 밑에 옮김
$ sudo mv nexus-3.29.2-02 /opt/
```
- port 변경
```
$ cd /opt/nexus-3.29.2-02/etc  
$ vi nexus-default.properties
```
![image](https://user-images.githubusercontent.com/57924258/105713708-c9139380-5f5e-11eb-8856-0153864721d5.png)
​
- 외부에서 사용해야 할 경우
 - 방화벽 오픈
```
firewall-cmd --permanent --zone=public --add-port=8082/tcp
firewall-cmd --reload
firewall-cmd --permanent --list-all
```
​
### nexus 계정정보 등록 및 nexus 계정 생성
```
sudo groupadd nexus && adduser nexus -g nexus
패스워드 정보도
```
- ```vi /opt/nexus-3.29.2-02/bin/nexus``` 실행  
``` RUN_AS_USER ``` 항목에 nexus 계정 등록
​
​
### 서비스 등록해서 자동 실행 시키기
- 다음의 명령어를 입력하여 리눅스 서비스에 등록
​
```
sudo vi /etc/systemd/system/nexus.service
```
```
[Unit]
Description=nexus service
After=network.target
 
[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus-3.15.2-01/bin/nexus start
ExecStop=/opt/nexus-3.15.2-01/bin/nexus stop
User=nexus
Restart=on-abort
TimeoutSec=600
 
[Install]
WantedBy=multi-user.target
```
```
sudo systemctl daemon-reload
sudo systemctl enable nexus.service
sudo systemctl start nexus.service
```
log는 ```tail -f /opt/sonatype-work/nexus3/log/nexus.log```를 참고하면 됨  
- 서비스 실행 후 커맨드 창
```
-------------------------------------------------
​
Started Sonatype Nexus OSS 3.29.2-02
​
-------------------------------------------------
```
​
- 접속
http://도메인:포트번호/ 화면을 통해 접속
​
![image](https://user-images.githubusercontent.com/57924258/105860784-4dcee200-6031-11eb-9edc-1ab01176bc8b.png)
​
## Spring 등의 외부 Repository를 Nexus에 등록
​
1. 먼저 UI에서 Blob Stores에 저장 될 저장소를 만든다. (test로 만듦)
2. 만든 후 설정-Repository 메뉴로 들어가 새로운 Repository를 생성한다.
​
![nexus-repositories-1](https://user-images.githubusercontent.com/57924258/106151653-eabf8580-61bf-11eb-8f16-192f08aa3a80.JPG)
![nexus-repositories-3](https://user-images.githubusercontent.com/57924258/106151658-ebf0b280-61bf-11eb-9fc3-dcb77ae14e56.JPG)
![repo-1](https://user-images.githubusercontent.com/57924258/106152823-36266380-61c1-11eb-8d0c-bf4f415e434e.JPG)
![repo-2](https://user-images.githubusercontent.com/57924258/106152826-37f02700-61c1-11eb-8b94-5849e9bc6bd5.JPG)
​
​
## Maven의 settings.xml 파일 추가하여 설치한 Nexus 서버로 미러링
​
- C:/user/.m2 폴더에 settings.xml 파일을 추가하여 다음과 같이 내용을 작성한다.
```
<?xml version="1.0" encoding="UTF-8"?>
<settings   xmlns="http://maven.apache.org/SETTINGS/1.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
 <mirrors>
   <mirror>
     <id>Maven Repo</id>
     <name>Maven Repo</name>
     <url>http://[url]/repository/maven-public/</url>
     <mirrorOf>central</mirrorOf>
   </mirror>
 </mirrors>
</settings>
```
​
## Spring Boot Web 프로젝트 구성
![intelli_1](https://user-images.githubusercontent.com/57924258/106151593-db403c80-61bf-11eb-8760-a5e344092659.JPG)
![intelli_2](https://user-images.githubusercontent.com/57924258/106151608-dda29680-61bf-11eb-8177-bd44abee0fed.JPG)
![intelli_3](https://user-images.githubusercontent.com/57924258/106151613-ded3c380-61bf-11eb-94c2-c5b3d8233e54.JPG)
![intelli_4](https://user-images.githubusercontent.com/57924258/106151616-e004f080-61bf-11eb-8a71-e18e1d66b32b.JPG)
![intelli_5](https://user-images.githubusercontent.com/57924258/106151620-e1ceb400-61bf-11eb-8a5f-e32c523287b8.JPG)
![intelli_6](https://user-images.githubusercontent.com/57924258/106151625-e3987780-61bf-11eb-9470-353b75ad491b.JPG)
​
## Spring Boot를 빌드하여 Nexus에서 dependency 다운로드하는지 확인
​
- 이 때 저는 spring repository 만 추가되어 있기 때문에 다른 패키지들을 다운로드 할 때 에러가 발생합니다.
- 그래서 실제로 maven install을 할 때는 이미 존재하는 maven-public repository를 이용했습니다.
- Maven clean install을 돌리면 log 파일을 다음과 같다.
```
"...
[INFO] Scanning for projects...
Downloading from Maven Repo: http://192.168.33.10:8089/repository/maven-public/org/springframework/boot/spring-boot-starter-parent/2.4.2/spring-boot-starter-parent-2.4.2.pom
Downloaded from Maven Repo: http://192.168.33.10:8089/repository/maven-public/org/springframework/boot/spring-boot-starter-parent/2.4.2/spring-boot-starter-parent-2.4.2.pom (0 B at 0 B/s)
Downloading from Maven Repo: http://192.168.33.10:8089/repository/maven-public/org/springframework/boot/spring-boot-dependencies/2.4.2/spring-boot-dependencies-2.4.2.pom
Downloaded from Maven Repo: http://192.168.33.10:8089/repository/maven-public/org/springframework/boot/spring-boot-dependencies/2.4.2/spring-boot-dependencies-2.4.2.pom (0 B at 0 B/s)
Downloading from Maven Repo: http://192.168.33.10:8089/repository/maven-public/com/datastax/oss/java-driver-bom/4.9.0/java-driver-bom-4.9.0.pom
Downloaded from Maven Repo: http://192.168.33.10:8089/repository/maven-public/com/datastax/oss/java-driver-bom/4.9.0/java-driver-bom-4.9.0.pom (0 B at 0 B/s)
Downloading from Maven Repo: http://192.168.33.10:8089/repository/maven-public/io/dropwizard/metrics/metrics-bom/4.1.17/metrics-bom-4.1.17.pom
Downloaded from Maven Repo: http://192.168.33.10:8089/repository/maven-public/io/dropwizard/metrics/metrics-bom/4.1.17/metrics-bom-4.1.17.pom (0 B at 0 B/s)
...
```
​
## maven deploy 커맨드로 내가 만든 Spring Boot의 빌드한 JAR 파일을 Nexus에 배포하기
- Repositories에 등록되어 있는 maven-snapshots repository를 이용하여 배포할 계획입니다.
- 정리하자면, maven-public repository에서 pom.xml에 있는 파일들을 미러링→ 해당 jar 파일들을 다운로드 → maven-snapshots repository에 배포한다는 계획입니다.
- 다음은 settings.xml에 deploy 설정을 추가한 내용입니다.
​
```
<?xml version="1.0" encoding="UTF-8"?>
<settings   xmlns="http://maven.apache.org/SETTINGS/1.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
           
   <mirrors>
       <mirror>
         <id>maven-public</id>
         <name>maven-public</name>
         <url>http://[maven repository url]:[maven repository port]/repository/maven-public/</url>
         <mirrorOf>central</mirrorOf>
       </mirror>
   </mirrors>
   <!-- pom.xml의 snapshotRepository id와 Nexus username 및 password를 입력해줍니다. -->
   <servers>
      <server>
         <id>nexus-snapshots</id>
         <username>admin</username>
         <password>****</password>
      </server>
   </servers>
</settings>
```
​
- Windows 10에서 pom.xml
​
```
... 생략
   <build>
       <plugins>
           ... 생략
           <plugin>
               <groupId>org.apache.maven.plugins</groupId>
               <artifactId>maven-deploy-plugin</artifactId>
               <version>${maven-deploy-plugin.version}</version>
               <configuration>
                   <skip>true</skip>
               </configuration>
           </plugin>
           <plugin>
               <groupId>org.sonatype.plugins</groupId>
               <artifactId>nexus-staging-maven-plugin</artifactId>
               <version>1.5.1</version>
               <executions>
                   <execution>
                       <id>default-deploy</id>
                       <phase>deploy</phase>
                       <goals>
                           <goal>deploy</goal>
                       </goals>
                   </execution>
               </executions>
               <configuration>
                   <serverId>nexus</serverId>
                   <nexusUrl>http://[maven repository url]:[maven repository port]/repository/maven-snapshots/</nexusUrl>
                   <skipStaging>true</skipStaging>
               </configuration>
           </plugin>
       </plugins>
   </build>
   <distributionManagement>
       <snapshotRepository>
           <id>nexus-snapshots</id>
           <url>http://[maven repository url]:[maven repository port]/repository/maven-snapshots/</url>
       </snapshotRepository>
   </distributionManagement>
... 생략
```
​
- 설정이 완료 되었다면, maven clean deploy -Dmaven.test.skip=true 명령어를 통해 maven을 실행시킵니다. (저는 Windows에 maven설치가 안되어 있어 spring boot 프로젝트의 mvnw 파일을 통해 mvn 명령어로 실행하였습니다.)
​
```
>mvn clean deploy -Dmaven.test.skip=true -e
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------------< com.example:demo >--------------------------
[INFO] Building demo 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-clean-plugin:3.1.0:clean (default-clean) @ demo ---
[INFO] Deleting C:\Users\hyunaoh\git\nexus-maven-demo\target
[INFO]
[INFO] --- maven-resources-plugin:3.2.0:resources (default-resources) @ demo ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Using 'UTF-8' encoding to copy filtered properties files.
[INFO] Copying 1 resource
[INFO] Copying 0 resource
[INFO]
[INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ demo ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to C:\Users\hyunaoh\git\nexus-maven-demo\target\classes
[INFO]
[INFO] --- maven-resources-plugin:3.2.0:testResources (default-testResources) @ demo ---
[INFO] Not copying test resources
[INFO]
[INFO] --- maven-compiler-plugin:3.8.1:testCompile (default-testCompile) @ demo ---
[INFO] Not compiling test sources
[INFO]
[INFO] --- maven-surefire-plugin:2.22.2:test (default-test) @ demo ---
[INFO] Tests are skipped.
[INFO]
[INFO] --- maven-jar-plugin:3.2.0:jar (default-jar) @ demo ---
[INFO] Building jar: C:\Users\hyunaoh\git\nexus-maven-demo\target\demo-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:2.4.2:repackage (repackage) @ demo ---
[INFO] Replacing main artifact with repackaged archive
[INFO]
[INFO] --- maven-install-plugin:2.5.2:install (default-install) @ demo ---
[INFO] Installing C:\Users\hyunaoh\git\nexus-maven-demo\target\demo-0.0.1-SNAPSHOT.jar to C:\Users\hyunaoh\.m2\repository\com\example\demo\0.0.1-SNAPSHOT\demo-0.0.1-SNAPSHOT.jar
[INFO] Installing C:\Users\hyunaoh\git\nexus-maven-demo\pom.xml to C:\Users\hyunaoh\.m2\repository\com\example\demo\0.0.1-SNAPSHOT\demo-0.0.1-SNAPSHOT.pom
[INFO]
[INFO] --- maven-deploy-plugin:2.8.2:deploy (default-deploy) @ demo ---
[INFO] Skipping artifact deployment
[INFO]
[INFO] --- nexus-staging-maven-plugin:1.5.1:deploy (default-deploy) @ demo ---
[INFO] Performing deferred deploys (gathering into "C:\Users\hyunaoh\git\nexus-maven-demo\target\nexus-staging\deferred")...
[INFO] Installing C:\Users\hyunaoh\git\nexus-maven-demo\target\demo-0.0.1-SNAPSHOT.jar to C:\Users\hyunaoh\git\nexus-maven-demo\target\nexus-staging\deferred\com\example\demo\0.0.1-SNAPSHOT\demo-0.0.1-SNAPSHOT.jar
[INFO] Installing C:\Users\hyunaoh\git\nexus-maven-demo\pom.xml to C:\Users\hyunaoh\git\nexus-maven-demo\target\nexus-staging\deferred\com\example\demo\0.0.1-SNAPSHOT\demo-0.0.1-SNAPSHOT.pom
[INFO] Deploying remotely...
[INFO] Bulk deploying locally gathered artifacts from directory:
[INFO]  * Bulk deploying locally gathered snapshot artifacts to URL http://192.168.33.10:8089/repository/maven-snapshots/
Downloading from nexus-snapshots: http://192.168.33.10:8089/repository/maven-snapshots/com/example/demo/0.0.1-SNAPSHOT/maven-metadata.xml
Uploading to nexus-snapshots: http://192.168.33.10:8089/repository/maven-snapshots/com/example/demo/0.0.1-SNAPSHOT/demo-0.0.1-20210130.101513-1.jar
Uploaded to nexus-snapshots: http://192.168.33.10:8089/repository/maven-snapshots/com/example/demo/0.0.1-SNAPSHOT/demo-0.0.1-20210130.101513-1.jar (8.5 MB at 872 kB/s)
Uploading to nexus-snapshots: http://192.168.33.10:8089/repository/maven-snapshots/com/example/demo/0.0.1-SNAPSHOT/demo-0.0.1-20210130.101513-1.pom
Uploaded to nexus-snapshots: http://192.168.33.10:8089/repository/maven-snapshots/com/example/demo/0.0.1-SNAPSHOT/demo-0.0.1-20210130.101513-1.pom (2.9 kB at 315 B/s)
Downloading from nexus-snapshots: http://192.168.33.10:8089/repository/maven-snapshots/com/example/demo/maven-metadata.xml
Uploading to nexus-snapshots: http://192.168.33.10:8089/repository/maven-snapshots/com/example/demo/0.0.1-SNAPSHOT/maven-metadata.xml
Uploaded to nexus-snapshots: http://192.168.33.10:8089/repository/maven-snapshots/com/example/demo/0.0.1-SNAPSHOT/maven-metadata.xml (765 B at 84 B/s)
Uploading to nexus-snapshots: http://192.168.33.10:8089/repository/maven-snapshots/com/example/demo/maven-metadata.xml
Uploaded to nexus-snapshots: http://192.168.33.10:8089/repository/maven-snapshots/com/example/demo/maven-metadata.xml (275 B at 30 B/s)
[INFO]  * Bulk deploy of locally gathered snapshot artifacts finished.
[INFO] Remote deploy finished with success.
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  41.025 s
[INFO] Finished at: 2021-01-30T19:15:50+09:00
[INFO] ------------------------------------------------------------------------
```
​
- 다음은 배포 완료 후 Nexus Repository 화면입니다.
- 해당 jar 파일이 들어와 있는 것을 확인 할 수 있습니다.
​
![image](https://user-images.githubusercontent.com/57924258/106357349-fd18fb00-6348-11eb-94b3-447ef791b363.png)
​
- http://192.168.33.10:8089/repository/maven-snapshots/com/example/demo/0.0.1-SNAPSHOT/demo-0.0.1-20210130.103414-1.pom 를 확인하면 다음과 같습니다. (아래는 생략)
​
![image](https://user-images.githubusercontent.com/57924258/106357375-16ba4280-6349-11eb-834b-0fe98152370c.png)
​
- maven deploy nexus 설정에 대해 자세히 알고 싶다면, https://www.baeldung.com/maven-deploy-nexus을 참고하셔도 됩니다.
​
## Nexus Repository에 소스까지 배포하기
​
pom.xml에 maven-source-plugin을 추가
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <parent>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-parent</artifactId>
       <version>2.4.2</version>
       <relativePath/> <!-- lookup parent from repository -->
   </parent>
   <groupId>com.example</groupId>
   <artifactId>demo</artifactId>
   <version>0.0.1-SNAPSHOT</version>
   <name>demo</name>
   <description>Demo project for Spring Boot</description>
   <properties>
       <java.version>1.8</java.version>
       <maven-deploy-plugin.version>2.8.2</maven-deploy-plugin.version>
   </properties>
   <dependencies>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter</artifactId>
       </dependency>
​
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-test</artifactId>
           <scope>test</scope>
       </dependency>
   </dependencies>
​
   <build>
       <plugins>
           <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
           </plugin>
           <plugin>
               <groupId>org.apache.maven.plugins</groupId>
               <artifactId>maven-source-plugin</artifactId>
               <version>3.2.1</version>
               <executions>
                   <execution>
                       <id>attach-sources</id>
                       <goals>
                           <goal>jar</goal>
                       </goals>
                   </execution>
               </executions>
           </plugin>
           <plugin>
               <groupId>org.apache.maven.plugins</groupId>
               <artifactId>maven-deploy-plugin</artifactId>
               <version>${maven-deploy-plugin.version}</version>
               <configuration>
                   <skip>true</skip>
               </configuration>
           </plugin>
           <plugin>
               <groupId>org.sonatype.plugins</groupId>
               <artifactId>nexus-staging-maven-plugin</artifactId>
               <version>1.5.1</version>
               <executions>
                   <execution>
                       <id>default-deploy</id>
                       <phase>deploy</phase>
                       <goals>
                           <goal>deploy</goal>
                       </goals>
                   </execution>
               </executions>
               <configuration>
                   <serverId>nexus</serverId>
                   <nexusUrl>http://192.168.33.10:8089/repository/maven-snapshots/</nexusUrl>
                   <skipStaging>true</skipStaging>
               </configuration>
           </plugin>
       </plugins>
   </build>
​
   <distributionManagement>
       <snapshotRepository>
           <id>nexus-snapshots</id>
           <url>http://192.168.33.10:8089/repository/maven-snapshots/</url>
       </snapshotRepository>
   </distributionManagement>
​
</project>
​
```
- 다음 화면은 source jar파일 추가된 걸 확인할 수 있음
![image](https://user-images.githubusercontent.com/57924258/106358310-de1d6780-634e-11eb-9a29-aeb3269f4a5f.png)
​
## release / snapshot
​
- SNAPSHOT : JAR 파일에 TIMESTAMP와 SEQUENCE가 붙으면 pom.xml 파일에 version이 SNAPSHOT 인 경우 -> 항상 최신 버전을 가지고 옴
- RELEASE : version을 1.0 1.0.1 이런식으로 fix 시키면 버전 업로드가 중지되는데, 이런 버전을 릴리즈 버전이라 함.
- Spring은 4.3.1.RELEASE 이렇게 쓰다가 최근에 5.2.0 이런식으로 바꾸었음
