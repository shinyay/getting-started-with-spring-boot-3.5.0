---
layout: page
title: "第 1 章　Spring Boot とは何か"
---

# 第 1 章　Spring Boot とは何か

---

## 1-1　プロジェクトの使命と歴史

### 1-1-1　誕生の背景

Spring Boot は「**スタンドアロンで本番運用可能な Spring アプリケーションを最小構成で“just run”できるようにする**」という思想の下、2014 年に誕生しました。Spring に詳しくない開発者でも短時間でクラウドや PaaS へデプロイできる――そんな“container-less web application architecture” への要求が JIRA チケットに寄せられたことがきっかけです。([spring.io][1])

### 1-1-2　公式ミッションと主要ゴール

公式サイトではミッションと“Primary goals”が明文化されています。

* **Radically faster on-boarding** : すぐにコードを書き始められる体験を提供
* **Opinionated, but flexible** : デフォルトは決め打ち、逸脱はいつでも可能
* **Non-functional features out-of-the-box** : 組み込みサーバー、外部設定、ヘルスチェック、メトリクスなどを標準搭載
* **No code generation / no XML** : 生成物や冗長な XML を必要としない([docs.spring.io][2])

### 1-1-3　リリース年表

| 年月      | バージョン      | 技術的トピック                                                                                 |
| ------- | ---------- | --------------------------------------------------------------------------------------- |
| 2014-04 | **1.0 GA** | Embedded Tomcat とスターター依存を導入。“コンテナレス”開発を実現。([spring.io][1])                              |
| 2018-03 | **2.0 GA** | Spring 5 / WebFlux を採用。Actuator 強化、Gradle 5 対応。([spring.io][3])                         |
| 2022-11 | **3.0 GA** | Java 17・Jakarta EE 9 ベースへ刷新。GraalVM ネイティブサポートを正式化。([spring.io][4], [spring.io][5])      |
| 2023-11 | **3.2 GA** | **Virtual Threads** と **Project CRaC** 初期対応、RestClient / JdbcClient 追加。([spring.io][6]) |
| 2025-05 | **3.5 GA** | 構造化ログ強化、Service Connection の SSL、Bean のバックグラウンド初期化などを提供。([spring.io][7])                |

> **ポイント** : 3.x 系以降は **“Java 17 以上必須”** が大前提。旧バージョンのランタイムでは動作しないため、移行時は Docker ベースイメージや CI/CD の JDK を必ず更新すること。

---

## 1-2　主な特徴と利点

| カテゴリー                                    | 概要                                                                                                              |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| **スターター依存**                              | `spring-boot-starter-*` を追加するだけで、Tomcat・JPA・Security などの依存／設定が一括管理される。                                          |
| **自動設定 (Auto-configuration)**            | クラスパスを解析し、条件アノテーション (`@Conditional*`) で最適な Bean を生成。「書かないときはゼロ設定、書きたいときは自由」なバランスを実現。                            |
| **組み込みサーバー**                             | Tomcat・Jetty・Undertow などを **“組み込み”** で提供。`java -jar` だけで HTTP サーバーが起動し、WAR 配布が不要。([docs.spring.io][8])          |
| **Production-ready Features (Actuator)** | `/actuator/health` や `/metrics` 等 30 以上のエンドポイントでヘルスチェック／メトリクス取得が可能。外形監視や Prometheus 連携も標準。([docs.spring.io][9]) |
| **Opinionated Defaults**                 | ロギング（Logback）、JSON（Jackson）、設定ファイル（application.yaml/properties）など実務で必要な選択肢を“良い塩梅”でプリセット。                        |
| **開発者体験 (DX)**                           | Spring Initializr、DevTools ホットリロード、CLI (Groovy スクリプト) によりプロトタイピングが高速化。([spring.io][5])                          |
| **クラウド／コンテナ連携**                          | Cloud Native Buildpacks（`bootBuildImage`）で安全な OCI イメージを生成し、Kubernetes 等に即デプロイ可能。                                |
| **ネイティブビルド**                             | GraalVM Native Image + Spring AOT により 100 ms 台で起動し、メモリを 1/10 まで削減できるケースも。                                       |
| **拡張性**                                  | “Starter + Auto-Config + Library” の 3 点セットで、社内共通機能をプラグインとして配布可能。                                                |

