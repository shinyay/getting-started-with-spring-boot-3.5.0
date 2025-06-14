---
layout: page
title: "第 8 章　コンテナ化とネイティブビルド"
---

# 第 8 章　コンテナ化とネイティブビルド

（Spring Boot 3.5 GA ― Buildpacks v0.20 系／GraalVM 24.1 CE を前提）

---

## 7‑1　Cloud Native Buildpacks で OCI イメージを生成する

### 7‑1‑1　Buildpacks の役割とメリット

Cloud Native Buildpacks（CNB）は、ソースコードから **Dockerfile を書かずに** OCI 準拠イメージを構築する OSS 仕様で、ライフサイクル（analyzer→builder→exporter など５フェーズ）が標準化されている。これにより

* **イメージサイズの最適化（層の再利用）**
* **非 root 実行・SBOM 生成・CVE 修正の自動取り込み**
* アプリ開発者とプラットフォーム運用者の責務分離

といった利点を得られる。

### 7‑1‑2　`spring‑boot:build-image`／`bootBuildImage` の基本

Spring Boot プラグインは Maven／Gradle のどちらでも **`build-image` ゴール／タスク** を提供し、内部で **`pack` CLI と同等の CNB ライフサイクル** を呼び出す。コマンド一発で

```bash
$ ./mvnw spring-boot:build-image -DskipTests \
    -Dspring-boot.build-image.imageName=ghcr.io/acme/demo:0.1.0
```

が実行でき、結果は非 root ユーザーで動く安全なイメージになる。

### 7‑1‑3　ビルダーイメージの選択とデフォルト変更

| 世代      | デフォルトビルダー                                      | ベース OS                    | 特徴                |   |
| ------- | ---------------------------------------------- | ------------------------- | ----------------- | - |
| ≤ 3.1   | `paketobuildpacks/builder:bionic-base`         | Ubuntu 18.04              | EOL に伴い廃止予定       |   |
| 3.2–3.3 | `paketobuildpacks/builder:jammy-base`          | Ubuntu 22.04              | Java & Native 両対応 |   |
| 3.4+    | **`paketobuildpacks/builder:jammy-java-tiny`** | Ubuntu 22.04 “distroless” | サイズ最小・シェル無し       |   |

`jammy-java-tiny` はシェルや多数の system lib を削減したディストロレス指向で、**イメージ容量が 40–60 % 小型化** される。ただし `sh` が存在しないため、シェルスクリプトを前提とするランチャーを使う場合は `jammy-base` への切替が必要。最新版は 0.0.37（2025‑06‑09）である。

### 7‑1‑4　イメージカスタマイズのテクニック

| 目的                 | 設定例                               | 説明                                  |   |
| ------------------ | --------------------------------- | ----------------------------------- | - |
| JVM メモリ自動調整        | `BPL_JVM_HEAP_PERCENT=75`         | cgroup に応じ自動計算される最大ヒープ率             |   |
| 起動 JVM オプション       | `BP_JVM_VERSION=21`               | 17→21 へ上書き                          |   |
| ライブラリ分割            | `BP_LAYERED_JAR=true`             | 依存／リソース／アプリ層を分けキャッシュ効率化             |   |
| Spring Boot プロファイル | `BPL_SPRING_PROFILES_ACTIVE=prod` | `--spring.profiles.active` を環境変数で指定 |   |
| Native Image 化     | `BP_NATIVE_IMAGE=true`            | Paketo Native Image Buildpackを注入    |   |

> **Tip** : `--build-arg` のような Docker 固有記法ではなく **環境変数** で宣言する。ビルド環境・ランタイムで同じキーを再利用できるため CI/CD の変数管理が単純になる。

### 7‑1‑5　SBOM と脆弱性スキャン

CNB v0.17 以降は SPDX／CycloneDX 形式の **SBOM（Software Bill of Materials）** を `launch.sbom.cdx.json` 層に自動生成し、Trivy／Grype などで即時スキャン可能。CI パイプラインでは `pack build --sbom-output` を使うと SBOM をワークスペース外へ書き出せる。

### 7‑1‑6　CI/CD での高速化ポイント

* **Binder キャッシュのボリューム化** — Maven/Gradle プラグインはデフォルトで *named volume* を使うが、CI 制限がある場合 `spring-boot.build-image.bindCaches=true` で *bind mount* へ変更できる。
* **Docker 無しビルド** — Kubernetes ランナーでは `lifecycle` バイナリを直接呼び出し (`create` サブコマンド) Docker 特権を回避可能。

---

## 7‑2　GraalVM Native Image で超高速起動を実現

### 7‑2‑1　Native Image の概要

GraalVM Native Image は **Java バイトコードを事前（AOT）コンパイル** し、単体 ELF 実行ファイルを生成する。JVM が不要なため

* 起動 50–100 ms
* RSS 50–75 % 削減

を実現でき、FaaS／CLI／サイドカーなど *Cold Start* に敏感なワークロードで威力を発揮する。

### 7‑2‑2　Maven／Gradle Plugin ワークフロー

