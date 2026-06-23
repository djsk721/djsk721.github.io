---
title: "NVIDIA API와 OpenClaw 기반 텔레그램 봇 구축 정리"
date: 2026-06-19
categories: [ai]
tags: [nvidia, telegram, openclaw, bot, api, gpu, telegram-bot]
---

> NVIDIA API 키는 공식 [NVIDIA 개발자 포털](https://build.nvidia.com/)에서 발급받을 수 있다.

---

## 개요

NVIDIA의 공식 API와 OpenClaw를 활용해 무료로 사용할 수 있는 텔레그램 봇을 구축할 수 있다.

---

## 준비물

- NVIDIA API 키가 있어야 한다.
- Telegram Bot Token이 필요하다. (@BotFather를 통해 발급 가능하다.)
- OpenClaw가 필요하다.

---

## NVIDIA API 키 발급 방법

1. [NVIDIA AI API 페이지](https://build.nvidia.com/)에 접속해 회원가입/로그인한다.
2. [API 관리 페이지](https://build.nvidia.com/settings/api-keys)에서 키를 생성한다.
3. 생성된 API_KEY를 복사한다.
    ```
    API_KEY: sk-abcdefg123456...
    ```

---

## 텔레그램 봇 생성 및 토큰 발급

1. 텔레그램 앱에서 @BotFather로 검색해 `/newbot` 명령어로 봇을 만든다.
2. 생성된 토큰이 제공된다.
    ```
    123456789:AAH-XXXXYYYYZZZZzzzzzz
    ```
3. 토큰과 봇 이름을 기록해둔다.

---

## OpenClaw를 통한 기본 텔레그램 봇 구현

OpenClaw로 간단하게 핸들러 및 API 연동이 가능하다.

설치 방법:
```
curl -fsSL https://openclaw.ai/install.sh | bash
```

설치 중 nvidia API 키를 입력한다. (nvidia provider 설정 시 에러가 발생하면 custom provider로 연결해야 한다.)

![API 설정 예시](/assets/images/posts/2026-06-19-Telegram-bot-with-nvidia-api/api_setup.png)

설치 과정을 따라가며 Telegram bot token을 입력한다.

![봇 설정 예시](/assets/images/posts/2026-06-19-Telegram-bot-with-nvidia-api/telegram_setup.png)

설정이 완료되면 아래와 같이 된다.

![설치 후](/assets/images/posts/2026-06-19-Telegram-bot-with-nvidia-api/result_openclaw.png)

텔레그램에서 /start로 채팅을 시작할 수 있다.

OpenClaw가 실행 중인 운영체제에 값을 입력하면 페어링이 완료된다.


이후 ~/.openclaw/openclaw.json 파일에서 설정을 변경하면 된다.
다만, 설정 변경하다 설정이 꼬이는 경우 있는데 아래를 참고하면 된다.

설정만 초기화하려면:
```
openclaw reset --scope config
```
모든 세션과 인증정보까지 초기화하려면:
```
openclaw reset --scope full --yes
```

---

## 실제 사용 예시

- 텔레그램에서 **/gpu** 명령어를 입력하면 최신 GPU 리스트와 상태를 받아볼 수 있다.
- OpenClaw로 명령어 추가 및 응답 커스터마이즈가 쉽다.
- NVIDIA의 여러 REST API(모델 추론, 드라이버 체크, 텐서RT 상태 등)와 연동할 수 있다.

---

## 확장 아이디어

- GPU별 실시간 사용량, 온도, 메모리 정보 모니터링이 가능하다.
- Stable Diffusion 등 AI/ML 결과를 텔레그램 알림으로 받을 수 있다.
- 특정 크론 트리거, 하드웨어 이상 탐지 시 즉각 알림이 가능하다.

---

## 요약

NVIDIA API, OpenClaw, 텔레그램을 연동하면 인프라/AI/GPU 상태 및 결과를 텔레그램 챗봇으로 실시간 확인할 수 있다.  
복잡한 시스템도 쉽게 자동화하고 모니터링할 수 있는 조합이다.

[OpenClaw]: https://openclaw.ai/ "OpenClaw"
[NVIDIA API 문서]: https://build.nvidia.com/ "NVIDIA API"
[텔레그램 BotFather 안내]: https://core.telegram.org/bots#botfather
