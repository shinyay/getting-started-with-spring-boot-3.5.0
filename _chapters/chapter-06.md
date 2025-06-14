---
layout: page
title: "第 6 章　Web アプリ開発 ― Spring MVC／WebFlux／HTTP クライアント"
---

# 第 6 章　Web アプリ開発 ― Spring MVC／WebFlux／HTTP クライアント

（Spring Boot 3.5 GA・Java 17+ 前提）

---

## 5‑1　Spring MVC ― テンプレート & 従来型サーブレットスタック

### 5‑1‑1　最小構成と自動設定

* `spring-boot-starter-web` を依存に追加すると、**DispatcherServlet・Jackson・Thymeleaf/Mustache の各 Bean** が自動登録される。([spring.io][1])
* コントローラは最上位パッケージ配下に配置し、`@GetMapping` 等で URL を割り当てる。

```java
@Controller
class HomeController {

  @GetMapping("/")
  String index(Model model) {
    model.addAttribute("msg", "Hello, Spring MVC!");
    return "index";        // src/main/resources/templates/index.html
  }
}
```

> **ポイント** : Boot 3.x では Jakarta Servlet 6.0 依存に移行しているため import は `jakarta.servlet.*` になる。

### 5‑1‑2　テンプレートエンジン選択

| エンジン          | 依存 (starter)                    | 特徴                                                                   |
| ------------- | ------------------------------- | -------------------------------------------------------------------- |
| **Thymeleaf** | `spring-boot-starter-thymeleaf` | HTML5 方言で自然テンプレートを保つ。レイアウトは \[Layout Dialect] で拡張可。([github.com][2]) |
| **Mustache**  | `spring-boot-starter-mustache`  | ロジックレス。1 ファイル＝1 View の静的サイト生成に向く。([docs.spring.io][3])               |

Boot が自動生成する `SpringTemplateEngine` / `Mustache.Compiler` は **キャッシュ ON** が既定。開発時は `spring.thymeleaf.cache=false` にして再読み込みを有効化する。

### 5‑1‑3　静的リソースと WebJar

* classpath :`/static`, `/public`, `/META-INF/resources` 直下は **「/」に直結** して公開される。([baeldung.com][4], [docs.spring.io][5])
* WebJar（例：`org.webjars.npm:bootstrap`）を利用すると  `/webjars/bootstrap/5.3.3/js/bootstrap.bundle.js` のように配信可能。CDN を嫌う企業内ネットワークで便利。

### 5‑1‑4　フォーム処理とバリデーション

1. DTO に `@NotNull`, `@Size` 等の Jakarta Bean Validation アノテーションを付与。
2. コントローラ引数に `@Valid` と `BindingResult` を置く。
3. エラーは `th:errors="*{field}"` で表示。

```java
@PostMapping("/signup")
String signup(@Valid @ModelAttribute UserForm form, BindingResult rs) {
  if (rs.hasErrors()) return "signup";
  service.create(form);
  return "redirect:/";
}
```

### 5‑1‑5　HttpMessageConverters とコンテンツネゴシエーション

* Boot は **23 種** の `HttpMessageConverter` を自動登録し、`Accept` ヘッダと拡張子で **JSON ↔ XML ↔ YAML** を切替える。
* カスタム MIME を追加したい場合は `WebMvcConfigurer#extendMessageConverters` を実装するだけでよい。

### 5‑1‑6　Virtual Threads 併用時の MVC

* `spring.threads.virtual.enabled=true`（Java 21 以上）で **リクエスト処理スレッドが仮想スレッド化** される。([docs.spring.io][6], [github.com][7])
* I/O 待ちが長いがリアクティブに書き換えるほどではない API では **同一コードでスループットを向上** できる。
* `@Async` や `@Scheduled` も自動で仮想スレッド実行になるため、スレッドプールサイズ調整の負荷が減る。

---

## 5‑2　Spring WebFlux ― リアクティブ & ノンブロッキング

### 5‑2‑1　WebFlux を選択する判断軸

