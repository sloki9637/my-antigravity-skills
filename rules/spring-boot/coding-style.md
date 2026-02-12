---
trigger: always_on
---

# Spring Boot + Spring Security + JPA Coding Rules

> These rules are **mandatory**.
> Do not infer intent. Follow exactly.

---

## 1. Responsibility Separation

- **Controller** MUST handle:
  - HTTP request/response mapping
  - Input validation (`@Valid`)
  - Calling Service methods
  - Returning `ResponseEntity` or DTO

- **Controller** MUST NOT:
  - Contain business logic
  - Directly access Repository
  - Perform data transformation
  - Handle transactions

- **Service** MUST handle:
  - Business logic
  - Transaction management (`@Transactional`)
  - Domain object orchestration
  - DTO ↔ Entity conversion

- **Service** MUST NOT:
  - Access `HttpServletRequest` or `HttpServletResponse`
  - Return Entity directly to Controller
  - Contain SQL or JPQL (use Repository)

- **Repository** MUST handle:
  - Data access logic
  - Custom queries (JPQL, QueryDSL)

- **Repository** MUST NOT:
  - Contain business logic
  - Perform DTO conversion

---

## 2. Directory Structure

```txt
src/main/java/com/example/project/
  global/
    config/              # Configuration classes
    security/            # Spring Security config, filters, handlers
    exception/           # Global exception classes and handler
    common/              # Common utilities, base entities
  domain/
    [feature]/
      controller/        # REST controllers
      service/           # Business logic
      repository/        # Data access
      entity/            # JPA entities
      dto/               # Request/Response DTOs
      enums/             # Domain-specific enums
```

Rules:

- All JPA entities **MUST** be in `entity/`
- All DTOs **MUST** be in `dto/`
- Security-related classes **MUST** be in `global/security/`
- Configuration classes **MUST** be in `global/config/`
- Domain packages **MUST** be organized by feature, not by layer
- Cross-cutting concerns **MUST** be in `global/`

---

## 3. Naming Conventions

### Classes

- Controller: `*Controller` (e.g., `CafeController`)
- Service interface: `*Service` (e.g., `CafeService`)
- Service impl: `*ServiceImpl` (e.g., `CafeServiceImpl`)
- Repository: `*Repository` (e.g., `CafeRepository`)
- Entity: domain noun (e.g., `Cafe`, `Member`)
- Request DTO: `*Request` (e.g., `CafeCreateRequest`)
- Response DTO: `*Response` (e.g., `CafeDetailResponse`)
- Exception: `*Exception` (e.g., `CafeNotFoundException`)
- Enum: domain noun (e.g., `CafeStatus`, `MemberRole`)

### Methods

- Service: business action (e.g., `registerCafe`, `cancelOrder`)
- Repository: `findBy*`, `existsBy*`, `deleteBy*`
- Controller: HTTP method mapping (e.g., `createCafe`, `getCafe`)

Examples:

- ✅ `CafeController`, `CafeService`, `CafeRepository`
- ✅ `CafeCreateRequest`, `CafeDetailResponse`
- ❌ `CafeDto`, `CafeReq`, `CafeRes`
- ❌ `CafeManager`, `CafeHelper` (ambiguous responsibility)

---

## 4. Entity Rules

### Base Entity

- Auditing fields **MUST** be in a `@MappedSuperclass` base entity
- `createdAt`, `updatedAt` **MUST** be present on all entities

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@Getter
public abstract class BaseEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

### Entity Design

- Entities **MUST** use `@Getter` only (Lombok)
- `@Setter` is **FORBIDDEN** on entities
- State changes **MUST** be through domain methods
- `@NoArgsConstructor(access = AccessLevel.PROTECTED)` is **MANDATORY**
- `@Builder` **MUST** be on a static factory method or constructor, not on the class

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Cafe extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private CafeStatus status;

    @Builder
    private Cafe(String name, CafeStatus status) {
        this.name = name;
        this.status = status;
    }

    // Domain method for state change
    public void close() {
        this.status = CafeStatus.CLOSED;
    }
}
```

### Relationship Mapping

- `@ManyToOne` **MUST** use `FetchType.LAZY`
- `@OneToMany` **MUST** use `FetchType.LAZY` (default)
- Bidirectional relationships **MUST** have a convenience method
- `cascade` **MUST** be explicitly defined, not `CascadeType.ALL` unless justified
- `orphanRemoval` **MUST** be explicitly set when needed

```java
@Entity
public class Menu extends BaseEntity {

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "cafe_id", nullable = false)
    private Cafe cafe;
}

