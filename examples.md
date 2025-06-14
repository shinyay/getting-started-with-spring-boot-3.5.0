---
layout: page
title: "サンプルコード"
description: "実践的なSpring Boot 3.5.0のサンプルコードとベストプラクティス"
---

<div class="toc">
  <h3 class="toc-title">目次</h3>
</div>

## Web アプリケーション

### REST API の基本実装

**HelloController.java**
```java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.*;
import org.springframework.http.ResponseEntity;

@RestController
@RequestMapping("/api")
public class HelloController {

    @GetMapping("/hello")
    public ResponseEntity<String> hello() {
        return ResponseEntity.ok("Hello, Spring Boot 3.5.0!");
    }
    
    @GetMapping("/hello/{name}")
    public ResponseEntity<Greeting> helloName(@PathVariable String name) {
        return ResponseEntity.ok(new Greeting("Hello", name));
    }
    
    @PostMapping("/hello")
    public ResponseEntity<Greeting> createGreeting(@RequestBody GreetingRequest request) {
        return ResponseEntity.ok(new Greeting(request.message(), request.name()));
    }
    
    public record Greeting(String message, String name) {}
    public record GreetingRequest(String message, String name) {}
}
```

### エラーハンドリング

**GlobalExceptionHandler.java**
```java
package com.example.demo.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleIllegalArgument(IllegalArgumentException ex) {
        return ResponseEntity
            .badRequest()
            .body(new ErrorResponse("INVALID_ARGUMENT", ex.getMessage()));
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred"));
    }
    
    public record ErrorResponse(String code, String message) {}
}
```

## データアクセス

### JPA エンティティとリポジトリ

**User.java**
```java
package com.example.demo.entity;

import jakarta.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String username;
    
    @Column(nullable = false)
    private String email;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt = LocalDateTime.now();
    
    // コンストラクタ、ゲッター、セッター
    public User() {}
    
    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }
    
    // getter/setter メソッド...
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
}
```

**UserRepository.java**
```java
package com.example.demo.repository;

import com.example.demo.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import java.util.Optional;
import java.util.List;

public interface UserRepository extends JpaRepository<User, Long> {
    
    Optional<User> findByUsername(String username);
    
    List<User> findByEmailContaining(String email);
    
    @Query("SELECT u FROM User u WHERE u.createdAt >= :date")
    List<User> findUsersCreatedAfter(@Param("date") LocalDateTime date);
    
    boolean existsByUsername(String username);
}
```

### サービス層の実装

**UserService.java**
```java
package com.example.demo.service;

import com.example.demo.entity.User;
import com.example.demo.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;

@Service
@Transactional
public class UserService {
    
    private final UserRepository userRepository;
    
    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public User createUser(String username, String email) {
        if (userRepository.existsByUsername(username)) {
            throw new IllegalArgumentException("ユーザー名は既に使用されています: " + username);
        }
        
        User user = new User(username, email);
        return userRepository.save(user);
    }
    
    @Transactional(readOnly = true)
    public Optional<User> findByUsername(String username) {
        return userRepository.findByUsername(username);
    }
    
    @Transactional(readOnly = true)
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
    
    public User updateUser(Long id, String email) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("ユーザーが見つかりません: " + id));
        
        user.setEmail(email);
        return userRepository.save(user);
    }
    
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

## 設定とプロパティ

### application.yml の例

```yaml
# データベース設定
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driverClassName: org.h2.Driver
    username: sa
    password: password
  
  h2:
    console:
      enabled: true
      path: /h2-console
  
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.H2Dialect

# サーバー設定
server:
  port: 8080
  servlet:
    context-path: /api

# ログ設定
logging:
  level:
    com.example.demo: DEBUG
    org.springframework.web: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE

# Actuator 設定
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,env
  endpoint:
    health:
      show-details: always

# カスタムプロパティ
app:
  name: Spring Boot Getting Started
  version: 3.5.0
  description: サンプルアプリケーション
