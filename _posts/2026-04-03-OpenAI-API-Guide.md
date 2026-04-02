---
title: "OpenAI Batch API 활용 가이드 (채팅/번역/분석)"
date: 2026-04-03
description: OpenAI API의 대용량 배치(chat/completions) 처리 실전 가이드. 파일 업로드, 배치 생성, 결과 조회까지 단계별 실습 예제 포함.
categories: [python]
tags: [openai, api, batch, 파이썬, 번역, 자동화, 분석, 대량처리]
---

## 1. API 키 설정

OpenAI API를 사용하려면 먼저 API 키가 필요합니다.

- 보통 환경변수나 설정파일을 통해 관리합니다. (예: python-dotenv, config 파일 등)

```python
from openai import OpenAI

client = OpenAI(api_key=config["OPEN_API_KEY"])
```

---

## 2. 배치 입력 파일 업로드

OpenAI 배치 API는 `.jsonl`(JSON Lines) 포맷의 입력 파일을 받아 처리합니다.<br>
**각 줄이 하나의 요청(request)이며, 아래처럼 messages 배열이 포함된 JSON 형태입니다.**

```json
{"messages": [{"role": "user", "content": "Translate: ST depression in lead II"}]}
{"messages": [{"role": "user", "content": "Translate: Right bundle branch block"}]}
```

파이썬으로 입력 파일을 API에 업로드하려면:

```python
# OpenAI API를 통해 파일을 업로드
batch_input_file = client.files.create(
    file=open("batchinput.jsonl", "rb"),
    purpose="batch"
)
batch_input_file_id = batch_input_file.id
```

---

## 3. 배치 생성

업로드된 파일 ID를 이용해 배치 작업을 생성합니다.<br>
- 사용할 endpoint (`/v1/chat/completions` 등)와, 처리 완료를 기다릴 기간(최대 24시간)을 지정할 수 있습니다.

```python
res = client.batches.create(
    input_file_id=batch_input_file_id,
    endpoint="/v1/chat/completions",
    completion_window="24h",
    metadata={
        "description": "ptb_xl_report_translation"
    }
)
```

응답 객체 `res`에는 배치 ID, 상태, 생성 시간 등의 정보가 담깁니다.

---

## 4. 결과 확인

배치가 완료되면 결과 파일은 OpenAI 서버 내 파일시스템에 저장됩니다.<br>
아래와 같이 상태와 output 파일 ID를 확인할 수 있습니다.

```python
batch = client.batches.retrieve(res.id)
print(batch.status)
print(batch.output_file_id)
```

**OpenAI Platform → Batches 탭**에서 결과 jsonl 파일을 다운로드할 수도 있습니다.

---

## 비고 (팁)

- **Batch Processing**은 수천~수만개 요청을 저렴하고 효율적으로 처리할 때 유용합니다.
- 반드시 입력 파일 포맷(JSONL) 및 각 요청의 구조(`{"messages": [...]}`)를 명확히 맞춰야 합니다.
- 결과 jsonl에는 success/failed 요청별 output이 한 줄씩 기록됩니다.
- GUI: OpenAI API 플랫폼(Dashboard)의 좌측 Batches 탭에서 직접 결과를 관리/다운로드할 수 있습니다.
- 비용은 단건 처리 대비 저렴(2024년 4월 기준), 속도는 처리량/대기열에 따라 수 분~수 시간.

---
참고: [OpenAI 공식 Batch API 소개](https://platform.openai.com/docs/guides/batch)
