# 第 9 章　テスト手法 — Spring Boot 3.5 時代の “確かな品質ゲート”

---

## 9‑1　単体テストの基本

### 9‑1‑1 `spring‑boot‑starter‑test` の内訳

Boot 3.5 の **`spring‑boot‑starter‑test`** は次の主なライブラリを同梱します。

* **JUnit Jupiter 5.11.x** – 標準テストエンジン
* **AssertJ 3.25+** – Fluent 断言 API
* **Hamcrest 2.2** – マッチャ DSL
* **Mockito 5.11** – モック生成／スタブ
* **JsonPath / Jackson‑Databind** – JSON 検証補助
* **Spring Test** – `MockMvc`, `TestContext` など ([mvnrepository.com][1])

依存は *`testImplementation`* スコープに自動追加され、**プロダクション成果物に一切影響しない** 点が安心材料です。

### 9‑1‑2　最小ユニットテストの作法

```java
@ExtendWith(MockitoExtension.class)
class PriceCalculatorTests {

  DiscountService disc = mock(DiscountService.class);

  @Test
  void totalIsCalculated() {
    when(disc.rateFor("VIP")).thenReturn(0.2);
    PriceCalculator calc = new PriceCalculator(disc);

    assertThat(calc.total(1_000)).isEqualTo(800);
    verify(disc).rateFor("VIP");
  }
}
```

* **Arrange–Act–Assert** が一目でわかる構造を維持。
* `@ExtendWith(MockitoExtension.class)` により **Mockito 用 JUnit 拡張**が自動注入。

### 9‑1‑3 `ApplicationContextRunner` で Auto‑config を検証

単体テスト層でも “設定どおり Bean が生えるか” を確かめたい場合は **`ApplicationContextRunner`** が最速。

```java
new ApplicationContextRunner()
  .withPropertyValues("mail.host=smtp.example.com")
  .withUserConfiguration(MailAutoConfiguration.class)
  .run(ctx -> assertThat(ctx).hasSingleBean(JavaMailSender.class));
```

* **起動 200–300 ms** 程度で済み、IDE での TDD が快適。

---

## 9‑2　テストスライスと Web レイヤー検証

### 9‑2‑1 `@WebMvcTest`（Servlet スタック）

```java
@WebMvcTest(controllers = OrderController.class)      // ①
class OrderControllerTests {

  @Autowired MockMvc mvc;

  @MockBean OrderService os;                          // ②

  @Test
  void happyPath() throws Exception {
    given(os.findOne(42)).willReturn(new OrderDto(42, 900));

    mvc.perform(get("/orders/42"))
       .andExpect(status().isOk())
       .andExpect(jsonPath("$.price").value(900));
  }
}
```

① **MVC 関連 Bean のみロード**、DataSource などは読み込まない
② `@MockBean` で依存サービスを差し替え ([docs.spring.io][2])

Boot 3.5 の `@WebMvcTest` はデフォルトで **Spring Security と MockMvc を自動構成**し、XSS・CSRF フィルターも本番同等に動作させられます。

### 9‑2‑2　代表的スライスアノテーション

| 用途         | アノテーション           | 主な自動構成                     |                       |
| ---------- | ----------------- | -------------------------- | --------------------- |
| WebFlux    | `@WebFluxTest`    | RouterFunction/WebClient   |                       |
| JPA        | `@DataJpaTest`    | EntityManager/DataSource   |                       |
| JSON       | `@JsonTest`       | ObjectMapper/JacksonTester |                       |
| RestClient | `@RestClientTest` | `RestClient` + WireMock    | ([docs.spring.io][3]) |

> **Tip** : スライスは **“狭く速く”**。テスト行数よりも「ロードする Bean の数」が速度を左右します。

### 9‑2‑3　セキュリティ付きスライス

`@WithMockUser` を併用するとテストごとに仮想認証情報が注入され、`403` → `200` の期待値だけ変えて安全確認ができます（第 8 章参照）。

---

## 9‑3　統合テストと Testcontainers 2.x

### 9‑3‑1 `@ServiceConnection` で “ゼロ設定” コンテナ

```java
@Testcontainers
@SpringBootTest
class RepositoryIT {

  @Container                                     // ①
  @ServiceConnection                              // ②
  static PostgreSQLContainer<?> db =
      new PostgreSQLContainer<>("postgres:16-alpine");

}
```

① テストクラス起動時に **Docker で Postgres を生成**
② `@ServiceConnection` により **JDBC URL／資格情報が自動で `DataSource` に注入** ([docs.spring.io][4], [docs.spring.io][5])

