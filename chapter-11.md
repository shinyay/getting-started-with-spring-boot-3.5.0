# 第 10 章　ベストプラクティス集

（対象バージョン : Spring Boot 3.5 GA／Java 17+）

---

## 10‑1　プロファイル戦略 ― 「環境差分」をコードレスに封じ込める

### 10‑1‑1　３層プロファイルモデル

| レイヤー        | 役割          | 代表例                            |
| ----------- | ----------- | ------------------------------ |
| **ベース**     | 全環境共通のデフォルト | `default` (暗黙)                 |
| **ファンクション** | 目的別の ON/OFF | `metrics`, `kafka`, `debug`    |
| **環境**      | 実行基盤ごと      | `local`, `test`, `stg`, `prod` |

*アプリ起動時に `local,metrics` のような **複合指定** が推奨*。役割を縦横に分離することで **“local = prod – x”** の単純差分に収束する。

### 10‑1‑2 `application-{profile}.yaml` だけに依存しない

| 手法                            | 使い分け                                                        |                            |
| ----------------------------- | ----------------------------------------------------------- | -------------------------- |
| **`spring.profiles.group.*`** | 「ON にしたら自動で別プロファイルも入る」例：`metrics → prometheus,file-logging` |                            |
| **Profile‑specific beans**    | 例：\`@Profile("kafka                                         | test & !prod")\` でコード単位の分岐 |
| **Property activation**       | `logging.level.root: ${LOG_LEVEL:INFO}` で **環境変数だけ** 変える    |                            |

> **ベストプラクティス** : 設定階層は **YAML < Env‑Var < CLI** を鉄則に。CI では `--spring.profiles.active=stg,metrics` を明示し「あと何が効いているのか」をログで確認する。

### 10‑1‑3　ランタイム切り替えと DevTools

* DevTools が有効な `local` プロファイルでのみ **自動再起動・LiveReload** を ON。
* `spring.devtools.livereload.enabled=false` を SCM に固定し、「LOCAL\_PC=true」の環境変数で反転させるとコミット汚染を防げる。

---

## 10‑2　外部設定 & 構成バリデーション

### 10‑2‑1 `@ConfigurationProperties` でタイプセーフ設定

```java
@ConfigurationProperties(prefix = "payment")
@Validated
record PaymentProps(@NotNull URI endpoint,
                    @Positive Duration timeout) { }
```

* **Immutable record+binder** が 3.x 推奨パターン。
* `spring-boot-configuration-processor` を `annotationProcessor` 依存に追加すると **IDE 補完** が生きる。

### 10‑2‑2　失敗を早く ― **Fail‑Fast 起動** にする

| 手段                            | 効果                                       |
| ----------------------------- | ---------------------------------------- |
| `@Validated` + JSR‑380 注釈     | 値域エラーを **起動時に阻止**                        |
| **FailureAnalyzer** 実装        | 例：秘密鍵が無い場合に *actionable message* を表示     |
| `spring.beaninfo.ignore=true` | JavaBean Introspector をスキップし 5–10 % 起動短縮 |

### 10‑2‑3　Config Data API と Secrets 管理

```yaml
spring:
  config:
    import:
      - vault://secret/data/payment
      - aws-parameterstore:/myapp/
      - env:MY_K8S_SECRET        # 3.5 の multi‑line 変数
