# DESIGN.md — djsk721.github.io 디자인 시스템 계약

## 1. 디자인 방향

깔끔한 라이트 테마의 한국어 기술 블로그. Slate 중성색 기반 + 단일 액센트(blue #2563eb).
시그니처 material: **라이트 페이지 위의 다크 에디터형 코드 블록** — 코드 중심 블로그의
핵심 콘텐츠(코드)에 시각적 무게를 부여한다. 모든 값은 아래 토큰에서만 가져온다.

## 2. Color Tokens

### Brand / Accent (단일 액센트 유지)
| Token | Value | 용도 |
|---|---|---|
| `--color-primary` | `#2563eb` | 링크, 활성 상태, 포인트 |
| `--color-primary-dark` | `#1d4ed8` | hover/focus |
| `--color-primary-light` | `#3b82f6` | border highlight |
| `--color-secondary` | `#64748b` | secondary 버튼 |

### Neutrals (slate 계열 통일 — warm/cool 혼용 금지)
| Token | Value |
|---|---|
| `--color-background` | `#ffffff` |
| `--color-background-alt` | `--color-background-alt` |
| `--color-surface` | `#ffffff` |
| `--color-text` | `#1e293b` |
| `--color-text-secondary` | `#475569` |
| `--color-text-muted` | `#64748b` |
| `--color-border` | `#e2e8f0` |
| `--color-border-light` | `#f1f5f9` |

### Code Block (신규 — 다크 에디터 재질)
| Token | Value | 용도 |
|---|---|---|
| `--code-bg` | `#0f172a` | 블록 배경 (slate-900) |
| `--code-border` | `#1e293b` | 블록 테두리 |
| `--code-text` | `#e2e8f0` | 기본 코드 텍스트 |
| `--code-header-bg` | `#16213b` | 언어 라벨 바 배경 |
| `--code-muted` | `#64748b` | 주석, 라인번호 |
| Syntax (Rouge 다크 테마 매핑) | | |
| `--syntax-keyword` | `#93c5fd` | keyword, tag |
| `--syntax-string` | `#86efac` | string, char |
| `--syntax-number` | `#fbbf24` | number, literal |
| `--syntax-comment` | `#7c8db0` | comment |
| `--syntax-function` | `#c4b5fd` | function/name |
| `--syntax-operator` | `#94a3b8` | operator, punctuation |
| 인라인 코드 | bg `#f1f5f9`, text `#334155`, border `#e2e8f0` | 라이트 유지 |

규칙: 코드 블록 내 파스텔 syntax 색상은 다크 배경 위 대비 ≥ 4.5:1만 사용.
이 밖의 새로운 hex 추가 금지 — 필요하면 먼저 이 파일에 토큰을 추가.

## 3. Typography

- Sans: `Apple SD Gothic Neo, Noto Sans KR, -apple-system, ...` (기존 스택 유지)
- Mono: `SF Mono, Fira Code, Roboto Mono, Monaco, Consolas, monospace`
- Scale: xs 0.75 / sm 0.875 / base 1 / lg 1.125 / xl 1.25 / 2xl 1.5 / 3xl 1.875 / 4xl 2.25rem
- 날짜·카운트 등 숫자에는 `font-variant-numeric: tabular-nums`
- 제목: `line-height-tight`(1.25) + weight 600–700, 본문 line-height-relaxed(1.75)
- 게시물 본문(`.post-content`) 최대 폭 제한 없음(그리드가 관리), 문단 폭 체감 ~65ch

## 4. Spacing / Radius / Shadow / Motion (기존 유지)

- Space: 1=4px … 12=48px 스케일
- Radius: sm 4 / md 8 / lg 12 / xl 16px. 컨테이너는 lg–xl, 내부 요소는 sm–md
- Shadow: sm/md/lg 기존 정의. shadow tint는 slate 기반(rgb(15 23 42 / x)) 허용
- Motion: fast 150ms / base 250ms / slow 350ms ease. **GPU-composited 속성만**
  (transform, opacity, filter). 레이아웃 속성 애니메이션 금지.
  `@media (prefers-reduced-motion: reduce)`에서 transition/animation 비활성.

## 5. Components & States

### Header / Nav
- sticky top, backdrop blur 유지. `.nav-current` = primary color + weight 600
- 모바일: 햄버거 → max-height 드롭다운 (기존 동작 유지)

### Post Card (홈)
- border + radius-lg + shadow-sm, hover: translateY(-2px)+shadow-md (transform만)
- 구성: **카테고리 칩**(신규, 좌상단) → 제목 → 날짜(tabular-nums) → 발췌 → 태그 칩들

### Chip (카테고리/태그 공용 프리미티브 — 신규)
- 패딩 4px×10px, radius-sm, bg background-alt, border border-light, font-xs
- 카테고리 칩은 primary tint variant 허용: bg `rgb(37 99 235 / .08)`, text primary-dark
- hover: primary bg + white text (기존 태그 hover 규칙 승계)

### Categories Archive (/categories/)
- 페이지 헤더: h1 + 총 게시물 수 표시
- 각 섹션: 카테고리명(h2) + 개수. 포스트 행은 리스트 row — hover 시 배경 alt 전환,
  링크 text-secondary→primary. 날짜 우측 정렬(tabular-nums), 모바일에서 세로 스택
- 필터 상태 메시지는 한국어 ("전체 카테고리 N개", "『X』 카테고리 글 보는 중") + aria-live
- 해시 필터 동작(기존 JS) 유지 — data-category-id 계약 그대로

### Sidebar
- 데스크톱(≥1024px): sticky 18rem 열, 모바일: 콘텐츠 하단. 기존 grid-area 계약 유지

### Post Page
- 메타: 날짜 · 카테고리 칩 · 읽는 시간(추정, 200wpm)
- 하단 네비게이션: **이전 글 ← / 홈 / → 다음 글 3분할**(신규 — 기존은 다음글만)
- taxonomy footer: 카테고리는 /categories/#slug 링크 칩, 태그는 #tag 텍스트

### Code Block (`.highlight pre`)
- radius-lg, padding 16×20, overflow-x auto, font-mono, font-size sm, line-height 1.7
- 상단 바: 언어명 소문자 라벨(code-muted), 우측 상단 모서리 radius 0 처리 없음(통일 lg)
- 인라인 `code`: 라이트 배경 유지(위 표)

### Blockquote (신규)
- 좌측 3px primary 보더, bg background-alt, radius-md, text-secondary

## 6. Accessibility Constraints

- 모든 인터랙티브 요소 `:focus-visible`에 2px primary outline + 2px offset
- 필터 상태 변경은 `aria-live="polite"` 영역으로 안내 (기존 유지)
- 본문 텍스트 대비 ≥ 4.5:1 (text-secondary on white = 7.6:1 ✅)
- 햄버거 버튼 aria-expanded/label 토글 (기존 유지)

## 7. Responsive Behavior

- Breakpoints: 640 / 767(mobile menu) / 1024(sidebar 전환)
- 카테고리 행·포스트 카드: <640px 세로 스택
- 콘텐츠 최대폭 `--layout-max-width: 90rem` 유지

## 8. Accepted Debt

- 다크모드 미지원 (추후 `prefers-color-scheme` 확장 여지)
- 페이지네이션(jekyll-paginate) 미사용 — 홈에 전체 목록 노출
- MathJax 전역 로드 → post 레이아웃으로 한정하는 것으로 완화