Boot 3.5 の Service Connection は **Kafka・Redis・MongoDB など 15+ テクノロジ**を標準サポートし、**SSL バンドル**も透過的に取り扱います ([infoq.com][6])。

### 9‑3‑2　SSL/E2E テストの強化

```java
@Container
@Ssl(bundle="local-trust")              // ①
@ServiceConnection
static KafkaContainer<?> kafka =
        new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:8"));

@Test
void produce_and_consume() {...}
```

① `@Ssl` で **証明書をコンテナ起動時に抽出 → テスト側へ束ねて注入** ([infoq.com][6])

### 9‑3‑3　動的プロパティとパラレル実行

* 旧式の `@DynamicPropertySource` は引き続き利用可能ですが、ServiceConnection 使用時は不要。
* **JUnit 5 Parallel Execution** (`junit.jupiter.execution.parallel.enabled=true`) と組み合わせる場合は **Reusable Containers** パターン（`startOnce()`）で大幅な時間短縮が可能 ([medium.com][7])。

---

## 9‑4　観測性テスト — メトリクス & トレース検証

### 9‑4‑1　`MeterRegistry` のアサーション

```java
@Autowired MeterRegistry registry;

@Test
void orderGaugeUpdated() {
  service.placeOrder(99);
  assertThat(registry.get("orders.submitted").counter().count())
      .isEqualTo(1);
}
```

* **Micrometer 直接 Assert** で数値の変化を即検証。
* レイテンシ分位数を確認したい場合は `metricsEndpoint.metric("http.server.requests", List.of("uri","/orders"))` を利用。

### 9‑4‑2　分散トレーシングの録画テスト

`@SpringBootTest` + `ZipkinContainer` を起動し、`/dependencies` API を Polling→Assert することで **Span 間の親子関係** を自動検証できる。

---

## 9‑5　ベストプラクティス集

| シチュエーション                 | 推奨事項                                                   | 背景               |
| ------------------------ | ------------------------------------------------------ | ---------------- |
| **テスト専用設定**              | `application-test.yaml` と `@ActiveProfiles("test")`    | 本番プロパティ汚染を防ぐ     |
| **Bean 再利用で高速化**         | `@DirtiesContext` を安易に多用しない                            | コンテナ再生成コスト増      |
| **乱数・現在時刻**              | `Clock` インタフェースを DI → モック                              | 再現性確保            |
| **Testcontainers キャッシュ** | `~/.testcontainers` → CI アーティファクト化                     | ネットワーク依存低減       |
| **Mutation Testing**     | Pitest + JUnit 5                                       | 分岐網羅率より実効カバレッジ   |
| **CI 並列化**               | Maven Surefire `junit5.parallel` + Reusable Containers | 30–50 % スループット向上 |

---

### まとめ

本章では **ユニット（JUnit + Mockito）→ スライス（`@WebMvcTest` 等）→ 統合（Testcontainers + Service Connection）** と段階的に深度を増すテスト戦略を体系化しました。Boot 3.5 の Testcontainers 連携により “データベースの起動すらコード 1 行” で完結し、SSL/メトリクスを含む実運用同等の E2E テストが容易になっています。次章では、これらテスト結果を最大化する **ベストプラクティス & AOT 最適化** をさらに掘り下げます。

[1]: https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test/3.5.0-M1?utm_source=chatgpt.com "spring-boot-starter-test » 3.5.0-M1 - Maven Repository"
[2]: https://docs.spring.io/spring-boot/3.5/api/java/org/springframework/boot/test/autoconfigure/web/servlet/WebMvcTest.html?utm_source=chatgpt.com "WebMvcTest (Spring Boot 3.5.0 API)"
[3]: https://docs.spring.io/spring-boot/appendix/test-auto-configuration/slices.html?utm_source=chatgpt.com "Test Slices :: Spring Boot"
[4]: https://docs.spring.io/spring-boot/api/java/org/springframework/boot/testcontainers/service/connection/ServiceConnection.html?utm_source=chatgpt.com "ServiceConnection (Spring Boot 3.5.0 API)"
[5]: https://docs.spring.io/spring-boot/reference/testing/testcontainers.html?utm_source=chatgpt.com "Testcontainers :: Spring Boot"
[6]: https://www.infoq.com/news/2025/05/spring-boot-3-5/?utm_source=chatgpt.com "Spring Boot 3.5 Delivers Improved Configuration, Containers, and ..."
[7]: https://medium.com/%40mrayandutta/reliable-spring-boot-integration-testing-with-testcontainers-2aaf2556c53e?utm_source=chatgpt.com "Reliable Spring Boot Integration Testing with Testcontainers - Medium"
