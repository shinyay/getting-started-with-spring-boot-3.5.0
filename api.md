---
layout: page
title: "API リファレンス"
description: "Spring Boot 3.5.0 の主要APIと設定オプションの詳細リファレンス"
---

<div class="toc">
  <h3 class="toc-title">目次</h3>
</div>

## アノテーション リファレンス

### コア アノテーション

#### @SpringBootApplication
```java
@SpringBootApplication
// 以下と同等
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

**属性:**
- `exclude` - 除外する自動設定クラス
- `excludeName` - 除外する自動設定クラス名
- `scanBasePackages` - コンポーネントスキャンのベースパッケージ
- `scanBasePackageClasses` - コンポーネントスキャンのベースクラス

**例:**
```java
@SpringBootApplication(
    exclude = {DataSourceAutoConfiguration.class},
    scanBasePackages = {"com.example.core", "com.example.web"}
)
```

#### @Configuration
設定クラスを示すアノテーション。

```java
@Configuration
public class AppConfig {
    
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    
    @Bean
    @Primary
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:h2:mem:primary")
            .build();
    }
}
```

### Web アノテーション

#### @RestController
```java
@RestController
@RequestMapping("/api/v1")
public class UserController {
    
    @GetMapping("/users")
    public List<User> getUsers() { }
    
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) { }
    
    @PostMapping("/users")
    public User createUser(@RequestBody @Valid User user) { }
    
    @PutMapping("/users/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody @Valid User user) { }
    
    @DeleteMapping("/users/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteUser(@PathVariable Long id) { }
}
```

#### @RequestMapping
**HTTP メソッド:**
- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping`

**パラメータ:**
- `value` / `path` - URL パス
- `method` - HTTP メソッド
- `params` - リクエストパラメータ条件
- `headers` - ヘッダー条件
- `consumes` - Content-Type 条件
- `produces` - Accept 条件

```java
@GetMapping(
    value = "/users",
    params = {"page", "size"},
    headers = "X-API-Version=1.0",
    produces = MediaType.APPLICATION_JSON_VALUE
)
```

### データアクセス アノテーション

#### @Entity / @Table
```java
@Entity
@Table(name = "users", 
       indexes = {@Index(columnList = "email")},
       uniqueConstraints = {@UniqueConstraint(columnNames = {"username"})})
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true, length = 50)
    private String username;
    
    @Column(nullable = false, length = 100)
    private String email;
    
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    private LocalDateTime updatedAt;
}
```

#### @Repository
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmail(@Param("email") String email);
    
    @Query(value = "SELECT * FROM users WHERE created_at > ?1", nativeQuery = true)
    List<User> findUsersCreatedAfter(LocalDateTime date);
    
    @Modifying
    @Query("UPDATE User u SET u.lastLogin = :date WHERE u.id = :id")
    void updateLastLogin(@Param("id") Long id, @Param("date") LocalDateTime date);
}
```

### テスト アノテーション

#### @SpringBootTest
```java
@SpringBootTest(
    classes = Application.class,
    webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT,
    properties = {"spring.profiles.active=test"}
)
class IntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @LocalServerPort
    private int port;
}
```

#### テストスライス
```java
// Web レイヤーのみテスト
@WebMvcTest(UserController.class)
class UserControllerTest {
    @Autowired private MockMvc mockMvc;
    @MockBean private UserService userService;
}

// データアクセスレイヤーのみテスト
@DataJpaTest
class UserRepositoryTest {
    @Autowired private TestEntityManager entityManager;
    @Autowired private UserRepository userRepository;
}

// JSON シリアライゼーションテスト
@JsonTest
class UserJsonTest {
    @Autowired private JacksonTester<User> json;
}
```

## 設定プロパティ リファレンス

### サーバー設定

```yaml
server:
  port: 8080                    # サーバーポート
  servlet:
    context-path: /api          # コンテキストパス
    session:
      timeout: 30m              # セッションタイムアウト
  compression:
    enabled: true               # レスポンス圧縮
    mime-types: text/html,text/xml,text/plain,text/css,text/javascript,application/javascript,application/json
  ssl:
    enabled: true               # SSL有効化
    key-store: classpath:keystore.p12
    key-store-password: secret
    key-store-type: PKCS12
```

### データソース設定

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: user
    password: password
    driver-class-name: org.postgresql.Driver
    hikari:
      maximum-pool-size: 20     # 最大プール数
      minimum-idle: 5           # 最小アイドル数
      connection-timeout: 30000 # 接続タイムアウト(ms)
      idle-timeout: 600000      # アイドルタイムアウト(ms)
      max-lifetime: 1800000     # 最大生存時間(ms)
      leak-detection-threshold: 60000 # リーク検知閾値(ms)
  
  jpa:
    hibernate:
      ddl-auto: validate        # none|validate|update|create|create-drop
    show-sql: true              # SQL出力
    properties:
      hibernate:
        format_sql: true        # SQL整形
        dialect: org.hibernate.dialect.PostgreSQLDialect
        jdbc:
          batch_size: 20        # バッチサイズ
        order_inserts: true     # INSERT順序最適化
        order_updates: true     # UPDATE順序最適化
```

### ログ設定

```yaml
logging:
  level:
    root: INFO
    com.example: DEBUG
    org.springframework.web: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: logs/application.log
  logback:
    rollingpolicy:
      max-file-size: 100MB
      max-history: 30
      total-size-cap: 3GB
```

