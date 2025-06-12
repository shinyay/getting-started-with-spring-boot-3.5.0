# 第 3 章　セットアップとプロジェクト作成

> **対象バージョン : Spring Boot 3.5.0**（Java 17+ が必須）

---

## 3‑1　Spring Initializr でのプロジェクト生成

### 3‑1‑1　Web UI（start.spring.io）の基本操作

1. ブラウザで `https://start.spring.io` を開く。
2. **Project** に *Maven* または *Gradle*、**Language** に *Java*／*Kotlin* を選択。
3. **Spring Boot** バージョンで *3.5.0* を指定。
4. **Dependencies** に `Spring Web`, `Actuator`, `Testcontainers` などを追加。
5. **Generate** をクリックして ZIP をダウンロード → 解凍すると **標準的な Maven／Gradle ディレクトリ構成** が得られる。

### 3‑1‑2　CLI からの生成（cURL／HTTPie）

インフラ自動化スクリプトや CI からは Web UI を介さず直接 ZIP を取得できる。

```bash
curl https://start.spring.io/starter.zip \
  -d type=gradle-project \
  -d bootVersion=3.5.0 \
  -d language=java \
  -d dependencies=web,actuator \
  -d packageName=com.example.demo \
  -o demo.zip
unzip demo.zip && cd demo
```

`curl start.spring.io` を叩くだけで **利用可能なパラメータ一覧**（project/type/dependencies など）がテキストで返るため、自動生成パイプラインに組み込みやすい。

### 3‑1‑3　IDE プラグイン連携

* **IntelliJ IDEA / Spring Assistant** – “Spring Initializr” ウィザードから同等の設定を GUI で選択してプロジェクトを作成。
* **Visual Studio Code / Spring Boot Extension Pack** – `Spring Initializr: Generate a Maven Project` コマンドが利用可能。
* **Spring Tools 4 (Eclipse)** – New → Spring Starter Project。

> **Tip**: いずれの方法でも **`pom.xml` または `build.gradle(.kts)` を生成**するだけなので、チーム標準の CI テンプレートに “生成済みビルドファイル” をコミットしておくと後続作業が安定する。

---

## 3‑2　ビルドシステム設定

### 3‑2‑1　Maven 基本構成

`spring-boot-starter-parent` を *parent POM* に設定すると、**依存 BOM 管理・Java 17・UTF‑8・`-parameters` オプション**などのデフォルトが自動適用される。

```xml
<project>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.5.0</version>
  </parent>

  <dependencies>
    <!-- Web API とテスト -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```



#### Spring Boot Maven Plugin

プラグインは **実行・再パッケージング・Docker/OCI イメージ化** を 1 本で担う。

| 代表ゴール                     | 用途                            | コマンド例                         |   |
| ------------------------- | ----------------------------- | ----------------------------- | - |
| `spring-boot:run`         | ソースのままアプリを起動（開発用ホットリロード対応）    | `mvn spring-boot:run`         |   |
| `spring-boot:repackage`   | 依存入りの実行 JAR/WAR を生成           | `mvn package`（既定で実行）          |   |
| `spring-boot:build-image` | **CNB** で OCI イメージを生成（非 root） | `mvn spring-boot:build-image` |   |

> **ベストプラクティス**: CI では `-DskipTests` でユニットテストを飛ばすのではなく、`mvn verify -Pci` のように **専用プロファイル**を切ってテスト対象を制御すると再現性が高い。

### 3‑2‑2　Gradle 基本構成

`plugins { id 'org.springframework.boot' version '3.5.0' }` と `io.spring.dependency-management` を宣言するだけで、**依存バージョンの一元管理**と **Boot 固有タスク** が追加される。

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.5.0'
    id 'io.spring.dependency-management' version '1.1.5'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
java { sourceCompatibility = JavaVersion.VERSION_17 }

repositories { mavenCentral() }

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```



*Gradle Plugin の特徴*

* Gradle 7.6.4 / 8.4 以降をサポート。
* `bootRun`・`bootJar`・`bootBuildImage` などのタスクを自動追加。
* **Configuration Cache** に対応し、増分ビルドが高速。([docs.spring.io][1])

| タスク              | 目的                    | 例                          |   |
| ---------------- | --------------------- | -------------------------- | - |
| `bootRun`        | ローカル実行                | `./gradlew bootRun`        |   |
| `bootJar`        | 実行 JAR を生成            | `./gradlew bootJar`        |   |
| `bootBuildImage` | OCI イメージ化（Buildpacks） | `./gradlew bootBuildImage` |   |

> **Tip**: `bootBuildImage --imageName ghcr.io/acme/demo:0.1.0` のように *タグ* を付けると、そのまま Docker レジストリに push しやすい。

### 3‑2‑3　プロジェクト構成とパッケージング

標準的なディレクトリ階層（`src/main/java`, `src/main/resources` 等）は **ビルドツールが自動検知**する。`@SpringBootApplication` を含むランチャークラスは通常 `com.example.demo` といった *ルートパッケージ直下* に置き、**コンポーネントスキャンの基点** とする。

---

## 3‑3　最小サンプルアプリの作成

### 3‑3‑1　ソースコード

<details><summary><code>DemoApplication.java</code></summary>

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication        // @Configuration + @EnableAutoConfiguration + @ComponentScan
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

</details>

<details><summary><code>HelloController.java</code></summary>

```java
package com.example.demo.web;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
class HelloController {

    @GetMapping("/")
    String index() {
        return "Greetings from Spring Boot!";
    }
}
```

</details>

### 3‑3‑2　実行手順

| ステップ          | Maven                                      | Gradle                                         |
| ------------- | ------------------------------------------ | ---------------------------------------------- |
| コンパイル＆テスト     | `mvn clean verify`                         | `./gradlew clean test`                         |
| 開発起動（ホットリロード） | `mvn spring-boot:run`                      | `./gradlew bootRun`                            |
| 本番 JAR 生成     | `mvn -Pprod package` *(例)*                 | `./gradlew bootJar`                            |
| 実行            | `java -jar target/demo-0.0.1-SNAPSHOT.jar` | `java -jar build/libs/demo-0.0.1-SNAPSHOT.jar` |
| コンテナイメージ化     | `mvn spring-boot:build-image`              | `./gradlew bootBuildImage`                     |

**動作確認**

```bash
curl http://localhost:8080/
# => Greetings from Spring Boot!
```

### 3‑3‑3　コード配置のベストプラクティス

| 層          | ディレクトリ例                       | ポイント                           |
| ---------- | ----------------------------- | ------------------------------ |
| Web/API    | `com.example.demo.web`        | `@RestController`, DTO         |
| Service    | `com.example.demo.service`    | ビジネスロジック。トランザクション境界をここに        |
| Repository | `com.example.demo.repository` | Spring Data インタフェースまたは JPA 実装  |
| Config     | `com.example.demo.config`     | `@Configuration` クラス。Bean 定義集中 |

> **Why**: ルートパッケージ配下でレイヤーを分けると、**`@ComponentScan` の範囲制御がシンプル** になり、テスト用スライスアノテーション（`@WebMvcTest` など）の *include/exclude 設定* も最小化できる。

---

## まとめ

本章では **Spring Initializr での雛形生成** から **Maven／Gradle の実践的なビルド設定**、そして **最小 “Hello World” アプリ** の作成と実行までを体系的に解説しました。ここで示したプロジェクト雛形とビルド知識を土台に、次章以降で **Auto‑configuration** や **スターター依存** の内部メカニズムを掘り下げていきます。

[1]: https://docs.spring.io/spring-boot/gradle-plugin/index.html "Gradle Plugin :: Spring Boot"