| 要件                      | MVC (Servlet) | WebFlux (Reactive) |
| ----------------------- | ------------- | ------------------ |
| 高同時接続 / SSE / WebSocket | △（従来スレッドモデル）  | ◎（ノンブロッキング I/O）    |
| レガシー同期ライブラリ利用           | ◎             | △（ブロッキング注意）        |
| 既存コード資産                 | ◎             | ▲（Mono/Flux へ書換え）  |

Virtual Threads で MVC の同時実行性が向上したとはいえ、**バックプレッシャー制御・ストリーミング応答** が必要な場合は WebFlux が依然有利 ([master-spring-ter.medium.com][8])。

### 5‑2‑2　アノテーション vs 機能的エンドポイント

#### アノテーション方式（従来型）

```java
@RestController
class GreetingHandler {

  @GetMapping("/hello/{name}")
  Mono<Greet> hello(@PathVariable String name) {
    return Mono.just(new Greet("Hi " + name));
  }
}
```

#### 機能的エンドポイント

```java
@Configuration
class Routes {

  @Bean
  RouterFunction<ServerResponse> router() {
    return RouterFunctions.route()
        .GET("/hello/{name}",
             req -> ServerResponse.ok()
                     .bodyValue("Hi " + req.pathVariable("name")))
        .build();
  }
}
```

* 関数型は **Bean 生成だけでルーティング** が完結し、AOT・Native Image でメモリ削減効果が大きい。([medium.com][9], [docs.spring.io][10])

### 5‑2‑3　Netty サーバーとチューニング

* `spring-boot-starter-webflux` ⇒ Reactor Netty が既定。HTTP/2 は自動有効（JDK 11+）([docs.spring.io][11])。
* スレッド数は `reactor.netty.ioWorkerCount`、メモリは `reactor.netty.pool.maxConnections` で制御。

### 5‑2‑4　Virtual Threads との共存シナリオ

* WebFlux の **ブロッキング回避サポート** は仮想スレッドにも適用され (`@Blocking` 相当の制御不要)。
* ただし **Reactor コンテキスト ⇆ 仮想スレッドの切替コスト** があるため、IO 待ちが短い高速ストリームでは従来のイベントループが優位。([medium.com][12], [medium.com][13])

### 5‑2‑5　Reactive Data & Test

* R2DBC／MongoDB Reactive Driver と組み合わせ、**完全ノンブロッキングパイプライン** を構築。
* `@WebFluxTest` + `WebTestClient` でエンドポイントを **バインドレスで高速テスト**。

---

## 5‑3　宣言的 HTTP クライアント ― RestClient / WebClient

### 5‑3‑1　RestClient（同期・ブロッキング）

* Spring Boot 3.2 で導入された **関数型 API**。`RestTemplate` より **ビルダー中心** かつ **レコード／kotlin data class** に自然マッピング。([docs.spring.io][14], [docs.spring.io][15])

```java
@Service
class CatService(RestClient.Builder builder) {

  private final RestClient client =
      builder.baseUrl("https://api.example.com").build();

  Cat find(String id) {
    return client.get().uri("/cats/{id}", id)
                 .retrieve().body(Cat.class);
  }
}
```

* **HttpMessageConverters** と `ClientHttpRequestFactory` は Boot が注入。タイムアウトは `builder.requestFactory(factory)` で個別指定。

### 5‑3‑2　WebClient（非同期・リアクティブ）

* `Mono` / `Flux` を返して **バックプレッシャー対応**。
* サーバーと同じ Reactor Netty を共有することで **接続プールを統合** し、メトリクスも併用できる。([docs.spring.io][15])

### 5‑3‑3　SSL バンドルとセキュア通信

* Boot 3.5 の **SSL Bundles** を `RestClientSsl` / `WebClientSsl` でバインドし、証明書設定を 1 行で適用可能。([docs.spring.io][16], [docs.spring.io][15])

```java
@Bean
RestClient secure(RestClient.Builder builder, RestClientSsl ssl) {
  return builder.apply(ssl.fromBundle("corp-tls")).build();
}
```

### 5‑3‑4　ベストプラクティス

