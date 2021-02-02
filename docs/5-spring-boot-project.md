# Spring Boot Project
## Maven BOM에 대한 소개
- Maven은 pom.xml이라는 파일을 통해 여러 dependency들을 등록하고 프로젝트를 빌드할 수 있는 기능이다.
- BOM(Bill Of Materials) 이란, 기존의 Maven POM이 제공하는 dependency들의 버전을 관리하고 업데이트해 주며 프로젝트를 빌드할 수 있는 기능이다.
- 그렇기에 BOM을 사용하는 이유는, 어떤 dependency들을 추가하더라도 버전에 대한 고민이 필요없기 때문이다.
- 가령, Spring 관련 dependency들을 추가한 뒤 다른 dependency들을 추가해야 하는 경우가 종종 있는데, 이러한 경우 버전 걱정없이 등록이 가능하다.

## Maven BOM의 특징
### Transitive Dependencies (종속성)
- 만약 2개의 dependency가 서로 다른 버전의 동일한 artifact에 종속되어 있다면, 어떤 버전이 포함되는게 맞을까?
- 정답은 "더 가까운 종속적 특성"을 지닌 버전이라고 할 수 있다. 예를 들어 설명해보겠다.
- A -> B -> C -> D 1.4  와  A -> E -> D 1.0 라는 종속적 특정이 있을 때, A의 프로젝트에 있는 D의 1.4 버전과 D의 1.0버전 중 포함되는 dependency는 1.0 버전이다. 왜냐하면, D 1.0이 A와 더 가까운 종속성을 지니고 있기 때문이다. 이를,  ```dependency mediation``` 이라고 한다.

## BOM의 사용법
### dependencyManagement 태그를 이용하여 다음의 dependency들을 등록함.
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
```
### BOM File의 사용
#### parent 태그를 사용하여 추가하는 방법과 dependency 태그에 추가하는 방법이다.
- parent 태그를 사용하여 추가
```
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
- dependency 태그에 추가 (프로젝트 규모가 크다면, 이 방식이 적합합니다)
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
### BOM Dependency 덮어쓰기
- artifact 버전의 우선순위는 다음과 같습니다
1. artifact 가 직접 version을 선언해놓은 경우
2. parent에 artifact의 버전이 존재하는 경우
3. import 된 pom과 import 된 파일들의 버전
4. dependecny mediation (버전 조정 기능)

### Spring BOM 
- 앞서 말한바와 같이 spring에 종속된 타사의 라이브러리, 혹은 다른 스프링 프로젝트를 가져올 때 오래된 버전을 가져올 수 있다. 혹은 직접 버전을 명시한 사실을 잊어버리게 되면 새로운 문제에 직면하게 된다.
- 그렇기에 이 문제를 극복하기 위해 메이븐은 BOM dependency의 컨셉을 도입했다.
- ```spring-framework-bom``` 을 dependencyManagement 섹션에 import 하면서 모든 Spring dependency들을 동일한 버전으로 등록한다.

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
- 그렇기에 더이상 특정버전들을 각 Spring artifact 들에 직접적으로 명시할 필요가 없다.
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

