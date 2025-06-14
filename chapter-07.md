# 第 6 章　運用・モニタリング

（Spring Boot 3.5 GA・Micrometer 1.14 系を前提）

---

## 6‑1　Actuator 基本 ― “Production‑ready” 機能群

### 6‑1‑1　Actuator を有効にする

* 依存に **`spring-boot-starter-actuator`** を追加すると 30 以上の組み込みエンドポイントが登録される。
* デフォルトで HTTP へ公開されるのは `health` のみ。その他は **`management.endpoints.web.exposure.include=*`** などで明示的に公開する ([docs.spring.io][1])。

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus
```

### 6‑1‑2　エンドポイント一覧（抜粋）

| ID           | 目的                   | 備考                                   |                       |
| ------------ | -------------------- | ------------------------------------ | --------------------- |
| `health`     | アプリ健全性               | Liveness/Readiness/Startup のグルーピング可  |                       |
| `metrics`    | Micrometer 収集メトリクス確認 | `/metrics/jvm.memory.used` 等         |                       |
| `prometheus` | Prometheus 形式 scrape | `micrometer-registry-prometheus` 依存要 |                       |
| `env`        | 外部設定の一覧              | 機密値はマスクされる                           |                       |
| `loggers`    | ログレベル変更              | POST で動的変更可                          |                       |
| `startup`    | 起動ステップのタイムライン        | `ApplicationStartup` を有効化時           |                       |
| `heapdump`   | ヒープダンプ生成             | セキュリティ上、本番では未公開推奨                    |                       |
| `shutdown`   | Graceful shutdown    | デフォルト無効                              | ([docs.spring.io][1]) |

### 6‑1‑3　セキュリティとアクセス制御

* Boot 3.4 から **アクセス制御モデル** が一新。`management.endpoints.access.default` に `none` を指定し、個別に `read-only`／`unrestricted` を付与する「許可リスト方式」が推奨される。

```yaml
management:
  endpoints:
    access:
      default: none
  endpoint:
    health.access: unrestricted
    prometheus.access: read-only
```

([docs.spring.io][1])

### 6‑1‑4　Health インジケータとグループ

* すべての `HealthIndicator` を **グループ化** し、パスや HTTP ステータスを用途別に分離できる（例：`/actuator/health/live` と `/healthz`）。
* HTTP マッピングは下記のようにカスタマイズ可能。

```yaml
management.endpoint.health:
  status:
    http-mapping:
      down: 503
      fatal: 503
  group.startup.additional-path: "server:/healthz"
```

([docs.spring.io][1])

---

## 6‑2　Observability アーキテクチャ

| レイヤー        | 実装                               | 説明                                                                  |
| ----------- | -------------------------------- | ------------------------------------------------------------------- |
| **Logging** | Logback + **Structured Logging** | 3.5 で JSON/ECS 出力を標準サポート ([spring.io][2], [github.com][3])          |
| **Metrics** | Micrometer 1.14                  | JVM/HTTP/DB など 250+ 系列を収集、Prometheus/OTel へエクスポート ([github.com][4]) |
| **Tracing** | Micrometer Tracing 1.4           | W3C Trace‑Context が既定、`Baggage` で属性伝搬 ([spring.io][5])              |

### 6‑2‑1　構造化ログ設定

```yaml
logging:
  structured:
    json:
      field-names:
        level: severity
      stacktrace:
        max-length: 20
```

*ECS (Elastic Common Schema) フォーマット* も `logging.structured.json.format=ecs` で一発切替。([github.com][3])

### 6‑2‑2　Prometheus 連携

1. 依存追加：`implementation 'io.micrometer:micrometer-registry-prometheus'`
2. `/actuator/prometheus` を公開し、Prometheus `scrape_configs` にパスを登録。
3. `management.metrics.distribution.percentiles-histogram.http.server.requests=true` で **レイテンシ分位数** を取得。([docs.spring.io][1])

### 6‑2‑3　OpenTelemetry でのトレーシング

```yaml
management:
  tracing:
    sampling.probability: 0.2  # 20 %
```

`micrometer-tracing-bridge-otel` と `opentelemetry-exporter-otlp` を追加すると、**OTLP gRPC** でコレクターへ即送信される。Context Propagation は自動設定。([spring.io][5])

---

## 6‑3　アプリケーション固有メトリクス

### 6‑3‑1　`@Timed` と `@Counted`

```java
@Timed(value = "order.submit", histogram = true, percentiles = {0.95})
@Counted("order.submit.count")
public void submitOrder(Order o) { ... }
```

* Micrometer 1.14 以降は AOP 依存が不要で、注釈を付けるだけで収集される。([stackoverflow.com][6])

### 6‑3‑2　MeterRegistry API

```java
public RecordController(MeterRegistry registry) {
  Gauge.builder("records.in.buffer", buffer, Deque::size)
       .tag("region", "ap-northeast-1")
       .register(registry);
}
```

* タグは**カードinality を意識**し、ユーザー ID 等の高変動値は避ける。

### 6‑3‑3　Long Task と分布統計

* `LongTaskTimer` でバッチ処理の所要時間を可視化。
* `DistributionSummary` により **ペイロードサイズ** や **金額** のヒストグラムを取得。

---

## 6‑4　Kubernetes／クラウド運用

### 6‑4‑1　Probe 用ヘルスグループ

```yaml
management.endpoint.health.group:
  live:
    include: "ping"
  ready:
    include: "db,messaging"
  startup:
    include: "startup"
