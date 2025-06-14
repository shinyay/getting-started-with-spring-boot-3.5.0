---
layout: page
title: "Getting Started with Spring Boot 3.5.0"
description: "A comprehensive guide to Spring Boot 3.5.0 with modern cloud-native Java development"
hero:
  title: "Getting Started with Spring Boot 3.5.0"
  description: "Master modern cloud-native Java development with Spring Boot 3.5.0 - from zero to production in minutes"
  buttons:
    - text: "Get Started"
      url: "/setup/"
      style: "primary"
      icon: "fas fa-rocket"
    - text: "View Chapters"
      url: "/chapters/"
      style: "secondary"
      icon: "fas fa-book"
---

<section class="features-section">
  <div class="container">
    <h2 class="section-title">なぜ Spring Boot 3.5.0 なのか？</h2>
    <div class="features-grid">
      <div class="feature-card card">
        <div class="card-body">
          <div class="feature-icon">
            <i class="fas fa-rocket"></i>
          </div>
          <h3 class="feature-title">高速起動</h3>
          <p class="feature-description">
            Virtual Threads と AOT コンパイルにより、従来比で大幅な起動時間短縮を実現。
            コンテナ環境での素早いスケールアップが可能です。
          </p>
        </div>
      </div>
      
      <div class="feature-card card">
        <div class="card-body">
          <div class="feature-icon">
            <i class="fas fa-shield-alt"></i>
          </div>
          <h3 class="feature-title">エンタープライズ対応</h3>
          <p class="feature-description">
            Jakarta EE 準拠により、エンタープライズ要件を満たす堅牢なアプリケーション開発。
            Spring Security 6.x による最新のセキュリティ対応。
          </p>
        </div>
      </div>
      
      <div class="feature-card card">
        <div class="card-body">
          <div class="feature-icon">
            <i class="fas fa-cloud"></i>
          </div>
          <h3 class="feature-title">クラウドネイティブ</h3>
          <p class="feature-description">
            Kubernetes、Docker、そしてサーバーレス環境への最適化。
            Buildpacks による効率的なコンテナイメージ作成をサポート。
          </p>
        </div>
      </div>
      
      <div class="feature-card card">
        <div class="card-body">
          <div class="feature-icon">
            <i class="fas fa-chart-line"></i>
          </div>
          <h3 class="feature-title">可観測性</h3>
          <p class="feature-description">
            Micrometer による包括的なメトリクス収集。
            分散トレーシングとログ集約による運用監視の完全対応。
          </p>
        </div>
      </div>
      
      <div class="feature-card card">
        <div class="card-body">
          <div class="feature-icon">
            <i class="fas fa-cogs"></i>
          </div>
          <h3 class="feature-title">自動設定</h3>
          <p class="feature-description">
            intelligent な自動設定により、設定ファイルの記述を最小化。
            開発者は業務ロジックに集中できます。
          </p>
        </div>
      </div>
      
      <div class="feature-card card">
        <div class="card-body">
          <div class="feature-icon">
            <i class="fas fa-users"></i>
          </div>
          <h3 class="feature-title">大規模開発</h3>
          <p class="feature-description">
            テストスライス、Testcontainers によるテスト効率化。
            チーム開発をサポートする豊富な開発者ツール群。
          </p>
        </div>
      </div>
    </div>
  </div>
</section>

<section class="quick-start-section">
  <div class="container">
    <h2 class="section-title">3 ステップで始める</h2>
    <div class="quick-start-grid">
      <div class="step-card card">
        <div class="card-header">
          <div class="step-number">1</div>
          <h3 class="card-title">プロジェクト作成</h3>
        </div>
        <div class="card-body">
          <p>Spring Initializr でプロジェクトを生成します。</p>
          <div class="highlight">
            <pre><code class="language-bash">curl https://start.spring.io/starter.zip \
  -d type=maven-project \
  -d language=java \
  -d bootVersion=3.5.0 \
  -d baseDir=demo \
  -d groupId=com.example \
  -d artifactId=demo \
  -d dependencies=web,actuator \
  -o demo.zip && unzip demo.zip</code></pre>
          </div>
        </div>
      </div>
      
      <div class="step-card card">
        <div class="card-header">
          <div class="step-number">2</div>
          <h3 class="card-title">アプリケーション作成</h3>
        </div>
        <div class="card-body">
          <p>簡単な REST API を作成します。</p>
          <div class="highlight">
            <pre><code class="language-java">@RestController
