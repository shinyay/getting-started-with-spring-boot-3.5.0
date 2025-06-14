---
layout: page
title: "第 5 章　データ永続化 ― JPA・Spring Data・R2DBC"
---

# 第 5 章　データ永続化 ― JPA・Spring Data・R2DBC

（Spring Boot 3.5 GA・Java 17+ 前提）

---

## 5‑1　JPA クイックスタート

### 5‑1‑1　基本セットアップ

Spring Boot で JPA を利用するには `spring-boot-starter-data-jpa` を依存に追加するだけで、**Hibernate・HikariCP・Transaction Manager** が自動設定されます。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 5‑1‑2　エンティティの定義

```java
package com.example.demo.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;
import java.time.LocalDateTime;

@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotNull
    @Size(min = 2, max = 50)
    @Column(nullable = false, length = 50)
    private String username;
    
    @NotNull
    @Size(min = 1, max = 100)
    @Column(nullable = false, length = 100)
    private String email;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }
    
    // コンストラクタ、getter、setter は省略
    
    public User() {}
    
    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }
    
    // getter/setter methods...
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

### 5‑1‑3　リポジトリの作成

```java
package com.example.demo.repository;

import com.example.demo.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    // クエリメソッド自動生成
    Optional<User> findByUsername(String username);
    
    List<User> findByEmailContaining(String emailPart);
    
    // カスタムクエリ
    @Query("SELECT u FROM User u WHERE u.username LIKE %:keyword% OR u.email LIKE %:keyword%")
    List<User> searchByKeyword(@Param("keyword") String keyword);
    
    // ネイティブクエリ
    @Query(value = "SELECT COUNT(*) FROM users WHERE created_at > ?1", nativeQuery = true)
    long countUsersCreatedAfter(LocalDateTime dateTime);
}
```

### 5‑1‑4　基本的な CRUD 操作

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
    
    @Autowired
    private UserRepository userRepository;
    
    // 作成
    public User createUser(String username, String email) {
        User user = new User(username, email);
        return userRepository.save(user);
    }
    
    // 読み取り
    @Transactional(readOnly = true)
    public Optional<User> findUserById(Long id) {
        return userRepository.findById(id);
    }
    
    @Transactional(readOnly = true)
    public List<User> findAllUsers() {
        return userRepository.findAll();
    }
    
    @Transactional(readOnly = true)
    public Optional<User> findUserByUsername(String username) {
        return userRepository.findByUsername(username);
    }
    
    // 更新
    public User updateUser(Long id, String newEmail) {
        return userRepository.findById(id)
            .map(user -> {
                user.setEmail(newEmail);
                return userRepository.save(user); // Dirty Checking で自動更新
            })
            .orElseThrow(() -> new RuntimeException("User not found"));
    }
    
    // 削除
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

---

## 5‑2　JPA ベストプラクティス

### 5‑2‑1　ローディング戦略

**遅延ローディング（Lazy Loading）** をデフォルトとし、必要な場合のみ **Eager Loading** や **@EntityGraph** を使用する：

```java
@Entity
public class Order {
    @Id
    @GeneratedValue
    private Long id;
    
    // 多対一は通常 EAGER だが、パフォーマンスを考慮して LAZY に変更
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User user;
    
    // 一対多は LAZY がデフォルト
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<OrderItem> items = new ArrayList<>();
}
```

**@EntityGraph による選択的フェッチ**：

```java
public interface OrderRepository extends JpaRepository<Order, Long> {
    
    @EntityGraph(attributePaths = {"user", "items"})
    @Query("SELECT o FROM Order o WHERE o.id = :id")
    Optional<Order> findByIdWithUserAndItems(@Param("id") Long id);
}
```

### 5‑2‑2　ページングとソート

```java
@Service
@Transactional(readOnly = true)
public class UserService {
    
    public Page<User> findUsersWithPaging(int page, int size, String sortBy) {
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
        return userRepository.findAll(pageable);
    }
    
    public Page<User> searchUsersWithPaging(String keyword, Pageable pageable) {
        return userRepository.findByUsernameContainingIgnoreCase(keyword, pageable);
    }
}
```

### 5‑2‑3　トランザクション境界の管理

```java
@Service
public class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private InventoryService inventoryService;
    
    @Transactional // メソッドレベルでのトランザクション管理
    public Order processOrder(OrderRequest request) {
        // 1. 在庫確認・減算（別サービス）
        inventoryService.reserveItems(request.getItems());
        
        // 2. 注文作成
        Order order = new Order(request.getUserId());
        request.getItems().forEach(item -> 
            order.addItem(new OrderItem(item.getProductId(), item.getQuantity()))
        );
        
        // 3. 保存
        return orderRepository.save(order);
    }
    
    @Transactional(readOnly = true) // 読み取り専用でパフォーマンス向上
    public List<Order> findUserOrders(Long userId) {
        return orderRepository.findByUserIdOrderByCreatedAtDesc(userId);
    }
}
```

---

## 5‑3　R2DBC による リアクティブデータアクセス

### 5‑3‑1　R2DBC セットアップ

非同期・ノンブロッキングなデータアクセスを実現する **R2DBC** は、WebFlux との組み合わせで真価を発揮します。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-r2dbc</artifactId>
</dependency>
<dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

**設定**：

```yaml
spring:
  r2dbc:
    url: r2dbc:h2:mem:///testdb
    username: sa
    password: 
  sql:
    init:
      mode: always
      schema-locations: classpath:schema.sql
