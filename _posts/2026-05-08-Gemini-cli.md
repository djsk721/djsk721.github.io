---
title: "GEMINI CLI 설치 및 사용법 가이드"
date: 2026-05-08
description: Gemini AI와 터미널에서 직접 상호작용할 수 있는 GEMINI CLI의 설치부터 기본 사용법까지 정리
categories: [ai]
tags: [gemini, cli, tool, tutorial]
---

## GEMINI CLI 설치 및 사용법

GEMINI CLI는 Gemini API와 상호작용할 수 있는 커맨드라인 툴입니다. 아래 절차에 따라 설치 및 인증, 기본 사용법을 정리

### 1. 설치 방법

**npm을 통한 설치 (권장)**
```bash
npm install -g @google/gemini-cli
```

*설치가 완료되면 `gemini --version` 명령어로 정상 설치 여부를 확인할 수 있음.*

---

### 2. 인증(Authentication) 설정

GEMINI CLI를 사용하려면 API Key를 발급받아야 함

1. [Google AI Studio](https://aistudio.google.com/app/apikey) 또는 [Gemini Developers Console](https://console.geminicli.com/)에 로그인
2. "API Keys" 메뉴에서 새 키를 생성
3. 생성된 키를 복사

**CLI에 API 키 등록하기**
```bash
gemini configure
# 또는 환경 변수 설정
export GEMINI_API_KEY='your_api_key_here'
```

---

### 3. 기본 사용 예시

**버전 확인**
```bash
gemini --version
```

**도움말 확인**
```bash
gemini --help
```

**대화형 모드 실행**
```bash
gemini
```

---

### 4. 주요 명령어 요약

| 명령어 | 설명 |
| :--- | :--- |
| `gemini` | 대화형 모드 시작 또는 프롬프트 실행 |
| `gemini configure` | API 키 등 환경설정 관리 |
| `gemini --version` | 현재 설치된 버전 확인 |
| `gemini --help` | 도움말 및 명령어 목록 확인 |

---

**더 자세한 사용법 및 옵션:**  
- 공식 문서: [https://ai.google.dev/docs](https://ai.google.dev/docs)
- 모든 명령어/옵션: `gemini --help`

