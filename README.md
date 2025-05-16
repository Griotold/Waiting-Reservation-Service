# BOBJOOL

![Image](https://github.com/user-attachments/assets/5598d19a-41bf-45bd-a492-daa093a6c656)

사용자들이 식당에서의 대기 시간을 줄이고, 효율적으로 예약을 관리할 수 있도록 돕는 온라인 식당 줄서기 및 예약 서비스입니다. 

# 인프라 설계도
![Image](https://github.com/user-attachments/assets/9cc37e82-2096-4b32-97ce-0bedd310e4ca)

# 기술 스택
<div align="center"> <h1>🛠 STACKS</h1> </div> <div align="center"> <!-- Language --> <img src="https://img.shields.io/badge/Java-17-007396?style=for-the-badge&logo=java&logoColor=white"/> <!-- Backend --> <img src="https://img.shields.io/badge/Spring%20Boot-3.3.1-6DB33F?style=for-the-badge&logo=springboot&logoColor=white"/> <img src="https://img.shields.io/badge/Spring-6DB33F?style=for-the-badge&logo=spring&logoColor=white"/> <img src="https://img.shields.io/badge/Spring%20Security-6DB33F?style=for-the-badge&logo=springsecurity&logoColor=white"/> <img src="https://img.shields.io/badge/QueryDSL-6DB33F?style=for-the-badge&logo=hibernate&logoColor=white"/> <!-- Spring Cloud --> <img src="https://img.shields.io/badge/Spring%20Cloud%20Eureka-6DB33F?style=for-the-badge&logo=spring&logoColor=white"/> <img src="https://img.shields.io/badge/Spring%20Cloud%20Gateway-6DB33F?style=for-the-badge&logo=spring&logoColor=white"/> <img src="https://img.shields.io/badge/Spring%20Cloud%20Config-6DB33F?style=for-the-badge&logo=spring&logoColor=white"/> <img src="https://img.shields.io/badge/Spring%20Cloud%20OpenFeign-6DB33F?style=for-the-badge&logo=spring&logoColor=white"/> <img src="https://img.shields.io/badge/Spring%20Cloud%20Circuit%20Breaker-6DB33F?style=for-the-badge&logo=spring&logoColor=white"/> <!-- Messaging & Infra --> <img src="https://img.shields.io/badge/Kafka-231F20?style=for-the-badge&logo=apachekafka&logoColor=white"/> <img src="https://img.shields.io/badge/Zookeeper-FF6D00?style=for-the-badge&logo=apachezookeeper&logoColor=white"/> <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white"/> <img src="https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white"/> <!-- Database & Cache --> <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white"/> <img src="https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white"/> <!-- Monitoring & Tracing --> <img src="https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white"/> <img src="https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white"/> <img src="https://img.shields.io/badge/Zipkin-CC0033?style=for-the-badge&logo=apache&logoColor=white"/> <!-- Test & Performance --> <img src="https://img.shields.io/badge/Junit5-25A162?style=for-the-badge&logo=java&logoColor=white"/> <img src="https://img.shields.io/badge/JMeter-D22128?style=for-the-badge&logo=apachejmeter&logoColor=white"/> </div>

# 내가 담당한 부분
## 1. 멀티 모듈 및 아키텍쳐 설계
- 공통으로 사용되는 코드, 설정들을 공통 모듈(common)에 따로 두어 관리
- 유지보수성을 높이고 코드의 중복성을 낮춤
### 1-1. waiting-reservation (루트 프로젝트)
1. settings.gradle
```
rootProject.name = 'waiting-reservation'
include 'common'
include 'eureka-server'
include 'gateway'
include 'config'
include 'auth'
include 'reservation'
include 'restaurant'
include 'queue'
include 'notification'
include 'payment'
```
2. build.gradle
```
plugins {
	id 'java'
	id 'org.springframework.boot' version '3.3.7'
	id 'io.spring.dependency-management' version '1.1.7'
}

subprojects {

	apply plugin: 'java'
	apply plugin: 'java-library'      
	apply plugin: 'org.springframework.boot'
	apply plugin: 'io.spring.dependency-management'

	group = 'com.bobjool'
	version = '0.0.1-SNAPSHOT'

	java {
		toolchain {
			languageVersion = JavaLanguageVersion.of(17)
		}
	}

	repositories {
		mavenCentral()
	}

	ext {
		set('springCloudVersion', "2023.0.4")
	}

	dependencies {
		implementation 'org.springframework.boot:spring-boot-starter-web'
		testImplementation 'org.springframework.boot:spring-boot-starter-test'
		compileOnly 'org.projectlombok:lombok'
		annotationProcessor 'org.projectlombok:lombok'
		testImplementation 'org.springframework.boot:spring-boot-starter-test'
		testRuntimeOnly 'org.junit.platform:junit-platform-launcher'

		//test lombok 사용
		testCompileOnly 'org.projectlombok:lombok'
		testAnnotationProcessor 'org.projectlombok:lombok'

		// test 시 h2 데이터베이스 사용
		testRuntimeOnly 'com.h2database:h2'
	}

	dependencyManagement {
		imports {
			mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
		}
	}

	tasks.named('test') {
		useJUnitPlatform()
	}

	tasks.withType(JavaCompile).configureEach {
		options.compilerArgs.add("-parameters")
	}

	bootJar.enabled = false
	jar.enabled = false
}
```
- 루트 프로젝트의 `build.gradle`은 하위 모듈의 공통 설정을 정한다.
- `spring-web` 과 `lombok`은 모든 하위 모듈에서 사용 가능하도록 설정

### 1-2. common 모듈 (공통 모듈)
- 공통으로 사용되는 코드, 설정들을 공통 모듈(common)에 따로 두어 관리
- 유지보수성을 높이고 코드의 중복성을 낮춤
1. build.gradle
```
jar.enabled = true

ext {
    set('querydslVersion', "5.0.0") 
}

dependencies {
    api 'org.springframework.boot:spring-boot-starter-data-jpa'

    // queryDsl
    implementation "com.querydsl:querydsl-jpa:${querydslVersion}:jakarta"
    annotationProcessor "com.querydsl:querydsl-apt:${querydslVersion}:jakarta"
    annotationProcessor "jakarta.annotation:jakarta.annotation-api"
    annotationProcessor "jakarta.persistence:jakarta.persistence-api"
}

def querydslSrcDir = 'src/main/generated'
clean {
    delete file(querydslSrcDir)
}
```
![image](https://github.com/user-attachments/assets/37337571-c8b1-4f84-b87a-00c55e2f6806)
- `common` 모듈 안에는  아래 항목들이 있고, 모든 하위 모듈에서 공통으로 사용하고 있다.
    - `BaseEntity`
    - `GlobalExceptionHandler`
    - `ErrorCode`
    - `ApiResponse`  : 공통 응답 DTO

## 2. 예약 및 결제 도메인 구현
- 예약 요청 후, 10분 이내에 결제가 이루어지지 않으면 예약 자동 취소
    - **Redis Keyspace Notification, TTL** 설정을 통해 구현
### 2-1. 예약 성공
![image](https://github.com/user-attachments/assets/8ff82bce-fb7a-4144-ad38-ea0891cab4ce)
1. 예약 요청을 받으면, 
    1. 레스토랑에 스케줄을 예약하고(`feignClient`), `PENDING`상태의 예약을 생성한다.
    2. `reservation.created`토픽에 이벤트를 발행한다.
2. 결제는 `reservation.created` 토픽의 이벤트를 받아서, 레디스의 `payment:autoCancel:{reservationId}` 에 10분 후 만료가 되도록 해서 저장한다.
3. 사용자가 예약 요청을 한 후, 10분 이내에 결제를 요청해서 성공하면, 
    1. 레디스에서 데이터를 지우고 
    2. `payment.completed` 토픽에 이벤트를 발행한다.
4. 예약은 `payment.completed` 토픽의 이벤트를 받아서, 예약을 `COMPLETE` 상태로 바꾼 후,
    1. `reservation.completed` 토픽에 이벤트를 발행한다.
5. 알림은 `reservation.completed` 토픽의 이벤트를 받아서, 예약 성공 알림 메세지를 보낸다.
### 2-2. 예약 실패 케이스 - 결제 실패한 경우
![image](https://github.com/user-attachments/assets/0f63e6da-8f82-408b-8517-35fb9e38d816)
1. 예약 요청을 받으면, 
    1. 레스토랑에 스케줄을 예약하고(`feignClient`), `PENDING`상태의 예약을 생성한다.
    2. `reservation.created`토픽에 이벤트를 발행한다.
2. 결제는 `reservation.created` 토픽의 이벤트를 받아서, 레디스의 `payment:autoCancel:{reservationId}` 에 10분 후 만료가 되도록 해서 저장한다.
3. 사용자가 예약 요청을 한 후, 10분 이내에 결제를 요청했지만, 결제가 실패하면(**PG사 결제 실패**)
    1. 레디스에서 데이터를 지우고 
    2. `payment.failed` 토픽에 이벤트를 발행한다.
4. 예약은 `payment.failed` 토픽의 이벤트를 받아서, 예약을 `FAIL` 상태로 바꾼 후,
    1. `reservation.failed` 토픽에 이벤트를 발행한다.
5. 알림은 `reservation.failed` 토픽의 이벤트를 받아서, 예약 실패 알림 메세지를 보낸다.
### 2-3. 예약 실패 케이스 - 결제 타임 아웃(10분 만료)
![image](https://github.com/user-attachments/assets/cdc9c93a-436e-4c42-971b-2ac3711f2682)
1. 예약 요청을 받으면, 
    1. 레스토랑에 스케줄을 예약하고(`feignClient`), `PENDING`상태의 예약을 생성한다.
    2. `reservation.created`토픽에 이벤트를 발행한다.
2. 결제는 `reservation.created` 토픽의 이벤트를 받아서, 레디스의 `payment:autoCancel:{reservationId}` 에 10분 후 만료가 되도록 해서 저장한다.
3. 사용자가 예약 요청을 한 후, 10분 이내에 결제를 요청하지 않으면,
    1. 레디스의 `payment:autoCancel:{reservationId}` 의 데이터가 삭제가 되고,
    2. `ExpirationListener`가 데이터 만료를 감지하여, 
    3. `payment.timeout` 토픽에 이벤트를 발행한다.
4. 예약은 `payment.timeout` 토픽의 이벤트를 받아서, 예약을 `FAIL` 상태로 바꾼 후,
    1. `reservation.failed` 토픽에 이벤트를 발행한다.
5. 알림은 `reservation.failed` 토픽의 이벤트를 받아서, 예약 실패 알림 메세지를 보낸다.
### 2-4. 예약 실패 케이스 - 레스토랑 스케쥴 예약 실패
![image](https://github.com/user-attachments/assets/717da1ed-c237-4cf5-8f4f-d32beab9e36d)
1. 예약 요청을 받으면, 
2. 레스토랑에 스케줄을 예약(`feignClient`) 시도하지만, **동시성 이슈**로 레스토랑 예약이 실패하고
3. **“예약이 불가능합니다.”** 응답 메세지를 보낸다.
## 3. CI/CD 파이프라인 구축
### 3-1. pull request 발생시 테스트 코드 수행
1. run-test.yml
```
# Actions 이름 github 페이지에서 볼 수 있다.
name: Run Test

# Event Trigger 특정 액션 (Push, Pull_Request)등이 명시한 Branch에서 일어나면 동작을 수행한다.
on:
  push:
    # 배열로 여러 브랜치를 넣을 수 있다.
    branches: [ develop, feat/*, refactor/*, chore/* ]
  # github pull request 생성시
  pull_request:
    branches:
      - develop # -로 여러 브랜치를 명시하는 것도 가능

  # 실제 어떤 작업을 실행할지에 대한 명시
jobs:
  test:
    runs-on: ubuntu-latest
#    strategy:
#      matrix:
#        service: [product]  # 테스트할 서비스들
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 17
        uses: actions/setup-java@v2
        with:
          distribution: 'adopt'
          java-version: '17'
      - name: Run All Tests
        run: |
          chmod +x ./gradlew
          ./gradlew clean test
```
- 테스트가 실패하면 빨간불, 통과하면 초록불
![image](https://github.com/user-attachments/assets/3c0339c4-9bf9-44dc-8db8-8c737ec0bb27)
### 3-2. 각 서비스별 CD 워크플로우 분리
#### 1. 문제 상황

![image](https://github.com/user-attachments/assets/56c4b412-a7b3-4255-8d70-c564c466ab3b)
- 기존 CD 워크플로우에서는 Github Actions가 실행될 때,
- 모든 EC2 인스턴스에 배포가 진행되어 모든 서비스가 동시에 재시동되는 문제가 발생했다.
- 이는 각 서비스에 대한 코드 수정이 이루어지지 않았음에도 불구하고,
- 불필요한 작업이 발생하여 배포 효율성이 떨어지는 문제이다.
#### 2. 해결 방법

![image](https://github.com/user-attachments/assets/e9cdfdbd-b010-4ea0-8b0f-276d51c5f790)
- 이 문제를 해결하기 위해, 서비스별로 CD 워크플로우를 분리하는 방식을 도입했다. 
- 이를 통해, 각 서비스의 코드가 수정될 때에만 해당 서비스가 배포되도록 설정하였다.
- 총 4개의 cd.yml 파일로 분리했다.
    - `eureka-gateway-cd.yml`
    - `auth-restaurant-cd.yml`
    - `reservation-payment-cd.yml`
    - `queue-notification-cd.yml`

