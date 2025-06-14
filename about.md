---
layout: page
title: "About"
description: "Spring Boot 3.5.0 ガイドについて"
---

## このガイドについて

**Getting Started with Spring Boot 3.5.0** は、現代のクラウドネイティブ Java 開発のための包括的なガイドです。Spring Boot 3.5.0 の最新機能から実運用まで、開発者が必要とするすべての知識を体系的に提供します。

### 🎯 目的

このガイドは以下の目的で作成されました：

- **学習効率の最大化**: 初心者から経験者まで、効率的に Spring Boot 3.5.0 を習得
- **実践的な知識**: 理論だけでなく、実際のプロジェクトで使える実践的なノウハウ
- **最新技術の習得**: Virtual Threads、AOT、GraalVM など最新機能の活用方法
- **運用ノウハウ**: 開発環境から本番環境まで、運用に必要な知識

### 📚 対象読者

| 読者層 | 前提知識 | 得られるもの |
|-------|---------|-------------|
| **Java 初心者** | 基本文法、Maven/Gradle | Spring Boot の基礎から応用まで |
| **Spring 2.x 経験者** | Spring DI/MVC の経験 | 3.x への移行知識、新機能の活用 |
| **アーキテクト** | システム設計経験 | マイクロサービス設計、運用設計 |
| **DevOps エンジニア** | インフラ、監視経験 | 可観測性、デプロイメント戦略 |

### 🛠 技術仕様

- **Spring Boot**: 3.5.0
- **Java**: 17 (LTS), 21 (LTS), 24 (EA)
- **ビルドツール**: Maven 3.9+, Gradle 8.0+
- **テストフレームワーク**: JUnit 5, Testcontainers
- **データベース**: PostgreSQL, H2, Redis
- **監視・可観測性**: Micrometer, Grafana, Zipkin

### 📖 章構成

<div class="chapter-overview">
  <div class="chapter-group">
    <h4><i class="fas fa-foundation"></i> 基礎編 (序章〜第3章)</h4>
    <ul>
      <li>Spring Boot の概念と歴史</li>
      <li>アーキテクチャとエコシステム</li>
      <li>環境セットアップとプロジェクト作成</li>
    </ul>
  </div>
  
  <div class="chapter-group">
    <h4><i class="fas fa-cogs"></i> 機能編 (第4章〜第8章)</h4>
    <ul>
      <li>自動設定とスターター</li>
      <li>データ永続化 (JPA, R2DBC)</li>
      <li>Web アプリケーション開発</li>
      <li>運用監視とメトリクス</li>
      <li>統合とメッセージング</li>
    </ul>
  </div>
  
  <div class="chapter-group">
    <h4><i class="fas fa-rocket"></i> 実践編 (第9章〜第13章)</h4>
    <ul>
      <li>セキュリティ実装</li>
      <li>包括的テスト戦略</li>
      <li>コンテナ化とクラウド</li>
      <li>リアクティブプログラミング</li>
      <li>本番運用とトラブルシューティング</li>
    </ul>
  </div>
</div>

### 🔄 更新履歴

#### 最新更新 (2025年6月14日)

<div class="update-highlights">
  <div class="update-item">
    <h5><i class="fas fa-plus-circle text-success"></i> 新規追加</h5>
    <ul>
      <li><strong>第5章 データ永続化</strong> - Spring Data JPA、R2DBC、Redis の包括的解説</li>
      <li><strong>Grafana Tempo 統合</strong> - 分散トレーシングの実装例</li>
      <li><strong>ECS準拠JSONログ</strong> - 本番環境でのログ出力設定</li>
      <li><strong>WebAuthn サンプル</strong> - 最新認証技術の実装例</li>
    </ul>
  </div>
  
  <div class="update-item">
    <h5><i class="fas fa-sync-alt text-primary"></i> 強化・更新</h5>
    <ul>
      <li><strong>Java 24 EA 対応</strong> - 最新Java版での動作確認</li>
      <li><strong>Testcontainers 並列実行</strong> - テスト効率化手法</li>
      <li><strong>ネイティブイメージビルド</strong> - GraalVM最適化設定</li>
      <li><strong>Spring Security 6.5</strong> - 最新セキュリティ機能</li>
    </ul>
  </div>
</div>

### 🤝 コントリビューション

このガイドは継続的に改善されています。以下の方法でご協力いただけます：

