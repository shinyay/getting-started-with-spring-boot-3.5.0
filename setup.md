---
layout: page
title: "環境セットアップ"
description: "Spring Boot 3.5.0 の開発環境をセットアップしましょう"
---

<div class="toc">
  <h3 class="toc-title">目次</h3>
</div>

## 必要な環境

Spring Boot 3.5.0 を使用するために必要な環境は以下の通りです：

<div class="alert alert-info">
  <strong>重要:</strong> Spring Boot 3.x では Java 17 以上が必須要件となります。
</div>

### Java Development Kit (JDK)

| Java バージョン | サポート状況 | 推奨度 |
|-------------|----------|------|
| **Java 17 (LTS)** | ✅ 完全サポート | ⭐⭐⭐ |
| **Java 21 (LTS)** | ✅ 完全サポート | ⭐⭐⭐ |
| **Java 24 (EA)** | ✅ 早期アクセス | ⭐⭐ |

#### JDK のインストール

**Windows:**
```bash
# Chocolatey を使用
choco install openjdk17

# または Scoop を使用
scoop install openjdk17
```

**macOS:**
```bash
# Homebrew を使用
brew install openjdk@17

# パスを通す
echo 'export PATH="/opt/homebrew/opt/openjdk@17/bin:$PATH"' >> ~/.zshrc
```

**Linux (Ubuntu/Debian):**
```bash
# OpenJDK 17 をインストール
sudo apt update
sudo apt install openjdk-17-jdk

# JAVA_HOME を設定
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

**インストール確認:**
```bash
java -version
javac -version
echo $JAVA_HOME
```

### ビルドツール

#### Maven

```bash
# Homebrew (macOS)
brew install maven

# Chocolatey (Windows)
choco install maven

# Ubuntu/Debian
sudo apt install maven

# 確認
mvn -version
```

#### Gradle

```bash
# Homebrew (macOS)
brew install gradle

# Chocolatey (Windows)
choco install gradle

# Ubuntu/Debian
sudo apt install gradle

# 確認
gradle -version
```

### IDE セットアップ

#### IntelliJ IDEA

1. **プラグインのインストール:**
   - Spring Boot
   - Spring Assistant
   - Lombok

2. **設定の推奨事項:**
   ```
   Settings > Build, Execution, Deployment > Compiler > Annotation Processors
   ✅ Enable annotation processing
   ```

3. **Live Templates の設定:**
   - `@RestController` クラス用テンプレート
   - `@Service` クラス用テンプレート

#### Visual Studio Code

**必要な拡張機能:**

```json
{
  "recommendations": [
    "redhat.java",
    "vscjava.vscode-java-pack",
    "vscjava.vscode-spring-boot",
    "pivotal.vscode-spring-boot",
    "redhat.vscode-xml"
  ]
}
```

**settings.json の推奨設定:**
```json
{
  "java.home": "/path/to/java-17",
  "spring-boot.ls.java.home": "/path/to/java-17",
  "java.configuration.runtimes": [
    {
      "name": "JavaSE-17",
      "path": "/path/to/java-17"
    }
  ]
}
```

### Docker 環境

Spring Boot アプリケーションのコンテナ化とテストに Docker を使用します。

#### Docker のインストール

**Windows/macOS:**
- [Docker Desktop](https://www.docker.com/products/docker-desktop) をダウンロード・インストール

**Linux:**
```bash
# Ubuntu の場合
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

**Docker Compose のインストール:**
```bash
# 最新版をインストール
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 確認
docker --version
docker-compose --version
```

## プロジェクト作成

### Spring Initializr を使用

#### Web UI での作成

