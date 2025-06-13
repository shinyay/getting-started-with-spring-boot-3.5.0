# 第 11 章　付録 ― 逆引きリファレンス集

---

## 11‑1　主要 `application.properties`／`application.yaml` 早見表

| カテゴリ              | プロパティ                                                                        | 既定値                       | 解説                                        |
| ----------------- | ---------------------------------------------------------------------------- | ------------------------- | ----------------------------------------- |
| **コアサーバー**        | `server.port`                                                                | `8080`                    | HTTP リスンポート                               |
|                   | `server.shutdown`                                                            | `IMMEDIATE`               | `GRACEFUL` に変更で Graceful Shutdown         |
| **データソース**        | `spring.datasource.url`                                                      | なし                        | JDBC 接続 URL                               |
|                   | `spring.sql.init.mode`                                                       | `embedded`                | `always` で RDB 初期スクリプトを常時実行               |
| **JPA/Hibernate** | `spring.jpa.hibernate.ddl-auto`                                              | なし                        | `validate` / `update` 等                   |
|                   | `spring.jpa.open-in-view`                                                    | `true`                    | MVC + JPA でのセッション維持                       |
| **Actuator**      | `management.endpoints.web.exposure.include`                                  | `health`                  | 公開 ID をカンマ区切りで列挙                          |
|                   | `management.endpoint.health.show-details`                                    | `never`                   | `when-authorized` で認可後のみ詳細表示              |
| **ログ**            | `logging.level.root`                                                         | `INFO`                    | `DEBUG` などに上書き                            |
|                   | `logging.structured.json.format`                                             | `default`                 | `ecs` で ECS‑JSON 出力 ([docs.spring.io][1]) |
| **プロファイル**        | `spring.profiles.active`                                                     | なし                        | `local,metrics` など複数可                     |
|                   | `spring.profiles.group.metrics`                                              | `prometheus,file-logging` | ネスト定義                                     |
| **AOT/Native**    | `spring.aot.enabled`                                                         | `false`                   | JVM モードでも AOT を有効化                        |
|                   | `spring.native.image.build-args`                                             | なし                        | 追加の `--enable-preview` 等を列挙               |
| **Observability** | `management.metrics.distribution.percentiles-histogram.http.server.requests` | `false`                   | P95/P99 収集を有効化                            |
|                   | `management.tracing.sampling.probability`                                    | `1.0`                     | 0.0–1.0 でサンプリング率調整                        |
| **Container**     | `spring.docker.compose.enabled`                                              | `true`                    | Docker Compose Runner を ON/OFF            |

*本表は Spring Boot 3.5.0 の **Common Application Properties** から運用頻度の高い項目を抽出しています。詳細は公式 Appendix を参照してください。([docs.spring.io][1])*

---

## 11‑2　スターター & 依存 BOM 対照表（抜粋）

| Starter Artifact                    | 主なトランジティブ依存                               | 用途例              |
| ----------------------------------- | ----------------------------------------- | ---------------- |
| `spring‑boot‑starter`               | spring‑boot, spring‑core, Logback, YAML   | 最小構成             |
| `spring‑boot‑starter‑web`           | **Tomcat**, Spring MVC, Jackson           | ブロッキング HTTP API  |
| `spring‑boot‑starter‑webflux`       | **Reactor Netty**, Spring WebFlux         | リアクティブ HTTP      |
| `spring‑boot‑starter‑data‑jpa`      | Spring Data JPA, Hibernate 6, HikariCP    | ORM／RDB          |
| `spring‑boot‑starter‑data‑mongodb`  | Spring Data MongoDB, Mongo Java Driver    | NoSQL            |
| `spring‑boot‑starter‑security`      | Spring Security 6.5                       | 認証・認可            |
| `spring‑boot‑starter‑oauth2‑client` | Spring Security OAuth2 Client             | OIDC Login       |
| `spring‑boot‑starter‑amqp`          | Spring AMQP, RabbitMQ Client              | RabbitMQ Pub/Sub |
| `spring‑boot‑starter‑kafka`         | Spring for Apache Kafka, Kafka Client 4.0 | Kafka Streams    |
| `spring‑boot‑starter‑batch`         | Spring Batch 5.1                          | バッチ処理            |
| `spring‑boot‑starter‑actuator`      | Micrometer, Actuator                      | 運用／監視            |
| `spring‑boot‑starter‑test`          | JUnit Jupiter, AssertJ, Mockito ほか        | テスト              |