#### Issues & Feedback
- [GitHub Issues](https://github.com/shinyay/getting-started-with-spring-boot-3.5.0/issues) でバグ報告や改善提案
- 誤字脱字、技術的な間違いの指摘
- 新しいトピックやサンプルコードのリクエスト

#### Pull Requests
- ドキュメントの修正・改善
- サンプルコードの追加・更新
- 翻訳やローカライゼーション

### 📞 サポート

#### コミュニティサポート
- **GitHub Discussions**: 技術的な質問や議論
- **Stack Overflow**: `spring-boot-3.5` タグで質問
- **Spring Community**: 公式コミュニティフォーラム

#### 公式リソース
- [Spring Boot 公式ドキュメント](https://docs.spring.io/spring-boot/docs/3.5.0/reference/html/)
- [Spring.io](https://spring.io/) - 公式サイト
- [Spring Boot GitHub](https://github.com/spring-projects/spring-boot)

### 🏆 謝辞

このガイドの作成にあたり、以下の方々とプロジェクトに感謝いたします：

#### Spring Team
Spring Boot および Spring Framework の開発チームの皆様

#### コミュニティ
- フィードバックをいただいた読者の皆様
- サンプルコードや改善提案をいただいたコントリビューターの皆様
- 技術レビューにご協力いただいた専門家の皆様

#### 参考文献・リソース
- Spring Boot Reference Documentation
- Spring Framework Documentation  
- Baeldung Spring Boot Tutorials
- Josh Long's Spring Boot Examples

### 📄 ライセンス

このドキュメントは [MIT License](https://opensource.org/licenses/MIT) の下で公開されています。

```
MIT License

Copyright (c) 2025 Shinya Yanagihara

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
```

### 🔮 今後の予定

#### 次回アップデート予定 (2025年Q3)
- **Project Loom 完全対応**: Virtual Threads の深堀り解説
- **Spring AI 統合**: 機械学習・AI機能の統合方法
- **Kubernetes Operator**: Spring Boot アプリのオペレーター開発
- **GraalVM ネイティブ最適化**: 起動時間とメモリ使用量の極限最適化

#### 長期計画
- **インタラクティブチュートリアル**: ブラウザ上で実行可能なサンプル
- **ビデオコンテンツ**: 主要機能の動画解説
- **多言語対応**: 英語版の提供
- **モバイル対応**: スマートフォンでの読みやすさ向上

---

<div class="contact-section">
  <h3><i class="fas fa-envelope"></i> お問い合わせ</h3>
  <p>ご質問、ご提案、ご指摘がございましたら、お気軽にお問い合わせください。</p>
  
  <div class="contact-methods">
    <div class="contact-method">
      <i class="fab fa-github"></i>
      <strong>GitHub</strong>
      <a href="https://github.com/shinyay" target="_blank" rel="noopener">@shinyay</a>
    </div>
    
    <div class="contact-method">
      <i class="fab fa-twitter"></i>
      <strong>Twitter</strong>
      <a href="https://twitter.com/yanashin18618" target="_blank" rel="noopener">@yanashin18618</a>
    </div>
    
    <div class="contact-method">
      <i class="fas fa-envelope"></i>
      <strong>Email</strong>
      <a href="mailto:shinya.yanagihara@gmail.com">shinya.yanagihara@gmail.com</a>
    </div>
  </div>
</div>

<style>
.chapter-overview {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
  margin: 2rem 0;
}

.chapter-group {
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
  padding: 1.5rem;
  border-radius: 8px;
  border-left: 4px solid #6db33f;
}

.chapter-group h4 {
  color: #34302d;
  margin-bottom: 1rem;
}

.chapter-group i {
  color: #6db33f;
  margin-right: 0.5rem;
}

.update-highlights {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1.5rem;
  margin: 1.5rem 0;
}

.update-item {
  background: white;
  padding: 1.5rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.update-item h5 i {
  margin-right: 0.5rem;
}

.text-success { color: #28a745; }
.text-primary { color: #007bff; }

.contact-section {
  background: linear-gradient(135deg, #34302d 0%, #2c2a27 100%);
  color: white;
  padding: 2rem;
  border-radius: 8px;
  margin-top: 3rem;
}

.contact-section h3 {
  color: white;
  margin-bottom: 1rem;
}

.contact-methods {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
  margin-top: 1.5rem;
}

.contact-method {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.contact-method i {
  color: #6db33f;
  width: 1.5rem;
}

.contact-method a {
  color: #6db33f;
  text-decoration: none;
}

.contact-method a:hover {
  color: #5fa134;
  text-decoration: underline;
}

@media (max-width: 768px) {
  .chapter-overview,
  .update-highlights {
    grid-template-columns: 1fr;
  }
  
  .contact-methods {
    grid-template-columns: 1fr;
  }
}
</style>