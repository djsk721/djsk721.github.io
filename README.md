# djsk721.github.io

개발자 김정현의 기술 블로그. Jekyll 4.3 기반 정적 사이트, GitHub Pages로 배포된다.

- 디자인 토큰·컴포넌트 계약은 [`DESIGN.md`](DESIGN.md) 참고

## 요구 사항

- Ruby 3.2 (`rbenv` 사용 중)
- Jekyll 4.3 및 의존 gem — `/home/jhkim/gems`에 설치되어 있음 (`gem list jekyll`로 확인)

## 로컬 실행

`.bundle/config`의 `BUNDLE_PATH`가 `~/.cache/djsk721-jekyll-bundle`(빈 디렉토리)을 가리키고 있어
`bundle exec jekyll serve`는 실패한다. 둘 중 하나로 해결:

### 방법 A — 바로 실행 (설정 수정 없음)

```bash
JEKYLL_NO_BUNDLER_REQUIRE=1 ruby /home/jhkim/gems/gems/jekyll-4.3.4/exe/jekyll serve
```

http://localhost:4000 접속. 기본 `--watch` 활성화라 파일 수정 시 자동 재빌드된다.

### 방법 B — bundle 정상화 (1회, 권장)

```bash
bundle config set --local path /home/jhkim/gems
bundle exec jekyll serve
```

이후로는 평소처럼 `bundle exec jekyll serve` 사용 가능.

## 빌드만 검증

```bash
JEKYLL_NO_BUNDLER_REQUIRE=1 ruby /home/jhkim/gems/gems/jekyll-4.3.4/exe/jekyll build
```

성공 기준: `done in N seconds` 출력, exit code 0. 결과물은 `_site/`에 생성된다.

## 테스트 체크리스트

수동 확인 포인트 (변경된 기능 기준):

| 페이지 | 확인 포인트 |
|---|---|
| 홈 `/` | 카드마다 카테고리 칩, 사이드바 "전체 카테고리", 카테고리 대소문자 중복(`AI`/`ai`) 없는지 |
| `/categories/` | 상단 "총 N개의 글 · M개의 카테고리", 섹션별 "N개의 글" |
| 카테고리 필터 | 사이드바 카테고리 클릭 또는 포스트 하단 칩 클릭 → 해당 섹션만 표시 + "「X」 카테고리의 글을 보고 있습니다." |
| 포스트 | 코드 블록 다크 스타일 + 우상단 언어 라벨 + 신택스 하이라이팅, 인용구 파란 보더, "약 N분 읽기", 하단 이전 글/홈/다음 글 |
| 모바일 (375px) | 햄버거 메뉴 열림/닫힘, 하단 네비 세로 스택 |

브라우저 개발자 도구 콘솔에 에러가 없는지도 함께 확인.

## 배포

`main`에 push하면 GitHub Pages가 자동 빌드·배포한다. 별도 서버 작업 불필요.

배포 전 로컬 빌드가 깨지지 않는지 먼저 확인할 것.

## 새 글 작성

`_posts/`에 `YYYY-MM-DD-제목.md` 형식으로 추가:

```markdown
---
title: "글 제목"
date: 2026-08-25
description: 검색/목록에 노출되는 요약 한두 문장.
categories: [devops]
tags: [docker, gpu]
---

본문...
```

규칙:

- **categories는 항상 소문자로 통일** (`AI`처럼 대문자를 섞으면 별개 카테고리로 분리되고 앵커 id가 중복된다)
- `description`은 홈 카드 발췌에 사용되므로 반드시 작성
- 카테고리 목록: `ai`, `devops`, `introduction`, `medical`, `mlops`, `paperreview`, `python`, `tutorial`

## 프로젝트 구조

```
_config.yml          # 사이트 설정 (exclude에 README 등 포함)
_layouts/            # default / page / post 레이아웃
_includes/           # header / sidebar / footer / head 부분 템플릿
_posts/              # 블로그 글 (파일명 = 날짜 + 슬러그)
assets/css/style.css # 유일한 스타일시트 (토큰은 DESIGN.md와 동기화)
categories.md        # /categories/ 페이지 (해시 기반 필터 포함)
index.md             # 홈 (포스트 카드 그리드)
DESIGN.md            # 디자인 시스템 계약 (토큰, 컴포넌트, 접근성)
```

## TODO

- 글 포스팅 주기(1주일)