```

* 認証情報は **Vault/KMS/Secrets Manager** へ。絶対に Git に置かない。
* **優先順序** : CLI > System prop > ENV > `application*` > Profile‑specific > Import。

### 10‑2‑4　リロード & ローリング更新

| 技術                                                     | 特徴                                                                                  |
| ------------------------------------------------------ | ----------------------------------------------------------------------------------- |
| **Spring Cloud Config**                                | `@ConfigurationProperties` + `@RefreshScope` でオンメモリ置換                               |
| **Kubernetes ConfigMap + `spring.cloud.kubernetes.*`** | `--spring.cloud.kubernetes.reload.strategy=restart` なら *Zero‑Downtime* Rolling と相性◎ |
| **CRaC (Checkpoint/Restore)**                          | Properties を載せ替えてもキャッシュから復元可能（Boot 3.2+）                                            |

---

## 10‑3　AOT 最適化と Project Leyden 展望

### 10‑3‑1　`spring.aot.enabled=true` がもたらす効果

| 効果                       | 内容                            | 規模感\*     |
| ------------------------ | ----------------------------- | --------- |
| **Bean 定義静的化**           | 反射呼び出し → 直接 `new`             | 起動 −15 %  |
| **クラスパススキャン削減**          | ターゲットパッケージをコード生成              | メモリ −10 % |
| **プロキシ生成 Ahead‑Of‑Time** | CGLIB → JDK Proxy → ジェネレートソース | メタ領域 −8 % |

\*Spring 社計測の “典型 50 Bean アプリ” 例。

```bash
./mvnw -Pnative -DspringAot
```

で **JVM モード** でも AOT コードが動くため、「起動高速化だけ欲しい」場合に最適。

### 10‑3‑2　AOT 向けコーディングガイド

| NG パターン              | 置き換え                        | 理由           |
| -------------------- | --------------------------- | ------------ |
| 乱用 `Class.forName()` | `ObjectProvider<T>`         | リフレクションヒント不要 |
| JDK ダイナミックプロキシ手動生成   | `@JdkProxyHint` + インタフェース設計 | 自動ヒントへ寄せる    |
| SpEL in YAML         | Java Config / `@Bean`       | AST 解釈コスト削減  |

### 10‑3‑3　Project Leyden と “Static Image”

* OpenJDK 22+ で議論中の **CDS + AOT → static image** パスは、Boot AOT とほぼ同一のリフレクション排除思想。
* 将来は **“Leyden target”** が Maven/Gradle プラグインに追加され、`nativeCompile` と並列運用される見込み。

> **見据えるべきポイント** : Boot‑AOT は **「Leyden へ向けた予習」**。成果物フォーマットが変わっても、どの Bean が反射を使うかを**今のうちに潰しておく**ことが移行コストを最小化する鍵。

---

## 10‑4　コードベース設計・運用ベストプラクティス（抜粋）

| カテゴリ                 | 推奨                                                        | Why               |
| -------------------- | --------------------------------------------------------- | ----------------- |
| **パッケージ構造**          | `web → service → repository → domain` の **縦分割**           | スライステスト時のスキャン制御が楽 |
| **DTO への MapStruct** | サービス層で `Mapper` インタフェースを DI                               | リフレクション不要＋Null 安全 |
| **共通ユーティリティ**        | `spring-boot-starter-<corp>` + Auto‑config                | コードコピーをスターターへ昇格   |
| **依存更新**             | Dependabot + `./mvnw versions:display-dependency-updates` | CVE を当日中に検知       |
| **静的解析**             | ErrorProne / SpotBugs AOT モジュールへ hook                     | Native ヒント漏れも同時発見 |

---

## 10‑5　観測性・ログ設計ベストプラクティス

| 項目                   | ベースライン                                                          |
| -------------------- | --------------------------------------------------------------- |
| ログ形式                 | `logging.structured.json.format=ecs` で **ECS‑JSON** 統一          |
| ID 伝搬                | `logging.structured.correlation.enabled=true` → `trace_id` 自動付与 |
| メトリクス Tag            | 不変 (`region`,`app`,`instance`) と変動 (`status`) を分離               |
| High‑cardinality ガード | `MeterFilter.deny(id -> id.getTag("userId")!=null)`             |
| 分布ヒストグラム             | HTTP, DB, Kafka すべてに `percentiles-histogram` を設定                |

---

### まとめ

本章では **プロファイル階層設計・外部設定の型安全化・AOT/Static Image を見据えた最適化** といった “アプリ全体の非機能品質” を高める指針を整理しました。次章の付録では、ここで触れたプロパティやスターターの **逆引きリファレンス** を一括掲載します。
