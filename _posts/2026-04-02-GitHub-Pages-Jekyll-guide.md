---
title: "GitHub Pages + Jekyll 가이드"
date: 2026-04-02
description: GitHub Pages 를 사용하여 Jekyll 블로그를 만드는 방법.
categories: [tutorial]
tags: [tutorial, github-pages, jekyll]
---

## 개요

이 포스트는 GitHub Pages 와 Jekyll 을 이용하여 블로그를 만드는 방법을 공유합니다.

## 설치 방법

### 1. 레포지토리 생성

GitHub 에서 새로운 레포지토리를 생성합니다.

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_USERNAME.github.io.git
cd YOUR_USERNAME.github.io
```

### 2. Jekyll 설정

`_config.yml` 파일을 생성하고 기본 설정을 합니다:

```yaml
title: Your Blog Name
description: Your blog description
baseurl: "/"
url: "https://yourusername.github.io"
```

### 3. 첫 포스트 작성

`_posts` 디렉토리에 마크다운 파일을 추가합니다:

```markdown
---
layout: post
title: "Hello World"
date: 2026-04-02
description: My first post!
tags: [introduction, jekyll]
---

내용 작성...
```

## 로컬에서 미리보기

```bash
bundle install
bundle exec jekyll serve
```

이제 `http://localhost:4000` 에서 확인 가능합니다.

## 배포

Git 커밋 후 푸시하면 자동으로 GitHub Pages 에 배포됩니다:

```bash
git add .
git commit -m "Update blog content"
git push origin main
```

5 분 이내로 사이트가 업데이트됩니다!