> **BOM との関係** : `spring‑boot‑dependencies:3.5.0` を **parent POM** または **Gradle Plugin** が自動適用し、上記すべての **ライブラリバージョンを統一** します。個別バージョン指定は基本的に不要です。([mvnrepository.com][2])

---

## 11‑3　Spring Boot CLI & DevTools リファレンス

### 11‑3‑1 CLI 導入

```bash
# SDKMAN!
$ sdk install springboot
$ spring --version
Spring CLI v3.5.0
```

*Homebrew (`brew install spring-boot`) や `curl -s https://get.sdkman.io` でも取得可*。([docs.spring.io][3])

### 11‑3‑2 主要 CLI コマンド

| コマンド                    | 説明                              | 代表オプション                                         |                       |
| ----------------------- | ------------------------------- | ----------------------------------------------- | --------------------- |
| `spring init`           | Start.spring.io へリクエストし ZIP を生成 | `--dependencies=web,actuator --java-version=17` |                       |
| `spring run`            | Groovy/Java/Kotlin ファイルを即実行     | `--watch` でホットリロード                              |                       |
| `spring encodepassword` | BCrypt ハッシュを生成                  | `--strength=12`                                 |                       |
| `spring shell`          | 対話式シェル                          | `--debug` で verbose                             | ([docs.spring.io][4]) |

### 11‑3‑3 DevTools 便利機能

| 機能               | プロパティ                                     | 備考                               |                       |
| ---------------- | ----------------------------------------- | -------------------------------- | --------------------- |
| **自動再起動**        | `spring.devtools.restart.enabled=true`    | `target/classes` 監視              |                       |
| **LiveReload**   | `spring.devtools.livereload.enabled=true` | ブラウザ拡張と連携                        |                       |
| **Remote Debug** | `spring.devtools.remote.secret=mytoken`   | `/actuator/tunnel` WebSocket     |                       |
| **属性トンネリング**     | `spring.devtools.add-properties=false`    | `spring.datasource.*` をローカルだけ上書き | ([docs.spring.io][5]) |

---

## 11‑4　代表的ビルドプラグイン坐標

| 項目                 | Maven                                               | Gradle                          | 用途                         |
| ------------------ | --------------------------------------------------- | ------------------------------- | -------------------------- |
| Spring Boot Plugin | `org.springframework.boot:spring-boot-maven-plugin` | `org.springframework.boot`      | JAR 再パッケージ / `build-image` |
| GraalVM Native     | `org.graalvm.buildtools.native:native-maven-plugin` | `org.graalvm.buildtools.native` | `nativeCompile`            |
| Docker Compose     | `io.spring.javaformat` (不要)                         | `springBootDockerCompose` タスク   | Testcontainers 代替          |

---

## 11‑5　コマンドライン “逆引き” チートシート

| 目的              | Maven                         | Gradle                     | CLI                 |
| --------------- | ----------------------------- | -------------------------- | ------------------- |
| プロジェクト雛形生成      | `curl start.spring.io/...`    | 同左                         | `spring init`       |
| ローカル実行          | `mvn spring-boot:run`         | `./gradlew bootRun`        | `spring run *.java` |
| FAT JAR 生成      | `mvn -Pprod package`          | `./gradlew bootJar`        | —                   |
| Buildpacks イメージ | `mvn spring-boot:build-image` | `./gradlew bootBuildImage` | —                   |
| ネイティブビルド        | `mvn -Pnative native:compile` | `./gradlew nativeCompile`  | —                   |

---

### おわりに

本付録では **設定プロパティ・スターター依存・CLI/DevTools** など「日々の開発術」に直結する逆引き情報をまとめました。書籍本文と合わせて活用することで、Spring Boot 3.5 の機能を **“思い出し 3 秒／調査ゼロ”** で引き出せる開発環境を構築できます。

[1]: https://docs.spring.io/spring-boot/appendix/application-properties/index.html?utm_source=chatgpt.com "Common Application Properties :: Spring Boot"
[2]: https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter/3.5.0?utm_source=chatgpt.com "org.springframework.boot » spring-boot-starter » 3.5.0"
[3]: https://docs.spring.io/spring-boot/installing.html?utm_source=chatgpt.com "Installing Spring Boot"
[4]: https://docs.spring.io/spring-boot/cli/using-the-cli.html?utm_source=chatgpt.com "Using the CLI :: Spring Boot"
[5]: https://docs.spring.io/spring-boot/cli/index.html?utm_source=chatgpt.com "Spring Boot CLI :: Spring Boot"