1. [start.spring.io](https://start.spring.io) にアクセス
2. 以下の設定を選択：
   - **Project:** Maven Project
   - **Language:** Java
   - **Spring Boot:** 3.5.0
   - **Group:** com.example
   - **Artifact:** spring-boot-getting-started
   - **Name:** Spring Boot Getting Started
   - **Package name:** com.example.springbootgettingstarted
   - **Packaging:** Jar
   - **Java:** 17

3. **Dependencies** を追加：
   - Spring Web
   - Spring Boot Actuator
   - Spring Data JPA
   - H2 Database
   - Testcontainers

4. **GENERATE** をクリックしてダウンロード

#### CLI での作成

```bash
curl https://start.spring.io/starter.zip \
  -d type=maven-project \
  -d language=java \
  -d bootVersion=3.5.0 \
  -d baseDir=spring-boot-getting-started \
  -d groupId=com.example \
  -d artifactId=spring-boot-getting-started \
  -d name="Spring Boot Getting Started" \
  -d description="Getting Started with Spring Boot 3.5.0" \
  -d packageName=com.example.springbootgettingstarted \
  -d packaging=jar \
  -d javaVersion=17 \
  -d dependencies=web,actuator,data-jpa,h2,testcontainers \
  -o spring-boot-getting-started.zip

unzip spring-boot-getting-started.zip
cd spring-boot-getting-started
```

### プロジェクト構造の確認

生成されたプロジェクトの構造を確認：

```
spring-boot-getting-started/
├── mvnw                              # Maven Wrapper (Unix)
├── mvnw.cmd                          # Maven Wrapper (Windows)
├── pom.xml                           # Maven 設定ファイル
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/springbootgettingstarted/
│   │   │       └── SpringBootGettingStartedApplication.java
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── static/               # 静的リソース
│   │       └── templates/            # テンプレートファイル
│   └── test/
│       └── java/
│           └── com/example/springbootgettingstarted/
│               └── SpringBootGettingStartedApplicationTests.java
└── target/                           # ビルド成果物 (自動生成)
```

## 動作確認

### アプリケーションの起動

```bash
# Maven Wrapper を使用 (推奨)
./mvnw spring-boot:run

# または直接 Maven を使用
mvn spring-boot:run

# または JAR ファイルから実行
./mvnw clean package
java -jar target/spring-boot-getting-started-0.0.1-SNAPSHOT.jar
```

### エンドポイントの確認

**アプリケーション確認:**
```bash
curl http://localhost:8080
# または ブラウザで http://localhost:8080 にアクセス
```

**Actuator エンドポイント確認:**
```bash
# ヘルスチェック
curl http://localhost:8080/actuator/health

# すべてのエンドポイント表示
curl http://localhost:8080/actuator
```

### Hello World API の追加

`src/main/java/com/example/springbootgettingstarted/HelloController.java` を作成：

```java
package com.example.springbootgettingstarted;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/")
    public String hello() {
        return "Hello, Spring Boot 3.5.0!";
    }

    @GetMapping("/api/greeting")
    public Greeting greeting() {
        return new Greeting("Hello", "Spring Boot 3.5.0 World!");
    }

    record Greeting(String message, String target) {}
}
```

**API テスト:**
```bash
# 基本挨拶
curl http://localhost:8080/

# JSON レスポンス
curl http://localhost:8080/api/greeting
```

## 開発環境の最適化

### IDE 設定の最適化

#### IntelliJ IDEA

1. **コード補完の強化:**
   ```
   Settings > Editor > General > Code Completion
   ✅ Show suggestions as you type
   ✅ Match case: First letter only
   ```

2. **Spring Boot 設定ファイルの補完:**
   ```
   Settings > Languages & Frameworks > Spring Boot
   ✅ Enable Spring Boot configuration file completion
   ```

#### Visual Studio Code

**launch.json の設定:**
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "java",
      "name": "Spring Boot-SpringBootGettingStartedApplication",
      "request": "launch",
      "cwd": "${workspaceFolder}",
      "mainClass": "com.example.springbootgettingstarted.SpringBootGettingStartedApplication",
      "projectName": "spring-boot-getting-started",
      "args": "",
      "envFile": "${workspaceFolder}/.env"
    }
  ]
}
```

### ホットリロード設定

**Spring Boot DevTools の追加:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

**application.properties の設定:**
```properties
# 開発時設定
spring.devtools.restart.enabled=true
spring.devtools.livereload.enabled=true
spring.thymeleaf.cache=false

# ログレベル設定
logging.level.com.example=DEBUG
logging.level.org.springframework.web=DEBUG
```

## トラブルシューティング

### よくある問題と解決方法

#### Java バージョンエラー
```
Error: LinkageError occurred while loading main class
```

**解決方法:**
```bash
# Java バージョン確認
java -version
javac -version

# JAVA_HOME 確認
echo $JAVA_HOME

# IntelliJ の場合
File > Project Structure > Project Settings > Project > Project SDK
```

#### ポート競合エラー
```
Web server failed to start. Port 8080 was already in use.
```

**解決方法:**
```bash
# ポート使用状況確認
lsof -i :8080  # macOS/Linux
netstat -ano | findstr :8080  # Windows

# 別のポートを使用
./mvnw spring-boot:run -Dspring-boot.run.arguments=--server.port=8081
```

#### Maven/Gradle ビルドエラー

**Maven の場合:**
```bash
# キャッシュクリア
./mvnw clean
rm -rf ~/.m2/repository

# 依存関係の再取得
./mvnw dependency:resolve
```

**Gradle の場合:**
```bash
# キャッシュクリア
./gradlew clean
rm -rf ~/.gradle/caches

# 依存関係の再取得
./gradlew build --refresh-dependencies
```

## 次のステップ

環境セットアップが完了したら、以下のページで詳細な機能を学習しましょう：

<div class="features-grid" style="margin-top: 2rem;">
  <div class="card">
    <div class="card-body">
      <h4><i class="fas fa-book"></i> チュートリアル</h4>
      <p>ステップバイステップでSpring Bootの基本を学習</p>
      <a href="{{ '/chapters/' | relative_url }}" class="btn btn-primary">チュートリアルを開始</a>
    </div>
  </div>
  
  <div class="card">
    <div class="card-body">
      <h4><i class="fas fa-code"></i> サンプルコード</h4>
      <p>実践的なコード例とベストプラクティス</p>
      <a href="{{ '/examples/' | relative_url }}" class="btn btn-primary">サンプルを見る</a>
    </div>
  </div>
  
  <div class="card">
    <div class="card-body">
      <h4><i class="fas fa-cogs"></i> API リファレンス</h4>
      <p>詳細なAPI仕様と設定オプション</p>
      <a href="{{ '/api/' | relative_url }}" class="btn btn-primary">API ドキュメント</a>
    </div>
  </div>
</div>