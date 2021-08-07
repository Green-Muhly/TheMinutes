# DB 변경 (H2 -> MySQL)

### 210803

## 문제상황:

AWS EC2 를 통해 ubuntu server 20.04 인스턴스 시작

java를 설치하고, 

h2 db를 받으려하는데 linux 환경에서 터미널을 통해 설치하지 못했다.

github을 통해 받았지만 browser가 없어서 실행되지 않았다.

firefox browser를 다운받아서 실행은 시켰지만,

여러모로 번거롭기도 하고 범용적인 DB를 활용하기 위해 MySQL로 변경하기로 결정

----

### 해결 방법:

Gradle로 Build 했기 때문에 기존 프로젝트에서 `application.yml` 를 수정했다.

```java
   
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/todolist
serverTimezone=UTC&characterEncoding=UTF-8
    username: ****
    password: *******

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        #        show_sql: true
        format_sql: true

logging.level:
  org.hibernate.SQL: debug
#  org.hibernate.type: trace
```

하지만 바로 연결되지 않았는데, 과거에 MySQL로 진행했었던 과제가 있어서 username과 password가 맞지 않았다. 그래서 MySQL을 재설치하고 실행하니 작동하였다.

----

JPA에서의 설정도 맞춰주기 위해 `persisence.xml` 또한 수정했다.

```java
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver"/>
            <property name="javax.persistence.jdbc.user" value="root"/>
            <property name="javax.persistence.jdbc.password" value="qwer1234"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql:tcp://localhost:3306/todolist"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQLDialect"/>

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="create" />
        </properties>
    </persistence-unit>
</persistence>
```



----

### AWS RDS 이용해서 배포하기

`application.yml`

```java
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://todolist.c55jrjs9yxoi.ap-northeast-2.rds.amazonaws.com:3306/todolist?serverTimezone=UTC&characterEncoding=UTF-8
    username: root
    password: qwer1234

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        #        show_sql: true
        format_sql: true
    database: mysql

logging.level:
  org.hibernate.SQL: debug
#  org.hibernate.type: trace
```

MySQL을 RDS에서 관리하고 EC2에서 구동하는 형식으로 변경.

RDS 인스턴스를 생성하고 엔드포인트를 받아와서 연결.

보안그룹을 따로 설정하여 EC2와 연결.
