---
layout: page
title: "第 9 章　セキュリティ統合 ― Spring Security 6.5 と Spring Boot 3.5"
---

# 第 9 章　セキュリティ統合 ― Spring Security 6.5 と Spring Boot 3.5

（Java 17+／Jakarta EE 10 前提）

---

## 8‑1　`spring‑boot‑starter‑security` がもたらす既定動作

| 挙動                          | 内容                                                         | 解除・変更方法                                        |                                        |
| --------------------------- | ---------------------------------------------------------- | ---------------------------------------------- | -------------------------------------- |
| **すべての HTTP エンドポイントを保護**    | 未認証アクセスは 401/302 を返す                                       | `authorizeHttpRequests` DSL で `permitAll()` 定義 |                                        |
| **ログインフォームまたは Basic 認証**    | コンテンツネゴシエーションに従い `/login` HTML か `WWW‑Authenticate: Basic` | `httpBasic()`／`formLogin()` を明示 or 無効化         |                                        |
| **インメモリユーザー `user/ランダムPW`** | 起動ログにパスワードを出力                                              | `UserDetailsService` Bean を上書き                 | ([docs.spring.io][1], [medium.com][2]) |

> **MEMO** : 3.5 でも“デフォルト全保護”ポリシーは不変。認証抜けを作らない安全側デフォルトとして設計されています。

---

## 8‑2　リクエストレベルのセキュリティ構成 ― `SecurityFilterChain` DSL

### 8‑2‑1　最小カスタム例

```java
@Configuration
@EnableMethodSecurity        // メソッドセキュリティを同時に有効化
class SecurityConfig {

  @Bean
  SecurityFilterChain http(HttpSecurity http) throws Exception {
    return http
        .authorizeHttpRequests(reg -> reg
            .requestMatchers("/", "/assets/**").permitAll()
            .requestMatchers("/admin/**").hasRole("ADMIN")
            .anyRequest().authenticated())
        .httpBasic(Customizer.withDefaults())
        .csrf(csrf -> csrf
            .ignoringRequestMatchers("/webhook/**"))
        .build();
  }
}
```

* 6 系で **`authorizeHttpRequests()`** が標準 DSL。2.x 時代の `antMatchers()` は削除済み。
* 静的リソース (`/assets/**`) を `permitAll()` で明示するのが基本型。

### 8‑2‑2　カスタマイザ・事前構成

| インタフェース                 | 用途                               | 例                                        |
| ----------------------- | -------------------------------- | ---------------------------------------- |
| `WebSecurityCustomizer` | **静的リソースを完全バイパス**（フィルターチェーンスキップ） | `.ignoring().requestMatchers("/img/**")` |
| `SecurityConfigurer`    | DSL にない独自設定を注入                   | JWT フィルターの前後関係調整                         |

---

## 8‑3　認証メカニズムの実装パターン

### 8‑3‑1　Basic／Form ログイン

* `httpBasic()`／`formLogin()` だけで実装可能。
* Boot 起動時に Common‑Logging へパスワードが出力されるので **本番環境では必ず自前ユーザーで上書き**。([docs.spring.io][3])

