# Kotlin 마이그레이션 계획서

## 목차
1. [개요](#1-개요)
2. [마이그레이션 전략](#2-마이그레이션-전략)
3. [단계별 마이그레이션 로드맵](#3-단계별-마이그레이션-로드맵)
4. [아키텍처 패턴 유지 방안](#4-아키텍처-패턴-유지-방안)
5. [Java vs Kotlin 매핑](#5-java-vs-kotlin-매핑)
6. [검증 전략](#6-검증-전략)
7. [빌드 및 의존성 관리](#7-빌드-및-의존성-관리)
8. [위험 요소 및 대응 방안](#8-위험-요소-및-대응-방안)
9. [마일스톤 및 체크포인트](#9-마일스톤-및-체크포인트)

---

## 1. 개요

### 1.1 마이그레이션 목표

**기본 원칙**: 아키텍처와 구조의 철학·원칙을 100% 유지하면서 코틀린의 장점 최대 활용

**목표**:
- ✅ 기존과 **정확히 동일한 동작** 보장 (100% 동작 호환성)
- ✅ 헥사고날 아키텍처의 **완벽한 유지**
- ✅ 레이어 분리 및 의존성 역전 원칙 **그대로 유지**
- ✅ 도메인 모델의 순수성 **유지**
- ✅ 비즈니스 로직 **변경 없음**
- ✅ 코틀린의 null safety, data class, extension function 등 **적극 활용**
- ✅ 코드 간결성 향상 (Java 184개 파일 → Kotlin 예상 140-150개 파일)
- ✅ 타입 안전성 강화

### 1.2 현재 코드베이스 현황

**언어**: Java 21
**프레임워크**: Spring Boot 3.4.6
**아키텍처**: Hexagonal Architecture (Ports & Adapters)
**파일 수**: 184개 Java 소스 파일, 9개 테스트 파일
**빌드 도구**: Gradle
**주요 기술**: QueryDSL, JWT, OAuth2, JPA, PostgreSQL/H2

**레이어 구조**:
```
Domain (순수 비즈니스 로직)
  ↓ 의존
Application (유스케이스)
  ↓ 의존
Infrastructure (어댑터) + Presentation (컨트롤러)
```

---

## 2. 마이그레이션 전략

### 2.1 점진적 마이그레이션 (Incremental Migration)

**이유**:
- Java와 Kotlin은 100% 상호운용 가능
- 한 번에 전체 변경 시 리스크가 너무 큼
- 레이어별로 순차 마이그레이션하며 지속적으로 검증
- 언제든지 롤백 가능

**접근법**: **Outside-In 전략** (외부에서 내부로)

```
1단계: Common (예외, 유틸리티) → 가장 독립적
2단계: Domain (도메인 모델) → 핵심 비즈니스 로직
3단계: Infrastructure (어댑터, 엔티티) → 외부 의존성
4단계: Application (서비스, 매퍼) → 유스케이스
5단계: Presentation (컨트롤러) → 외부 인터페이스
6단계: 테스트 코드 마이그레이션
```

**각 단계마다**:
- ✅ 해당 레이어 완전 마이그레이션
- ✅ 전체 테스트 스위트 통과 확인
- ✅ 통합 테스트 실행
- ✅ Git 커밋 (원자적 변경)

### 2.2 Java-Kotlin 공존 전략

**Gradle 설정**:
```kotlin
plugins {
    kotlin("jvm") version "2.1.0"
    kotlin("plugin.spring") version "2.1.0"
    kotlin("plugin.jpa") version "2.1.0"
    kotlin("kapt") version "2.1.0"  // QueryDSL용
}

// Java + Kotlin 동시 컴파일
sourceSets {
    main {
        java.srcDirs("src/main/java", "src/main/kotlin")
    }
    test {
        java.srcDirs("src/test/java", "src/test/kotlin")
    }
}
```

**디렉토리 구조** (마이그레이션 중):
```
src/
├── main/
│   ├── java/          # 아직 마이그레이션 안 된 파일
│   ├── kotlin/        # 마이그레이션 완료된 파일
│   └── resources/
└── test/
    ├── java/
    ├── kotlin/
    └── resources/
```

**최종 구조** (마이그레이션 완료 후):
```
src/
├── main/
│   ├── kotlin/        # 모든 소스 파일
│   └── resources/
└── test/
    ├── kotlin/
    └── resources/
```

---

## 3. 단계별 마이그레이션 로드맵

### 3.1 Phase 0: 준비 작업

**작업 항목**:

1. **Gradle 설정 업데이트**
   - Kotlin 플러그인 추가
   - Kotlin 표준 라이브러리 추가
   - Kotlin 컴파일러 옵션 설정
   - kapt 플러그인 추가 (QueryDSL, Lombok → Kotlin 호환)

2. **의존성 검증**
   - Spring Boot 3.4.6의 Kotlin 호환성 확인 ✅ (공식 지원)
   - QueryDSL Kotlin 지원 확인 ✅ (kotlin-jpa 플러그인)
   - JWT, OAuth2 등 라이브러리 호환성 확인

3. **CI/CD 파이프라인 수정**
   - Kotlin 컴파일 추가
   - 테스트 실행 확인

4. **기준선(Baseline) 설정**
   - 현재 모든 테스트 통과 확인
   - 테스트 커버리지 측정
   - API 동작 스냅샷 생성 (컨트랙트 테스트 기준)

**검증 기준**:
- ✅ Gradle 빌드 성공
- ✅ 모든 기존 테스트 통과
- ✅ 애플리케이션 정상 기동

**예상 소요 시간**: 2-4시간

---

### 3.2 Phase 1: Common Layer 마이그레이션

**대상 파일** (우선순위 순):

```
common/
├── exception/
│   ├── ErrorType.kt                 # enum → enum class
│   ├── BusinessException.kt         # 표준 예외
│   ├── ErrorResponse.kt             # data class
│   └── GlobalExceptionHandler.kt    # @RestControllerAdvice
├── entity/
│   └── BaseEntity.kt                # abstract class
├── jwt/
│   ├── JwtProperties.kt             # @ConfigurationProperties
│   ├── JwtTokenProvider.kt          # component
│   └── JwtAuthenticationFilter.kt
└── security/
    ├── UserPrincipal.kt             # data class
    ├── CustomAccessDeniedHandler.kt
    └── CustomAuthenticationEntryPoint.kt
```

**마이그레이션 예시**:

**Before (Java)**:
```java
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

**After (Kotlin)**:
```kotlin
enum class ErrorType(
    val httpStatus: HttpStatus,
    val code: String,
    val message: String
) {
    POPUP_NOT_FOUND(HttpStatus.NOT_FOUND, "POPUP_NOT_FOUND", "팝업을 찾을 수 없습니다"),
    INVALID_PEOPLE_COUNT(HttpStatus.BAD_REQUEST, "INVALID_PEOPLE_COUNT", "인원수는 1~6명이어야 합니다")
}
```

**Before (Java)**:
```java
@Getter
public class BusinessException extends RuntimeException {
    private final ErrorType errorType;
    private final String additionalInfo;

    public BusinessException(ErrorType errorType) {
        super(errorType.getMessage());
        this.errorType = errorType;
        this.additionalInfo = null;
    }

    public BusinessException(ErrorType errorType, String additionalInfo) {
        super(errorType.getMessage());
        this.errorType = errorType;
        this.additionalInfo = additionalInfo;
    }
}
```

**After (Kotlin)**:
```kotlin
class BusinessException(
    val errorType: ErrorType,
    val additionalInfo: String? = null
) : RuntimeException(errorType.message)
```

**검증 방법**:
1. ✅ 전체 테스트 스위트 실행 (`./gradlew test`)
2. ✅ 예외 처리 통합 테스트
3. ✅ API 에러 응답 검증

**예상 소요 시간**: 4-6시간

---

### 3.3 Phase 2: Domain Layer 마이그레이션

**대상 파일**:

```
domain/
├── model/
│   ├── Member.kt                    # data class (Java record → Kotlin data class)
│   ├── popup/
│   │   ├── Popup.kt                # class (비즈니스 로직 포함)
│   │   ├── PopupStatus.kt          # enum class
│   │   ├── PopupCategory.kt        # value object (inline value class)
│   │   ├── OpeningHours.kt         # value object
│   │   ├── Location.kt
│   │   ├── PopupSchedule.kt
│   │   └── PopupDisplay.kt
│   ├── waiting/
│   │   ├── Waiting.kt              # data class (불변 + 비즈니스 로직)
│   │   ├── WaitingStatus.kt        # enum class
│   │   └── WaitingQuery.kt         # sealed interface
│   ├── notification/
│   │   ├── Notification.kt
│   │   ├── NotificationStatus.kt
│   │   └── DomainEvent.kt          # sealed interface
│   └── ban/
│       ├── Ban.kt
│       └── BanType.kt
└── port/
    ├── WaitingPort.kt              # interface
    ├── PopupPort.kt                # interface
    ├── MemberPort.kt               # interface
    ├── NotificationPort.kt         # interface
    └── EmailSendPort.kt            # interface
```

**핵심 변환 예시**:

#### **Java Record → Kotlin Data Class**

**Before (Java Record)**:
```java
public record Waiting(
    Long id,
    Popup popup,
    String waitingPersonName,
    Member member,
    String contactEmail,
    Integer peopleCount,
    Integer waitingNumber,
    WaitingStatus status,
    LocalDateTime registeredAt,
    LocalDateTime enteredAt,
    LocalDateTime canEnterAt,
    Integer expectedWaitingTimeMinutes,
    Integer initialWaitingNumber
) {
    // Compact constructor (validation)
    public Waiting {
        if (peopleCount < 1 || peopleCount > 6) {
            throw new BusinessException(ErrorType.INVALID_PEOPLE_COUNT);
        }
        if (waitingPersonName.length() < 2 || waitingPersonName.length() > 20) {
            throw new BusinessException(ErrorType.INVALID_WAITING_PERSON_NAME);
        }
    }

    // Business logic
    public Waiting enter() {
        if (status != WaitingStatus.WAITING) {
            throw new BusinessException(ErrorType.WAITING_ALREADY_ENTERED);
        }
        return new Waiting(
            id, popup, waitingPersonName, member, contactEmail, peopleCount,
            waitingNumber, WaitingStatus.VISITED, registeredAt,
            LocalDateTime.now(), canEnterAt, expectedWaitingTimeMinutes,
            initialWaitingNumber
        );
    }

    public Waiting minusWaitingNumber() {
        return new Waiting(
            id, popup, waitingPersonName, member, contactEmail, peopleCount,
            waitingNumber - 1, status, registeredAt, enteredAt, canEnterAt,
            expectedWaitingTimeMinutes, initialWaitingNumber
        );
    }
}
```

**After (Kotlin Data Class)**:
```kotlin
data class Waiting(
    val id: Long?,
    val popup: Popup,
    val waitingPersonName: String,
    val member: Member,
    val contactEmail: String,
    val peopleCount: Int,
    val waitingNumber: Int,
    val status: WaitingStatus,
    val registeredAt: LocalDateTime,
    val enteredAt: LocalDateTime?,
    val canEnterAt: LocalDateTime?,
    val expectedWaitingTimeMinutes: Int?,
    val initialWaitingNumber: Int
) {
    init {
        require(peopleCount in 1..6) {
            throw BusinessException(ErrorType.INVALID_PEOPLE_COUNT)
        }
        require(waitingPersonName.length in 2..20) {
            throw BusinessException(ErrorType.INVALID_WAITING_PERSON_NAME)
        }
    }

    fun enter(): Waiting {
        require(status == WaitingStatus.WAITING) {
            throw BusinessException(ErrorType.WAITING_ALREADY_ENTERED)
        }
        return copy(
            status = WaitingStatus.VISITED,
            enteredAt = LocalDateTime.now()
        )
    }

    fun minusWaitingNumber(): Waiting = copy(waitingNumber = waitingNumber - 1)

    fun markAsNoShow(): Waiting = copy(status = WaitingStatus.NO_SHOW)

    fun markAsCanEnter(): Waiting = copy(canEnterAt = LocalDateTime.now())
}
```

**개선 포인트**:
- ✅ `copy()` 메서드로 불변 객체 업데이트 간소화
- ✅ `init` 블록에서 유효성 검증
- ✅ Null safety (`?` 타입)
- ✅ `require()` 함수로 전제조건 검증

#### **Sealed Class → Sealed Interface**

**Before (Java Sealed Class)**:
```java
public sealed interface WaitingQuery {
    record ForWaitingId(Long waitingId) implements WaitingQuery {}
    record ForPopup(Long popupId, WaitingStatus status) implements WaitingQuery {}
    record ForVisitHistory(Long memberId, Integer size, Long lastWaitingId, String status) implements WaitingQuery {}
    record ForDuplicateCheck(Long memberId, Long popupId, LocalDate date) implements WaitingQuery {}
    record ForStatus(WaitingStatus status) implements WaitingQuery {}
    record ForCanEnterWaiting() implements WaitingQuery {}

    static WaitingQuery forWaitingId(Long id) { return new ForWaitingId(id); }
}
```

**After (Kotlin Sealed Interface)**:
```kotlin
sealed interface WaitingQuery {
    data class ForWaitingId(val waitingId: Long) : WaitingQuery
    data class ForPopup(val popupId: Long, val status: WaitingStatus?) : WaitingQuery
    data class ForVisitHistory(
        val memberId: Long,
        val size: Int,
        val lastWaitingId: Long?,
        val status: String?
    ) : WaitingQuery
    data class ForDuplicateCheck(
        val memberId: Long,
        val popupId: Long,
        val date: LocalDate
    ) : WaitingQuery
    data class ForStatus(val status: WaitingStatus) : WaitingQuery
    data object ForCanEnterWaiting : WaitingQuery

    companion object {
        fun forWaitingId(id: Long): WaitingQuery = ForWaitingId(id)
    }
}
```

**개선 포인트**:
- ✅ `data object` 활용 (파라미터 없는 케이스)
- ✅ Companion object로 팩토리 메서드
- ✅ Null safety

#### **Port Interface**

**Before (Java)**:
```java
public interface WaitingPort {
    Waiting save(Waiting waiting);
    List<Waiting> saveAll(List<Waiting> waitings);
    List<Waiting> findByQuery(WaitingQuery query);
    Integer getNextWaitingNumber(Long popupId);
    Optional<Waiting> findByMemberIdAndPopupId(Long memberId, Long popupId);
}
```

**After (Kotlin)**:
```kotlin
interface WaitingPort {
    fun save(waiting: Waiting): Waiting
    fun saveAll(waitings: List<Waiting>): List<Waiting>
    fun findByQuery(query: WaitingQuery): List<Waiting>
    fun getNextWaitingNumber(popupId: Long): Int
    fun findByMemberIdAndPopupId(memberId: Long, popupId: Long): Waiting?
}
```

**개선 포인트**:
- ✅ `Optional<T>` → `T?` (Kotlin null safety)
- ✅ 타입 추론 제거 (명시적 반환 타입)

**검증 방법**:
1. ✅ 도메인 로직 단위 테스트 (`WaitingTest.kt`)
2. ✅ 불변성 검증 (모든 도메인 객체 불변)
3. ✅ 비즈니스 규칙 검증 (enter, minusWaitingNumber 등)
4. ✅ JPA 어노테이션 없음 확인 (순수성 유지)

**예상 소요 시간**: 8-12시간

---

### 3.4 Phase 3: Infrastructure Layer 마이그레이션

**대상 파일**:

```
infrastructure/
├── persistence/
│   ├── adapter/
│   │   ├── WaitingPortAdapter.kt
│   │   ├── PopupPortAdapter.kt
│   │   ├── MemberPortAdapter.kt
│   │   └── ... (총 10개 어댑터)
│   ├── entity/
│   │   ├── WaitingEntity.kt
│   │   ├── MemberEntity.kt
│   │   ├── popup/
│   │   │   ├── PopupEntity.kt
│   │   │   ├── PopupContentEntity.kt
│   │   │   └── ...
│   │   └── ...
│   ├── mapper/
│   │   ├── WaitingEntityMapper.kt
│   │   └── ...
│   └── repository/
│       ├── WaitingJpaRepository.kt
│       ├── PopupJpaRepository.kt
│       └── ...
└── external/
    ├── NotificationSseAdapter.kt
    └── GmailSmtpEmailAdapter.kt
```

**핵심 변환 예시**:

#### **JPA Entity**

**Before (Java)**:
```java
@Entity
@Table(name = "waiting")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
public class WaitingEntity extends BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "popup_id", nullable = false)
    private PopupEntity popup;

    @Column(name = "waiting_person_name", nullable = false, length = 20)
    private String waitingPersonName;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id", nullable = false)
    private MemberEntity member;

    @Column(name = "people_count", nullable = false)
    private Integer peopleCount;

    @Column(name = "waiting_number", nullable = false)
    private Integer waitingNumber;

    @Enumerated(EnumType.STRING)
    @Column(name = "status", nullable = false)
    private WaitingStatus status;
}
```

**After (Kotlin)**:
```kotlin
@Entity
@Table(name = "waiting")
class WaitingEntity(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long? = null,

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "popup_id", nullable = false)
    val popup: PopupEntity,

    @Column(name = "waiting_person_name", nullable = false, length = 20)
    val waitingPersonName: String,

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id", nullable = false)
    val member: MemberEntity,

    @Column(name = "people_count", nullable = false)
    val peopleCount: Int,

    @Column(name = "waiting_number", nullable = false)
    var waitingNumber: Int,  // var: 업데이트 가능

    @Enumerated(EnumType.STRING)
    @Column(name = "status", nullable = false)
    var status: WaitingStatus  // var: 업데이트 가능
) : BaseEntity()
```

**주의사항**:
- ⚠️ JPA 엔티티는 `data class` 사용 불가 (프록시 이슈)
- ⚠️ `no-arg` 플러그인 필요 (JPA 기본 생성자)
- ⚠️ 업데이트 필드는 `var`, 불변 필드는 `val`
- ✅ Nullable 필드는 `?` 사용

#### **Port Adapter (QueryDSL 포함)**

**Before (Java)**:
```java
@Repository
@RequiredArgsConstructor
public class WaitingPortAdapter implements WaitingPort {
    private final WaitingJpaRepository waitingJpaRepository;
    private final WaitingEntityMapper waitingEntityMapper;
    private final JPAQueryFactory jpaQueryFactory;

    @Override
    public List<Waiting> findByQuery(WaitingQuery query) {
        return switch (query) {
            case WaitingQuery.ForWaitingId q ->
                waitingJpaRepository.findById(q.waitingId())
                    .map(entity -> waitingEntityMapper.toDomain(entity))
                    .stream().toList();
            case WaitingQuery.ForPopup q -> {
                var predicate = QWaitingEntity.waitingEntity.popup.id.eq(q.popupId());
                if (q.status() != null) {
                    predicate = predicate.and(QWaitingEntity.waitingEntity.status.eq(q.status()));
                }
                yield jpaQueryFactory.selectFrom(QWaitingEntity.waitingEntity)
                    .where(predicate)
                    .orderBy(QWaitingEntity.waitingEntity.waitingNumber.asc())
                    .fetch()
                    .stream()
                    .map(waitingEntityMapper::toDomain)
                    .toList();
            }
            // ... other cases
        };
    }
}
```

**After (Kotlin)**:
```kotlin
@Repository
class WaitingPortAdapter(
    private val waitingJpaRepository: WaitingJpaRepository,
    private val waitingEntityMapper: WaitingEntityMapper,
    private val jpaQueryFactory: JPAQueryFactory
) : WaitingPort {

    override fun findByQuery(query: WaitingQuery): List<Waiting> {
        return when (query) {
            is WaitingQuery.ForWaitingId ->
                waitingJpaRepository.findById(query.waitingId)
                    .map { waitingEntityMapper.toDomain(it) }
                    .map { listOf(it) }
                    .orElse(emptyList())

            is WaitingQuery.ForPopup -> {
                val qWaiting = QWaitingEntity.waitingEntity
                var predicate = qWaiting.popup.id.eq(query.popupId)

                query.status?.let {
                    predicate = predicate.and(qWaiting.status.eq(it))
                }

                jpaQueryFactory.selectFrom(qWaiting)
                    .where(predicate)
                    .orderBy(qWaiting.waitingNumber.asc())
                    .fetch()
                    .map { waitingEntityMapper.toDomain(it) }
            }
            // ... other cases
        }
    }
}
```

**개선 포인트**:
- ✅ `when` 표현식 (switch → when)
- ✅ Smart casting (`is` 타입 체크)
- ✅ `?.let` null-safe 호출
- ✅ `map` 람다 간소화

#### **Entity Mapper**

**Before (Java)**:
```java
@Component
@RequiredArgsConstructor
public class WaitingEntityMapper {
    private final PopupEntityMapper popupEntityMapper;
    private final MemberEntityMapper memberEntityMapper;

    public Waiting toDomain(WaitingEntity entity) {
        return new Waiting(
            entity.getId(),
            popupEntityMapper.toDomain(entity.getPopup()),
            entity.getWaitingPersonName(),
            memberEntityMapper.toDomain(entity.getMember()),
            entity.getContactEmail(),
            entity.getPeopleCount(),
            entity.getWaitingNumber(),
            entity.getStatus(),
            entity.getRegisteredAt(),
            entity.getEnteredAt(),
            entity.getCanEnterAt(),
            entity.getExpectedWaitingTimeMinutes(),
            entity.getInitialWaitingNumber()
        );
    }

    public WaitingEntity toEntity(Waiting domain, PopupEntity popup, MemberEntity member) {
        return new WaitingEntity(
            domain.id(),
            popup,
            domain.waitingPersonName(),
            member,
            domain.contactEmail(),
            domain.peopleCount(),
            domain.waitingNumber(),
            domain.status()
        );
    }
}
```

**After (Kotlin)**:
```kotlin
@Component
class WaitingEntityMapper(
    private val popupEntityMapper: PopupEntityMapper,
    private val memberEntityMapper: MemberEntityMapper
) {
    fun toDomain(entity: WaitingEntity): Waiting = Waiting(
        id = entity.id,
        popup = popupEntityMapper.toDomain(entity.popup),
        waitingPersonName = entity.waitingPersonName,
        member = memberEntityMapper.toDomain(entity.member),
        contactEmail = entity.contactEmail,
        peopleCount = entity.peopleCount,
        waitingNumber = entity.waitingNumber,
        status = entity.status,
        registeredAt = entity.registeredAt,
        enteredAt = entity.enteredAt,
        canEnterAt = entity.canEnterAt,
        expectedWaitingTimeMinutes = entity.expectedWaitingTimeMinutes,
        initialWaitingNumber = entity.initialWaitingNumber
    )

    fun toEntity(domain: Waiting, popup: PopupEntity, member: MemberEntity): WaitingEntity =
        WaitingEntity(
            id = domain.id,
            popup = popup,
            waitingPersonName = domain.waitingPersonName,
            member = member,
            peopleCount = domain.peopleCount,
            waitingNumber = domain.waitingNumber,
            status = domain.status
        )
}
```

**개선 포인트**:
- ✅ Named arguments (가독성)
- ✅ Expression body (`=`)
- ✅ Getter 호출 간소화 (`entity.id` vs `entity.getId()`)

**검증 방법**:
1. ✅ 레포지토리 테스트 (`WaitingPortAdapterTest.kt`)
2. ✅ QueryDSL 쿼리 정확성 검증
3. ✅ Entity ↔ Domain 매핑 테스트
4. ✅ N+1 문제 확인 (배치 fetch size)

**예상 소요 시간**: 10-14시간

---

### 3.5 Phase 4: Application Layer 마이그레이션

**대상 파일**:

```
application/
├── service/
│   ├── WaitingService.kt          # 핵심 (266줄)
│   ├── PopupService.kt
│   ├── NotificationService.kt
│   ├── WaitingNotificationService.kt
│   └── ... (총 17개 서비스)
├── dto/
│   ├── waiting/
│   │   ├── WaitingCreateRequest.kt   # data class
│   │   ├── WaitingCreateResponse.kt  # data class
│   │   └── ...
│   ├── popup/
│   ├── notification/
│   └── oauth/
└── mapper/
    ├── WaitingDtoMapper.kt
    └── ...
```

**핵심 변환 예시**:

#### **Service Class**

**Before (Java)**:
```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class WaitingService {
    private final WaitingPort waitingPort;
    private final PopupPort popupPort;
    private final MemberPort memberPort;
    private final BanPort banPort;
    private final WaitingDtoMapper waitingDtoMapper;
    private final WaitingNotificationService waitingNotificationService;

    @Transactional
    public WaitingCreateResponse createWaiting(Long popupId, WaitingCreateRequest request, Long memberId) {
        // 1. 팝업 조회
        Popup popup = popupPort.findById(popupId)
            .orElseThrow(() -> new BusinessException(ErrorType.POPUP_NOT_FOUND));

        // 2. 운영 시간 검증
        if (!popup.isOpenAt(LocalDateTime.now())) {
            throw new BusinessException(ErrorType.POPUP_NOT_OPENED);
        }

        // 3. 제재 확인
        if (banPort.isBanned(memberId, popupId)) {
            throw new BusinessException(ErrorType.BANNED_MEMBER);
        }

        // 4. 중복 신청 확인
        var existingWaiting = waitingPort.findByMemberIdAndPopupId(memberId, popupId);
        if (existingWaiting.isPresent()) {
            throw new BusinessException(ErrorType.DUPLICATE_WAITING);
        }

        // 5. 대기 생성
        Member member = memberPort.findById(memberId)
            .orElseThrow(() -> new BusinessException(ErrorType.MEMBER_NOT_FOUND));

        Integer nextNumber = waitingPort.getNextWaitingNumber(popupId);

        Waiting waiting = new Waiting(
            null, popup, request.waitingPersonName(), member,
            request.contactEmail(), request.peopleCount(),
            nextNumber, WaitingStatus.WAITING,
            LocalDateTime.now(), null, null, null, nextNumber
        );

        Waiting savedWaiting = waitingPort.save(waiting);

        // 6. 알림 발송
        waitingNotificationService.sendWaitingConfirmedNotification(savedWaiting);

        return waitingDtoMapper.toCreateResponse(savedWaiting);
    }
}
```

**After (Kotlin)**:
```kotlin
@Service
@Transactional(readOnly = true)
class WaitingService(
    private val waitingPort: WaitingPort,
    private val popupPort: PopupPort,
    private val memberPort: MemberPort,
    private val banPort: BanPort,
    private val waitingDtoMapper: WaitingDtoMapper,
    private val waitingNotificationService: WaitingNotificationService
) {

    @Transactional
    fun createWaiting(popupId: Long, request: WaitingCreateRequest, memberId: Long): WaitingCreateResponse {
        // 1. 팝업 조회
        val popup = popupPort.findById(popupId)
            ?: throw BusinessException(ErrorType.POPUP_NOT_FOUND)

        // 2. 운영 시간 검증
        require(popup.isOpenAt(LocalDateTime.now())) {
            throw BusinessException(ErrorType.POPUP_NOT_OPENED)
        }

        // 3. 제재 확인
        require(!banPort.isBanned(memberId, popupId)) {
            throw BusinessException(ErrorType.BANNED_MEMBER)
        }

        // 4. 중복 신청 확인
        val existingWaiting = waitingPort.findByMemberIdAndPopupId(memberId, popupId)
        require(existingWaiting == null) {
            throw BusinessException(ErrorType.DUPLICATE_WAITING)
        }

        // 5. 대기 생성
        val member = memberPort.findById(memberId)
            ?: throw BusinessException(ErrorType.MEMBER_NOT_FOUND)

        val nextNumber = waitingPort.getNextWaitingNumber(popupId)

        val waiting = Waiting(
            id = null,
            popup = popup,
            waitingPersonName = request.waitingPersonName,
            member = member,
            contactEmail = request.contactEmail,
            peopleCount = request.peopleCount,
            waitingNumber = nextNumber,
            status = WaitingStatus.WAITING,
            registeredAt = LocalDateTime.now(),
            enteredAt = null,
            canEnterAt = null,
            expectedWaitingTimeMinutes = null,
            initialWaitingNumber = nextNumber
        )

        val savedWaiting = waitingPort.save(waiting)

        // 6. 알림 발송
        waitingNotificationService.sendWaitingConfirmedNotification(savedWaiting)

        return waitingDtoMapper.toCreateResponse(savedWaiting)
    }
}
```

**개선 포인트**:
- ✅ `val`로 불변 변수 선언
- ✅ `?:` Elvis operator로 null 처리
- ✅ `require()` 전제조건 검증
- ✅ Named arguments (가독성)
- ✅ Type inference (`var` → `val`)

**추가 개선 - Extension Function** (선택적):
```kotlin
// Extension function for validation
fun Popup.validateOperatingHours() {
    require(isOpenAt(LocalDateTime.now())) {
        throw BusinessException(ErrorType.POPUP_NOT_OPENED)
    }
}

// Usage
popup.validateOperatingHours()
```

#### **DTO (Data Transfer Object)**

**Before (Java)**:
```java
public record WaitingCreateRequest(
    @NotBlank(message = "대기자 이름은 필수입니다")
    @Size(min = 2, max = 20, message = "이름은 2~20자여야 합니다")
    String waitingPersonName,

    @NotBlank(message = "이메일은 필수입니다")
    @Email(message = "이메일 형식이 올바르지 않습니다")
    String contactEmail,

    @NotNull(message = "인원수는 필수입니다")
    @Min(value = 1, message = "최소 1명 이상이어야 합니다")
    @Max(value = 6, message = "최대 6명까지 가능합니다")
    Integer peopleCount
) {}
```

**After (Kotlin)**:
```kotlin
data class WaitingCreateRequest(
    @field:NotBlank(message = "대기자 이름은 필수입니다")
    @field:Size(min = 2, max = 20, message = "이름은 2~20자여야 합니다")
    val waitingPersonName: String,

    @field:NotBlank(message = "이메일은 필수입니다")
    @field:Email(message = "이메일 형식이 올바르지 않습니다")
    val contactEmail: String,

    @field:NotNull(message = "인원수는 필수입니다")
    @field:Min(value = 1, message = "최소 1명 이상이어야 합니다")
    @field:Max(value = 6, message = "최대 6명까지 가능합니다")
    val peopleCount: Int
)
```

**주의사항**:
- ⚠️ Bean Validation 어노테이션에 `@field:` 사용 필요 (Kotlin property에 적용)

**검증 방법**:
1. ✅ 서비스 레이어 테스트 (`WaitingServiceTest.kt`)
2. ✅ 트랜잭션 경계 확인
3. ✅ DTO 매핑 테스트
4. ✅ 비즈니스 로직 검증 (중복, 제재, 알림 등)

**예상 소요 시간**: 12-16시간

---

### 3.6 Phase 5: Presentation Layer 마이그레이션

**대상 파일**:

```
presentation/
└── controller/
    ├── WaitingController.kt
    ├── PopupController.kt
    ├── NotificationController.kt
    ├── OAuthController.kt
    ├── MemberController.kt
    ├── AdminController.kt
    ├── ImageController.kt
    └── handler/
        ├── OAuth2SuccessHandler.kt
        └── OAuth2FailureHandler.kt
```

**핵심 변환 예시**:

**Before (Java)**:
```java
@RestController
@RequestMapping("/api")
@RequiredArgsConstructor
@Tag(name = "대기 관리", description = "팝업 대기 관련 API")
public class WaitingController {
    private final WaitingService waitingService;

    @PostMapping("/popups/{popupId}/waitings")
    @Operation(summary = "대기 신청", description = "팝업에 대기 신청을 합니다")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "대기 신청 성공"),
        @ApiResponse(responseCode = "400", description = "잘못된 요청"),
        @ApiResponse(responseCode = "404", description = "팝업을 찾을 수 없음")
    })
    public ResponseEntity<ApiResponse<WaitingCreateResponse>> createWaiting(
        @PathVariable Long popupId,
        @RequestBody @Valid WaitingCreateRequest request,
        @AuthenticationPrincipal UserPrincipal principal
    ) {
        WaitingCreateResponse response = waitingService.createWaiting(
            popupId, request, principal.getMemberId()
        );
        return ResponseEntity.ok(new ApiResponse<>("대기 신청 성공", response));
    }

    @GetMapping("/members/me/visit-history")
    @Operation(summary = "방문 내역 조회")
    public ResponseEntity<ApiResponse<List<WaitingVisitHistoryResponse>>> getVisitHistory(
        @RequestParam(required = false) Integer size,
        @RequestParam(required = false) Long lastWaitingId,
        @RequestParam(required = false) String status,
        @AuthenticationPrincipal UserPrincipal principal
    ) {
        List<WaitingVisitHistoryResponse> response = waitingService.getVisitHistory(
            principal.getMemberId(), size, lastWaitingId, status
        );
        return ResponseEntity.ok(new ApiResponse<>("조회 성공", response));
    }
}
```

**After (Kotlin)**:
```kotlin
@RestController
@RequestMapping("/api")
@Tag(name = "대기 관리", description = "팝업 대기 관련 API")
class WaitingController(
    private val waitingService: WaitingService
) {

    @PostMapping("/popups/{popupId}/waitings")
    @Operation(summary = "대기 신청", description = "팝업에 대기 신청을 합니다")
    @ApiResponses(
        ApiResponse(responseCode = "200", description = "대기 신청 성공"),
        ApiResponse(responseCode = "400", description = "잘못된 요청"),
        ApiResponse(responseCode = "404", description = "팝업을 찾을 수 없음")
    )
    fun createWaiting(
        @PathVariable popupId: Long,
        @RequestBody @Valid request: WaitingCreateRequest,
        @AuthenticationPrincipal principal: UserPrincipal
    ): ResponseEntity<ApiResponse<WaitingCreateResponse>> {
        val response = waitingService.createWaiting(
            popupId, request, principal.memberId
        )
        return ResponseEntity.ok(ApiResponse("대기 신청 성공", response))
    }

    @GetMapping("/members/me/visit-history")
    @Operation(summary = "방문 내역 조회")
    fun getVisitHistory(
        @RequestParam(required = false) size: Int?,
        @RequestParam(required = false) lastWaitingId: Long?,
        @RequestParam(required = false) status: String?,
        @AuthenticationPrincipal principal: UserPrincipal
    ): ResponseEntity<ApiResponse<List<WaitingVisitHistoryResponse>>> {
        val response = waitingService.getVisitHistory(
            principal.memberId, size, lastWaitingId, status
        )
        return ResponseEntity.ok(ApiResponse("조회 성공", response))
    }
}
```

**개선 포인트**:
- ✅ Nullable 파라미터 (`Int?`, `Long?`)
- ✅ Property 접근 (`principal.memberId`)
- ✅ 타입 추론 (`val response =`)

**검증 방법**:
1. ✅ 통합 테스트 (MockMvc 또는 RestAssured)
2. ✅ API 계약 테스트 (Swagger 문서 검증)
3. ✅ 인증/인가 테스트

**예상 소요 시간**: 6-8시간

---

### 3.7 Phase 6: 테스트 코드 마이그레이션

**대상 파일**:

```
src/test/kotlin/
├── Demo2ApplicationTests.kt
├── QuerydslTest.kt
├── ParallelExecutionTest.kt
├── application/
│   ├── mapper/
│   │   └── WaitingDtoMapperTest.kt
│   └── service/
│       ├── WaitingServiceTest.kt      # 핵심 (930줄)
│       └── EmailTemplateServiceTest.kt
├── domain/
│   └── model/
│       └── waiting/
│           └── WaitingTest.kt
└── infrastructure/
    └── persistence/
        ├── adapter/
        │   └── WaitingPortAdapterTest.kt
        └── mapper/
            └── MemberEntityMapperTest.kt
```

**핵심 변환 예시**:

**Before (Java + JUnit 5)**:
```java
@SpringBootTest
@Transactional
class WaitingServiceTest {
    @Autowired
    private WaitingService waitingService;

    @MockBean
    private WaitingPort waitingPort;

    @MockBean
    private PopupPort popupPort;

    @Test
    @DisplayName("대기 신청 성공 - 첫 번째 대기자")
    void createWaiting_success_firstWaiter() {
        // Given
        Long popupId = 1L;
        Long memberId = 1L;

        Popup popup = Popup.builder()
            .id(popupId)
            .name("테스트 팝업")
            .status(PopupStatus.ACTIVE)
            .build();

        Member member = new Member(memberId, "test@example.com", "테스터");

        WaitingCreateRequest request = new WaitingCreateRequest(
            "홍길동", "hong@example.com", 2
        );

        given(popupPort.findById(popupId)).willReturn(Optional.of(popup));
        given(memberPort.findById(memberId)).willReturn(Optional.of(member));
        given(waitingPort.getNextWaitingNumber(popupId)).willReturn(1);
        given(waitingPort.findByMemberIdAndPopupId(memberId, popupId))
            .willReturn(Optional.empty());

        // When
        WaitingCreateResponse response = waitingService.createWaiting(
            popupId, request, memberId
        );

        // Then
        assertThat(response.waitingNumber()).isEqualTo(1);
        assertThat(response.peopleCount()).isEqualTo(2);
        assertThat(response.status()).isEqualTo(WaitingStatus.WAITING);

        verify(waitingPort, times(1)).save(any(Waiting.class));
    }

    @Test
    @DisplayName("대기 신청 실패 - 팝업 없음")
    void createWaiting_fail_popupNotFound() {
        // Given
        Long popupId = 999L;

        given(popupPort.findById(popupId)).willReturn(Optional.empty());

        // When & Then
        assertThatThrownBy(() -> waitingService.createWaiting(
            popupId, new WaitingCreateRequest("홍길동", "hong@example.com", 2), 1L
        ))
            .isInstanceOf(BusinessException.class)
            .hasFieldOrPropertyWithValue("errorType", ErrorType.POPUP_NOT_FOUND);
    }
}
```

**After (Kotlin + JUnit 5 + Kotest Assertions)**:
```kotlin
@SpringBootTest
@Transactional
class WaitingServiceTest {
    @Autowired
    private lateinit var waitingService: WaitingService

    @MockBean
    private lateinit var waitingPort: WaitingPort

    @MockBean
    private lateinit var popupPort: PopupPort

    @Test
    @DisplayName("대기 신청 성공 - 첫 번째 대기자")
    fun `createWaiting success - first waiter`() {
        // Given
        val popupId = 1L
        val memberId = 1L

        val popup = Popup.builder()
            .id(popupId)
            .name("테스트 팝업")
            .status(PopupStatus.ACTIVE)
            .build()

        val member = Member(memberId, "test@example.com", "테스터")

        val request = WaitingCreateRequest(
            waitingPersonName = "홍길동",
            contactEmail = "hong@example.com",
            peopleCount = 2
        )

        given(popupPort.findById(popupId)).willReturn(popup)
        given(memberPort.findById(memberId)).willReturn(member)
        given(waitingPort.getNextWaitingNumber(popupId)).willReturn(1)
        given(waitingPort.findByMemberIdAndPopupId(memberId, popupId))
            .willReturn(null)

        // When
        val response = waitingService.createWaiting(popupId, request, memberId)

        // Then
        assertThat(response.waitingNumber).isEqualTo(1)
        assertThat(response.peopleCount).isEqualTo(2)
        assertThat(response.status).isEqualTo(WaitingStatus.WAITING)

        verify(waitingPort, times(1)).save(any())
    }

    @Test
    @DisplayName("대기 신청 실패 - 팝업 없음")
    fun `createWaiting fail - popup not found`() {
        // Given
        val popupId = 999L

        given(popupPort.findById(popupId)).willReturn(null)

        // When & Then
        assertThatThrownBy {
            waitingService.createWaiting(
                popupId,
                WaitingCreateRequest("홍길동", "hong@example.com", 2),
                1L
            )
        }
            .isInstanceOf(BusinessException::class.java)
            .hasFieldOrPropertyWithValue("errorType", ErrorType.POPUP_NOT_FOUND)
    }
}
```

**개선 포인트**:
- ✅ Backtick 테스트 이름 (한글 + 공백)
- ✅ `lateinit var` for Spring injection
- ✅ `val` for immutable test data
- ✅ Named arguments
- ✅ Type inference

**추가 개선 - Kotest (선택적)**:
```kotlin
// Kotest의 StringSpec 스타일
class WaitingServiceKotestTest : StringSpec({

    lateinit var waitingService: WaitingService
    lateinit var waitingPort: WaitingPort

    beforeTest {
        waitingPort = mockk()
        waitingService = WaitingService(waitingPort, ...)
    }

    "대기 신청 성공 - 첫 번째 대기자" {
        // Given
        val popup = Popup.builder().id(1L).name("테스트").build()
        every { popupPort.findById(1L) } returns popup

        // When
        val response = waitingService.createWaiting(1L, request, 1L)

        // Then
        response.waitingNumber shouldBe 1
        response.status shouldBe WaitingStatus.WAITING
    }
})
```

**검증 방법**:
1. ✅ 전체 테스트 스위트 실행 (`./gradlew test`)
2. ✅ 코드 커버리지 유지 (기존과 동일 수준)
3. ✅ 병렬 실행 확인

**예상 소요 시간**: 8-12시간

---

## 4. 아키텍처 패턴 유지 방안

### 4.1 헥사고날 아키텍처 유지

**원칙**:
```
✅ 도메인 레이어는 외부 의존성 완전 배제
✅ 포트 인터페이스를 통한 의존성 역전
✅ 어댑터는 인프라 레이어에만 존재
✅ 레이어 간 명확한 경계 유지
```

**Kotlin 구현**:
```kotlin
// Domain Layer - 순수 비즈니스 로직
package com.example.demo.domain.model.waiting

data class Waiting(...) {
    // NO JPA, NO Spring annotations
    fun enter(): Waiting = copy(status = WaitingStatus.VISITED)
}

// Domain Layer - Port (interface)
package com.example.demo.domain.port

interface WaitingPort {
    fun save(waiting: Waiting): Waiting
}

// Infrastructure Layer - Adapter (implementation)
package com.example.demo.infrastructure.persistence.adapter

@Repository
class WaitingPortAdapter(...) : WaitingPort {
    override fun save(waiting: Waiting): Waiting {
        // JPA, QueryDSL 등 인프라 기술 사용
    }
}
```

### 4.2 레이어 분리 규칙

**Package 구조 유지**:
```
com.example.demo
├── domain/              # 도메인 (순수)
│   ├── model/           # 도메인 모델
│   └── port/            # 포트 인터페이스
├── application/         # 애플리케이션 (유스케이스)
│   ├── service/         # 서비스
│   ├── dto/             # DTO
│   └── mapper/          # DTO 매퍼
├── infrastructure/      # 인프라 (어댑터)
│   ├── persistence/     # 영속성 어댑터
│   └── external/        # 외부 시스템 어댑터
├── presentation/        # 프레젠테이션
│   └── controller/      # REST 컨트롤러
└── common/              # 공통 (예외, 유틸리티)
```

**의존성 방향**:
```
Presentation → Application → Domain ← Infrastructure
                                ↑
                              Common
```

### 4.3 불변성 유지

**Java**:
```java
public record Waiting(...) {
    public Waiting enter() {
        return new Waiting(...);  // 새 인스턴스 생성
    }
}
```

**Kotlin**:
```kotlin
data class Waiting(...) {
    fun enter(): Waiting = copy(status = WaitingStatus.VISITED)  // copy() 메서드
}
```

✅ **동일한 불변성 보장**

---

## 5. Java vs Kotlin 매핑

### 5.1 언어 기능 매핑표

| Java 기능 | Kotlin 변환 | 비고 |
|-----------|-------------|------|
| `record` | `data class` | 불변 데이터 클래스 |
| `sealed interface` | `sealed interface` | 타입 안전 계층 구조 |
| `Optional<T>` | `T?` | Null safety |
| `switch` (pattern matching) | `when` | 더 강력한 패턴 매칭 |
| `@RequiredArgsConstructor` | Primary constructor | 생성자 자동 생성 |
| `@Getter` | Property | Getter 자동 생성 |
| `@NoArgsConstructor` | `no-arg` plugin | JPA용 기본 생성자 |
| `@AllArgsConstructor` | Primary constructor | 모든 필드 생성자 |
| `@Builder` | `copy()` or `@Builder` | 빌더 패턴 (DSL 가능) |
| `Stream API` | Collection functions | `map`, `filter`, `reduce` 등 |
| `static` method | Companion object | 정적 메서드/필드 |
| `final` | `val` | 불변 변수 |
| Anonymous class | Lambda or object | 간결한 익명 객체 |

### 5.2 Spring 어노테이션 매핑

| Java | Kotlin | 비고 |
|------|--------|------|
| `@Service` | `@Service` | 동일 |
| `@Repository` | `@Repository` | 동일 |
| `@Component` | `@Component` | 동일 |
| `@Transactional` | `@Transactional` | 동일 (open 함수에만 적용) |
| `@Valid` | `@Valid` | 동일 |
| Bean Validation (`@NotNull`) | `@field:NotNull` | 필드 타겟 지정 |
| `@Autowired` | Constructor injection | Primary constructor 권장 |

### 5.3 JPA 어노테이션 주의사항

**Java**:
```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class WaitingEntity { ... }
```

**Kotlin**:
```kotlin
// build.gradle.kts
plugins {
    kotlin("plugin.jpa") version "2.1.0"  // no-arg 자동 생성
}

// WaitingEntity.kt
@Entity
class WaitingEntity(...) {
    // no-arg plugin이 자동으로 기본 생성자 생성
}
```

**주의**:
- ⚠️ JPA 엔티티는 `data class` 사용 금지 (프록시 이슈)
- ⚠️ `open` 키워드 필요 (kotlin-allopen 플러그인 자동 처리)
- ⚠️ Lazy loading 필드는 `lateinit var` 또는 nullable

---

## 6. 검증 전략

### 6.1 단계별 검증 체크리스트

**각 Phase 완료 후 필수 검증**:

#### **1. 컴파일 검증**
```bash
./gradlew clean build
```
- ✅ 컴파일 에러 0개
- ✅ 경고(Warning) 최소화

#### **2. 테스트 검증**
```bash
./gradlew test
```
- ✅ 기존 테스트 100% 통과
- ✅ 테스트 커버리지 유지 (라인 커버리지 ≥ 기존 수준)

#### **3. 통합 테스트**
```bash
./gradlew bootRun
```
- ✅ 애플리케이션 정상 기동
- ✅ 헬스체크 통과 (`/actuator/health`)
- ✅ Swagger UI 정상 접근 (`/swagger-ui.html`)

#### **4. API 계약 테스트**
- ✅ 모든 엔드포인트 응답 형식 동일
- ✅ HTTP 상태 코드 동일
- ✅ 에러 응답 형식 동일

#### **5. 데이터베이스 검증**
- ✅ 스키마 변경 없음 (DDL 동일)
- ✅ 쿼리 결과 동일 (QueryDSL)
- ✅ N+1 문제 없음

### 6.2 회귀 테스트 전략

**목표**: 기존과 **정확히 동일한 동작** 보장

#### **방법 1: Golden Master Testing**

**개념**: Java 버전의 API 응답을 "golden master"로 저장하고, Kotlin 버전과 비교

**구현**:
```kotlin
@Test
fun `API 응답 회귀 테스트`() {
    // 1. Java 버전 응답 (golden master)
    val expectedJson = """
    {
      "message": "대기 신청 성공",
      "data": {
        "waitingId": 1,
        "waitingNumber": 1,
        "peopleCount": 2,
        "status": "WAITING"
      }
    }
    """.trimIndent()

    // 2. Kotlin 버전 응답
    val response = mockMvc.perform(
        post("/api/popups/1/waitings")
            .contentType(MediaType.APPLICATION_JSON)
            .content(requestJson)
    ).andReturn().response.contentAsString

    // 3. JSON 비교
    JSONAssert.assertEquals(expectedJson, response, JSONCompareMode.STRICT)
}
```

#### **방법 2: Property-Based Testing (선택적)**

**도구**: Kotest Property Testing

```kotlin
@Test
fun `대기 번호는 항상 양수`() = checkAll(
    iterations = 100,
    Arb.int(1..100)  // 팝업 ID
) { popupId ->
    val waiting = createWaiting(popupId)
    waiting.waitingNumber shouldBeGreaterThan 0
}
```

#### **방법 3: Snapshot Testing**

**도구**: Approval Tests (ApprovalTests.Java)

```kotlin
@Test
fun `대기 생성 응답 스냅샷 테스트`() {
    val response = waitingService.createWaiting(1L, request, 1L)
    Approvals.verify(objectMapper.writeValueAsString(response))
}
```

### 6.3 성능 회귀 테스트

**목표**: 성능 저하 없음 (± 5% 이내)

**측정 항목**:
1. API 응답 시간 (p50, p95, p99)
2. 데이터베이스 쿼리 수
3. 메모리 사용량
4. 애플리케이션 시작 시간

**도구**:
- JMeter / Gatling (부하 테스트)
- Spring Boot Actuator (메트릭)
- JProfiler / YourKit (프로파일링)

**예시**:
```kotlin
@Test
fun `대기 생성 성능 테스트`() {
    val startTime = System.currentTimeMillis()

    repeat(100) {
        waitingService.createWaiting(1L, request, it.toLong())
    }

    val elapsedTime = System.currentTimeMillis() - startTime

    // 100개 생성이 1초 이내 완료되어야 함
    assertThat(elapsedTime).isLessThan(1000)
}
```

### 6.4 검증 자동화

**CI/CD 파이프라인에 포함**:

```yaml
# .github/workflows/kotlin-migration-validation.yml
name: Kotlin Migration Validation

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up JDK 21
        uses: actions/setup-java@v3
        with:
          java-version: 21

      - name: Build with Gradle
        run: ./gradlew clean build

      - name: Run tests
        run: ./gradlew test

      - name: Check test coverage
        run: ./gradlew jacocoTestReport

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3

      - name: Integration tests
        run: ./gradlew bootRun & sleep 10 && curl http://localhost:8080/actuator/health
```

---

## 7. 빌드 및 의존성 관리

### 7.1 Gradle 설정 변경

**Before (build.gradle - Java)**:
```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.4.6'
    id 'io.spring.dependency-management' version '1.1.7'
}

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

dependencies {
    // Spring Boot
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'

    // QueryDSL
    implementation 'io.github.openfeign.querydsl:querydsl-jpa:6.10.1'
    annotationProcessor 'io.github.openfeign.querydsl:querydsl-apt:6.10.1:jakarta'

    // Lombok
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    // Test
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

**After (build.gradle.kts - Kotlin)**:
```kotlin
plugins {
    kotlin("jvm") version "2.1.0"
    kotlin("plugin.spring") version "2.1.0"      // allopen for @Component, @Service, etc.
    kotlin("plugin.jpa") version "2.1.0"         // no-arg for @Entity
    kotlin("kapt") version "2.1.0"               // Annotation processing
    id("org.springframework.boot") version "3.4.6"
    id("io.spring.dependency-management") version "1.1.7"
}

group = "com.example"
version = "0.0.1-SNAPSHOT"

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

kotlin {
    compilerOptions {
        freeCompilerArgs.addAll("-Xjsr305=strict")  // Strict null safety
        jvmTarget.set(org.jetbrains.kotlin.gradle.dsl.JvmTarget.JVM_21)
    }
}

// Java + Kotlin 동시 컴파일 (마이그레이션 중)
sourceSets {
    main {
        java.srcDirs("src/main/java", "src/main/kotlin")
    }
    test {
        java.srcDirs("src/test/java", "src/test/kotlin")
    }
}

dependencies {
    // Kotlin
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib")
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")  // JSON serialization

    // Spring Boot
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    implementation("org.springframework.boot:spring-boot-starter-security")
    implementation("org.springframework.boot:spring-boot-starter-oauth2-client")

    // QueryDSL (Kotlin 지원)
    implementation("io.github.openfeign.querydsl:querydsl-jpa:6.10.1")
    kapt("io.github.openfeign.querydsl:querydsl-apt:6.10.1:jakarta")
    kapt("org.springframework.boot:spring-boot-configuration-processor")  // @ConfigurationProperties

    // Database
    runtimeOnly("org.postgresql:postgresql")
    runtimeOnly("com.h2database:h2")

    // JWT
    implementation("io.jsonwebtoken:jjwt-api:0.12.6")
    runtimeOnly("io.jsonwebtoken:jjwt-impl:0.12.6")
    runtimeOnly("io.jsonwebtoken:jjwt-jackson:0.12.6")

    // Documentation
    implementation("org.springdoc:springdoc-openapi-starter-webmvc-ui:2.8.9")

    // Test
    testImplementation("org.springframework.boot:spring-boot-starter-test") {
        exclude(group = "org.junit.vintage", module = "junit-vintage-engine")
    }
    testImplementation("io.kotest:kotest-runner-junit5:5.8.0")
    testImplementation("io.kotest:kotest-assertions-core:5.8.0")
    testImplementation("io.mockk:mockk:1.13.9")  // Kotlin mocking library
}

tasks.withType<Test> {
    useJUnitPlatform()

    // 병렬 실행
    maxParallelForks = Runtime.getRuntime().availableProcessors()
    forkEvery = 1
}

// QueryDSL 생성 파일 디렉토리
kotlin.sourceSets.main {
    kotlin.srcDir("build/generated/source/kapt/main")
}
```

### 7.2 추가 의존성

**Kotlin 전용**:
```kotlin
// Jackson Kotlin 모듈 (JSON 직렬화)
implementation("com.fasterxml.jackson.module:jackson-module-kotlin")

// Kotest (선택적 - 더 나은 테스트 DSL)
testImplementation("io.kotest:kotest-runner-junit5:5.8.0")
testImplementation("io.kotest:kotest-assertions-core:5.8.0")

// MockK (선택적 - Kotlin 전용 모킹 라이브러리)
testImplementation("io.mockk:mockk:1.13.9")
```

### 7.3 Kotlin 컴파일러 플러그인

**allopen (Spring 지원)**:
```kotlin
// Spring 어노테이션이 붙은 클래스를 자동으로 open으로 만듦
// @Component, @Service, @Repository, @Controller 등
kotlin("plugin.spring") version "2.1.0"
```

**no-arg (JPA 지원)**:
```kotlin
// @Entity가 붙은 클래스에 자동으로 no-arg 생성자 추가
kotlin("plugin.jpa") version "2.1.0"
```

**kapt (Annotation Processing)**:
```kotlin
// QueryDSL, Lombok 등 어노테이션 프로세싱
kotlin("kapt") version "2.1.0"
```

---

## 8. 위험 요소 및 대응 방안

### 8.1 위험 요소

| 위험 | 영향도 | 확률 | 대응 방안 |
|------|--------|------|-----------|
| **1. JPA Entity 프록시 이슈** | 높음 | 중간 | - JPA 엔티티에 `data class` 사용 금지<br>- `no-arg`, `allopen` 플러그인 필수<br>- Lazy loading 테스트 철저 |
| **2. QueryDSL 호환성 문제** | 중간 | 낮음 | - kapt 플러그인 사용<br>- Q클래스 생성 확인<br>- 기존 쿼리와 결과 비교 테스트 |
| **3. Null safety 오해** | 중간 | 중간 | - `Optional<T>` → `T?` 매핑 정확히<br>- Platform types 주의<br>- `!!` 연산자 최소화 |
| **4. 성능 저하** | 중간 | 낮음 | - 벤치마크 테스트<br>- Inline 함수 활용<br>- Reflection 최소화 |
| **5. Spring AOP 이슈** | 중간 | 낮음 | - `open` 함수만 AOP 적용됨 (allopen 플러그인)<br>- `@Transactional` 검증 |
| **6. 테스트 누락** | 높음 | 중간 | - 각 Phase마다 전체 테스트 실행<br>- 코드 커버리지 모니터링<br>- 회귀 테스트 자동화 |
| **7. 팀 학습 곡선** | 낮음 | 높음 | - Kotlin 교육 세션<br>- 코드 리뷰 강화<br>- 스타일 가이드 작성 |
| **8. 롤백 어려움** | 높음 | 낮음 | - Git branch 전략 (feature branch)<br>- 각 Phase마다 커밋<br>- 점진적 마이그레이션 (Java-Kotlin 공존) |

### 8.2 구체적 대응 방안

#### **1. JPA Entity 프록시 이슈**

**문제**:
```kotlin
// ❌ 잘못된 예시
@Entity
data class WaitingEntity(...)  // data class는 final 메서드 생성 → 프록시 불가
```

**해결**:
```kotlin
// ✅ 올바른 예시
@Entity
class WaitingEntity(...)  // 일반 class + allopen 플러그인

// build.gradle.kts
kotlin("plugin.jpa") version "2.1.0"  // no-arg, allopen 자동 적용
```

**검증**:
```kotlin
@Test
fun `Lazy loading 테스트`() {
    val waiting = waitingRepository.findById(1L).orElseThrow()

    // Lazy loaded 필드 접근
    assertDoesNotThrow {
        waiting.popup.name  // 프록시가 정상 작동해야 함
    }
}
```

#### **2. QueryDSL 호환성**

**설정**:
```kotlin
// build.gradle.kts
dependencies {
    implementation("io.github.openfeign.querydsl:querydsl-jpa:6.10.1")
    kapt("io.github.openfeign.querydsl:querydsl-apt:6.10.1:jakarta")
}

kotlin.sourceSets.main {
    kotlin.srcDir("build/generated/source/kapt/main")  // Q클래스 경로
}
```

**검증**:
```bash
./gradlew clean build
ls build/generated/source/kapt/main/com/example/demo/infrastructure/persistence/entity
# QWaitingEntity.java, QPopupEntity.java 등 확인
```

#### **3. Null Safety**

**Platform Types 주의**:
```kotlin
// Java 코드에서 @Nullable, @NotNull 없는 경우 → Platform Type (T!)
val member: Member = memberPort.findById(1L)  // ⚠️ Platform type
member.name  // NPE 가능

// ✅ 명시적 null 처리
val member: Member? = memberPort.findById(1L)  // Nullable
member?.name  // Safe call
```

**JSR-305 엄격 모드**:
```kotlin
// build.gradle.kts
kotlin {
    compilerOptions {
        freeCompilerArgs.addAll("-Xjsr305=strict")  // Platform type 경고
    }
}
```

#### **4. 성능 최적화**

**Inline 함수 활용**:
```kotlin
// Higher-order function을 inline으로 → 오버헤드 제거
inline fun <T> transaction(block: () -> T): T {
    // Transaction 로직
    return block()
}
```

**Reflection 최소화**:
```kotlin
// ❌ Reflection 사용
val kClass = Waiting::class
val properties = kClass.memberProperties

// ✅ 직접 접근
waiting.waitingNumber
```

#### **5. Spring AOP 검증**

**테스트**:
```kotlin
@Test
fun `@Transactional AOP 적용 확인`() {
    // Given
    val waiting = createWaiting()

    // When
    assertThrows<RuntimeException> {
        waitingService.enterWaiting(waiting.id!!)
        throw RuntimeException("Rollback test")
    }

    // Then
    val result = waitingRepository.findById(waiting.id!!)
    assertThat(result).isEmpty  // Rollback 되어야 함
}
```

---

## 9. 마일스톤 및 체크포인트

### 9.1 전체 마일스톤

```
┌─────────────────────────────────────────────────────────────┐
│  Milestone 0: 준비 (2-4시간)                                │
│  - Gradle 설정                                              │
│  - 기준선 설정                                               │
│  ✅ 검증: 빌드 성공, 모든 테스트 통과                        │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Milestone 1: Common Layer (4-6시간)                        │
│  - 예외 처리 (ErrorType, BusinessException)                 │
│  - JWT, Security                                            │
│  ✅ 검증: 예외 처리 테스트 통과                             │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Milestone 2: Domain Layer (8-12시간)                       │
│  - 도메인 모델 (Waiting, Popup, Notification, etc.)         │
│  - 포트 인터페이스                                           │
│  ✅ 검증: 도메인 로직 테스트 통과, 순수성 유지               │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Milestone 3: Infrastructure Layer (10-14시간)              │
│  - JPA 엔티티                                                │
│  - Port 어댑터 (QueryDSL)                                    │
│  - 외부 어댑터 (SSE, Email)                                  │
│  ✅ 검증: 레포지토리 테스트, 쿼리 결과 동일                  │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Milestone 4: Application Layer (12-16시간)                 │
│  - 서비스 (17개)                                             │
│  - DTO, 매퍼                                                 │
│  ✅ 검증: 서비스 테스트 (930줄) 통과                         │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Milestone 5: Presentation Layer (6-8시간)                  │
│  - 컨트롤러 (9개)                                            │
│  - OAuth 핸들러                                              │
│  ✅ 검증: API 통합 테스트, Swagger 문서 확인                 │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Milestone 6: 테스트 코드 (8-12시간)                        │
│  - 9개 테스트 파일                                           │
│  ✅ 검증: 전체 테스트 스위트 통과, 커버리지 유지             │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│  Milestone 7: 최종 검증 및 정리 (4-6시간)                   │
│  - 회귀 테스트                                               │
│  - 성능 테스트                                               │
│  - 문서 업데이트                                             │
│  - Java 파일 삭제                                            │
│  ✅ 검증: 100% 동작 호환성 확인                              │
└─────────────────────────────────────────────────────────────┘
```

**총 예상 시간**: **54-78시간** (약 7-10일, 1인 기준)

### 9.2 체크포인트

**각 Milestone 완료 후 필수 확인 사항**:

#### ✅ **Checkpoint 1: 빌드 성공**
```bash
./gradlew clean build
```
- [ ] 컴파일 에러 0개
- [ ] Warning 최소화

#### ✅ **Checkpoint 2: 테스트 통과**
```bash
./gradlew test
```
- [ ] 모든 기존 테스트 통과
- [ ] 신규 테스트 추가 (Kotlin 특화)
- [ ] 코드 커버리지 ≥ 기존 수준

#### ✅ **Checkpoint 3: 통합 테스트**
```bash
./gradlew bootRun
```
- [ ] 애플리케이션 정상 기동
- [ ] `/actuator/health` 응답 200 OK
- [ ] `/swagger-ui.html` 접근 가능
- [ ] 주요 API 엔드포인트 테스트 (Postman/Insomnia)

#### ✅ **Checkpoint 4: 코드 품질**
```bash
./gradlew detekt  # Kotlin 정적 분석 (선택적)
```
- [ ] Kotlin 스타일 가이드 준수
- [ ] `!!` 연산자 최소화 (0개 권장)
- [ ] Magic number 제거
- [ ] 함수 길이 적절 (≤ 30줄 권장)

#### ✅ **Checkpoint 5: 문서 업데이트**
- [ ] README.md 업데이트 (Kotlin 빌드 방법)
- [ ] API 문서 확인 (Swagger)
- [ ] 아키텍처 문서 업데이트 (`docs/architecture/`)

#### ✅ **Checkpoint 6: Git 커밋**
```bash
git add .
git commit -m "feat: Migrate [Layer] to Kotlin"
git push origin feature/kotlin-migration
```
- [ ] 원자적 커밋 (레이어별)
- [ ] 커밋 메시지 명확
- [ ] PR 생성 및 리뷰

---

## 10. 추가 권장 사항

### 10.1 Kotlin 스타일 가이드

**공식 가이드 준수**: [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html)

**주요 규칙**:
1. **Naming**:
   - 클래스: PascalCase (`WaitingService`)
   - 함수/변수: camelCase (`createWaiting`)
   - 상수: UPPER_SNAKE_CASE (`MAX_WAITING_COUNT`)

2. **들여쓰기**: 4칸 (기존 Java와 동일)

3. **함수 길이**: ≤ 30줄 권장

4. **Null safety**:
   - `!!` 연산자 최소화 (0개 목표)
   - `?.let` 활용
   - Elvis 연산자 `?:` 활용

5. **Expression body**:
   ```kotlin
   // ✅ 간단한 함수
   fun isActive(): Boolean = status == PopupStatus.ACTIVE

   // ❌ 복잡한 로직
   fun createWaiting() = waitingPort.save(...)  // Block body 사용
   ```

### 10.2 도구 추천

**1. IntelliJ IDEA**: Kotlin 공식 IDE (최고의 지원)

**2. Detekt**: Kotlin 정적 분석
```kotlin
// build.gradle.kts
plugins {
    id("io.gitlab.arturbosch.detekt") version "1.23.4"
}

detekt {
    buildUponDefaultConfig = true
    config.setFrom("$projectDir/config/detekt.yml")
}
```

**3. ktlint**: Kotlin 코드 포매터
```bash
./gradlew ktlintFormat
```

**4. Kotest**: 강력한 테스트 프레임워크 (선택적)

### 10.3 교육 자료

**팀 학습**:
1. [Kotlin 공식 문서](https://kotlinlang.org/docs/home.html)
2. [Kotlin for Java Developers (Coursera)](https://www.coursera.org/learn/kotlin-for-java-developers)
3. [Spring Boot with Kotlin](https://spring.io/guides/tutorials/spring-boot-kotlin/)
4. 내부 세션: "Java → Kotlin 마이그레이션 Best Practices"

---

## 11. 결론

### 11.1 요약

이 계획서는 **Java 21 + Spring Boot 3.4.6** 기반의 **헥사고날 아키텍처**를 **Kotlin**으로 마이그레이션하는 상세한 로드맵을 제공합니다.

**핵심 원칙**:
1. ✅ **점진적 마이그레이션** (레이어별 순차 전환)
2. ✅ **아키텍처 철학 100% 유지** (헥사고날, 레이어 분리, 도메인 순수성)
3. ✅ **동작 호환성 보장** (기존과 정확히 동일한 동작)
4. ✅ **철저한 검증** (테스트, 회귀, 성능)

**예상 효과**:
- 📉 코드 라인 수 감소 (184개 → 140-150개 파일, ~20% 감소)
- 🛡️ Null safety 강화 (런타임 NPE 감소)
- 🚀 개발 생산성 향상 (간결한 문법, 강력한 표준 라이브러리)
- 📈 유지보수성 향상 (불변성, 타입 안전성)

**예상 기간**: **7-10일** (1인 기준, 54-78시간)

### 11.2 Next Steps

1. **Phase 0 시작**: Gradle 설정 및 기준선 설정
2. **Common Layer 마이그레이션**
3. **단계별 진행** (Domain → Infrastructure → Application → Presentation → Test)
4. **지속적 검증** (각 Phase마다 전체 테스트)
5. **최종 검증 및 배포**

---

**문서 버전**: 1.0
**작성일**: 2025-11-27
**작성자**: Claude (Kotlin Migration Planning Agent)
**상태**: Draft → Review 대기
