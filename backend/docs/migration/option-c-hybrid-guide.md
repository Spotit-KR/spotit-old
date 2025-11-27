# 옵션 C: 하이브리드 접근 가이드

## 목차
1. [개요](#1-개요)
2. [의사결정 기준](#2-의사결정-기준)
3. [재사용 가치 평가](#3-재사용-가치-평가)
4. [선택적 마이그레이션 전략](#4-선택적-마이그레이션-전략)
5. [단계별 실행 계획](#5-단계별-실행-계획)
6. [재사용 코드 변환 가이드](#6-재사용-코드-변환-가이드)
7. [1인 개발 최적화](#7-1인-개발-최적화)
8. [체크리스트](#8-체크리스트)

---

## 1. 개요

### 1.1 하이브리드 접근이란?

**정의**: 새 레포지토리에서 시작하되, **기존 코드에서 재사용 가치가 높은 부분만** 선택적으로 Kotlin으로 변환하여 복사

```
기존 프로젝트 (Java)
    ↓ 선택적 복사 + Kotlin 변환
새 프로젝트 (Kotlin)
    ↓ 새 요구사항 추가
MVP 완성
```

**옵션 B와의 차이**:
- 옵션 B: 100% 새로 작성
- **옵션 C**: 좋은 코드는 재사용 (30-40%), 나머지 새로 작성

### 1.2 이 접근을 선택하는 경우

✅ **다음 조건이 해당되면 하이브리드를 추천합니다**:

- [ ] 기존 기능 중 **30-50%** 재사용 가능
- [ ] 기존 아키텍처가 **우수**함 (헥사고날)
- [ ] 기존 **공통 모듈**이 잘 설계됨 (예외 처리, Security 등)
- [ ] 일부 **도메인 개념**은 유지됨 (팝업, 회원 등)
- [ ] 기존 데이터 일부 마이그레이션 필요
- [ ] **개발 시간 단축** 원함 (옵션 B보다 20-30% 빠름)
- [ ] 1인 개발로 **검증된 코드**를 활용하고 싶음

### 1.3 장단점

**장점**:
- ✅ 검증된 코드 재사용 → 버그 감소
- ✅ 개발 시간 단축 (옵션 B 대비 20-30%)
- ✅ 좋은 패턴 유지 (헥사고날 아키텍처 등)
- ✅ 불필요한 코드는 배제 → 깨끗한 시작
- ✅ 마이그레이션보다 리스크 낮음

**단점**:
- ⚠️ 코드 선별 작업 필요 (판단 시간)
- ⚠️ 일부 레거시 패턴 유입 가능성
- ⚠️ 변환 과정에서 버그 유입 가능

**예상 일정**:
- 옵션 A (전체 마이그레이션): 54-78시간
- 옵션 B (새로 작성): 90-120시간 (100% 새로 작성)
- **옵션 C (하이브리드)**: **60-80시간** (재사용 30-40%)

---

## 2. 의사결정 기준

### 2.1 재사용 가능 범위 분석

기획서를 받으면 다음 표를 작성하세요:

| 레이어 | 기존 코드 | 재사용 가능? | 재사용률 | 작업 |
|--------|-----------|--------------|----------|------|
| **Common** | 예외 처리, JWT, Security | ✅ 높음 | 80% | Kotlin 변환 후 복사 |
| **Domain** | Waiting, Popup, Member | 🔶 중간 | 40% | 개념은 재사용, 필드 수정 |
| **Application** | 17개 서비스 | ❌ 낮음 | 20% | 새로 작성 (비즈니스 변경) |
| **Infrastructure** | Port Adapter, Entity | 🔶 중간 | 30% | 일부 재사용 |
| **Presentation** | 9개 컨트롤러 | ❌ 낮음 | 10% | 새로 작성 (API 변경) |

**전체 재사용률**: **30-40%**

**결론**: 하이브리드 접근 적합 ✅

### 2.2 의사결정 플로우차트

```
[기존 기능 유지율?]
    ├─ > 70% → 옵션 A (마이그레이션)
    ├─ 30-70% → [기존 아키텍처 품질?]
    │               ├─ 우수 → 옵션 C (하이브리드) ✅
    │               └─ 나쁨 → 옵션 B (새 레포)
    └─ < 30% → 옵션 B (새 레포)
```

---

## 3. 재사용 가치 평가

### 3.1 평가 기준 (4가지 축)

각 코드 모듈을 다음 기준으로 평가:

```
1. 품질 (Quality): 코드 품질, 테스트 커버리지
2. 적합성 (Relevance): 새 요구사항과의 일치도
3. 독립성 (Independence): 다른 모듈과의 결합도
4. 복잡도 (Complexity): 변환 난이도
```

**점수**: 각 1-5점, 합계 20점 만점

**재사용 판단**:
- **16점 이상**: ✅ 반드시 재사용
- **12-15점**: 🔶 선택적 재사용
- **11점 이하**: ❌ 새로 작성

### 3.2 기존 코드 평가표

| 모듈 | 품질 | 적합성 | 독립성 | 복잡도 | 합계 | 결정 |
|------|------|--------|--------|--------|------|------|
| **ErrorType + BusinessException** | 5 | 5 | 5 | 5 | **20** | ✅ 재사용 |
| **GlobalExceptionHandler** | 5 | 5 | 5 | 4 | **19** | ✅ 재사용 |
| **JwtTokenProvider** | 5 | 5 | 4 | 4 | **18** | ✅ 재사용 |
| **OAuth2 로그인** | 4 | 5 | 4 | 4 | **17** | ✅ 재사용 |
| **Member 도메인** | 5 | 5 | 4 | 5 | **19** | ✅ 재사용 |
| **Popup 도메인** | 5 | 4 | 4 | 4 | **17** | ✅ 재사용 (수정) |
| **Waiting 도메인** | 5 | 2 | 3 | 3 | **13** | 🔶 개념만 참고 |
| **WaitingService** | 5 | 1 | 2 | 2 | **10** | ❌ 새로 작성 |
| **QueryDSL 설정** | 4 | 5 | 5 | 3 | **17** | ✅ 재사용 |
| **BaseEntity (Auditing)** | 5 | 5 | 5 | 5 | **20** | ✅ 재사용 |
| **WaitingNotificationService** | 4 | 1 | 2 | 2 | **9** | ❌ 새로 작성 |
| **SSE Adapter** | 4 | 2 | 4 | 3 | **13** | 🔶 WebSocket으로 교체 |

**재사용 결정 요약**:
- ✅ **반드시 재사용 (8개)**: 예외 처리, JWT, OAuth, Member, Popup, QueryDSL, BaseEntity
- 🔶 **선택적 재사용 (2개)**: Waiting (개념만), SSE (기술 변경)
- ❌ **새로 작성 (2개)**: Service 레이어, 컨트롤러

---

## 4. 선택적 마이그레이션 전략

### 4.1 3단계 접근법

```
Phase 1: 공통 모듈 복사 (1-2일)
   → 예외 처리, Security, JWT
   → 기존 코드 → Kotlin 변환 → 복사

Phase 2: 재사용 도메인 복사 (2-3일)
   → Member, Popup (필요시 수정)
   → 기존 코드 → Kotlin 변환 → 수정 → 복사

Phase 3: 신규 기능 개발 (10-15일)
   → 새 도메인, 서비스, API
   → 처음부터 Kotlin으로 작성
```

### 4.2 복사 vs 새로 작성 기준

**복사 (Copy & Convert)**:
- 코드 품질 우수
- 로직 변경 불필요 (또는 소폭)
- 독립적 (다른 모듈과 결합도 낮음)
- 예: ErrorType, JWT, Member

**참고 (Reference)**:
- 개념은 유사하나 구현 변경 필요
- 비즈니스 로직 일부 변경
- 예: Waiting → Reservation (개념은 유사)

**새로 작성 (Rewrite)**:
- 비즈니스 로직 대폭 변경
- API 스펙 변경
- 의존성 많음
- 예: Service, Controller

---

## 5. 단계별 실행 계획

### 5.1 Phase 0: 프로젝트 초기화 (0.5일)

```bash
# 1. 새 레포지토리 생성
mkdir spotit-v2
cd spotit-v2
git init

# 2. Spring Initializr로 프로젝트 생성 (Kotlin)
curl https://start.spring.io/starter.zip \
  -d type=gradle-project-kotlin \
  -d language=kotlin \
  -d bootVersion=3.4.6 \
  -d dependencies=web,data-jpa,security,oauth2-client,validation,postgresql,h2,actuator \
  -o backend.zip
unzip backend.zip
rm backend.zip

# 3. 기존 프로젝트를 별도 디렉토리에 클론 (참고용)
cd ..
git clone <기존-레포-URL> spotit-old
```

**디렉토리 구조**:
```
workspace/
├── spotit-v2/        # 새 프로젝트 (Kotlin)
│   └── backend/
└── spotit-old/       # 기존 프로젝트 (Java, 참고용)
    └── backend/
```

### 5.2 Phase 1: 공통 모듈 복사 (1-2일)

#### 1.1 예외 처리 시스템

**작업 순서**:
```bash
# 1. 기존 코드 확인
cat spotit-old/backend/src/main/java/com/example/demo/common/exception/ErrorType.java

# 2. AI 에이전트에게 변환 요청
"다음 Java 코드를 Kotlin으로 변환해줘:
[ErrorType.java 코드 붙여넣기]

요구사항:
- enum class 사용
- 기존 로직 100% 유지"

# 3. 변환된 코드를 새 프로젝트에 복사
# spotit-v2/backend/src/main/kotlin/com/spotit/backend/common/exception/ErrorType.kt

# 4. 동일하게 BusinessException, GlobalExceptionHandler 변환
```

**체크리스트**:
- [ ] ErrorType.kt (60+ 에러 타입)
- [ ] BusinessException.kt
- [ ] ErrorResponse.kt
- [ ] GlobalExceptionHandler.kt
- [ ] 테스트 코드 작성
- [ ] 빌드 성공 확인

#### 1.2 Security & JWT

**복사 대상**:
- [ ] JwtProperties.kt
- [ ] JwtTokenProvider.kt
- [ ] JwtAuthenticationFilter.kt
- [ ] UserPrincipal.kt
- [ ] SecurityConfig.kt

**변환 포인트**:
```java
// Java - Lombok
@Getter
@RequiredArgsConstructor
public class JwtTokenProvider {
    private final JwtProperties jwtProperties;
}

// Kotlin - Primary constructor
class JwtTokenProvider(
    private val jwtProperties: JwtProperties
) {
}
```

#### 1.3 기타 공통 모듈

**복사 대상**:
- [ ] BaseEntity.kt (JPA Auditing)
- [ ] ApiResponse.kt (통일된 응답 형식)

**예상 시간**: 1-2일 (변환 + 테스트)

### 5.3 Phase 2: 재사용 도메인 복사 (2-3일)

#### 2.1 Member 도메인 (거의 변경 없음)

**복사 대상**:
```
domain/model/Member.kt          # Java record → Kotlin data class
domain/port/MemberPort.kt       # interface
infrastructure/persistence/entity/MemberEntity.kt
infrastructure/persistence/adapter/MemberPortAdapter.kt
infrastructure/persistence/repository/MemberJpaRepository.kt
```

**변환 예시**:
```java
// Java Record
public record Member(
    Long id,
    String email,
    String nickname,
    String profileImageUrl,
    String provider,
    String providerId
) {}

// Kotlin Data Class
data class Member(
    val id: Long?,
    val email: String,
    val nickname: String,
    val profileImageUrl: String?,
    val provider: String,
    val providerId: String
)
```

#### 2.2 Popup 도메인 (일부 수정)

**복사 후 수정**:
```kotlin
// 기존 필드 유지
data class Popup(
    val id: Long?,
    val name: String,
    val location: Location,
    val schedule: PopupSchedule,

    // 신규 필드 추가 (기획에 따라)
    val aiRecommendationScore: Double? = null,  // AI 추천 점수
    val maxReservationsPerDay: Int? = null      // 일일 예약 제한
)
```

**체크리스트**:
- [ ] 기존 Popup.kt 복사
- [ ] 새 필드 추가
- [ ] 비즈니스 로직 검토 (유지 or 수정)
- [ ] PopupEntity.kt 복사 + 수정
- [ ] PopupPort, Adapter 복사

#### 2.3 Waiting → Reservation (개념 참고)

**접근법**: 새로 작성하되 기존 구조 참고

```kotlin
// 기존 Waiting.kt 참고 (복사 X)
// 새로 작성: Reservation.kt

data class Reservation(
    val id: Long?,
    val popup: Popup,
    val member: Member,
    val reservationCode: String,     // QR 코드 (신규)
    val reservedDateTime: LocalDateTime,  // 예약 시간 (신규)
    val peopleCount: Int,            // 기존과 동일
    val status: ReservationStatus,   // CONFIRMED, CHECKED_IN (변경)
    val createdAt: LocalDateTime,
    val checkedInAt: LocalDateTime? = null
) {
    // 기존 Waiting의 검증 로직 참고
    init {
        require(peopleCount in 1..6) {
            throw BusinessException(ErrorType.INVALID_PEOPLE_COUNT)
        }
    }

    // 기존 enter() 메서드 참고 → checkIn() 메서드로 변경
    fun checkIn(): Reservation {
        require(status == ReservationStatus.CONFIRMED) {
            throw BusinessException(ErrorType.INVALID_RESERVATION_STATUS)
        }
        return copy(
            status = ReservationStatus.CHECKED_IN,
            checkedInAt = LocalDateTime.now()
        )
    }
}
```

**예상 시간**: 2-3일 (도메인 3개 기준)

### 5.4 Phase 3: 신규 기능 개발 (10-15일)

이 단계는 **옵션 B와 동일**하게 진행:
- 새 도메인 (Review, Notification 등)
- 새 서비스 (ReservationService, PopupService 등)
- 새 API (컨트롤러)
- 테스트 코드

**차이점**: 기존 공통 모듈을 활용하므로 **20-30% 빠름**

---

## 6. 재사용 코드 변환 가이드

### 6.1 자동 변환 도구 활용

**IntelliJ IDEA** (추천):
1. 기존 Java 파일 열기
2. `Code` → `Convert Java File to Kotlin File`
3. 자동 변환된 코드 검토
4. 수동 최적화

**장점**:
- ✅ 90% 자동 변환
- ✅ 구문 오류 없음

**단점**:
- ⚠️ Idiomatic Kotlin 아님 (수동 최적화 필요)
- ⚠️ Null safety 명시적 처리 필요

### 6.2 수동 최적화 체크리스트

**자동 변환 후 반드시 수정**:

#### 1. Null Safety
```kotlin
// ❌ 자동 변환 (Platform type)
val member: Member = memberPort.findById(id)

// ✅ 수동 최적화
val member: Member = memberPort.findById(id)
    ?: throw BusinessException(ErrorType.MEMBER_NOT_FOUND)
```

#### 2. Optional 제거
```kotlin
// ❌ 자동 변환 (Optional 유지)
fun findById(id: Long): Optional<Member>

// ✅ 수동 최적화
fun findById(id: Long): Member?
```

#### 3. Data Class 활용
```kotlin
// ❌ 자동 변환 (일반 class)
class MemberDto {
    var id: Long? = null
    var email: String? = null
}

// ✅ 수동 최적화
data class MemberDto(
    val id: Long?,
    val email: String
)
```

#### 4. Collection Functions
```kotlin
// ❌ 자동 변환 (Java Stream)
val result = list.stream()
    .filter { it.isActive() }
    .map { it.name }
    .collect(Collectors.toList())

// ✅ 수동 최적화
val result = list
    .filter { it.isActive }
    .map { it.name }
```

#### 5. When Expression
```kotlin
// ❌ 자동 변환 (if-else)
val result = if (status == Status.ACTIVE) {
    "활성"
} else if (status == Status.INACTIVE) {
    "비활성"
} else {
    "알 수 없음"
}

// ✅ 수동 최적화
val result = when (status) {
    Status.ACTIVE -> "활성"
    Status.INACTIVE -> "비활성"
    else -> "알 수 없음"
}
```

### 6.3 AI 에이전트 활용 변환

**효과적인 프롬프트**:
```
다음 Java 코드를 Idiomatic Kotlin으로 변환해줘:

[Java 코드 붙여넣기]

요구사항:
1. data class 사용
2. Null safety (? 타입)
3. Optional → Nullable 변환
4. Stream API → Collection functions
5. when 표현식 사용
6. 기존 로직 100% 유지

변환 후 주요 변경사항을 요약해줘.
```

**검증**:
```
변환된 코드를 검토해줘:

1. Null safety 제대로 처리?
2. 불변성 유지 (val)?
3. Kotlin 스타일 가이드 준수?
4. 기존 로직과 동일?

[변환된 Kotlin 코드 붙여넣기]
```

---

## 7. 1인 개발 최적화

### 7.1 우선순위 설정

**하이브리드 접근 시 우선순위**:

```
P0 (최우선 - 복사)
   ├─ 예외 처리 (ErrorType, BusinessException)
   ├─ JWT (JwtTokenProvider)
   └─ Member 도메인

P1 (재사용)
   ├─ Security 설정
   ├─ OAuth2 로그인
   ├─ Popup 도메인 (수정)
   └─ QueryDSL 설정

P2 (참고)
   ├─ Waiting 도메인 (개념만)
   └─ 알림 시스템 (구조만)

P3 (새로 작성)
   ├─ Service 레이어
   ├─ Controller
   └─ 신규 기능
```

### 7.2 시간 배분 (총 60-80시간)

| 단계 | 작업 | 시간 | 누적 |
|------|------|------|------|
| Phase 0 | 프로젝트 초기화 | 4시간 | 4시간 |
| Phase 1 | 공통 모듈 복사 (변환) | 12-16시간 | 16-20시간 |
| Phase 2 | 도메인 복사 (수정) | 16-20시간 | 32-40시간 |
| Phase 3 | 신규 기능 개발 | 28-40시간 | 60-80시간 |

**옵션 B 대비**: **25-30% 시간 절감**

### 7.3 AI 에이전트 활용 극대화

**Phase 1 (복사) - AI 역할**: 변환기
```
"다음 Java 코드를 Kotlin으로 변환해줘"
→ 기계적 변환 작업 → AI가 매우 잘함 ✅
```

**Phase 2 (수정) - AI 역할**: 리팩터링 도우미
```
"이 Kotlin 코드에 [신규 필드]를 추가하고, [비즈니스 로직]을 수정해줘"
→ 기존 코드 기반 수정 → AI가 잘함 ✅
```

**Phase 3 (신규) - AI 역할**: 코드 생성기
```
"기존 ReservationService를 참고해서 ReviewService를 생성해줘"
→ 패턴 기반 생성 → AI가 매우 잘함 ✅
```

**효율성**: 옵션 B보다 AI 활용도 **+40%** (복사/변환 작업)

---

## 8. 체크리스트

### 8.1 시작 전

- [ ] 기획서 분석 완료
- [ ] 재사용 가치 평가표 작성
- [ ] 재사용 모듈 목록 확정
- [ ] 새로 작성할 모듈 목록 확정
- [ ] 우선순위 설정 (P0, P1, P2, P3)

### 8.2 Phase 1 (공통 모듈 복사)

- [ ] ErrorType, BusinessException 변환 완료
- [ ] GlobalExceptionHandler 변환 완료
- [ ] JWT 모듈 변환 완료
- [ ] Security 설정 변환 완료
- [ ] OAuth2 로그인 변환 완료
- [ ] 기본 빌드 성공
- [ ] 테스트 작성 및 통과

### 8.3 Phase 2 (도메인 복사)

- [ ] Member 도메인 변환 완료
- [ ] Popup 도메인 변환 + 수정 완료
- [ ] 신규 도메인 (Reservation 등) 설계
- [ ] 도메인 레이어 테스트 통과
- [ ] 헥사고날 아키텍처 구조 확인

### 8.4 Phase 3 (신규 개발)

- [ ] Service 레이어 구현
- [ ] Controller 구현
- [ ] 통합 테스트 작성
- [ ] API 문서 작성 (Swagger)
- [ ] 성능 테스트
- [ ] 보안 검토

### 8.5 최종 확인

- [ ] 기존 코드 참조 제거 (더 이상 필요 없음)
- [ ] 불필요한 주석 제거
- [ ] 코드 포매팅 (ktlint)
- [ ] 전체 테스트 통과
- [ ] 문서화 완료
- [ ] 배포 준비 (Docker, CI/CD)

---

## 9. 실전 예시

### 9.1 ErrorType 변환 실습

**Step 1: 기존 코드 확인**
```java
// ErrorType.java
@Getter
@RequiredArgsConstructor
public enum ErrorType {
    POPUP_NOT_FOUND(HttpStatus.NOT_FOUND, "POPUP_NOT_FOUND", "팝업을 찾을 수 없습니다"),
    INVALID_PEOPLE_COUNT(HttpStatus.BAD_REQUEST, "INVALID_PEOPLE_COUNT", "인원수는 1~6명이어야 합니다");

    private final HttpStatus httpStatus;
    private final String code;
    private final String message;
}
```

**Step 2: AI에게 변환 요청**
```
다음 Java enum을 Kotlin으로 변환해줘:
[코드 붙여넣기]

Kotlin enum class 사용, Lombok 제거
```

**Step 3: AI 응답 (변환 결과)**
```kotlin
// ErrorType.kt
enum class ErrorType(
    val httpStatus: HttpStatus,
    val code: String,
    val message: String
) {
    POPUP_NOT_FOUND(HttpStatus.NOT_FOUND, "POPUP_NOT_FOUND", "팝업을 찾을 수 없습니다"),
    INVALID_PEOPLE_COUNT(HttpStatus.BAD_REQUEST, "INVALID_PEOPLE_COUNT", "인원수는 1~6명이어야 합니다")
}
```

**Step 4: 새 프로젝트에 복사**
```bash
# 파일 생성
touch src/main/kotlin/com/spotit/backend/common/exception/ErrorType.kt

# 변환된 코드 붙여넣기
# (에디터에서 수동 또는 AI 에이전트 통해)

# 빌드 확인
./gradlew build
```

**Step 5: 테스트 작성**
```kotlin
class ErrorTypeTest {
    @Test
    fun `ErrorType should have correct HttpStatus`() {
        assertThat(ErrorType.POPUP_NOT_FOUND.httpStatus).isEqualTo(HttpStatus.NOT_FOUND)
    }
}
```

### 9.2 Member 도메인 변환 실습

**Step 1: 기존 코드 (Java Record)**
```java
public record Member(
    Long id,
    String email,
    String nickname,
    String profileImageUrl,
    String provider,
    String providerId
) {}
```

**Step 2: Kotlin 변환**
```kotlin
data class Member(
    val id: Long?,
    val email: String,
    val nickname: String,
    val profileImageUrl: String?,
    val provider: String,
    val providerId: String
)
```

**Step 3: 포트 인터페이스 변환**
```java
// Java
public interface MemberPort {
    Member save(Member member);
    Optional<Member> findById(Long id);
    Optional<Member> findByEmail(String email);
}

// Kotlin
interface MemberPort {
    fun save(member: Member): Member
    fun findById(id: Long): Member?
    fun findByEmail(email: String): Member?
}
```

**Step 4: 어댑터 변환 (복잡)**
```kotlin
// MemberPortAdapter.kt
@Repository
class MemberPortAdapter(
    private val memberJpaRepository: MemberJpaRepository,
    private val memberEntityMapper: MemberEntityMapper
) : MemberPort {

    override fun save(member: Member): Member {
        val entity = memberEntityMapper.toEntity(member)
        val saved = memberJpaRepository.save(entity)
        return memberEntityMapper.toDomain(saved)
    }

    override fun findById(id: Long): Member? {
        return memberJpaRepository.findById(id)
            .map { memberEntityMapper.toDomain(it) }
            .orElse(null)
    }
}
```

---

## 10. 성공 사례 시나리오

### 10.1 시나리오: 팝업 예약 시스템 (실제 예시)

**배경**:
- 기존: 대기열 시스템 (Waiting)
- 신규: 예약 시스템 (Reservation) + AI 추천

**재사용 결정**:

| 모듈 | 재사용? | 이유 |
|------|---------|------|
| ErrorType | ✅ 100% | 품질 우수, 범용적 |
| JWT | ✅ 100% | 변경 불필요 |
| Member | ✅ 100% | 동일 |
| Popup | ✅ 80% | 일부 필드 추가 |
| Waiting | 🔶 개념만 | 예약으로 변경 |
| WaitingService | ❌ 0% | 비즈니스 완전 변경 |

**실행**:
1. **Day 1**: ErrorType, JWT 복사 (12시간)
2. **Day 2-3**: Member, Popup 복사 + 수정 (16시간)
3. **Day 4-5**: Reservation 도메인 신규 작성 (기존 Waiting 참고) (16시간)
4. **Day 6-10**: Service, Controller 신규 작성 (40시간)

**결과**:
- 총 84시간 (약 10.5일)
- 옵션 B (새로 작성) 대비 **30% 시간 절감**
- 옵션 A (전체 마이그레이션) 대비 **리스크 50% 감소**

---

## 11. FAQ

### Q1. 기존 코드를 복사하면 레거시가 유입되지 않나요?

**A**: 선별적 복사이므로 **좋은 코드만** 가져옵니다.
- ✅ 품질 평가 (3.2 참고)
- ✅ 16점 이상만 복사
- ✅ Kotlin 변환 시 최적화

### Q2. 변환 과정에서 버그가 발생하면?

**A**: 단계별 검증으로 방지합니다.
- ✅ 각 모듈 변환 후 테스트 작성
- ✅ 빌드 성공 확인
- ✅ 통합 테스트

### Q3. 옵션 B보다 얼마나 빠른가요?

**A**: **20-30% 시간 절감**
- 옵션 B: 90-120시간
- 옵션 C: 60-80시간
- 절감: 30-40시간 (공통 모듈 재사용)

### Q4. AI 에이전트가 변환을 잘못하면?

**A**: 수동 검증 단계가 있습니다.
- ✅ AI 변환 → 수동 검토 (6.2 체크리스트)
- ✅ 테스트 코드 작성
- ✅ 기존 Java 코드와 비교

### Q5. 어떤 경우에 하이브리드가 최적인가요?

**A**: 다음 조건 충족 시:
- 기존 재사용률 **30-50%**
- 기존 아키텍처 **우수**
- **1인 개발**로 시간 중요
- 검증된 코드 활용 희망

---

## 12. 결론

### 12.1 하이브리드 접근 요약

**핵심**:
- 새 레포지토리에서 시작
- 좋은 코드만 선별적으로 복사 (30-40%)
- 나머지는 새로 작성 (60-70%)

**장점**:
- ✅ 옵션 B 대비 **20-30% 빠름**
- ✅ 옵션 A 대비 **리스크 50% 낮음**
- ✅ 검증된 코드 재사용
- ✅ 깨끗한 코드베이스

**적합한 경우**:
- 재사용률 30-50%
- 기존 아키텍처 우수
- 1인 개발 + AI 에이전트

**예상 일정**: **60-80시간** (7.5-10일)

### 12.2 Next Steps

1. **재사용 가치 평가** (3.2 표 작성)
2. **재사용 모듈 목록 확정**
3. **Phase 1 시작** (공통 모듈 복사)
4. **단계별 진행** (Phase 2, 3)
5. **지속적 검증** (각 단계마다 테스트)

---

**문서 버전**: 1.0
**작성일**: 2025-11-27
**대상**: 1인 개발자 + AI 에이전트
**예상 소요 시간**: 60-80시간 (7.5-10일)
**재사용률**: 30-40%
