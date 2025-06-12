# 第 2 章　バージョン体系とライフサイクル

---

## 2‑1　リリース列車とサポート期間

### 2‑1‑1　バージョニング規則

Spring Boot は **MAJOR.MINOR.PATCH** 方式を採用します。

| 数字    | 役割              | 例             | 互換性指針                                         |
| ----- | --------------- | ------------- | --------------------------------------------- |
| MAJOR | 大規模仕様変更・下位互換の破壊 | 2 → 3         | Jakarta EE 9 への移行など大掛かりな変更が入るため追加作業が必要        |
| MINOR | 機能追加・非破壊の改善     | 3.4 → 3.5     | API の追加や依存バージョン更新。Deprecated API は残るが将来削除され得る |
| PATCH | バグ修正・微小改善       | 3.5.0 → 3.5.1 | 下位互換を壊さない緊急修正・CVE 対応                          |

### 2‑1‑2　リリースの cadence

* **マイナー版** … 約 6 か月ごと (5–6 月・11 月)
* **パッチ版** … 6–8 週間ごと
* リリース日は Spring Framework／Cloud と同期し “リリース列車 (Release Train)” で輸送される。

### 2‑1‑3　サポートポリシー

2025 年 2 月発表の新ポリシーでは、

* OSS（無償）: マイナーごとに **13 か月**
* エンタープライズ: マイナーごとに **25 か月**
* 各メジャー世代の **最終マイナー (例: 3.5)** はさらに **+5 年** の延長サポート＝合計 7 年([spring.io][1], [spring.io][2])

> **ポイント**: バグ修正だけでなく **依存ライブラリの CVE 対応** もサポート期間に依存するため、運用中のバージョンを必ず把握すること。

#### 3.x 系リリースとサポート期限一覧

| 系列      | GA (初回リリース)                    | OSS サポート終了\*   | Enterprise 終了  | LTS 追加終了       |
| ------- | ------------------------------ | -------------- | -------------- | -------------- |
| 3.0     | 2022‑11‑24([spring.io][3])     | 2023‑12‑31     | 2024‑12‑31     | —              |
| 3.1     | 2023‑05‑18([spring.io][4])     | 2024‑06‑30     | 2025‑06‑30     | —              |
| 3.2     | 2023‑11‑23([spring.io][5])     | 2024‑12‑31     | 2025‑12‑31     | —              |
| 3.3     | 2024‑05‑23([spring.io][6])     | 2025‑06‑30     | 2026‑06‑30     | —              |
| 3.4     | 2024‑11‑21([spring.io][7])     | 2025‑12‑31     | 2026‑12‑31     | —              |
| **3.5** | **2025‑05‑22**([spring.io][8]) | **2026‑06‑30** | **2027‑06‑30** | **2032‑06‑30** |

\*13 か月ルールに従い月末へ丸めた日付を記載。

---

## 2‑2　Spring Boot 3.5 (最新安定版) の新機能

### 2‑2‑1　構造化ログの強化

* `logging.structured.json.*` で **JSON ログ** を標準化し、スタックトレース長の制御や ECS 準拠フォーマットを追加。

```yaml
logging:
  structured:
    json:
      stacktrace:
        max-length: 20               # 行数上限
        format: truncated           # full / truncated
```

([github.com][9])

### 2‑2‑2　Service Connection の SSL 対応

Kafka／Redis など 7 つのサービスに対し `spring.ssl.bundle.*` を介した **双方向 TLS** を Buildpacks・Testcontainers 両方で自動ON。([github.com][9])

### 2‑2‑3　環境変数から複数プロパティを一括ロード

```bash
export MY_CFG=$'my.var1=v1\nmy.var2=v2'
```

```properties
spring.config.import=env:MY_CFG
```

で `Environment` に即反映。Docker Secrets／Kubernetes ConfigMap と相性◎。([github.com][9])

### 2‑2‑4　Bean バックグラウンド初期化

`bootstrapExecutor` が自動生成され、Bean 作成を非同期化して **起動高速化**。メトリクスは Actuator `startup` で確認可能。([github.com][9])

### 2‑2‑5　Servlet／Filter 登録アノテーション

従来の `ServletRegistrationBean` に代わり宣言的に登録可能。

```java
@Bean
@FilterRegistration(name="audit", urlPatterns="/api/*", order=0)
AuditFilter audit() { return new AuditFilter(); }
```

([github.com][9])

### 2‑2‑6　依存エコシステムの更新

| プロジェクト           | 3.4 → 3.5       | 備考                   |                                   |
| ---------------- | --------------- | -------------------- | --------------------------------- |
| Spring Framework | 6.1 → 6.2       | LTS 世代               |                                   |
| Spring Data      | 2024.0 → 2025.0 | 新しい Release Train    |                                   |
| Spring Security  | 6.4 → 6.5       | OAuth2 Client 自動構成拡充 |                                   |
| Apache Pulsar    | 3.3 → 4.0       | LTS クライアントへ移行        | ([spring.io][8], [github.com][9]) |

