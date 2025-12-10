# Docker Compose 를 활용해 컨테이너 관리하기



## Docker Compose 를 사용하는 이유&#x20;

### Docker Compose 란?&#x20;

여러 개의 Docker 컨테이너들을 하나의 서비스로 정의하고 구성해 하나의 묶음으로 관리할 수 있게 해주는 툴이다.&#x20;

### Docker Compose 를 사용하는 이유

1. 여러 개의 컨테이너를 관리하는 데 용이&#x20;
   1. 여러 개의 컨테이너로 이루어진 복잡한 애플리케이션을 한 번에 관리할 수 있게 해준다.&#x20;
   2. 여러 개의 컨테이너를 하나의 환경에서 실행하고 관리하는 데 도움이 된다.&#x20;
2. 복잡한 명령어로 실행시키던 걸 간소화 시킬 수 있다.&#x20;
   1. 이전에 MySQL 이미지를 컨테이너로 실행시킬 때 아래와 같은 명령어를 실행시켰다.&#x20;
   2. 너무 복잡하다;;&#x20;
   3. Docker Compose 를 사용하면 위와 같이 컨테이너를 실행시킬 때마다 복잡한 명령어를 입력하지 않아도 된다.&#x20;

```shellscript
$ docker run -e MYSQL_ROOT_PASSWORD=password123 -p 33063306 -v /Users/jaeseong/Documents/Develop/docker-practice/my_data:/var/lib/mysql
```

## 자주 사용하는 Docker Compose CLI 명령어

> 아래 명령어들은 compose.yml 이 존재하는 디렉토리에서 실행시켜야 한다.&#x20;

### compose 파일 작성&#x20;

#### compose.yml&#x20;

```yml
services: 
    webserver:
        container_name: webserver
        image: nginx 
        ports: 
            -80:80
```

### compose.yml 에서 정의한 컨테이너 실행&#x20;

```shellscript
$ docker compose up    # 포그라운드에서 실행
$ docker compose up -d # 백그라운드에서 실행
```

### Docker compose 로 실행시킨 컨테이너 확인하기&#x20;

```shellscript
# compose.yml 에 정의된 컨테이너 중 실행 중인 컨테이너만 보여준다. 
$ docker compose ps 

# compose.yml 에 정의된 모든 컨테이너를 보여준다. 
$ docker compose ps -a 
```

### 컨테이너를 실행하기 전에 이미지 재빌드하기

```shellscript
$ docker compose up --build # 포그라운드에서 실행 
$ docker compose up --build -d # 백그라운드에서 실행
```

* `compose.yml` 에서 정의한 이미지 파일에서 코드가 변경되었을 경우, 이미지를 다시 빌드해서 컨테이너를 실행시켜야 코드 변경된 부분이 적용된다.&#x20;
* 그러므로 이럴 때에는 `--build` 옵션을 추가해서 사용해야 한다.&#x20;

### 이미지 다운받기 / 업데이트하기&#x20;

```shellscript
$ docker compose pull 
```

* `compose.yml` 에서 정의된 이미지를 다운 받거나 업데이트 한다.&#x20;
  * 로컬 환경에 이미지가 없다면 이미지를 다운 받는다.&#x20;
  * 로컬 환경에 이미지가 있는데, Dockerhub 의 이미지와 다른 이미지일 경우 이미지를 업데이트 한다.&#x20;

### Docker Compose 에서 이용한 컨테이너 종료하기&#x20;

```shellscript
$ docker compose down 
```

## Spring Boot, MySQL 컨테이너 동시에 띄워보기

### Spring Boot, MySQL 컨테이너 동시에 띄워보기&#x20;

#### 1. Spring Boot 프로젝트 세팅&#x20;

<figure><img src="../../../../.gitbook/assets/image (262).png" alt=""><figcaption></figcaption></figure>

* Java 17 버전을 선택&#x20;
* Dependencies 는 `Spring Boot DevTools`, `Spring Web`, `Spring Data JPA`, `MySQL Driver`  선택&#x20;