**インメモリユーザーの詳細設定例**：

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public UserDetailsService userDetailsService() {
        // パスワードエンコーダーを使用して安全にパスワードを保存
        PasswordEncoder encoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
        
        UserDetails admin = User.builder()
            .username("admin")
            .password(encoder.encode("admin123")) // 本番では強力なパスワードを使用
            .roles("ADMIN", "USER")
            .authorities("READ_USERS", "WRITE_USERS", "DELETE_USERS")
            .accountNonExpired(true)
            .accountNonLocked(true)
            .credentialsNonExpired(true)
            .disabled(false)
            .build();
            
        UserDetails user = User.builder()
            .username("user")
            .password(encoder.encode("user123"))
            .roles("USER")
            .authorities("READ_USERS")
            .build();
            
        UserDetails readonly = User.builder()
            .username("readonly")
            .password(encoder.encode("readonly123"))
            .roles("VIEWER")
            .authorities("READ_USERS")
            .build();
        
        return new InMemoryUserDetailsManager(admin, user, readonly);
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        // Spring Security 6.4/6.5 推奨: DelegatingPasswordEncoder
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/users/**").hasAnyRole("USER", "ADMIN")
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/**").hasAuthority("READ_USERS")
                .requestMatchers(HttpMethod.POST, "/api/**").hasAuthority("WRITE_USERS")
                .requestMatchers(HttpMethod.DELETE, "/api/**").hasAuthority("DELETE_USERS")
                .anyRequest().authenticated()
            )
            .httpBasic(basic -> basic
                .realmName("Demo API")
                .authenticationEntryPoint(customAuthenticationEntryPoint())
            )
            .formLogin(form -> form
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard", true)
                .failureUrl("/login?error=true")
                .permitAll()
            )
            .logout(logout -> logout
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login?logout=true")
                .invalidateHttpSession(true)
                .deleteCookies("JSESSIONID")
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
                .maximumSessions(1)
                .maxSessionsPreventsLogin(false)
            )
            .build();
    }
    
    @Bean
    public AuthenticationEntryPoint customAuthenticationEntryPoint() {
        return (request, response, authException) -> {
            response.setStatus(HttpStatus.UNAUTHORIZED.value());
            response.setContentType("application/json");
            response.getWriter().write("""
                {
                    "error": "Unauthorized",
                    "message": "Authentication required",
                    "timestamp": "%s"
                }
                """.formatted(Instant.now()));
        };
    }
}
```

**Spring Security バージョン別の重要な違い**：

| バージョン | 主な変更点 | 移行時の注意点 |
|-----------|-----------|-------------|
| **Spring Security 6.4** | `@EnableMethodSecurity` デフォルト有効化 | `@EnableGlobalMethodSecurity` は非推奨 |
| **Spring Security 6.5** | Passkeys (WebAuthn) 正式サポート | `webAuthn()` DSL 追加 |
| | AuthorizationManager API 強化 | `access()` メソッドでより柔軟な認可制御 |
| | SSL Bundle 統合 | mTLS 設定の簡素化 |

> **注意**: Spring Security 6.4 と 6.5 では `SecurityFilterChain` の DSL が一部変更されています。Spring Boot 3.5 では 6.5 系が標準ですが、6.4 からの移行時は DSL の変更点を確認してください。

### 8‑3‑2　Passkeys（WebAuthn） ― 6.5 の目玉機能

```java
@Bean
SecurityFilterChain passkey(HttpSecurity http,
        RelyingPartyRegistrationRepository rp) throws Exception {

  http
    .authenticationProvider(
        WebAuthnAuthenticationProvider.withDefaults(rp))
    .webAuthn(Customizer.withDefaults());   // DSL 追加（6.4+）

  return http.build();
}
```

* **パスワードレスかつフィッシング耐性** の高い認証方式。
* 6.5 で **JDBC 永続化**・`HttpMessageConverter` カスタマイズが拡充。([docs.spring.io][4], [docs.spring.io][5])

> **実装サンプル**: WebAuthn の完全な実装例は[こちらのサンプルアプリケーション](https://github.com/spring-projects/spring-security-samples/tree/main/servlet/spring-boot/java/authentication/webauthn)を参照してください。登録・認証フローの詳細な実装と設定例が含まれています。

### 8‑3‑3　OAuth 2.0 / OIDC クライアント

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          github:
            client-id: ${GITHUB_ID}
            client-secret: ${GITHUB_SECRET}
```

* Boot が `AuthorizationCodeGrantRequestEntityConverter` を自動構成し、`/login/oauth2/code/*` をエンドポイントに割り当てる。
* 6.5 で **DPoP（Proof‑of‑Possession）** へ実験的対応。([docs.spring.io][5])

### 8‑3‑4　JWT（Resource Server）

```java
@Bean
SecurityFilterChain api(HttpSecurity http) throws Exception {
  http.oauth2ResourceServer(oauth2 -> oauth2
        .jwt(Customizer.withDefaults()));
  return http.build();
}
```

* `spring.security.oauth2.resourceserver.jwt.issuer-uri` へ OIDC Discovery URL を指定するだけで JWK が自動解決。

### 8‑3‑5　サービス間通信の TLS バンドル

* Boot 3.5 の **SSL Bundles** を `spring.ssl.bundle.*` に定義し、**RestClient/WebClient でも同一 Bundle** を再利用可能（第 5 章参照）。
* Server 側は同じ Bundle 名を `server.ssl.bundle` に設定するだけ。外部設定で証明書をローテーションしやすい。([medium.com][2])

---

## 8‑4　認可（Authorization）の多層モデル

### 8‑4‑1　URL ベース

* `authorizeHttpRequests` の *最左優先* ルールに注意。具体パス → ワイルドカード → `anyRequest()` の順に並べる。

### 8‑4‑2　メソッドレベル ― `@EnableMethodSecurity`

| アノテーション                                                       | 用途             | スコープ        |
| ------------------------------------------------------------- | -------------- | ----------- |
| `@PreAuthorize("hasRole('ADMIN')")`                           | 前置評価           | メソッド or クラス |
| `@PostAuthorize("returnObject.owner == authentication.name")` | 戻り値評価          | メソッド        |
| `@SecurityRequirement` (OpenAPI)                              | Swagger UI 表示用 | メソッドパラメータ   |

6.0 で旧 `@EnableGlobalMethodSecurity` は削除、**`@EnableMethodSecurity` が必須**。([docs.spring.io][6])

### 8‑4‑3　権限モデル設計ガイド

1. **Role ＝ coarse‑grained**（例：`ADMIN`, `USER`）
2. **Authority ＝ fine‑grained**（例：`project:read`, `invoice:write`）
3. 属性ベースアクセス制御 (ABAC) は *Spring Authorization Server* か **OPA Gatekeeper** 等の外部 PDP を推奨。

---

## 8‑5　リアクティブスタック（WebFlux）のセキュリティ

```java
@Bean
SecurityWebFilterChain reactive(ServerHttpSecurity http) {
  return http
     .authorizeExchange(ex -> ex
        .pathMatchers("/stream/**").hasAuthority("SSE")
        .anyExchange().authenticated())
     .oauth2Login(Customizer.withDefaults())
     .build();
}
```

* フィルターは **ノンブロッキング** に再実装されており、`Mono<Authentication>` を利用。
* `ServerSecurityContextRepository` で **JWT キャッシュレス認証** を実装すると高スループット。

---

## 8‑6　セキュリティテスト

### 8‑6‑1　ユニット／スライステスト

```java
@WebMvcTest(controllers = AdminController.class)
@WithMockUser(username="alice",roles="ADMIN")   // ①
class AdminControllerTests {

  @Autowired
  MockMvc mvc;

  @Test
  void returns200() throws Exception {
    mvc.perform(get("/admin/panel"))
       .andExpect(status().isOk());
  }
}
```

* `@WithMockUser` は **SecurityContext を自動注入** し、テスト速度を最速化。 ([medium.com][7])
* WebFlux は `@WebFluxTest` + `@WithMockUser` + `WebTestClient`.

### 8‑6‑2　統合テスト

* `@SpringBootTest(webEnvironment=RANDOM_PORT)` + `TestRestTemplate` で **フィルターチェーンをフル起動**。
* OAuth2 クライアントは **WireMock** or **Testcontainers Keycloak** を使用してコールバックフローを再現。

### 8‑6‑3　SecurityContext 漏洩防止

JUnit 5 の **`@DirtiesContext`** を併用し、テスト毎の認証情報キャッシュをクリアする。

---

## 8‑7　運用とベストプラクティス

| カテゴリ          | 推奨設定                                                               | 理由                  |
| ------------- | ------------------------------------------------------------------ | ------------------- |
| **CSRF**      | API 専用サービスは `csrf.disable()`                                       | JSON API＋JWT では冗長   |
| **セッション固定攻撃** | `http.sessionManagement().sessionFixation().migrateSession()`      | デフォルト有効・明示すると可読性向上  |
| **ヘッダー保護**    | `http.headers(h -> h.contentSecurityPolicy("default-src 'self'"))` | DOM XSS を抑止         |
| **認証情報の外部化**  | `spring.security.user.*` を本番で使わない                                  | 誤コミット防止             |
| **秘密情報管理**    | Vault/KMS → `config.import=vault:` or Bundle                       | `.env` 直書き禁止        |
| **CVE 対応**    | Boot & Security を **常に同時アップグレード**                                  | Runtime 依存を BOM が統括 |

---

### まとめ

本章では **`starter‑security` による既定動作から、Spring Security 6.5 の最新機能（Passkeys・DPoP・SSL Bundles）** まで、認証・認可・テスト・運用を一気通貫で解説した。次章では **テスト手法全般** に焦点を移し、ユニット／スライス／統合テストの体系的アプローチを深堀りする。

[1]: https://docs.spring.io/spring-boot/reference/web/spring-security.html?utm_source=chatgpt.com "Spring Security :: Spring Boot"
[2]: https://medium.com/%40alxkm/spring-security-basic-setup-in-a-spring-boot-application-49f0b0556847?utm_source=chatgpt.com "Spring Security: Basic Setup in a Spring Boot Application - Medium"
[3]: https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/basic.html?utm_source=chatgpt.com "Basic Authentication :: Spring Security"
[4]: https://docs.spring.io/spring-security/reference/servlet/authentication/passkeys.html?utm_source=chatgpt.com "Passkeys :: Spring Security"
[5]: https://docs.spring.io/spring-security/reference/whats-new.html?utm_source=chatgpt.com "What's New in Spring Security 6.5"
[6]: https://docs.spring.io/spring-security/reference/servlet/authorization/method-security.html?utm_source=chatgpt.com "Method Security - Spring"
[7]: https://medium.com/%40kjavaman12/testing-securitycontextholder-in-spring-security-tests-with-withmockuser-38ce8060088b?utm_source=chatgpt.com "Testing SecurityContextHolder in Spring Security Tests with ..."