@Entity
public class Cafe extends BaseEntity {

    @OneToMany(mappedBy = "cafe", cascade = CascadeType.PERSIST, orphanRemoval = true)
    private List<Menu> menus = new ArrayList<>();

    public void addMenu(Menu menu) {
        menus.add(menu);
        menu.setCafe(this);
    }
}
```

### ID Strategy

- **MUST** use `GenerationType.IDENTITY` for MySQL/MariaDB
- **MUST** use `GenerationType.SEQUENCE` for PostgreSQL/Oracle
- `@GeneratedValue` **MUST** be explicitly specified

---

## 5. DTO Rules

- Request/Response DTO **MUST** be separate classes
- DTO **MUST NOT** extend or embed Entity
- Entity **MUST NOT** be exposed in API response directly
- DTO conversion **MUST** happen in Service layer or DTO's static factory method

### Request DTO

- **MUST** include validation annotations
- **SHOULD** use `record` for immutable request objects

```java
public record CafeCreateRequest(
    @NotBlank(message = "카페 이름은 필수입니다")
    @Size(max = 100)
    String name,

    @NotBlank(message = "주소는 필수입니다")
    String address
) {}
```

### Response DTO

- **MUST** have a static factory method `from(Entity)` or `of(...)`
- **SHOULD** use `record` for immutable response objects

```java
public record CafeDetailResponse(
    Long id,
    String name,
    String address,
    CafeStatus status,
    LocalDateTime createdAt
) {
    public static CafeDetailResponse from(Cafe cafe) {
        return new CafeDetailResponse(
            cafe.getId(),
            cafe.getName(),
            cafe.getAddress(),
            cafe.getStatus(),
            cafe.getCreatedAt()
        );
    }
}
```

---

## 6. Controller Rules

- **MUST** use `@RestController`
- **MUST** define base path with `@RequestMapping`
- **MUST** return `ResponseEntity<T>`
- **MUST** use `@Valid` for request body validation
- Path variables **MUST** use descriptive names

```java
@RestController
@RequestMapping("/api/v1/cafes")
@RequiredArgsConstructor
public class CafeController {

    private final CafeService cafeService;