#### 2. 간단한 코드 작성&#x20;

**AppController**&#x20;

<pre class="language-java"><code class="lang-java"><strong>@RestController
</strong>public class AppController {
  @GetMapping("/")
  public String home() {
    return "Hello, World!";
  }
}
</code></pre>

#### 3. application.yml 에 DB 연결을 위한 정보 작성하기&#x20;

application.yml&#x20;

```yml
spring: 
    datasource: 
        url: jdbc:mysql://localhost:3306/mydb
        username: root
        password: pwd1234
        driver-class-name: com.mysql.cj.jdbc.Driver
```

#### 4. Dockerfile 작성하기&#x20;

Dockerfile&#x20;

```docker
FROM openjdk:17-jdk

COPY build/libs/*SNAPSHOT.jar /app.jar

ENTRYPOINT "java", "-jar", "/app.jar"]
```

#### 5. compose.yml 파일 작성하기&#x20;

**compose.yml**

```yml
services:
    my-server:
        build: . 
        ports: 
            -8080:8080
        depens_on:
            my-db:
                condition:  service_healthy
    my-db:
        image: mysql
        environment:
            MYSQL_ROOT_PASSWORD: pwd1234
            MYSQL_DATABASE: mydb
        volumes: 
            - ./mysql_data:/var/lib/mysql
        ports: 
            - 3306:3306
        healthcheck:
            test: ["CMD", "mysqladmin", "ping"]
            interval: 5s 
            retries: 10
```

#### 6. Spring Boot 프로젝트 빌드하기&#x20;

```shellscript
$ ./gradlew clean build 

$ docker compose up -d --build

$ docker compose ps 
$ docker ps 
$ docker logs [Container ID]
```

* Spring Boot 컨테이너 로그를 확인해보면 아래와 같이 에러 메시지가 떠있다. 아래 에러 메시지는 DB 와 연결이 제대로 이루어지지 않았다는 의미이다.&#x20;
* 왜그럴까?&#x20;

### 컨테이너로 실행시킨 Spring Boot 가 MySQL 에 연결이 안 되는 이유

#### 컨테이너로 실행시킨 Spring Boot 가 MySQL 에 연결이 안 되는 이유

<figure><img src="../../../../.gitbook/assets/image (263).png" alt=""><figcaption></figcaption></figure>

* 각각의 컨테이너는 자신만의 네트워크망과 IP 주소를 가지고 있다. 호스트 컴퓨터 입장에서 localhost 는 호스트 컴퓨터를 가리키지만, Spring Boot 컨테이너 입장에서 localhost 는 Spring Boot 컨테이너를 가리킨다.&#x20;
* 그런데, Spring Boot 의 코드를 작성할 때 DB 정보를 아래와 같이 입력했었다. Spring Boot 가 실행되는환경인 컨테이너 입장에서 `localhost:3306` 이라는 주소는, Spring Boot 컨테이너 내부에 있는 3306 포트와 연결을 시도하게 된다. 하지만 Spring Boot 가 실행되는 컨테이너 내부의  3306 포트에는 아무것도 실행되고 있지 않다.&#x20;
* 이러한 구조상의 문제 때문에, Spring Boot 가 MySQL 에 연결이 안되고 있었던 것이다.&#x20;

#### Spring Boot(application.yml) 의 DB 정보를 아래와 같이 수정한 뒤 시도해보기&#x20;

**application.yml**&#x20;

```yml
spring: 
    datasource: 
        url: jdbc:mysql://my-db:3306/mydb
        username: root
        password: pwd1234
        driver-class-name: com.mysql.cj.jdbc.Driver
```

* `compose.yml` MySQL 서비스 이름을 my-db 로 설정해두었다.&#x20;
* **Spring Boot 내 `application.yml` 파일에서 해당 서비스 이름을 사용하면 `compose.yml` 에서 정의한 서비스와 연결을 시도하게 되고,**&#x20;
* 실행해보면 정상 연결되는 것을 확인할 수 있다.&#x20;