| 項目                        | RestClient                                              | WebClient                                   |
| ------------------------- | ------------------------------------------------------- | ------------------------------------------- |
| 共通ヘッダ                     | `builder.defaultHeader()`                               | `builder.defaultHeader()`                   |
| ログ／監視                     | `RestClientCustomizer` → `ExchangeFilterFunction` で集中管理 | `ExchangeFilterFunction`                    |
| Retries / Circuit Breaker | Resilience4j Decorators                                 | `reactor.retry` or Resilience4j             |
| テスト                       | `MockRestServiceServer`                                 | `WebClient.builder().exchangeFunction(...)` |

> **Virtual Threads × RestClient** : `spring.threads.virtual.enabled=true` で **ブロッキング呼び出しもスレッド浪費を抑制**。外部 API 呼び出しを大量に並列化する際に特に有効。([stackoverflow.com][17])

---

### まとめ

本章では **Servlet 型 Spring MVC** と **Reactive 型 WebFlux** の使い分け、さらに **新世代 HTTP クライアント (RestClient/WebClient)** を一気通貫で解説しました。次章では Actuator・Micrometer を軸に **運用・モニタリング** 機能を深掘りします。

[1]: https://spring.io/guides/gs/spring-boot?utm_source=chatgpt.com "Getting Started | Building an Application with Spring Boot"
[2]: https://github.com/andbin/spring-boot3-thymeleaf-basic-demo?utm_source=chatgpt.com "Spring Boot 3 – Thymeleaf Basic Demo - GitHub"
[3]: https://docs.spring.io/spring-boot/api/java/org/springframework/boot/web/servlet/view/MustacheView.html?utm_source=chatgpt.com "MustacheView (Spring Boot 3.5.0 API)"
[4]: https://www.baeldung.com/spring-mvc-static-resources?utm_source=chatgpt.com "Serve Static Resources with Spring - Baeldung"
[5]: https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-config/static-resources.html?utm_source=chatgpt.com "Static Resources :: Spring Framework"
[6]: https://docs.spring.io/spring-boot/reference/features/task-execution-and-scheduling.html?utm_source=chatgpt.com "Task Execution and Scheduling :: Spring Boot"
[7]: https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.2.0-M1-Release-Notes?utm_source=chatgpt.com "Spring Boot 3.2.0 M1 Release Notes - GitHub"
[8]: https://master-spring-ter.medium.com/java-virtual-threads-are-here-do-we-still-need-webflux-40250d51c78e?utm_source=chatgpt.com "Java Virtual Threads Are Here: Do We Still Need WebFlux?"
[9]: https://medium.com/%40AlexanderObregon/registering-functional-endpoints-with-spring-boot-and-routerfunction-4dbac9a4e61b?utm_source=chatgpt.com "Using RouterFunction in Spring Boot - Medium"
[10]: https://docs.spring.io/spring-framework/reference/web/webflux-functional.html?utm_source=chatgpt.com "Functional Endpoints :: Spring Framework"
[11]: https://docs.spring.io/spring-boot/how-to/webserver.html?utm_source=chatgpt.com "Embedded Web Servers :: Spring Boot"
[12]: https://medium.com/%40anand34577/virtual-threads-in-spring-boot-3-2-a-game-changer-for-java-applications-43e6a85c3104?utm_source=chatgpt.com "Virtual Threads in Spring Boot 3.2: A Game-Changer for Java Applications"
[13]: https://medium.com/%40lgianlucaster/mastering-virtual-threads-in-java-and-spring-boot-many-requests-few-threads-nobody-waits-too-cae73f657f84?utm_source=chatgpt.com "Mastering Virtual Threads in Java and Spring Boot — Many requests, few ..."
[14]: https://docs.spring.io/spring-boot/reference/io/rest-client.html?utm_source=chatgpt.com "Calling REST Services :: Spring Boot"
[15]: https://docs.spring.io/spring-boot/reference/io/rest-client.html "Calling REST Services :: Spring Boot"
[16]: https://docs.spring.io/spring-boot/3.5/api/java/org/springframework/boot/autoconfigure/web/client/RestClientSsl.html?utm_source=chatgpt.com "RestClientSsl (Spring Boot 3.5.0 API)"
[17]: https://stackoverflow.com/questions/78012861/parallel-service-calls-with-spring-boot-3-2-and-virtual-threads?utm_source=chatgpt.com "Parallel service calls with Spring Boot 3.2 and virtual threads"