    @PostMapping
    public ResponseEntity<CafeDetailResponse> createCafe(
            @Valid @RequestBody CafeCreateRequest request) {
        CafeDetailResponse response = cafeService.createCafe(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }

    @GetMapping("/{cafeId}")
    public ResponseEntity<CafeDetailResponse> getCafe(
            @PathVariable Long cafeId) {
        CafeDetailResponse response = cafeService.getCafe(cafeId);
        return ResponseEntity.ok(response);
    }
}
```

---

## 7. Service Rules

- **MUST** use interface + implementation pattern when polymorphism is needed
- **MUST** use `@RequiredArgsConstructor` for dependency injection
- **MUST** use `@Transactional(readOnly = true)` at class level for read-heavy services
- Write operations **MUST** use `@Transactional`
- **MUST NOT** catch exceptions silently

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class CafeService {

    private final CafeRepository cafeRepository;

    @Transactional
    public CafeDetailResponse createCafe(CafeCreateRequest request) {
        Cafe cafe = Cafe.builder()
                .name(request.name())
                .build();
        Cafe savedCafe = cafeRepository.save(cafe);
        return CafeDetailResponse.from(savedCafe);
    }

    public CafeDetailResponse getCafe(Long cafeId) {
        Cafe cafe = cafeRepository.findById(cafeId)
                .orElseThrow(() -> new CafeNotFoundException(cafeId));
        return CafeDetailResponse.from(cafe);
    }
}
```

---

## 8. Repository Rules

- **MUST** extend `JpaRepository<Entity, IdType>`
- Custom queries **MUST** use `@Query` with JPQL or QueryDSL
- Native queries are **DISCOURAGED** unless performance-critical
- Bulk operations **MUST** use `@Modifying` with `clearAutomatically = true`
- Pagination **MUST** use `Pageable` parameter

```java
public interface CafeRepository extends JpaRepository<Cafe, Long> {

    Optional<Cafe> findByName(String name);

    boolean existsByName(String name);

    @Query("SELECT c FROM Cafe c WHERE c.status = :status")
    List<Cafe> findAllByStatus(@Param("status") CafeStatus status);

    @Query("SELECT c FROM Cafe c JOIN FETCH c.menus WHERE c.id = :id")
    Optional<Cafe> findByIdWithMenus(@Param("id") Long id);

    @Modifying(clearAutomatically = true)
    @Query("UPDATE Cafe c SET c.status = :status WHERE c.id = :id")
    int updateStatus(@Param("id") Long id, @Param("status") CafeStatus status);
}
```

---

## 9. Exception Handling

- **MUST** use a global exception handler (`@RestControllerAdvice`)
- Custom exceptions **MUST** extend a common base exception
- HTTP status codes **MUST** be semantically correct
- Error response **MUST** have a consistent format

```java
@Getter
public abstract class BusinessException extends RuntimeException {

    private final HttpStatus status;

    protected BusinessException(String message, HttpStatus status) {
        super(message);
        this.status = status;
    }
}

public class CafeNotFoundException extends BusinessException {

    public CafeNotFoundException(Long cafeId) {
        super("카페를 찾을 수 없습니다. ID: " + cafeId, HttpStatus.NOT_FOUND);
    }
}
```

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(BusinessException e) {
        return ResponseEntity.status(e.getStatus())
                .body(ErrorResponse.of(e.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(
            MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().stream()
                .map(FieldError::getDefaultMessage)
                .collect(Collectors.joining(", "));
        return ResponseEntity.badRequest()
                .body(ErrorResponse.of(message));
    }
}
```

---

## 10. Spring Security Rules

### Configuration

- **MUST** use `SecurityFilterChain` bean (not `WebSecurityConfigurerAdapter`)
- **MUST** disable CSRF for stateless API (JWT-based)
- **MUST** set `SessionCreationPolicy.STATELESS` for JWT
- CORS **MUST** be explicitly configured

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtAuthenticationFilter;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
                .csrf(AbstractHttpConfigurer::disable)
                .sessionManagement(session ->
                        session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/api/v1/auth/**").permitAll()
                        .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                        .anyRequest().authenticated())
                .addFilterBefore(jwtAuthenticationFilter,
                        UsernamePasswordAuthenticationFilter.class)
                .build();
    }
}
```

### Authentication

- JWT filter **MUST** extend `OncePerRequestFilter`
- Password **MUST** be encoded with `BCryptPasswordEncoder`
- Sensitive information **MUST NOT** be included in JWT payload
- Token expiration **MUST** be configured externally (`application.yml`)

### Authorization

- Method-level security **MUST** use `@PreAuthorize` when needed
- Role-based access **MUST** use `hasRole()` or `hasAuthority()`
- Custom permission checks **MUST** be in a separate component

```java
@PreAuthorize("hasRole('ADMIN') or @cafePermissionEvaluator.isOwner(#cafeId)")
public CafeDetailResponse updateCafe(Long cafeId, CafeUpdateRequest request) {
    // ...
}
```

---

## 11. Transaction Rules

- Read-only operations **MUST** use `@Transactional(readOnly = true)`
- Write operations **MUST** use `@Transactional`
- Transaction boundaries **MUST** be at Service layer
- `@Transactional` on private methods is **FORBIDDEN** (proxy limitation)
- Self-invocation of `@Transactional` methods is **FORBIDDEN**

---

## 12. N+1 Problem Prevention

- **MUST** use `JOIN FETCH` or `@EntityGraph` for eager loading when needed
- **MUST** verify query count in tests for critical paths
- **MUST** use `@BatchSize` or `default_batch_fetch_size` for collection loading
- Lazy loading outside transaction is **FORBIDDEN** (use DTO projection or fetch join)

```java
// Fetch Join
@Query("SELECT c FROM Cafe c JOIN FETCH c.menus WHERE c.id = :id")
Optional<Cafe> findByIdWithMenus(@Param("id") Long id);

// EntityGraph
@EntityGraph(attributePaths = {"menus"})
Optional<Cafe> findWithMenusById(Long id);
```

---

## 13. Configuration Rules

- Secrets **MUST NOT** be hardcoded (use environment variables or external config)
- Profile-specific config **MUST** use `application-{profile}.yml`
- `application.yml` **MUST** contain only default/shared settings
- Custom properties **MUST** use `@ConfigurationProperties`

```yaml
# application.yml
spring:
  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        default_batch_fetch_size: 100
    open-in-view: false
```

- `open-in-view` **MUST** be `false`
- `ddl-auto` **MUST** be `validate` or `none` in production

---

## 14. Testing Rules

### Unit Tests

- Service tests **MUST** use `@ExtendWith(MockitoExtension.class)`
- **MUST** mock dependencies with `@Mock` and `@InjectMocks`
- Test method names **MUST** describe behavior in Korean or English

```java
@ExtendWith(MockitoExtension.class)
class CafeServiceTest {

    @Mock
    private CafeRepository cafeRepository;

    @InjectMocks
    private CafeService cafeService;

    @Test
    @DisplayName("카페를 정상적으로 생성한다")
    void createCafe_success() {
        // given
        CafeCreateRequest request = new CafeCreateRequest("테스트 카페", "서울시");
        Cafe cafe = Cafe.builder().name("테스트 카페").build();
        given(cafeRepository.save(any(Cafe.class))).willReturn(cafe);

        // when
        CafeDetailResponse response = cafeService.createCafe(request);

        // then
        assertThat(response.name()).isEqualTo("테스트 카페");
        then(cafeRepository).should().save(any(Cafe.class));
    }
}
```

### Integration Tests

- **MUST** use `@SpringBootTest` for full context tests
- Repository tests **MUST** use `@DataJpaTest`
- **MUST** use `given / when / then` pattern
- Test data **MUST** be isolated per test

---

## 15. Dependency Injection Rules

- **MUST** use constructor injection (via `@RequiredArgsConstructor`)
- `@Autowired` field injection is **FORBIDDEN**
- `@Setter` injection is **FORBIDDEN**
- Circular dependencies are **FORBIDDEN** (redesign if encountered)

```java
// ✅ GOOD - Constructor injection
@Service
@RequiredArgsConstructor
public class CafeService {
    private final CafeRepository cafeRepository;
    private final MemberRepository memberRepository;
}

// ❌ BAD - Field injection
@Service
public class CafeService {
    @Autowired
    private CafeRepository cafeRepository;
}
```

---

## 16. API Versioning

- API paths **MUST** include version prefix: `/api/v1/`
- Version changes **MUST** be documented
- Breaking changes **MUST** increment the version number

---

## 17. Logging Rules

- **MUST** use SLF4J (`@Slf4j` from Lombok)
- **MUST NOT** use `System.out.println`
- Sensitive data **MUST NOT** be logged
- Log level guidelines:
  - `ERROR`: unexpected failures
  - `WARN`: recoverable issues
  - `INFO`: significant business events
  - `DEBUG`: development-time diagnostics

---

## 18. Rule Priority

1. **Security** - No vulnerabilities allowed
2. **Correctness** - Code must work as intended
3. **Maintainability** - Clean architecture boundaries
4. **Performance** - Optimize when necessary
5. **Convenience** - DX improvements (lowest priority)

---

## 19. Enforcement

- Any code violating these rules **MUST** be rejected
- No exceptions unless explicitly documented
- Rules trump personal preferences