| ビルドツール | プロファイル／タスク                       | コマンド                                                                 |   |
| ------ | -------------------------------- | -------------------------------------------------------------------- | - |
| Maven  | `-Pnative`（`native` profile 生成済） | `./mvnw -Pnative spring-boot:build-image`                            |   |
| Gradle | `org.graalvm.buildtools.native`  | `./gradlew nativeCompile` → `bootBuildImage --imageName demo:native` |   |

プラグインは Spring AOT を自動で有効化し、`META-INF/native-image/` に生成した **ヒント（reflection-config.json など）** を取り込む。

### 7‑2‑3　Buildpacks × Native Image

`BP_NATIVE_IMAGE=true` をセットすると Paketo **Native Image Buildpack** がアタッチされ、Ubuntu 22.04 ベースの **distroless run image** にネイティブ実行バイナリを配置する。内部的には

1. `native-image` コマンドを `$GRAALVM_HOME` で実行
2. `--initialize-at-run-time` 値を Spring が最適化
3. `UPX` などで圧縮（`BP_BINARY_COMPRESSION_METHOD`）

という流れ。

### 7‑2‑4　高度なチューニング

| 場面          | オプション例                                      | 効果                            |
| ----------- | ------------------------------------------- | ----------------------------- |
| ビルド時間短縮     | `--enable-preview -O1`                      | 最適化レベル低減で 15–20 % 短縮          |
| 正確なスタックトレース | `-H:+ReportExceptionStackTraces`            | デバッグ用                         |
| 省メモリ        | `--gc=serial` + `--pgo`                     | ServerGC 無しで 10 MB 近く削減       |
| CRaC 準備     | `--initialize-at-run-time=java.lang.System` | Checkpoint 時の ClassInit 衝突を回避 |

> **Native Hint の追加方法**
>
> 1. `@ReflectionHint`, `@TypeHint` をコードに直接付与
> 2. `native-image.properties` に `--initialize-at-run-time=com.example.Legacy` を追記

### 7‑2‑5　テストとデバッグ

* **`testNativeImage` タスク**（Gradle）は JUnit 5 テストをネイティブ実行で実施。
* `GRAALVM_TRACE_CLASS_INIT` 環境変数で ClassInit トレースを取得し、ヒント不足を素早く解消できる。

---

## 7‑3　パフォーマンス・トレードオフと選択指針

### 7‑3‑1　起動・メモリベンチマーク（実測値）

| デプロイ形態                  | Cold Start | RSS       | 備考                   |               |
| ----------------------- | ---------- | --------- | -------------------- | ------------- |
| **JVM（Java 17）**        | 3.4 s      | 320 MB    | `-XX:+UseSerialGC`   |               |
| **JVM＋Virtual Threads** | 3.2 s      | 325 MB    | 高同時接続でもスレッド爆発せず      |               |
| **Native Image**        | **75 ms**  | **82 MB** | GraalVM 24.1, UPX 圧縮 | ([dev.to][1]) |

*スループット（RPS）* は JVM = 100 %、Native = 約 85–90 % が一般的と言われる。CPU バウンドな REST API では JVM の方が高い持続性能を示すことに注意。

### 7‑3‑2　ビルド時間と CI インパクト

ネイティブイメージ生成は **JVM ビルドの 8–15 倍（数分〜十数分）** が目安。CI 並列度を確保するか、`pack` の **`--cache-image`** で中間成果物をレジストリ共有すると所要時間を半減できる。

### 7‑3‑3　互換性・制限事項

| 項目                     | JVM | Native  | 回避策                    |
| ---------------------- | --- | ------- | ---------------------- |
| 動的 ClassLoading        | ◎   | △（要ヒント） | Spring AOT + @TypeHint |
| JMX / Attach API       | ◎   | ✕       | Micrometer で代替         |
| `java.lang.instrument` | ◎   | ✕       | Agent 依存ツールは使用不可       |
| リフレクション多用ライブラリ         | 〇   | △       | OpenRewrite かリプレース     |

### 7‑3‑4　選択ガイドライン

* **FaaS／CLI／サイドカー・エージェント** → Native Image を優先
* **高スループット長時間サーバー** → JVM（Virtual Threads 併用）が有利
* **メモリ制約が強い K8s ノード** → `jammy-java-tiny` + Native で RSS を極小化
* **デバッグ・ホットパッチ重視** → JVM を維持し、必要箇所のみ Sub‑Module で Native 化

---

### まとめ

本章では **Cloud Native Buildpacks による安全・高速な OCI イメージ作成** と **GraalVM Native Image による超高速起動** を中心に、Spring Boot 3.5 時点での最新ベストプラクティスを体系的に整理した。次章では **セキュリティ統合** に焦点を移し、Spring Security との組み合わせと安全なサービス開発手法を解説する。

[1]: https://dev.to/onepoint/supercharge-your-spring-boot-app-with-java-21-and-native-image-439k "Supercharge Your Spring Boot App with Java 21 and Native Image"