```

* `live` を `/actuator/health/live` に、`ready` を管理ポートへ公開すると *Liveness ↔ Readiness* を明確に分離できる。([docs.spring.io][1])

### 6‑4‑2　Service Connection とメトリクス

Boot 3.5 の **Service Connection** は **SSL/TLS の自動バンドル** と **Actuator メトリクス登録** を行い、Testcontainers・Buildpacks でも同一設定が利用可能 ([spring.io][2])。

### 6‑4‑3　Helm / Kustomize テンプレート例（抜粋）

```yaml
livenessProbe:
  httpGet:
    path: /actuator/health/live
    port: 8080
readinessProbe:
  httpGet:
    path: /actuator/health/ready
    port: 8080
```

Kubernetes の **startupProbe** が長時間必要な場合、`management.endpoint.health.group.startup.additional-path=server:/healthz` を活用する。([stackoverflow.com][7])

---

## 6‑5　運用時プロパティ ― 推奨ベースライン

| カテゴリ      | 推奨値                                                                               | 理由                     |
| --------- | --------------------------------------------------------------------------------- | ---------------------- |
| 管理ポート分離   | `management.server.port=9000`                                                     | 本番トラフィックと Actuator を隔離 |
| エンドポイント公開 | `management.endpoints.web.exposure.include=health,metrics,prometheus`             | 最小公開                   |
| ヘルス詳細     | `management.endpoint.health.show-details=when-authorized`                         | 機密情報保護                 |
| ログ上限      | `logging.structured.json.stacktrace.max-length=20`                                | 大量ログ抑止                 |
| JVM 計測    | `management.metrics.enable.jvm=true`                                              | GC/メモリ監視               |
| HTTP 分位数  | `management.metrics.distribution.percentiles-histogram.http.server.requests=true` | P95/P99 を取得            |

---

## 6‑6　起動高速化と可視化

### 6‑6‑1　Bean バックグラウンド初期化

* 3.5 では `bootstrapExecutor` が自動生成され、**Singleton Bean を非同期で構築**。大規模アプリで *10–30 %* 起動短縮を確認。([infoq.com][8])

### 6‑6‑2　`startup` エンドポイントでタイムライン分析

```bash
curl http://localhost:9000/actuator/startup | jq '.timeline[] | {bean:.name,duration:.duration}'
```

ステップ単位の時間を把握し、**ボトルネック Bean を特定**できる。([docs.spring.io][1])

---

## 6‑7　トラブルシューティングとベストプラクティス

1. **メトリクスが倍増**している場合 → 重複 `MeterRegistry` Bean を検出 (`/actuator/beans`).
2. **Health が DOWN から復帰しない** → キャッシュされている可能性。`management.endpoint.health.cache.time-to-live=10s` を調整。
3. **Prometheus でメモリリーク** → High‑cardinality タグを削減、`MeterFilter.deny(id -> id.getTag("userId") != null)` を実装。
4. **Structured Logging でフィールド欠落** → JSON レイアウト変更後は *必ず* ログパイプライン側でスキーマを更新。

---

### まとめ

本章では **Actuator エンドポイントの公開・保護から、Micrometer によるメトリクス／トレーシング／構造化ログまで**、Spring Boot 3.5 の運用・監視機能を横断的に整理しました。次章では **コンテナ化 & ネイティブビルド** に焦点を移し、Buildpacks や GraalVM によるデプロイ最適化を解説します。

[1]: https://docs.spring.io/spring-boot/reference/actuator/endpoints.html "Endpoints :: Spring Boot"
[2]: https://spring.io/blog/2025/05/22/spring-boot-3-5-0-available-now?utm_source=chatgpt.com "Spring Boot 3.5.0 available now"
[3]: https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.5.0-RC1-Release-Notes?utm_source=chatgpt.com "Spring Boot 3.5.0 RC1 Release Notes - GitHub"
[4]: https://github.com/spring-projects/spring-boot/releases?utm_source=chatgpt.com "Releases · spring-projects/spring-boot - GitHub"
[5]: https://spring.io/blog/2022/10/12/observability-with-spring-boot-3?utm_source=chatgpt.com "Observability with Spring Boot 3"
[6]: https://stackoverflow.com/questions/79530553/observability-with-java-spring-boot-actuator-and-micrometer-annotation-counted?utm_source=chatgpt.com "Observability with java spring boot actuator and micrometer ..."
[7]: https://stackoverflow.com/questions/68007519/how-to-configure-kubernetes-startup-probe-with-spring-actuator?utm_source=chatgpt.com "How to configure Kubernetes startup probe with Spring Actuator"
[8]: https://www.infoq.com/news/2025/05/spring-boot-3-5/?utm_source=chatgpt.com "Spring Boot 3.5 Delivers Improved Configuration, Containers, and ..."