### Actuator 設定

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus,loggers,env
        exclude: shutdown
      base-path: /actuator
      cors:
        allowed-origins: "*"
        allowed-methods: GET,POST
  endpoint:
    health:
      show-details: always
      show-components: always
    info:
      enabled: true
    metrics:
      enabled: true
    prometheus:
      enabled: true
  metrics:
    export:
      prometheus:
        enabled: true
    distribution:
      percentiles-histogram:
        http.server.requests: true
      slo:
        http.server.requests: 50ms,100ms,200ms,300ms,500ms,1s
```

## Spring Boot スターター一覧

### Web 開発

| スターター | 説明 | 主要依存関係 |
|-----------|------|-------------|
| `spring-boot-starter-web` | Web アプリケーション開発 | Spring MVC, Tomcat, Jackson |
| `spring-boot-starter-webflux` | リアクティブ Web 開発 | Spring WebFlux, Netty |
| `spring-boot-starter-websocket` | WebSocket サポート | Spring WebSocket |
| `spring-boot-starter-thymeleaf` | Thymeleaf テンプレート | Thymeleaf |
| `spring-boot-starter-mustache` | Mustache テンプレート | Mustache |

### データアクセス

| スターター | 説明 | 主要依存関係 |
|-----------|------|-------------|
| `spring-boot-starter-data-jpa` | JPA データアクセス | Spring Data JPA, Hibernate |
| `spring-boot-starter-data-r2dbc` | リアクティブデータアクセス | Spring Data R2DBC |
| `spring-boot-starter-data-redis` | Redis データアクセス | Spring Data Redis, Lettuce |
| `spring-boot-starter-data-mongodb` | MongoDB データアクセス | Spring Data MongoDB |
| `spring-boot-starter-data-elasticsearch` | Elasticsearch | Spring Data Elasticsearch |

### メッセージング

| スターター | 説明 | 主要依存関係 |
|-----------|------|-------------|
| `spring-boot-starter-amqp` | RabbitMQ メッセージング | Spring AMQP |
| `spring-boot-starter-kafka` | Kafka メッセージング | Spring Kafka |
| `spring-boot-starter-artemis` | Apache Artemis | Spring JMS |

### セキュリティ

| スターター | 説明 | 主要依存関係 |
|-----------|------|-------------|
| `spring-boot-starter-security` | Spring Security | Spring Security |
| `spring-boot-starter-oauth2-client` | OAuth2 クライアント | Spring Security OAuth2 |
| `spring-boot-starter-oauth2-resource-server` | OAuth2 リソースサーバー | Spring Security OAuth2 |

### テスト

| スターター | 説明 | 主要依存関係 |
|-----------|------|-------------|
| `spring-boot-starter-test` | テストサポート | JUnit 5, Mockito, AssertJ |
| `spring-boot-testcontainers` | Testcontainers | Testcontainers |

### 監視・運用

| スターター | 説明 | 主要依存関係 |
|-----------|------|-------------|
| `spring-boot-starter-actuator` | アプリケーション監視 | Spring Boot Actuator |
| `spring-boot-starter-validation` | バリデーション | Hibernate Validator |

## 自動設定クラス

### よく使用される自動設定

```java
// データソース自動設定の除外
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)

// 複数の自動設定除外
@SpringBootApplication(exclude = {
    DataSourceAutoConfiguration.class,
    HibernateJpaAutoConfiguration.class,
    JpaRepositoriesAutoConfiguration.class
})

// 設定プロパティでの除外
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

### 主要な自動設定クラス

| クラス | 説明 | 条件 |
|--------|------|------|
| `DataSourceAutoConfiguration` | データソース設定 | `DataSource` クラスが存在 |
| `JpaRepositoriesAutoConfiguration` | JPA リポジトリ | `@EnableJpaRepositories` 不在 |
| `WebMvcAutoConfiguration` | Spring MVC | Web アプリケーション |
| `SecurityAutoConfiguration` | Spring Security | Security 依存関係存在 |
| `ActuatorAutoConfiguration` | Actuator エンドポイント | Actuator 依存関係存在 |

## カスタムプロパティ

### @ConfigurationProperties の使用

```java
@ConfigurationProperties(prefix = "app")
@Component
public class AppProperties {
    
    private String name;
    private String version;
    private Security security = new Security();
    private List<String> servers = new ArrayList<>();
    
    // getters and setters...
    
    public static class Security {
        private boolean enabled = true;
        private String algorithm = "SHA-256";
        
        // getters and setters...
    }
}
```

対応する設定:
```yaml
app:
  name: My Application
  version: 1.0.0
  security:
    enabled: true
    algorithm: SHA-256
  servers:
    - server1.example.com
    - server2.example.com
```

## プロファイル設定

### プロファイル別設定ファイル

```
application.yml                 # デフォルト設定
application-development.yml     # 開発環境
application-staging.yml         # ステージング環境
application-production.yml      # 本番環境
```

### プロファイル固有の Bean

```java
@Configuration
public class DatabaseConfig {
    
    @Bean
    @Profile("development")
    public DataSource developmentDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }
    
    @Bean
    @Profile("production")
    public DataSource productionDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:postgresql://prod-db:5432/myapp")
            .build();
    }
}
```

### 条件付き設定

```java
@Configuration
@ConditionalOnProperty(
    value = "app.feature.enabled",
    havingValue = "true",
    matchIfMissing = false
)
public class FeatureConfig {
    
    @Bean
    @ConditionalOnMissingBean
    public FeatureService featureService() {
        return new DefaultFeatureService();
    }
}
```

この API リファレンスを活用して、Spring Boot 3.5.0 アプリケーションの開発を効率的に進めてください。