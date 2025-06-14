---
layout: page
title: "序章　― Spring Boot 3.5 が拓く “クラウド標準 Java” の現在地"
---

# 序章　― Spring Boot 3.5 が拓く “クラウド標準 Java” の現在地

---

## 0‑1　本書の目的

本書は **Spring Boot 3.5** を軸に、モノリスからマイクロサービス、コンテナ／サーバーレス、そして将来の Project Leyden に至るまで——
**「現代 Java アプリケーションが備えるべき機能すべて」** を一冊で体系化することを目指します。
公式ドキュメントの網羅性と、現場で鍛え上げられたベストプラクティスの両輪を融合し、

* **ゼロから学ぶ開発者** が “最短３ステップ” で Hello World をデプロイできる。
* **既に運用中のチーム** が「移行・運用・観測・最適化」の知識ギャップを 30 分で埋められる。

——そんな **“実践＆逆引きハイブリッド”** のリファレンスを提供します。

---

## 0‑2　対象読者と前提知識

| 読者像                   | 想定スキル                        | 本書で得られるもの                                 |
| --------------------- | ---------------------------- | ----------------------------------------- |
| **Java／Kotlin 初級者**   | 基本文法・Maven/Gradle が触れる       | 最小構成から本番稼働までの “一本道”                       |
| **Spring 2.x 実務者**    | DI / MVC の経験あり               | 3.x の Jakarta 移行、Virtual Threads、ネイティブビルド |
| **SRE／プラットフォームエンジニア** | Kubernetes／Observability を担当 | Actuator・Micrometer・Buildpacks の運用設計      |

> **必須前提** : Java 17 以上がインストール済みであること。
> **推奨** : Docker／Docker Compose、Git に触れた経験があると実習がスムーズです。
> 
> ### 環境要件
> * **Java**: 17 (LTS), 21 (LTS) または 24 EA（本書のコード例は全バージョンで検証済み）
> * **Spring Boot**: 3.5.0 以上
> * **ビルドツール**: Maven 3.9+ または Gradle 8.5+
> * **コンテナ**: Docker 20.10+ および Docker Compose V2
> * **OS**: Windows 10+, macOS 12+, Linux (Ubuntu 20.04+ 推奨)
>
> ### リンク
> * **課題管理**: [GitHub Issues](https://github.com/awesome-springboot-3.5-book/issues)
> * **議論フォーラム**: [GitHub Discussions](https://github.com/awesome-springboot-3.5-book/discussions)

---

## 0‑3　Spring Boot 3.5 とエコシステムの現在地

2025 年現在、クラウドネイティブ化とランタイム高速化の潮流はさらに加速し、

* **構造化ログ & OpenTelemetry** による観測性の標準化
* **Virtual Threads + Reactive** によるスケーラビリティの両立
* **GraalVM Native Image & CRaC** によるサーバーレス最適化
* **Cloud Native Buildpacks** によるサプライチェーン強化

が急速に普及しています。Spring Boot 3.5 はこれらを “設定ゼロ／スターター１行” で統合し、**エンタープライズ Java の DX を一段押し上げるプラットフォーム** へ進化しました。

---

## 0‑4　本書の構成と読み進め方

| 章          | 概要                    | キーワード                    |
| ---------- | --------------------- | ------------------------ |
| **第 1 章**  | プロジェクトの使命と歴史          | Boot 誕生、3.x の進化          |
| **第 2 章**  | バージョン体系・サポート          | リリース列車、3.5 新機能           |
| **第 3 章**  | セットアップ & 雛形           | Initializr、ビルド設定         |
| **第 4 章**  | Auto‑config / Starter | 条件判定、独自スターター             |
| **第 5 章**  | データ永続化                | JPA・Spring Data・R2DBC   |
| **第 6 章**  | Web 開発                | MVC・WebFlux・RestClient   |
| **第 7 章**  | 運用・モニタリング             | Actuator、Micrometer、OTel |
| **第 8 章**  | コンテナ & ネイティブ          | Buildpacks、GraalVM       |
| **第 9 章**  | セキュリティ統合              | Security 6.5、Passkeys    |
| **第 10 章** | テスト手法                 | スライス、Testcontainers      |
| **第 11 章** | ベストプラクティス集            | プロファイル、AOT               |
| **第 12 章** | 逆引き付録                 | プロパティ一覧、スターター表           |

* **入門者** : 第 3→4→6 章でアプリを動かし、第 7 章で運用観点を学ぶ流れが最短ルート。
* **経験者** : 必要なテーマを逆引きできるよう、章立てを“機能別タイル”にしています。

---

## 0‑5　ハンズオン環境とサンプルコード

* **リポジトリ** : [https://github.com/awesome‑springboot‑3.5‑book](https://github.com/awesome‑springboot‑3.5‑book)
* **ブランチ構成** :

  * `chapter‑03/` – 雛形プロジェクト
  * `chapter‑07/native/` – Native Image 例
  * `chapter‑09/testcontainers/` – E2E テスト一式
* **Docker** : `docker compose up` だけで Postgres／Keycloak／Zipkin が起動し、書籍の全サンプルを再現可能。

---

## 0‑6　学習を最速化する３つの Tips

1. **Actuator を常時 ON** —— 開発段階から `/actuator/health` を叩き、設定変更の影響を可視化。
2. **Testcontainers で “本物” をテスト** —— Mock ではなく Docker 上の DB/Kafka を CI で並列起動。
3. **構成バリデーションを厳格に** —— `@ConfigurationProperties` + `@Validated` で早期に失敗させ、デプロイ後の “環境差分バグ” をゼロへ。

---

## 0‑7　本書の表記規約

* **コマンド** : `$` から始まるシェル例示。
* **コード** : Java は *record*、Kotlin は *data class* を優先採用。
* **プロパティ** : `snake.case` をそのまま記載し、環境変数は `SPRING_CONFIG_IMPORT` のように大文字＋アンダースコアで表記。
* **注釈** : “💡Tip” はハンズオン促進、“⚠ 注意” は移行時の落とし穴を示す。

---

### 本書で得られるゴール

この序章を読み終えた時点で、読者は

* **何が学べるか** — Spring Boot 3.5 の全領域をカバーした学習ロードマップ
* **どう学ぶか** — 章ごとのハンズオンと逆引きの使い方
* **なぜ学ぶか** — クラウドネイティブ Java に必要な非機能要件の重要性

を具体的にイメージできるはずです。では、次ページから **Spring Boot の核心** に飛び込みましょう。
