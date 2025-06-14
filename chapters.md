---
layout: page
title: "目次"
description: "Spring Boot 3.5.0 完全ガイドの全章一覧"
---

<div class="chapters-grid">
  {% assign sorted_chapters = site.chapters | sort: 'path' %}
  {% for chapter in sorted_chapters %}
  <div class="chapter-card card">
    <div class="card-body">
      <div class="chapter-number">
        {% assign chapter_num = chapter.path | split: '/' | last | split: '-' | first | remove: 'chapter' %}
        {% if chapter_num == '00' %}
          序章
        {% else %}
          第{{ chapter_num }}章
        {% endif %}
      </div>
      <h3 class="chapter-title">
        <a href="{{ chapter.url | relative_url }}">{{ chapter.title | default: chapter.name }}</a>
      </h3>
      <div class="chapter-content">
        {{ chapter.content | strip_html | truncatewords: 30 }}
      </div>
      <div class="chapter-meta">
        <a href="{{ chapter.url | relative_url }}" class="btn btn-primary btn-sm">
          <i class="fas fa-book-open"></i> 読む
        </a>
      </div>
    </div>
  </div>
  {% endfor %}
</div>

<style>
.chapters-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
  gap: 2rem;
  margin-top: 2rem;
}

.chapter-card {
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.chapter-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.chapter-number {
  display: inline-block;
  background: linear-gradient(135deg, #6db33f 0%, #5fa134 100%);
  color: white;
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-size: 0.9rem;
  font-weight: 600;
  margin-bottom: 1rem;
}

.chapter-title {
  font-size: 1.25rem;
  margin-bottom: 1rem;
}

.chapter-title a {
  color: #34302d;
  text-decoration: none;
}

.chapter-title a:hover {
  color: #6db33f;
}

.chapter-content {
  color: #5f6368;
  line-height: 1.6;
  margin-bottom: 1.5rem;
}

.chapter-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

@media (max-width: 768px) {
  .chapters-grid {
    grid-template-columns: 1fr;
    gap: 1.5rem;
  }
}
</style>

## 学習の進め方

このガイドは以下の順序で学習することを推奨します：

<div class="learning-path">
  <div class="path-step">
    <div class="step-icon"><i class="fas fa-play"></i></div>
    <div class="step-content">
      <h4>1. 基礎理解</h4>
      <p><strong>序章〜第3章</strong>で Spring Boot の概念と環境セットアップを学習</p>
    </div>
  </div>
  
  <div class="path-step">
    <div class="step-icon"><i class="fas fa-cogs"></i></div>
    <div class="step-content">
      <h4>2. 機能習得</h4>
      <p><strong>第4章〜第8章</strong>で自動設定、データアクセス、Web開発の実践</p>
    </div>
  </div>
  
  <div class="path-step">
    <div class="step-icon"><i class="fas fa-shield-alt"></i></div>
    <div class="step-content">
      <h4>3. 本格運用</h4>
      <p><strong>第9章〜第13章</strong>でセキュリティ、テスト、デプロイメントを学習</p>
    </div>
  </div>
</div>

<style>
.learning-path {
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
  padding: 2rem;
  border-radius: 8px;
  margin: 2rem 0;
}

.path-step {
  display: flex;
  align-items: flex-start;
  margin-bottom: 1.5rem;
  
  &:last-child {
    margin-bottom: 0;
  }
}

.step-icon {
  width: 50px;
  height: 50px;
  background: linear-gradient(135deg, #6db33f 0%, #5fa134 100%);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 1.2rem;
  margin-right: 1.5rem;
  flex-shrink: 0;
}

.step-content h4 {
  color: #34302d;
  margin-bottom: 0.5rem;
}

.step-content p {
  color: #5f6368;
  margin: 0;
  line-height: 1.6;
}
</style>