public class HelloController {
    @GetMapping("/")
    public String hello() {
        return "Hello, Spring Boot 3.5.0!";
    }
}</code></pre>
          </div>
        </div>
      </div>
      
      <div class="step-card card">
        <div class="card-header">
          <div class="step-number">3</div>
          <h3 class="card-title">起動・デプロイ</h3>
        </div>
        <div class="card-body">
          <p>アプリケーションを起動して、すぐに動作確認。</p>
          <div class="highlight">
            <pre><code class="language-bash">./mvnw spring-boot:run
# または
java -jar target/demo-0.0.1-SNAPSHOT.jar</code></pre>
          </div>
        </div>
      </div>
    </div>
    
    <div class="text-center" style="margin-top: 2rem;">
      <a href="{{ '/setup/' | relative_url }}" class="btn btn-primary btn-lg">
        <i class="fas fa-play"></i> 詳細なセットアップガイドを見る
      </a>
    </div>
  </div>
</section>

<section class="latest-updates-section">
  <div class="container">
    <h2 class="section-title">最新の更新情報</h2>
    <div class="updates-grid">
      <div class="update-card card">
        <div class="card-body">
          <h4 class="update-title">
            <i class="fas fa-database"></i>
            データ永続化の章を新規追加
          </h4>
          <p class="update-date">2025年6月14日更新</p>
          <p>Spring Data JPA、R2DBC、Redis を使った現代的なデータアクセス手法を網羅的に解説。</p>
          <a href="{{ '/chapters/' | relative_url }}" class="btn btn-sm btn-outline">詳細を見る</a>
        </div>
      </div>
      
      <div class="update-card card">
        <div class="card-body">
          <h4 class="update-title">
            <i class="fas fa-chart-bar"></i>
            可観測性とメトリクス強化
          </h4>
          <p class="update-date">2025年6月14日更新</p>
          <p>Grafana Tempo、Zipkin との連携例と、ECS 準拠 JSON ログ出力の実装方法を追加。</p>
          <a href="{{ '/chapters/' | relative_url }}" class="btn btn-sm btn-outline">詳細を見る</a>
        </div>
      </div>
      
      <div class="update-card card">
        <div class="card-body">
          <h4 class="update-title">
            <i class="fas fa-vial"></i>
            Testcontainers 最新対応
          </h4>
          <p class="update-date">2025年6月14日更新</p>
          <p>並列実行とキャッシュ機能を活用した効率的なテスト環境構築手法を更新。</p>
          <a href="{{ '/chapters/' | relative_url }}" class="btn btn-sm btn-outline">詳細を見る</a>
        </div>
      </div>
    </div>
  </div>
</section>

<style>
.section-title {
  text-align: center;
  font-size: 2.5rem;
  color: #34302d;
  margin-bottom: 3rem;
  font-weight: 700;
}

.quick-start-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
  margin-bottom: 3rem;
}

.step-card .card-header {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.step-number {
  width: 40px;
  height: 40px;
  background: linear-gradient(135deg, #6db33f 0%, #5fa134 100%);
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 1.2rem;
}

.updates-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
}

.update-title {
  color: #34302d;
  margin-bottom: 0.5rem;
  
  i {
    color: #6db33f;
    margin-right: 0.5rem;
  }
}

.update-date {
  font-size: 0.9rem;
  color: #5f6368;
  margin-bottom: 1rem;
}

.quick-start-section,
.latest-updates-section {
  padding: 4rem 0;
}

.features-section {
  padding: 4rem 0;
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
}

@media (max-width: 768px) {
  .section-title {
    font-size: 2rem;
  }
  
  .quick-start-grid,
  .updates-grid {
    grid-template-columns: 1fr;
  }
}
</style>