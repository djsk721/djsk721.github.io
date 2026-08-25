# DESIGN.md — djsk721.github.io 디자인 시스템 계약

## 1. 디자인 방향

**Notion · Linear · GitHub Docs처럼 정보 밀도와 가독성을 우선하는 기술 문서형 UI.**
화려함보다 "빠르게 훑고, 찾고, 읽는다". Slate 중성색 + 단일 액센트(blue), 기본 상태는 flat —
테두리·그림자·카드 최소화, hover에서만 subtle 피드백. 라이트/다크 모드 모두 지원하며
다크는 pure black이 아닌 GitHub 계열 dark surface.

## 2. Color Tokens

### Light (기본)
| Token | Value | 용도 |
|---|---|---|
| `--color-primary` | `#2563eb` | 링크, 활성 상태 |
| `--color-primary-dark` | `#1d4ed8` | hover/focus |
| `--color-primary-light` | `#3b82f6` | border highlight |
| `--color-background` | `#ffffff` | 페이지 배경 |
| `--color-background-alt` | `#f8fafc` | hover 배경, 인용구 |
| `--color-surface` | `#ffffff` | (flat 원칙상 background와 동일) |
| `--color-text` | `#1e293b` | 본문/제목 |
| `--color-text-secondary` | `#475569` | 보조 텍스트 |
| `--color-text-muted` | `#64748b` | 메타/날짜 |
| `--color-border` | `#e2e8f0` | hairline 구분선 |
| `--color-border-light` | `#f1f5f9` | 칩 테두리 |

### Dark (GitHub 계열 — pure black 금지)
| Token | Value |
|---|---|
| `--color-primary` | `#4493f8` (GitHub dark link blue) |
| `--color-primary-dark` | `#6cb0ff` (hover — 다크에서는 밝아진다) |
| `--color-background` | `#0d1117` |
| `--color-background-alt` | `#151b23` |
| `--color-surface` | `#161b22` |
| `--color-text` | `#e6edf3` |
| `--color-text-secondary` | `#b6c2cf` |
| `--color-text-muted` | `#8b949e` |
| `--color-border` | `#30363d` |
| `--color-border-light` | `#21262d` |

### Code Block
라이트: bg `#0f172a` / border `#1e293b` / text `#e2e8f0` / header `#16213b` / muted `#7c8db0`
다크: bg `#161b22` / border `#30363d` (토큰만 교체, 구조 동일)
Syntax (공통): keyword `#93c5fd` · string `#86efac` · number `#fbbf24` · comment `#7c8db0` · function `#c4b5fd` · operator `#94a3b8`
인라인 코드: 라이트 bg `#f1f5f9` text `#334155` / 다크 bg `rgb(110 118 129 / .25)` text `#e6edf3`

테마 스위칭: `:root` (라이트) + `html[data-theme="dark"]` 오버라이드.
`color-scheme` 속성 함께 토글. 새 hex 추가 전 이 파일에 먼저 등록.

## 3. Layout

| 컨테이너 | 폭 | 용도 |
|---|---|---|
| `--width-home` | `76rem` (1216px) | 홈/카테고리 등 목록 페이지 (사이드바 + 리스트) |
| `--width-post` | `46rem` (736px) | 게시글 본문 열 |
| `--width-toc` | `15rem` (240px) | 본문 우측 TOC 열 (≥1200px만) |

- 헤더 내부도 `--width-home` 정렬
- 포스트 페이지: 좌측 카테고리 사이드바 대신 본문(46rem) + 우측 TOC 2열
- 목록 페이지: 좌측 사이드바(15rem) + 콘텐츠

## 4. Typography

- Sans: `Apple SD Gothic Neo, Noto Sans KR, -apple-system, ...` / Mono: `SF Mono, Fira Code, ...`
- **한글 규칙**: `word-break: keep-all` + `overflow-wrap: break-word` (어절 단위 줄바꿈)
- 본문 line-height 1.8 (한글 가독), 제목 1.25, 코드 1.7
- Scale: xs .75 / sm .875 / base 1 / lg 1.125 / xl 1.25 / 2xl 1.5 / 3xl 1.875 / 4xl 2.25rem
- 위계: 제목 text+600 / 본문 secondary+400 / 메타 muted+sm / 날짜 tabular-nums
- 날짜·카테고리·읽기시간 등 metadata는 아이콘 없이 텍스트만, `·` 구분