---

## 1-3　Spring Boot 3.x の位置付けと進化ポイント

### 1-3-1　技術スタックの刷新

* **Java 17 / Jakarta EE 9+** がミニマム要件。`javax.*` パッケージは完全に **`jakarta.*`** へ移行。([spring.io][5])
* Spring Framework 6 を基盤に **AOT** と **Observability** が強化され、GraalVM でも JVM でも同じ構成で動作。

### 1-3-2　3.x 系マイナーリリースの主要機能

| バージョン                | ハイライト                                                                                                              |
| -------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **3.0 (2022-11)**    | GraalVM ネイティブ GA、`RFC 7807` 準拠の Error Handling、Docker Compose サポート。([spring.io][4])                                |
| **3.1 (2023-05)**    | **Service Connections** の導入、Kubernetes 健康診断向けプローブ自動設定、DCR（Docker Compose Runner）。                                  |
| **3.2 (2023-11)**    | **Virtual Threads (Java 21)**、Project CRaC、RestClient / JdbcClient、Observability API 拡充。([spring.io][6])           |
| **3.3 / 3.4 (2024)** | Micrometer 1.14 系、Buildpack v0.10、SSL バンドル自動再読み込み、Testcontainers 2 系統合。                                            |
| **3.5 (2025-05)**    | 構造化ログ (`application/logback-spring.xml` 無しで JSON 出力)、Bean のバックグラウンド初期化、環境変数からの Properties 読み込み簡素化。([spring.io][7]) |

### 1-3-3　エコシステムへの影響

* **仮想スレッド対応**により従来の Reactive／非同期コードを簡潔化できる反面、監視ツールやライブラリのスレッド管理 API 互換性を要確認。
* GraalVM ネイティブは **“スモールフットプリント × 高速起動”** を実現する一方、動的プロキシやリフレクションを多用するアプリでは Native Hint の追加が必須。
* Jakarta 移行で **“import 変更だけ”** と侮らず、依存ライブラリが jakarta 対応済みかを CI で常に検査すること。

---

### まとめ

本章では **Spring Boot のミッション・歴史・主要機能** を俯瞰しました。次章以降は、ここで触れた概念を具体的に深掘りし、3.x 系の最新 API とベストプラクティスを体系的に解説していきます。

[1]: https://spring.io/blog/2014/04/01/spring-boot-1-0-ga-released "Spring Boot 1.0 GA Released"
[2]: https://docs.spring.io/spring-boot/index.html "Spring Boot :: Spring Boot"
[3]: https://spring.io/blog/2018/03/01/spring-boot-2-0-goes-ga?utm_source=chatgpt.com "Spring Boot 2.0 goes GA"
[4]: https://spring.io/blog/2022/11/24/spring-boot-3-0-goes-ga?utm_source=chatgpt.com "Spring Boot 3.0 Goes GA"
[5]: https://spring.io/blog/2022/05/24/preparing-for-spring-boot-3-0?utm_source=chatgpt.com "Preparing for Spring Boot 3.0"
[6]: https://spring.io/blog/2023/11/23/spring-boot-3-2-0-available-now "Spring Boot 3.2.0 available now"
[7]: https://spring.io/blog/2025/05/22/spring-boot-3-5-0-available-now "Spring Boot 3.5.0 available now"
[8]: https://docs.spring.io/spring-boot/docs/2.0.0.M2/reference/html/boot-features-developing-web-applications.html?utm_source=chatgpt.com "27. Developing web applications - Spring"
[9]: https://docs.spring.io/spring-boot/reference/actuator/index.html?utm_source=chatgpt.com "Production-ready Features :: Spring Boot"