```

### 設定プロパティクラス

**AppProperties.java**
```java
package com.example.demo.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    
    private String name;
    private String version;
    private String description;
    
    // getter/setter
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getVersion() { return version; }
    public void setVersion(String version) { this.version = version; }
    
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
}
```

## テスト実装

### コントローラーテスト

**HelloControllerTest.java**
```java
package com.example.demo.controller;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(HelloController.class)
class HelloControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void hello_ShouldReturnGreeting() throws Exception {
        mockMvc.perform(get("/api/hello"))
            .andExpect(status().isOk())
            .andExpect(content().string("Hello, Spring Boot 3.5.0!"));
    }

    @Test
    void helloName_ShouldReturnPersonalizedGreeting() throws Exception {
        mockMvc.perform(get("/api/hello/John"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.message").value("Hello"))
            .andExpect(jsonPath("$.name").value("John"));
    }

    @Test
    void createGreeting_ShouldCreateNewGreeting() throws Exception {
        String requestJson = """
            {
                "message": "Hi there",
                "name": "Alice"
            }
            """;

        mockMvc.perform(post("/api/hello")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestJson))
            .andExpected(status().isOk())
            .andExpect(jsonPath("$.message").value("Hi there"))
            .andExpect(jsonPath("$.name").value("Alice"));
    }
}
```

### サービステスト

**UserServiceTest.java**
```java
package com.example.demo.service;

import com.example.demo.entity.User;
import com.example.demo.repository.UserRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Test
    void createUser_ShouldCreateNewUser() {
        // Given
        String username = "testuser";
        String email = "test@example.com";
        User savedUser = new User(username, email);
        savedUser.setId(1L);

        when(userRepository.existsByUsername(username)).thenReturn(false);
        when(userRepository.save(any(User.class))).thenReturn(savedUser);

        // When
        User result = userService.createUser(username, email);

        // Then
        assertNotNull(result);
        assertEquals(username, result.getUsername());
        assertEquals(email, result.getEmail());
        verify(userRepository).existsByUsername(username);
        verify(userRepository).save(any(User.class));
    }

    @Test
    void createUser_ShouldThrowException_WhenUsernameExists() {
        // Given
        String username = "existinguser";
        String email = "test@example.com";

        when(userRepository.existsByUsername(username)).thenReturn(true);

        // When & Then
        IllegalArgumentException exception = assertThrows(
            IllegalArgumentException.class,
            () -> userService.createUser(username, email)
        );

        assertTrue(exception.getMessage().contains("ユーザー名は既に使用されています"));
        verify(userRepository).existsByUsername(username);
        verify(userRepository, never()).save(any(User.class));
    }
}
```

## Docker対応

### Dockerfile

```dockerfile
# Multi-stage build
FROM eclipse-temurin:17-jdk-alpine AS builder

WORKDIR /app
COPY . .
RUN ./mvnw clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:17-jre-alpine

RUN addgroup -g 1001 -S spring && \
    adduser -u 1001 -S spring -G spring

WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar

USER spring:spring

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/springboot
      - SPRING_DATASOURCE_USERNAME=postgres
      - SPRING_DATASOURCE_PASSWORD=password
    depends_on:
      - db
    networks:
      - spring-network

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: springboot
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - spring-network

volumes:
  postgres_data:

networks:
  spring-network:
    driver: bridge
```

## GitHub Actions CI/CD

### .github/workflows/ci.yml

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
    - uses: actions/checkout@v4
    
    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
        
    - name: Cache Maven packages
      uses: actions/cache@v3
      with:
        path: ~/.m2
        key: ${{ "{{" }} runner.os }}-m2-${{ "{{" }} hashFiles('**/pom.xml') }}
        restore-keys: ${{ "{{" }} runner.os }}-m2
        
    - name: Run tests
      run: ./mvnw clean test
      
    - name: Build application
      run: ./mvnw clean package -DskipTests
      
    - name: Upload JAR
      uses: actions/upload-artifact@v3
      with:
        name: jar-artifact
        path: target/*.jar

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
      
    - name: Login to DockerHub
      uses: docker/login-action@v3
      with:
        username: ${{ "{{" }} secrets.DOCKER_USERNAME }}
        password: ${{ "{{" }} secrets.DOCKER_PASSWORD }}
        
    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: your-username/spring-boot-app:latest,your-username/spring-boot-app:${{ "{{" }} github.sha }}
```

これらのサンプルコードを参考に、実際の Spring Boot 3.5.0 アプリケーションを構築してください。各例は独立して動作するように作られており、プロジェクトの要件に応じてカスタマイズできます。