```

### 5‑3‑2　リアクティブエンティティとリポジトリ

```java
package com.example.demo.entity;

import org.springframework.data.annotation.Id;
import org.springframework.data.relational.core.mapping.Table;
import java.time.LocalDateTime;

@Table("products")
public class Product {
    
    @Id
    private Long id;
    
    private String name;
    private Double price;
    private LocalDateTime createdAt;
    
    // constructors, getters, setters...
    public Product() {}
    
    public Product(String name, Double price) {
        this.name = name;
        this.price = price;
        this.createdAt = LocalDateTime.now();
    }
    
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public Double getPrice() { return price; }
    public void setPrice(Double price) { this.price = price; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
}
```

```java
package com.example.demo.repository;

import com.example.demo.entity.Product;
import org.springframework.data.r2dbc.repository.Query;
import org.springframework.data.r2dbc.repository.R2dbcRepository;
import org.springframework.stereotype.Repository;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@Repository
public interface ProductRepository extends R2dbcRepository<Product, Long> {
    
    Flux<Product> findByNameContaining(String name);
    
    Flux<Product> findByPriceBetween(Double minPrice, Double maxPrice);
    
    @Query("SELECT * FROM products WHERE price > :price ORDER BY price DESC")
    Flux<Product> findExpensiveProducts(Double price);
    
    @Query("SELECT COUNT(*) FROM products WHERE price < :maxPrice")
    Mono<Long> countAffordableProducts(Double maxPrice);
}
```

### 5‑3‑3　リアクティブサービス

```java
package com.example.demo.service;

import com.example.demo.entity.Product;
import com.example.demo.repository.ProductRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@Service
public class ProductService {
    
    @Autowired
    private ProductRepository productRepository;
    
    public Mono<Product> createProduct(String name, Double price) {
        Product product = new Product(name, price);
        return productRepository.save(product);
    }
    
    public Flux<Product> findAllProducts() {
        return productRepository.findAll();
    }
    
    public Mono<Product> findProductById(Long id) {
        return productRepository.findById(id);
    }
    
    public Flux<Product> searchProducts(String keyword) {
        return productRepository.findByNameContaining(keyword);
    }
    
    public Flux<Product> findProductsInPriceRange(Double minPrice, Double maxPrice) {
        return productRepository.findByPriceBetween(minPrice, maxPrice);
    }
    
    public Mono<Product> updateProduct(Long id, String newName, Double newPrice) {
        return productRepository.findById(id)
            .map(product -> {
                product.setName(newName);
                product.setPrice(newPrice);
                return product;
            })
            .flatMap(productRepository::save);
    }
    
    public Mono<Void> deleteProduct(Long id) {
        return productRepository.deleteById(id);
    }
}
```

### 5‑3‑4　WebFlux コントローラーとの統合

```java
package com.example.demo.web;

import com.example.demo.entity.Product;
import com.example.demo.service.ProductService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    @Autowired
    private ProductService productService;
    
    @GetMapping
    public Flux<Product> getAllProducts() {
        return productService.findAllProducts();
    }
    
    @GetMapping("/{id}")
    public Mono<Product> getProduct(@PathVariable Long id) {
        return productService.findProductById(id);
    }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Mono<Product> createProduct(@RequestBody Product product) {
        return productService.createProduct(product.getName(), product.getPrice());
    }
    
    @PutMapping("/{id}")
    public Mono<Product> updateProduct(@PathVariable Long id, @RequestBody Product product) {
        return productService.updateProduct(id, product.getName(), product.getPrice());
    }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public Mono<Void> deleteProduct(@PathVariable Long id) {
        return productService.deleteProduct(id);
    }
    
    @GetMapping("/search")
    public Flux<Product> searchProducts(@RequestParam String keyword) {
        return productService.searchProducts(keyword);
    }
}
```

---

## まとめ

本章では Spring Boot 3.5 での **データ永続化** の主要アプローチを解説しました：

* **JPA/Hibernate** - 従来型の ORM による同期データアクセス
* **Spring Data JPA** - リポジトリパターンによる定型処理の自動化
* **R2DBC** - リアクティブプログラミングモデルでの非同期データアクセス

次章では、これらのデータアクセス層を活用した **Web アプリケーション開発** について詳しく見ていきます。

[1]: https://docs.spring.io/spring-boot/docs/current/reference/html/data.html "Spring Boot Data Documentation"
[2]: https://spring.io/projects/spring-data-jpa "Spring Data JPA"
[3]: https://r2dbc.io/ "R2DBC Specification"
[4]: https://docs.spring.io/spring-data/r2dbc/docs/current/reference/html/ "Spring Data R2DBC Reference"