---

## 2‑3　移行ガイド概要

### 2‑3‑1　2.x → 3.x への主要変更点

| 項目                    | 影響                              | 対策                                            |                                     |
| --------------------- | ------------------------------- | --------------------------------------------- | ----------------------------------- |
| **Java 17 必須**        | Java 11 以前の CI／Docker イメージが起動不可 | ビルド環境を JDK 17+ に更新                            |                                     |
| **Jakarta EE 9 名前空間** | `javax.*` → `jakarta.*` に全面置換   | IDE のマイグレーション／OpenRewrite レシピを利用              |                                     |
| Spring Framework 6 依存 | 古い拡張ライブラリが NoClassDefFound      | 互換バージョンの有無を確認                                 |                                     |
| Spring Security 6     | すべての dispatch type に認可適用        | `spring.security.filter.dispatcher-types` で調整 |                                     |
| Deprecated API の削除    | ビルド時にコンパイルエラー                   | `-Werror`＋Properties Migrator で事前検知           | ([spring.io][10], [github.com][11]) |

**推奨手順**

1. 既存コードを 2.7.x 最新へ更新しテストを緑にする。
2. `--release=17` でコンパイルし Jakarta 互換性警告を解消。
3. `spring-boot-properties-migrator` を一時依存に追加し、起動ログでリネームされたプロパティを洗い出す。
4. サードパーティー依存をすべて Jakarta 対応版へ上げ、`javax.*` が transitive で残っていないか `mvn dependency:tree` で確認。
5. Boot 3.x にバージョンアップし、統合・E2E テストを実行。

### 2‑3‑2　3.4 → 3.5 で注意すべき breaking changes

| 変更点                                        | 影響              | 回避／対応                                         |                   |
| ------------------------------------------ | --------------- | --------------------------------------------- | ----------------- |
| `heapdump` エンドポイントが `access=NONE` デフォルト    | ヒープダンプ取得が不可     | Actuator で明示的に公開＋アクセス権設定                      |                   |
| `.enabled` 系プロパティ値が厳格化 (`true`/`false` のみ) | `"on"` などの値が無効  | 値を置換                                          |                   |
| `TaskExecutor` の旧エイリアス `taskExecutor` 削除   | 名前で参照していたコードが失敗 | `applicationTaskExecutor` に修正 or BeanAlias 追加 |                   |
| Deprecated API の削除 (3.3 で予告分)              | コンパイルエラー        | IDE で “@Deprecated(since="3.3")” ラベルを検索       | ([github.com][9]) |

### 2‑3‑3　安全なアップグレードチェックリスト

1. **CI でフルテスト** — 単体・統合・契約テストを含めグリーンであること。
2. **`--dry-run` デプロイ** — Blue/Green か Shadow Traffic で実トラフィックを当てる。
3. **Actuator** で起動時間・メモリ・GC 挙動が旧版と大差ないか数値比較。
4. **監視エージェント** (APM, OpenTelemetry) が新しい Bean 定義に追随しているか確認。
5. OSS サポート期限を再計算し、運用部門へ共有。

---

### まとめ

本章では **リリーススケジュール** と **サポートライフサイクル**、さらに **最新版 3.5 のアップデート概要** と **移行時の落とし穴** を整理しました。次章ではプロジェクト作成とビルド設定を掘り下げ、実際に 3.5 系アプリを立ち上げる手順を解説していきます。

[1]: https://spring.io/blog/2025/02/13/support-policy-updates "Updates to Spring Support Policy"
[2]: https://spring.io/support-policy "Spring | Support Policy"
[3]: https://spring.io/blog/2022/11/24/spring-boot-3-0-goes-ga?utm_source=chatgpt.com "Spring Boot 3.0 Goes GA"
[4]: https://spring.io/blog/2023/05/18/spring-boot-3-1-0-available-now?utm_source=chatgpt.com "Spring Boot 3.1.0 available now"
[5]: https://spring.io/blog/2023/11/23/spring-boot-3-2-0-available-now?utm_source=chatgpt.com "Spring Boot 3.2.0 available now"
[6]: https://spring.io/blog/2024/05/23/spring-boot-3-3-0-available-now?utm_source=chatgpt.com "Spring Boot 3.3.0 available now"
[7]: https://spring.io/blog/2024/11/21/spring-boot-3-4-0-available-now?utm_source=chatgpt.com "Spring Boot 3.4.0 available now"
[8]: https://spring.io/blog/2025/05/22/spring-boot-3-5-0-available-now "Spring Boot 3.5.0 available now"
[9]: https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.5-Release-Notes "Spring Boot 3.5 Release Notes · spring-projects/spring-boot Wiki · GitHub"
[10]: https://spring.io/blog/2022/05/24/preparing-for-spring-boot-3-0 "Preparing for Spring Boot 3.0"
[11]: https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide "Spring Boot 3.0 Migration Guide · spring-projects/spring-boot Wiki · GitHub"
