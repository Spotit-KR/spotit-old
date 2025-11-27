# 옵션 B: 새 레포지토리 부트스트랩 가이드

## 목차
1. [개요](#1-개요)
2. [의사결정 기준](#2-의사결정-기준)
3. [프로젝트 초기화](#3-프로젝트-초기화)
4. [기본 아키텍처 설정](#4-기본-아키텍처-설정)
5. [1인 개발 + AI 에이전트 최적화](#5-1인-개발--ai-에이전트-최적화)
6. [개발 우선순위 및 로드맵](#6-개발-우선순위-및-로드맵)
7. [기존 프로젝트에서 참고할 것](#7-기존-프로젝트에서-참고할-것)
8. [데이터 마이그레이션 전략](#8-데이터-마이그레이션-전략)
9. [AI 에이전트 활용 전략](#9-ai-에이전트-활용-전략)

---

## 1. 개요

### 1.1 이 가이드를 선택하는 경우

✅ **다음 조건이 모두 해당되면 새 레포지토리를 추천합니다**:

- [ ] 기존 기능 중 **50% 미만** 유지
- [ ] 기존 데이터 연동이 **제한적** (선택적 마이그레이션 가능)
- [ ] 새로운 비즈니스 요구사항이 **대폭 변경**됨
- [ ] 깨끗한 코드베이스로 시작하고 싶음
- [ ] **1인 개발**로 빠른 프로토타이핑이 필요
- [ ] 기술 부채 없는 최신 기술 스택 원함

### 1.2 핵심 원칙 (1인 개발 + AI 에이전트 최적화)

```
🎯 생산성 최우선
   - 최소한의 보일러플레이트
   - AI 에이전트가 이해하기 쉬운 구조
   - 명확한 컨벤션

🚀 빠른 프로토타이핑
   - MVP 우선 개발
   - 핵심 기능부터 구현
   - 점진적 확장

🛡️ 검증된 패턴 재사용
   - 기존 프로젝트의 좋은 패턴 차용
   - 하지만 불필요한 복잡도는 제거

🤖 자동화 극대화
   - CI/CD 조기 구축
   - 코드 생성 자동화
   - 테스트 자동화
```

### 1.3 예상 효과

**기존 마이그레이션 (옵션 A) 대비**:
- ⏱️ **초기 개발 기간**: +20% (하지만 기술 부채 0)
- 🎯 **요구사항 적합도**: +40% (새 요구사항에 최적화)
- 🐛 **버그 발생률**: -50% (검증된 패턴 + 깨끗한 시작)
- 🚀 **장기 유지보수성**: +70% (기술 부채 없음)
- 🧠 **AI 에이전트 생산성**: +50% (명확한 구조)

---

## 2. 의사결정 기준

### 2.1 기능 변경 분석 템플릿

기획서를 받으면 다음 표를 작성하세요:

| 기존 기능 | 신규 기획 | 재사용 가능? | 변경 정도 | 우선순위 |
|-----------|-----------|--------------|-----------|----------|
| 대기 신청 | 예약 시스템 | 부분 (도메인 개념) | 70% 변경 | P0 (MVP) |
| 대기 번호 발급 | QR 코드 체크인 | 불가 | 100% 변경 | P0 (MVP) |
| SSE 실시간 알림 | WebSocket 푸시 | 불가 | 100% 변경 | P1 |
| Email 알림 | Email + SMS | 가능 | 30% 변경 | P1 |
| 팝업 검색 | AI 추천 + 검색 | 부분 (검색만) | 80% 변경 | P0 (MVP) |
| OAuth 로그인 (Kakao) | OAuth (Kakao, Google) | 가능 | 20% 변경 | P0 (MVP) |
| 관리자 대시보드 | 없음 | 불필요 | - | - |
| 제재 시스템 | 없음 | 불필요 | - | - |

**분석**:
- 재사용 가능: 2개 (25%)
- 부분 재사용: 2개 (25%)
- 완전 변경: 3개 (37.5%)
- 불필요: 2개 (12.5%)

**결론**: 재사용률 **25%** → **새 레포지토리 추천**

### 2.2 데이터 마이그레이션 필요성

| 데이터 | 이관 필요? | 방법 | 복잡도 |
|--------|-----------|------|--------|
| 회원 (Member) | Yes | SQL export/import | 낮음 |
| OAuth 연동 정보 | Yes | SQL export/import | 낮음 |
| 팝업 기본 정보 | Yes | API 동기화 | 중간 |
| 대기 이력 | No (참고용만) | Read-only 조회 | 낮음 |
| 알림 이력 | No | 불필요 | - |
| 제재 이력 | No | 불필요 | - |

**결론**: 핵심 데이터만 이관 (회원, OAuth) → **새 레포지토리 가능**

---

## 3. 프로젝트 초기화

### 3.1 프로젝트 생성 (Spring Initializr)

**방법 1: Web UI**
- https://start.spring.io/
- Project: **Gradle - Kotlin**
- Language: **Kotlin**
- Spring Boot: **3.4.6**
- Java: **21**
- Packaging: **Jar**
- Group: `com.spotit`
- Artifact: `backend`
- Dependencies:
  - Spring Web
  - Spring Data JPA
  - Spring Security
  - OAuth2 Client
  - Validation
  - PostgreSQL Driver
  - H2 Database (test/dev)
  - Spring Boot Actuator

**방법 2: CLI (추천 - AI 에이전트 친화적)**

```bash
# 새 디렉토리 생성
mkdir spotit-v2
cd spotit-v2

# Spring Initializr CLI
curl https://start.spring.io/starter.zip \
  -d type=gradle-project-kotlin \
  -d language=kotlin \
  -d bootVersion=3.4.6 \
  -d baseDir=backend \
  -d groupId=com.spotit \
  -d artifactId=backend \
  -d name=spotit-backend \
  -d packageName=com.spotit.backend \
  -d javaVersion=21 \
  -d dependencies=web,data-jpa,security,oauth2-client,validation,postgresql,h2,actuator \
  -o backend.zip

unzip backend.zip
rm backend.zip
cd backend
```

### 3.2 디렉토리 구조 초기 설정

```bash
mkdir -p src/main/kotlin/com/spotit/backend/{domain,application,infrastructure,presentation,common}
mkdir -p src/main/kotlin/com/spotit/backend/domain/{model,port}
mkdir -p src/main/kotlin/com/spotit/backend/application/{service,dto,mapper}
mkdir -p src/main/kotlin/com/spotit/backend/infrastructure/{persistence,external}
mkdir -p src/main/kotlin/com/spotit/backend/infrastructure/persistence/{adapter,entity,mapper,repository}
mkdir -p src/main/kotlin/com/spotit/backend/presentation/controller
mkdir -p src/main/kotlin/com/spotit/backend/common/{exception,security,config}
mkdir -p src/test/kotlin/com/spotit/backend
mkdir -p docs/{architecture,api,business-logic}
```

**최종 구조**:
```
backend/
├── src/
│   ├── main/
│   │   ├── kotlin/com/spotit/backend/
│   │   │   ├── domain/           # 순수 비즈니스 로직
│   │   │   │   ├── model/        # 도메인 모델
│   │   │   │   └── port/         # 포트 인터페이스
│   │   │   ├── application/      # 유스케이스
│   │   │   │   ├── service/
│   │   │   │   ├── dto/
│   │   │   │   └── mapper/
│   │   │   ├── infrastructure/   # 어댑터
│   │   │   │   ├── persistence/
│   │   │   │   └── external/
│   │   │   ├── presentation/     # REST API
│   │   │   │   └── controller/
│   │   │   └── common/           # 공통 모듈
│   │   └── resources/
│   └── test/
├── docs/
│   ├── architecture/
│   ├── api/
│   └── business-logic/
├── build.gradle.kts
└── settings.gradle.kts
```

### 3.3 Gradle 설정 (build.gradle.kts)

```kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
    kotlin("jvm") version "2.1.0"
    kotlin("plugin.spring") version "2.1.0"
    kotlin("plugin.jpa") version "2.1.0"
    kotlin("kapt") version "2.1.0"
    id("org.springframework.boot") version "3.4.6"
    id("io.spring.dependency-management") version "1.1.7"
}

group = "com.spotit"
version = "0.0.1-SNAPSHOT"

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

repositories {
    mavenCentral()
}

dependencies {
    // Kotlin
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib")
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")

    // Spring Boot
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    implementation("org.springframework.boot:spring-boot-starter-security")
    implementation("org.springframework.boot:spring-boot-starter-oauth2-client")
    implementation("org.springframework.boot:spring-boot-starter-actuator")

    // Database
    runtimeOnly("org.postgresql:postgresql")
    runtimeOnly("com.h2database:h2")

    // QueryDSL
    implementation("com.querydsl:querydsl-jpa:5.1.0:jakarta")
    kapt("com.querydsl:querydsl-apt:5.1.0:jakarta")
    kapt("org.springframework.boot:spring-boot-configuration-processor")

    // JWT
    implementation("io.jsonwebtoken:jjwt-api:0.12.6")
    runtimeOnly("io.jsonwebtoken:jjwt-impl:0.12.6")
    runtimeOnly("io.jsonwebtoken:jjwt-jackson:0.12.6")

    // API Documentation
    implementation("org.springdoc:springdoc-openapi-starter-webmvc-ui:2.8.9")

    // Monitoring
    runtimeOnly("io.micrometer:micrometer-registry-prometheus")

    // Test
    testImplementation("org.springframework.boot:spring-boot-starter-test") {
        exclude(group = "org.junit.vintage", module = "junit-vintage-engine")
    }
    testImplementation("org.springframework.security:spring-security-test")
    testImplementation("io.kotest:kotest-runner-junit5:5.8.0")
    testImplementation("io.kotest:kotest-assertions-core:5.8.0")
    testImplementation("io.mockk:mockk:1.13.9")
}

kotlin {
    compilerOptions {
        freeCompilerArgs.addAll("-Xjsr305=strict")
        jvmTarget.set(org.jetbrains.kotlin.gradle.dsl.JvmTarget.JVM_21)
    }
}

allOpen {
    annotation("jakarta.persistence.Entity")
    annotation("jakarta.persistence.MappedSuperclass")
    annotation("jakarta.persistence.Embeddable")
}

tasks.withType<KotlinCompile> {
    kotlinOptions {
        freeCompilerArgs += "-Xjsr305=strict"
        jvmTarget = "21"
    }
}

tasks.withType<Test> {
    useJUnitPlatform()
}
```

### 3.4 application.yml 기본 설정

```yaml
spring:
  application:
    name: spotit-backend

  profiles:
    active: local

  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password:

  jpa:
    hibernate:
      ddl-auto: create-drop
    properties:
      hibernate:
        format_sql: true
        default_batch_fetch_size: 100
    show-sql: true

  h2:
    console:
      enabled: true

springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui.html
    operationsSorter: method

logging:
  level:
    com.spotit: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
```

---

## 4. 기본 아키텍처 설정

### 4.1 예외 처리 시스템 (공통 모듈 우선)

기존 프로젝트의 훌륭한 예외 처리 시스템을 Kotlin으로 재구현:

**ErrorType.kt**
```kotlin
package com.spotit.backend.common.exception

import org.springframework.http.HttpStatus

enum class ErrorType(
    val httpStatus: HttpStatus,
    val code: String,
    val message: String
) {
    // Common
    INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "INTERNAL_SERVER_ERROR", "서버 내부 오류가 발생했습니다"),
    INVALID_INPUT(HttpStatus.BAD_REQUEST, "INVALID_INPUT", "잘못된 입력입니다"),

    // Auth
    AUTHENTICATION_REQUIRED(HttpStatus.UNAUTHORIZED, "AUTHENTICATION_REQUIRED", "인증이 필요합니다"),
    INVALID_TOKEN(HttpStatus.UNAUTHORIZED, "INVALID_TOKEN", "유효하지 않은 토큰입니다"),

    // Member
    MEMBER_NOT_FOUND(HttpStatus.NOT_FOUND, "MEMBER_NOT_FOUND", "회원을 찾을 수 없습니다"),

    // Popup
    POPUP_NOT_FOUND(HttpStatus.NOT_FOUND, "POPUP_NOT_FOUND", "팝업을 찾을 수 없습니다"),

    // TODO: 기획에 따라 에러 타입 추가
}
```

**BusinessException.kt**
```kotlin
package com.spotit.backend.common.exception

class BusinessException(
    val errorType: ErrorType,
    val additionalInfo: String? = null
) : RuntimeException(errorType.message)
```

**GlobalExceptionHandler.kt**
```kotlin
package com.spotit.backend.common.exception

import org.springframework.http.ResponseEntity
import org.springframework.web.bind.annotation.ExceptionHandler
import org.springframework.web.bind.annotation.RestControllerAdvice
import java.time.LocalDateTime

@RestControllerAdvice
class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException::class)
    fun handleBusinessException(e: BusinessException): ResponseEntity<ErrorResponse> {
        val response = ErrorResponse(
            code = e.errorType.code,
            message = e.errorType.message,
            additionalInfo = e.additionalInfo,
            timestamp = LocalDateTime.now()
        )
        return ResponseEntity.status(e.errorType.httpStatus).body(response)
    }
}

data class ErrorResponse(
    val code: String,
    val message: String,
    val additionalInfo: String?,
    val timestamp: LocalDateTime
)
```

### 4.2 Security 기본 설정

**SecurityConfig.kt**
```kotlin
package com.spotit.backend.common.security

import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import org.springframework.security.config.annotation.web.builders.HttpSecurity
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity
import org.springframework.security.config.http.SessionCreationPolicy
import org.springframework.security.web.SecurityFilterChain

@Configuration
@EnableWebSecurity
class SecurityConfig {

    @Bean
    fun filterChain(http: HttpSecurity): SecurityFilterChain {
        http
            .csrf { it.disable() }
            .sessionManagement { it.sessionCreationPolicy(SessionCreationPolicy.STATELESS) }
            .authorizeHttpRequests { auth ->
                auth
                    .requestMatchers("/api/auth/**", "/api/public/**").permitAll()
                    .requestMatchers("/swagger-ui/**", "/api-docs/**").permitAll()
                    .requestMatchers("/actuator/health").permitAll()
                    .anyRequest().authenticated()
            }

        return http.build()
    }
}
```

### 4.3 헥사고날 아키텍처 템플릿

**도메인 레이어 예시** (새 요구사항에 맞게 수정 필요):

```kotlin
package com.spotit.backend.domain.model

import com.spotit.backend.common.exception.BusinessException
import com.spotit.backend.common.exception.ErrorType
import java.time.LocalDateTime

// 예시: 예약 도메인 (기존 대기 → 예약 시스템으로 변경되었다고 가정)
data class Reservation(
    val id: Long?,
    val popup: Popup,
    val member: Member,
    val reservationCode: String,  // QR 코드용 고유 코드
    val reservedDateTime: LocalDateTime,
    val peopleCount: Int,
    val status: ReservationStatus,
    val createdAt: LocalDateTime,
    val checkedInAt: LocalDateTime? = null
) {
    init {
        require(peopleCount in 1..6) {
            throw BusinessException(ErrorType.INVALID_INPUT, "인원수는 1~6명이어야 합니다")
        }
    }

    fun checkIn(): Reservation {
        require(status == ReservationStatus.CONFIRMED) {
            throw BusinessException(ErrorType.INVALID_INPUT, "확정된 예약만 체크인 가능합니다")
        }
        return copy(
            status = ReservationStatus.CHECKED_IN,
            checkedInAt = LocalDateTime.now()
        )
    }

    fun cancel(): Reservation {
        require(status == ReservationStatus.CONFIRMED) {
            throw BusinessException(ErrorType.INVALID_INPUT, "확정된 예약만 취소 가능합니다")
        }
        return copy(status = ReservationStatus.CANCELLED)
    }
}

enum class ReservationStatus {
    CONFIRMED,      // 예약 확정
    CHECKED_IN,     // 체크인 완료
    CANCELLED,      // 취소
    NO_SHOW         // 노쇼
}
```

**포트 인터페이스**:
```kotlin
package com.spotit.backend.domain.port

import com.spotit.backend.domain.model.Reservation

interface ReservationPort {
    fun save(reservation: Reservation): Reservation
    fun findById(id: Long): Reservation?
    fun findByReservationCode(code: String): Reservation?
    fun findByMemberId(memberId: Long): List<Reservation>
}
```

---

## 5. 1인 개발 + AI 에이전트 최적화

### 5.1 AI 에이전트 친화적 코드 스타일

**원칙**:
1. **명확한 네이밍** - AI가 의도를 쉽게 파악
2. **작은 함수** - 한 함수는 하나의 책임만
3. **주석 최소화** - 코드 자체로 설명
4. **일관된 패턴** - 동일한 패턴 반복 사용

**예시**:

```kotlin
// ❌ 나쁜 예 - 복잡하고 AI가 이해하기 어려움
fun processReservation(id: Long, action: String, data: Map<String, Any>): Any {
    return when (action) {
        "checkin" -> { /* 복잡한 로직 */ }
        "cancel" -> { /* 복잡한 로직 */ }
        else -> throw Exception()
    }
}

// ✅ 좋은 예 - 명확하고 AI가 쉽게 이해
fun checkInReservation(reservationId: Long): Reservation {
    val reservation = reservationPort.findById(reservationId)
        ?: throw BusinessException(ErrorType.RESERVATION_NOT_FOUND)

    val checkedIn = reservation.checkIn()
    return reservationPort.save(checkedIn)
}

fun cancelReservation(reservationId: Long): Reservation {
    val reservation = reservationPort.findById(reservationId)
        ?: throw BusinessException(ErrorType.RESERVATION_NOT_FOUND)

    val cancelled = reservation.cancel()
    return reservationPort.save(cancelled)
}
```

### 5.2 코드 생성 템플릿 (AI 프롬프트용)

**프롬프트 예시 1: 새 도메인 생성**
```
다음 요구사항으로 도메인 모델을 생성해줘:

도메인: Review (리뷰)
필드:
- id: Long?
- popup: Popup
- member: Member
- rating: Int (1~5)
- content: String (1~500자)
- images: List<String> (최대 5개)
- createdAt: LocalDateTime

비즈니스 규칙:
- rating은 1~5만 가능
- content는 1~500자
- images는 최대 5개

헥사고날 아키텍처로 다음을 생성:
1. domain/model/Review.kt
2. domain/port/ReviewPort.kt
3. infrastructure/persistence/entity/ReviewEntity.kt
4. infrastructure/persistence/adapter/ReviewPortAdapter.kt
5. application/service/ReviewService.kt
6. application/dto/ReviewCreateRequest.kt
7. presentation/controller/ReviewController.kt
```

**프롬프트 예시 2: CRUD 자동 생성**
```
Reservation 도메인에 대한 CRUD API를 생성해줘:

엔드포인트:
- POST /api/reservations - 예약 생성
- GET /api/reservations/{id} - 예약 조회
- GET /api/reservations - 내 예약 목록 조회
- DELETE /api/reservations/{id} - 예약 취소

인증: JWT 토큰 필요 (POST, DELETE)

기존 프로젝트 패턴 참고:
- Service: @Transactional
- Controller: ResponseEntity<ApiResponse<T>>
- DTO: validation 어노테이션
```

### 5.3 개발 자동화 스크립트

**scaffold.sh** - 도메인 스캐폴딩 스크립트
```bash
#!/bin/bash
# scaffold.sh - 새 도메인 디렉토리 구조 자동 생성

DOMAIN_NAME=$1

if [ -z "$DOMAIN_NAME" ]; then
    echo "Usage: ./scaffold.sh <domain-name>"
    exit 1
fi

BASE_PATH="src/main/kotlin/com/spotit/backend"

# 디렉토리 생성
mkdir -p "$BASE_PATH/domain/model/$DOMAIN_NAME"
mkdir -p "$BASE_PATH/domain/port"
mkdir -p "$BASE_PATH/application/service"
mkdir -p "$BASE_PATH/application/dto/$DOMAIN_NAME"
mkdir -p "$BASE_PATH/infrastructure/persistence/entity/$DOMAIN_NAME"
mkdir -p "$BASE_PATH/infrastructure/persistence/adapter"
mkdir -p "$BASE_PATH/presentation/controller"

echo "✅ Scaffolded domain: $DOMAIN_NAME"
echo "📁 Created directories:"
echo "  - domain/model/$DOMAIN_NAME"
echo "  - application/dto/$DOMAIN_NAME"
echo "  - infrastructure/persistence/entity/$DOMAIN_NAME"
```

**사용법**:
```bash
chmod +x scaffold.sh
./scaffold.sh review      # Review 도메인 구조 생성
./scaffold.sh notification # Notification 도메인 구조 생성
```

### 5.4 AI 에이전트 워크플로우

```
1. 기획 분석
   ↓
   [AI에게 기획서 요약 요청]
   "이 기획서를 분석해서 필요한 도메인, API 엔드포인트, 우선순위를 정리해줘"

2. 도메인 모델 설계
   ↓
   [AI에게 도메인 생성 요청]
   "Reservation 도메인을 헥사고날 아키텍처로 생성해줘 (위 프롬프트 사용)"

3. API 구현
   ↓
   [AI에게 CRUD 생성 요청]
   "Reservation CRUD API를 생성해줘"

4. 테스트 작성
   ↓
   [AI에게 테스트 생성 요청]
   "ReservationService 테스트를 작성해줘 (Given-When-Then 패턴)"

5. 문서화
   ↓
   [AI에게 문서 생성 요청]
   "Reservation API 문서를 작성해줘 (Swagger 어노테이션 포함)"
```

---

## 6. 개발 우선순위 및 로드맵

### 6.1 MVP (Minimum Viable Product) 정의

**Phase 0: 인프라 (1-2일)**
- [x] 프로젝트 초기화
- [ ] 공통 모듈 (예외, Security)
- [ ] CI/CD 파이프라인
- [ ] 로컬 개발 환경 (Docker Compose)

**Phase 1: 인증/인가 (2-3일)**
- [ ] OAuth2 로그인 (Kakao, Google)
- [ ] JWT 발급/검증
- [ ] Member 도메인
- [ ] API: POST /api/auth/login, GET /api/members/me

**Phase 2: 핵심 도메인 (3-5일)**
기획에 따라 달라지지만, 예시:
- [ ] Popup 도메인 (팝업 정보)
- [ ] Reservation 도메인 (예약 시스템)
- [ ] API: POST /api/reservations, GET /api/reservations/{id}

**Phase 3: 검색/추천 (3-4일)**
- [ ] Popup 검색 (키워드, 카테고리, 위치)
- [ ] AI 추천 (외부 API 연동 또는 간단한 알고리즘)
- [ ] API: GET /api/popups/search, GET /api/popups/recommendations

**Phase 4: 알림 (2-3일)**
- [ ] Notification 도메인
- [ ] WebSocket/SSE 실시간 알림
- [ ] Email/SMS 알림
- [ ] API: GET /api/notifications, POST /api/notifications/read

**Phase 5: 관리 기능 (2-3일)**
- [ ] 내 예약 내역 조회
- [ ] 예약 취소
- [ ] 리뷰 작성 (있는 경우)
- [ ] API: GET /api/reservations, DELETE /api/reservations/{id}

**총 예상 기간**: 13-20일 (1인 기준, AI 에이전트 활용 시)

### 6.2 개발 우선순위 매트릭스

```
           중요도
           ↑
           |
    P0     |  P1
  (MVP)    | (중요)
-----------|----------→ 긴급도
    P2     |  P3
  (나중에) | (불필요)
           |
```

**P0 (MVP - 최우선)**:
- 인증/인가
- 핵심 예약 기능
- 팝업 기본 CRUD
- 검색 (기본)

**P1 (중요 - MVP 이후)**:
- 실시간 알림
- AI 추천
- 리뷰 시스템

**P2 (나중에)**:
- 관리자 대시보드
- 통계/분석
- 고급 검색 필터

**P3 (불필요/보류)**:
- 기존 프로젝트의 제재 시스템 (기획에 없으면)
- 복잡한 대기열 관리 (예약제로 변경 시)

---

## 7. 기존 프로젝트에서 참고할 것

### 7.1 재사용 가능한 패턴

**✅ 꼭 가져올 것**:

1. **예외 처리 시스템**
   - ErrorType enum (60+ 에러 타입)
   - BusinessException
   - GlobalExceptionHandler
   - 통일된 ErrorResponse 형식

2. **헥사고날 아키텍처 구조**
   - Domain / Application / Infrastructure / Presentation 레이어
   - Port & Adapter 패턴
   - 도메인 모델의 순수성 (JPA 어노테이션 배제)

3. **Security 구조**
   - JWT 발급/검증 로직
   - OAuth2 로그인 플로우
   - UserPrincipal 패턴

4. **QueryDSL 활용 패턴**
   - 동적 쿼리 작성
   - N+1 문제 해결 (batch fetch size)

5. **API 응답 포맷**
   ```kotlin
   data class ApiResponse<T>(
       val message: String,
       val data: T
   )
   ```

6. **테스트 패턴**
   - Given-When-Then 구조
   - MockBean 활용
   - 도메인 로직 단위 테스트

### 7.2 참고만 하고 수정할 것

**⚠️ 수정 필요**:

1. **도메인 모델**
   - Waiting → Reservation (또는 새 요구사항에 맞게)
   - 비즈니스 규칙 재검토
   - 새 기획에 맞게 필드 조정

2. **알림 시스템**
   - SSE → WebSocket (더 나은 실시간 경험)
   - 하이브리드 알림 (즉시 + 스케줄) → 이벤트 기반으로 단순화

3. **Sealed Class 쿼리 패턴**
   - 좋은 패턴이지만, 복잡도 검토 후 선택적 사용

### 7.3 버릴 것

**❌ 가져오지 않을 것**:

1. **제재 시스템** (Ban) - 새 기획에 없으면 불필요
2. **복잡한 대기 번호 관리** - 예약제로 단순화
3. **관리자 세션 토큰** - JWT 통일
4. **노쇼 2회 제한** - 새 요구사항 확인 후 결정

---

## 8. 데이터 마이그레이션 전략

### 8.1 마이그레이션 필요 데이터 분석

**필수 마이그레이션**:
- Member (회원 기본 정보)
- OAuth 연동 정보 (Kakao ID 등)

**선택적 마이그레이션**:
- Popup 기본 정보 (재사용 가능한 경우)

**마이그레이션 불필요**:
- Waiting 이력 (새 시스템과 호환 안 됨)
- Notification 이력
- Ban 이력

### 8.2 마이그레이션 방법

**방법 1: SQL Export/Import (단순 데이터)**

```sql
-- 기존 DB에서 Export
SELECT id, email, nickname, profile_image_url, provider, provider_id, created_at
FROM member
WHERE deleted_at IS NULL;

-- CSV 저장 후 새 DB로 Import
COPY member(id, email, nickname, profile_image_url, provider, provider_id, created_at)
FROM '/path/to/member.csv'
DELIMITER ','
CSV HEADER;
```

**방법 2: API 동기화 (복잡한 데이터)**

```kotlin
// 마이그레이션 전용 API (일회성)
@RestController
@RequestMapping("/api/migration")
class MigrationController(
    private val memberService: MemberService
) {

    @PostMapping("/members")
    fun migrateMembers(@RequestBody members: List<MemberMigrationDto>) {
        members.forEach { memberService.createMemberFromMigration(it) }
    }
}
```

**방법 3: 하이브리드 (추천)**
- 회원 데이터: SQL Export/Import (빠름)
- 팝업 데이터: API 동기화 (검증 가능)

### 8.3 마이그레이션 체크리스트

- [ ] 기존 DB 백업
- [ ] 마이그레이션 스크립트 작성
- [ ] 테스트 환경에서 마이그레이션 실행
- [ ] 데이터 정합성 검증 (count, sample 조회)
- [ ] 롤백 계획 수립
- [ ] 프로덕션 마이그레이션 실행
- [ ] 새 시스템에서 데이터 확인

---

## 9. AI 에이전트 활용 전략

### 9.1 효과적인 AI 프롬프트 패턴

**패턴 1: 컨텍스트 제공**
```
나는 팝업 스토어 예약 시스템을 개발 중이야.
헥사고날 아키텍처를 사용하고, Kotlin + Spring Boot 3.4.6이야.

다음 요구사항으로 Reservation 도메인을 생성해줘:
[요구사항]

기존 프로젝트의 Waiting 도메인을 참고해:
[기존 코드 붙여넣기]
```

**패턴 2: 단계별 요청**
```
1단계: Reservation 도메인 모델만 먼저 생성해줘
(AI 응답 확인)

2단계: 이제 ReservationPort 인터페이스를 생성해줘
(AI 응답 확인)

3단계: ReservationPortAdapter를 생성해줘 (QueryDSL 사용)
```

**패턴 3: 예시 기반 요청**
```
다음과 같은 형식으로 ReviewService를 생성해줘:

[WaitingService.kt 일부 붙여넣기]

동일한 패턴으로:
- @Transactional 사용
- Port를 통한 데이터 접근
- BusinessException 던지기
- DTO Mapper 사용
```

### 9.2 코드 리뷰 자동화

**AI에게 코드 리뷰 요청**:
```
다음 코드를 리뷰해줘. 체크 포인트:
1. 헥사고날 아키텍처 원칙 준수?
2. Null safety 제대로 사용?
3. 비즈니스 로직이 도메인 레이어에 있나?
4. 테스트 가능한 구조?
5. 성능 이슈 (N+1 등)?

[코드 붙여넣기]
```

### 9.3 문서 자동 생성

**AI에게 문서 생성 요청**:
```
다음 API들에 대한 문서를 작성해줘:

형식:
- HTTP Method, Path
- Request Body (예시 JSON)
- Response Body (예시 JSON)
- 에러 케이스

API 목록:
- POST /api/reservations
- GET /api/reservations/{id}
- DELETE /api/reservations/{id}

[Controller 코드 붙여넣기]
```

### 9.4 AI 에이전트 한계 인지

**AI가 잘하는 것**:
- ✅ 반복적인 CRUD 코드 생성
- ✅ 보일러플레이트 작성
- ✅ 테스트 코드 스캐폴딩
- ✅ 리팩토링 제안
- ✅ 문서 작성

**AI가 못하는 것** (사람이 직접):
- ❌ 비즈니스 요구사항 분석
- ❌ 아키텍처 중요 결정
- ❌ 성능 최적화 (프로파일링 필요)
- ❌ 복잡한 비즈니스 로직 설계
- ❌ 보안 취약점 최종 검증

---

## 10. 빠른 시작 가이드 (Quick Start)

### 10.1 1일차: 프로젝트 초기화

```bash
# 1. 프로젝트 생성
curl https://start.spring.io/starter.zip \
  -d type=gradle-project-kotlin \
  -d language=kotlin \
  -d bootVersion=3.4.6 \
  -d dependencies=web,data-jpa,security,oauth2-client,validation,postgresql,h2,actuator \
  -o backend.zip
unzip backend.zip

# 2. 디렉토리 구조 생성
mkdir -p src/main/kotlin/com/spotit/backend/{domain,application,infrastructure,presentation,common}
# (상세 명령어는 3.2 참고)

# 3. 공통 모듈 생성 (AI 활용)
# ErrorType, BusinessException, GlobalExceptionHandler 생성

# 4. 첫 빌드 및 실행
./gradlew bootRun
```

### 10.2 2-3일차: 인증/인가

```bash
# 1. Member 도메인 생성
./scaffold.sh member

# 2. AI에게 요청
"Member 도메인을 생성해줘:
- id, email, nickname, profileImageUrl, provider, providerId
- OAuth2 로그인 지원 (Kakao)"

# 3. JWT 설정
# (기존 프로젝트의 JWT 코드 참고)

# 4. 테스트
curl -X POST http://localhost:8080/api/auth/login
```

### 10.3 4-5일차: 핵심 도메인

```bash
# 1. Popup 도메인 생성
./scaffold.sh popup

# 2. Reservation 도메인 생성
./scaffold.sh reservation

# 3. CRUD API 구현 (AI 활용)

# 4. 통합 테스트
./gradlew test
```

### 10.4 1주일 차: MVP 완성

- [x] 인증/인가
- [x] Popup CRUD
- [x] Reservation CRUD
- [x] 기본 검색
- [ ] 배포 (Docker + AWS/GCP)

---

## 11. 체크리스트

### 11.1 프로젝트 시작 전

- [ ] 기획서 분석 완료
- [ ] 기능 변경 범위 확인 (50% 미만?)
- [ ] 데이터 마이그레이션 범위 확인
- [ ] MVP 범위 정의
- [ ] 우선순위 설정 (P0, P1, P2, P3)

### 11.2 개발 중

- [ ] 헥사고날 아키텍처 준수
- [ ] 도메인 모델 순수성 유지
- [ ] 테스트 작성 (최소 서비스 레이어)
- [ ] API 문서 작성 (Swagger)
- [ ] 에러 처리 일관성

### 11.3 배포 전

- [ ] 전체 테스트 통과
- [ ] 통합 테스트 통과
- [ ] 성능 테스트 (기본 부하 테스트)
- [ ] 보안 검토 (OWASP Top 10)
- [ ] 문서화 완료
- [ ] CI/CD 파이프라인 구축
- [ ] 모니터링 설정 (Actuator, Prometheus)

---

## 12. 결론

### 12.1 요약

**새 레포지토리를 선택하는 이유**:
- 기존 기능 유지율 < 50%
- 기존 데이터 연동 제한적
- 깨끗한 코드베이스
- 1인 개발 + AI 에이전트 최적화

**핵심 원칙**:
- 생산성 최우선 (최소 보일러플레이트)
- MVP 빠른 프로토타이핑
- 검증된 패턴 재사용 (헥사고날 아키텍처)
- AI 에이전트 적극 활용

**예상 일정**: 13-20일 (MVP 기준, 1인 개발)

### 12.2 성공 요인

1. **명확한 우선순위** - P0 (MVP) 집중
2. **작은 단위 개발** - 도메인별 완성 후 다음 단계
3. **AI 에이전트 활용** - 반복 작업 자동화
4. **지속적 검증** - 작은 단위로 테스트하며 개발
5. **기존 패턴 재사용** - 바퀴를 재발명하지 않기

---

**문서 버전**: 1.0
**작성일**: 2025-11-27
**대상**: 1인 개발자 + AI 에이전트
**예상 소요 시간**: 13-20일 (MVP)
