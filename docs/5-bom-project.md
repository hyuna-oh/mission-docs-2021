
# Spring BOM

## 기본설정
- IntelliJ라는 IDE 툴로 Spring Initializr를 이용하여 Spring Boot Starter 프로젝트 새로 생성하였습니다.
- Spring Web 프로젝트로, Maven, Java, jdk 1.8 버전을 사용하였습니다.
- 추가 한 Dependency 항목 : Spring Web, Rest Repository, PostgreSQL Driver, MySQL Driver, Rest Repository HAL Explorer, Spring HATEOAS

![image](https://user-images.githubusercontent.com/57924258/107136931-f68f1280-694a-11eb-89d5-a2fc445210ce.png)
![image](https://user-images.githubusercontent.com/57924258/107136932-f8f16c80-694a-11eb-8760-6b927d297c3e.png)
![image](https://user-images.githubusercontent.com/57924258/107136934-fbec5d00-694a-11eb-8360-3c365f1dc6b0.png)
![image](https://user-images.githubusercontent.com/57924258/107136937-027ad480-694b-11eb-8821-9a98bb46b0e8.png)

- 다음은 초기의 pom.xml 파일 내용입니다.
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
    <artifactId>spring-boot-integration-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-boot-integration-demo</name>
    <description>Spring Boot + Data REST + JSP 등 통합 프로젝트</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-rest</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-hateoas</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-rest-hal-explorer</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

현재로선 문제가 없어보이지만, 만약 여기에 spring boot와 관련 없는 dependency를 추가 하게 된다면, 버전과 관련한 충돌 이슈가 발생할 수 있습니다.
또한, pom이 메인 pom.xml 파일과 서브 pom.xml 파일로 나뉘어져 있다면 parent 태그가 있을 경우 상속 때문에 문제가 발생할 수 있습니다.
이를 해결하기 위해 나온 것이 Maven BOM 방식입니다. Maven BOM 방식은 아래에서 설명드리겠습니다.

## Maven BOM 이란?
- 먼저, Maven은 pom.xml이라는 파일을 통해 여러 dependency들을 등록하고 프로젝트를 빌드할 수 있는 기능입니다.
- BOM(Bill Of Materials) 이란, 기존의 Maven POM이 제공하는 dependency들의 버전을 정의하고 관리하는 기능을 제공합니다.
- 그렇기에 BOM을 사용하는 이유는, 버전 관리에 용이하며 parent 태그로 인해 발생하는 문제를 해결할 수 있습니다.
- 가령, Spring Boot 관련 dependency들을 추가한 뒤 다른 dependency들을 추가해야 하는 경우가 종종 있는데, 이러한 경우 버전 걱정없이 등록이 가능합니다.

## Maven BOM의 특징
### Transitive Dependencies (dependency들의 종속성)
- 만약 2개의 dependency가 서로 다른 버전의 동일한 artifact에 종속되어 있다면, 어떤 버전이 포함되는 게 맞을까?
- 정답은 "더 가까운 종속적 특성"을 지닌 버전이라고 할 수 있습니다. 예를 들어 설명해보겠습니다.
- A -> B -> C -> D 1.4 와 A -> E -> D 1.0 라는 종속적 특정이 있을 때, A의 프로젝트에 있는 D의 1.4 버전과 D의 1.0버전 중 포함되는 dependency는 1.0 버전입니다. 
- 왜냐하면, D 1.0이 A와 더 가까운 종속성을 지니고 있기 때문입니다. 이를, dependency mediation 이라고 합니다.

### BOM Dependency의 우선순위
- 동일한 artifact 버전의 우선순위는 다음과 같습니다
1. artifact에 직접 version을 명시해놓은 경우
2. parent에 artifact의 버전이 이미 존재하는 경우
3. import 된 pom과 import 된 파일들의 버전
4. dependecny mediation 

## BOM의 사용법 
1. dependencyManagement 태그를 이용하여 다음의 dependency들을 등록합니다.
```
<project ...>
	
    <modelVersion>4.0.0</modelVersion>
    <groupId>baeldung</groupId>
    <artifactId>Baeldung-BOM</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>BaelDung-BOM</name>
    <description>parent pom</description>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>test</groupId>
                <artifactId>a</artifactId>
                <version>1.2</version>
            </dependency>
            <dependency>
                <groupId>test</groupId>
                <artifactId>b</artifactId>
                <version>1.0</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>test</groupId>
                <artifactId>c</artifactId>
                <version>1.0</version>
                <scope>compile</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
2. BOM File을 사용합니다.
두가지 방법이 있으므로 하나를 선택해서 작업하시면 됩니다.
1) parent 태그를 사용하여 추가하는 방법입니다.
<project ...>
    <modelVersion>4.0.0</modelVersion>
    <groupId>baeldung</groupId>
    <artifactId>Test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Test</name>
    <parent>
        <groupId>baeldung</groupId>
        <artifactId>Baeldung-BOM</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
</project>
```
- 하지만 이 방식은 큰 프로젝트의 경우 Maven POM을 상속하기도 하고 Sub POM을 두는데 Parent로 Spring을 추가해버리면 상속을 할 수 없게 되며, Sub POM으로 묶을수가 없습니다. 
- 그러므로, 단일 Maven POM으로 된 프로젝트라면 크게 문제가 없는데 Sub POM이 있으면 그때부터 문제가 생깁니다. 

2. dependency 태그에 추가하는 방법입니다. (프로젝트 규모가 크다면, 이 방식이 적합합니다) 
```
<project ...>
    <modelVersion>4.0.0</modelVersion>
    <groupId>baeldung</groupId>
    <artifactId>Test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Test</name>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>baeldung</groupId>
                <artifactId>Baeldung-BOM</artifactId>
                <version>0.0.1-SNAPSHOT</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

## BOM 방식을 이용하여 spring을 등록
- spring-framework-bom 을 dependencyManagement 섹션에 import 하면서 모든 Spring dependency들을 동일한 버전으로 등록합니다.
```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-framework-bom</artifactId>
            <version>4.3.8.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

- 그렇기에 특정버전들을 각 Spring artifact 들에 직접적으로 명시할 필요가 없습니다.
```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
    </dependency>
<dependencies>
```
- 실제 pom.xml 코드에 dependencyManagement 태그에 추가해보겠습니다. (앞으로 이 방식으로 BOM 예제를 진행하겠습니다.)
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>spring-boot-integration-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-boot-integration-demo</name>
    <description>Spring Boot + Data REST + JSP 등 통합 프로젝트</description>

    <properties>
		<!-- jdk 1.8에 맞는 compiler source 및 target 버전 지정 -->
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>utf-8</project.build.sourceEncoding>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <!-- Import dependency management from Spring Boot -->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.4.2</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-rest</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-hateoas</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-rest-hal-explorer</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
				<!--  기존 JAR 및 WAR 아카이브를 다시 패키징하여 명령어로 사용할 수 있도록 합니다. -->
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```
- mvn dependency:tree 명령어를 통해 종속 관계 및 버전 정보를 확인할 수 있습니다.
```
[INFO] com.example:spring-boot-integration-demo:jar:0.0.1-SNAPSHOT
[INFO] +- org.springframework.boot:spring-boot-starter:jar:2.4.2:compile
[INFO] |  +- org.springframework.boot:spring-boot:jar:2.4.2:compile
[INFO] |  |  \- org.springframework:spring-context:jar:5.3.3:compile
[INFO] |  +- org.springframework.boot:spring-boot-autoconfigure:jar:2.4.2:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter-logging:jar:2.4.2:compile
[INFO] |  |  +- ch.qos.logback:logback-classic:jar:1.2.3:compile
[INFO] |  |  |  \- ch.qos.logback:logback-core:jar:1.2.3:compile
[INFO] |  |  +- org.apache.logging.log4j:log4j-to-slf4j:jar:2.13.3:compile
[INFO] |  |  |  \- org.apache.logging.log4j:log4j-api:jar:2.13.3:compile
[INFO] |  |  \- org.slf4j:jul-to-slf4j:jar:1.7.30:compile
[INFO] |  +- jakarta.annotation:jakarta.annotation-api:jar:1.3.5:compile
[INFO] |  +- org.springframework:spring-core:jar:5.3.3:compile
[INFO] |  |  \- org.springframework:spring-jcl:jar:5.3.3:compile
[INFO] |  \- org.yaml:snakeyaml:jar:1.27:compile
[INFO] +- org.springframework.boot:spring-boot-starter-data-rest:jar:2.4.2:compile
[INFO] |  \- org.springframework.data:spring-data-rest-webmvc:jar:3.4.3:compile
[INFO] |     +- org.springframework.data:spring-data-rest-core:jar:3.4.3:compile
[INFO] |     |  +- org.springframework:spring-tx:jar:5.3.3:compile
[INFO] |     |  +- org.springframework.data:spring-data-commons:jar:2.4.3:compile
[INFO] |     |  \- org.atteo:evo-inflector:jar:1.2.2:compile
[INFO] |     +- com.fasterxml.jackson.core:jackson-databind:jar:2.11.4:compile
[INFO] |     |  \- com.fasterxml.jackson.core:jackson-core:jar:2.11.4:compile
[INFO] |     \- com.fasterxml.jackson.core:jackson-annotations:jar:2.11.4:compile
[INFO] +- org.springframework.boot:spring-boot-starter-hateoas:jar:2.4.2:compile
[INFO] |  \- org.springframework.hateoas:spring-hateoas:jar:1.2.3:compile
[INFO] |     +- org.springframework:spring-aop:jar:5.3.3:compile
[INFO] |     +- org.springframework:spring-beans:jar:5.3.3:compile
[INFO] |     \- org.springframework.plugin:spring-plugin-core:jar:2.0.0.RELEASE:compile
[INFO] +- org.springframework.boot:spring-boot-starter-web:jar:2.4.2:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter-json:jar:2.4.2:compile
[INFO] |  |  +- com.fasterxml.jackson.datatype:jackson-datatype-jdk8:jar:2.11.4:compile
[INFO] |  |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.11.4:compile
[INFO] |  |  \- com.fasterxml.jackson.module:jackson-module-parameter-names:jar:2.11.4:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter-tomcat:jar:2.4.2:compile
[INFO] |  |  +- org.apache.tomcat.embed:tomcat-embed-core:jar:9.0.41:compile
[INFO] |  |  +- org.glassfish:jakarta.el:jar:3.0.3:compile
[INFO] |  |  \- org.apache.tomcat.embed:tomcat-embed-websocket:jar:9.0.41:compile
[INFO] |  +- org.springframework:spring-web:jar:5.3.3:compile
[INFO] |  \- org.springframework:spring-webmvc:jar:5.3.3:compile
[INFO] |     \- org.springframework:spring-expression:jar:5.3.3:compile
[INFO] +- org.springframework.data:spring-data-rest-hal-explorer:jar:3.4.3:compile
[INFO] |  \- org.slf4j:slf4j-api:jar:1.7.30:compile
[INFO] +- mysql:mysql-connector-java:jar:8.0.22:runtime
[INFO] +- org.postgresql:postgresql:jar:42.2.18:runtime
[INFO] |  \- org.checkerframework:checker-qual:jar:3.5.0:runtime
[INFO] \- org.springframework.boot:spring-boot-starter-test:jar:2.4.2:test
[INFO]    +- org.springframework.boot:spring-boot-test:jar:2.4.2:test
[INFO]    +- org.springframework.boot:spring-boot-test-autoconfigure:jar:2.4.2:test
[INFO]    +- com.jayway.jsonpath:json-path:jar:2.4.0:compile
[INFO]    |  \- net.minidev:json-smart:jar:2.3:compile
[INFO]    |     \- net.minidev:accessors-smart:jar:1.2:compile
[INFO]    |        \- org.ow2.asm:asm:jar:5.0.4:compile
[INFO]    +- jakarta.xml.bind:jakarta.xml.bind-api:jar:2.3.3:test
[INFO]    |  \- jakarta.activation:jakarta.activation-api:jar:1.2.2:test
[INFO]    +- org.assertj:assertj-core:jar:3.18.1:test
[INFO]    +- org.hamcrest:hamcrest:jar:2.2:test
[INFO]    +- org.junit.jupiter:junit-jupiter:jar:5.7.0:test
[INFO]    |  +- org.junit.jupiter:junit-jupiter-api:jar:5.7.0:test
[INFO]    |  |  +- org.apiguardian:apiguardian-api:jar:1.1.0:test
[INFO]    |  |  +- org.opentest4j:opentest4j:jar:1.2.0:test
[INFO]    |  |  \- org.junit.platform:junit-platform-commons:jar:1.7.0:test
[INFO]    |  +- org.junit.jupiter:junit-jupiter-params:jar:5.7.0:test
[INFO]    |  \- org.junit.jupiter:junit-jupiter-engine:jar:5.7.0:test
[INFO]    |     \- org.junit.platform:junit-platform-engine:jar:1.7.0:test
[INFO]    +- org.mockito:mockito-core:jar:3.6.28:test
[INFO]    |  +- net.bytebuddy:byte-buddy:jar:1.10.19:test
[INFO]    |  +- net.bytebuddy:byte-buddy-agent:jar:1.10.19:test
[INFO]    |  \- org.objenesis:objenesis:jar:3.1:test
[INFO]    +- org.mockito:mockito-junit-jupiter:jar:3.6.28:test
[INFO]    +- org.skyscreamer:jsonassert:jar:1.5.0:test
[INFO]    |  \- com.vaadin.external.google:android-json:jar:0.0.20131108.vaadin1:test
[INFO]    +- org.springframework:spring-test:jar:5.3.3:test
[INFO]    \- org.xmlunit:xmlunit-core:jar:2.7.0:test
[INFO] ------------------------------------------------------------------------
```

### pom.xml에서 spring-boot-maven-plugin이란 무엇일까?
- build 태그 안에 존재하는 spring-boot-maven-plugin은 build를 수행할 때 관여하는 플러그인 입니다.
- 이 플러그인이 구체적으로 어떤 역할을 하는지 알아보기위해 pom.xml에서 해당 플러그인을 지우고 mvn clean package 명령어를 실행한 뒤 생성된 jar 파일을 실행해 보겠습니다.
![image](https://user-images.githubusercontent.com/57924258/107136959-1cb4b280-694b-11eb-8c4a-4eff6705624c.png)
- Manifest 속성이 없다하니, 생성 된 jar 파일 내부의 MANIFEST.MF 파일을 살펴보겠습니다.
- (여기서 Manifest는 jar, war 등의 파일로 패키징 할 때 사용한 설정들을 저장해 놓은 파일 정도로 생각하시면 될 것 같습니다.)
```
Manifest-Version: 1.0
Archiver-Version: Plexus Archiver
Built-By: hyunaoh
Created-By: Apache Maven 3.6.3
Build-Jdk: 1.8.0_252
```
- 다음은 디렉토리 구조입니다.
```
.
├── META-INF
│   ├── MANIFEST.MF
│   └── maven
│       └── com.example
│           └── spring-boot-integration-demo
│               ├── pom.properties
│               └── pom.xml
├── application.yml
└── com
    └── example
        └── spring
            └── SpringBootIntegrationDemoApplication.class
```
- 이번에는 plugin을 설정한 후 mvn clean package 실행 후 자바로 생성된 jar 파일을 실행시켜 보겠습니다.
![image](https://user-images.githubusercontent.com/57924258/107136962-2807de00-694b-11eb-8cf4-628f02b49ccc.png)

- 에러없이 실행되는 것을 확인 할 수 있습니다.
- 무슨 차이가 있는 것인지 MANIFEST.MF 파일을 살펴보겠습니다.
- 추가된 속성들에 대한 정보를 살펴보면, Spring Boot 관련 라이브러리와 Main Class에 대한 정보들이 추가됐음을 확인할 수 있습니다.
```
Manifest-Version: 1.0
Spring-Boot-Classpath-Index: BOOT-INF/classpath.idx
Archiver-Version: Plexus Archiver
Built-By: hyunaoh
Spring-Boot-Layers-Index: BOOT-INF/layers.idx
Start-Class: com.example.spring.SpringBootIntegrationDemoApplication
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Spring-Boot-Version: 2.4.2
Created-By: Apache Maven 3.6.3
Build-Jdk: 1.8.0_252
Main-Class: org.springframework.boot.loader.JarLauncher
```
- 이외에도 디렉토리 구조가 바뀐 것을 확인 할 수 있습니다.
```
.
├── BOOT-INF
│   ├── classes
│   │   ├── application.yml
│   │   └── com
│   ├── classpath.idx
│   ├── layers.idx
│   └── lib
│       ├── accessors-smart-1.2.jar
│       ├── asm-5.0.4.jar
│       ├── checker-qual-3.5.0.jar
..  중략 ...
│       ├── spring-web-5.3.3.jar
│       ├── spring-webmvc-5.3.3.jar
│       ├── tomcat-embed-core-9.0.41.jar
│       └── tomcat-embed-websocket-9.0.41.jar
├── META-INF
│   ├── MANIFEST.MF
│   └── maven
│       └── com.example
└── org
    └── springframework
        └── boot	
```

- 관련하여 세부적인 내용은 https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/ 를 참조하시면 됩니다.
## Maven에 HikariCP dependency 추가
- HikariCP란 DataBase Connection Pool 의 종류 중 하나입니다.
- (다른 DBCP들보다 빠르고 가벼워 많이들 사용한다고 합니다.)
- DBCP라고 하는데, 이는 외부 라이브러리의 Connection 객체들을 저장해둔 pool로 connection을 계속 생성하는 것이 아닌 빌려쓸 수 있는 저장소라고 생각하시면 됩니다.
- 상단의 pom.xml에 다음 dependency를 pom.xml에 추가하겠습니다.
```
        <dependency>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
        </dependency>
```
- 추가한 후 tree구조를 살펴 보면 다음과 같습니다.
```
... 생략...
[INFO] +- com.zaxxer:HikariCP:jar:3.4.5:compile
... 생략...
```
- 현재 3.4.5 버전이 설치 된 것을 확인 할 수 있습니다.
- 그렇다면 이 버전의 정보는 어디서 가져올 수 있는것일까요?
- 다시 pom.xml의 dependencyManagement 태그 안에 있던 spring-boot-dependencies를 보면 hikaricp version 정보를 확인할 수 있습니다.
![image](https://user-images.githubusercontent.com/57924258/107136963-2b9b6500-694b-11eb-9456-807734a9d162.png)

## Maven 사용시 기타 참고사항
- 참고로 maven 사용 전에 Always update snapshots 설정을 반드시 체크하고 수행하시기 바랍니다..
- 안 그러면 pom.xml에서 수정된 항목이 제대로 반영되지 않아 큰 곤란을 겪을 수 있습니다.. 휴..ㅠㅠㅋ
![image](https://user-images.githubusercontent.com/57924258/107136964-2e965580-694b-11eb-9a04-cd9cb779ff09.png)