## 5. Components & States

### Post List Row (홈 — 카드 그리드 대체)
- flat row, 행 구분은 `border-bottom` hairline. 박스/그림자/둥근테두리 없음
- 구조: `[chip] 제목 ······ 날짜` / 아래 설명(sm, muted, 1줄 clamp) / 아래 태그
- 태그는 **최대 2개 + `+N`** 축약 (홈 한정)
- hover: row 배경 `--color-background-alt`만 (translate/그림자 금지), 제목 색 유지
- hero: 좌측 정렬, 배경 그라디언트 제거(flat), 제목 3xl + 설명 + "총 N개의 글" 메타

### Header (sticky)
- 높이 4rem, 하단 hairline, backdrop blur
- nav-current: 텍스트 색 + **하단 2px 인디케이터** (색상만으로 구분 금지)
- 우측: 검색 버튼(`Ctrl K` 키 힌트 표시, 아이콘+텍스트) · 테마 토글(아이콘 버튼)
- 모바일: 햄버거 + 검색/테마 아이콘 유지

### Command Palette (Ctrl/Cmd+K)
- overlay: `rgba(0,0,0,.5)` 배경 + 중앙 패널(radius-lg, surface, border, shadow-lg)
- 입력 즉시 필터(제목·설명·카테고리·태그 대상, 대소문자 무시), 결과 없으면 빈 상태 문구
- 키보드: ↑↓ 이동 / Enter 이동 / Esc 닫기 / 클릭 닫기. 활성 항목 배경 alt
- 열림 때 body 스크롤 잠금, 닫히면 해제. 검색 데이터는 인라인 JSON (`#search-data`)

### TOC (포스트)
- 데스크톱 ≥1200px: 우측 sticky(top = header+여백), h2/h3 트리(h3 들여쓰기), xs 크기
- scrollspy: 현재 섹션 링크 `--color-text` + 좌측 2px primary 인디케이터, 나머지 muted
- 모바일/<1200px: 본문 상단 `<details>` 접이식 ("목차" 요약, 열면 같은 리스트)
- heading에 id 없으면 JS가 부여

### Chip
- 패딩 3px 10px, radius-sm, xs, border-light. 카테고리 variant: primary tint
- hover: primary bg + white. 필터바의 활성 칩은 `is-active` (primary 채움)

### Categories 페이지
- 상단 칩 필터바: `전체 N` + 카테고리별 `이름 N` — 클릭 시 해당 섹션만 표시, URL hash 동기화(기존 딥링크 유지)
- 섹션/행 구조는 유지, 행 hover는 배경 alt

### Sidebar (목록 페이지)
- flat: 카드 테두리/그림자 제거, 제목 + 링크 리스트 + 우측 개수(muted)
- sticky top = header + 여백

### Code / Table / 반응형 오버플로
- 코드 블록: radius-lg, x-scroll, 언어 라벨 우상단 (기존 유지)
- 표: block + x-scroll + 터치 스크롤, 셀 내 코드는 wrap
- 태그 목록/칩: wrap. 인라인 코드: `overflow-wrap: anywhere`

## 6. Accessibility Constraints

- `:focus-visible` 2px primary outline + 2px offset (다크에선 밝은 blue)
- 필터/검색 결과 수 변화는 aria-live 영역 안내, 팔레트 입력은 combobox 패턴(aria-expanded/activedescendant)
- 테마 토글은 `aria-label` + `aria-pressed`, 시스템 설정 기본 + localStorage 기억
- 대비 ≥ 4.5:1 (다크 muted #8b949e on #0d1117 = 5.0:1 ✅)
- `prefers-reduced-motion` 존중 (기존 블록 유지)

## 7. Motion

- 최소 원칙: hover/active/transition 150ms 위주, transform/opacity만
- row hover는 배경색만. 등장 애니메이션/스크롤 트리거 금지

## 8. Accepted Debt

- 검색은 클라이언트 필터(전문 검색 아님) — 30여 편 규모에 충분
- 페이지네이션 미사용 — 홈 전체 목록
- MathJax post 한정 로드
