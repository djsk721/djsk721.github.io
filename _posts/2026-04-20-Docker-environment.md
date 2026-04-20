---
title: "Open WebUI, Pipelines, n8n 환경설정 및 Docker Compose 가이드"
date: 2026-04-20
description: Open WebUI, Pipelines, n8n의 Docker 기반 통합 환경 구축 및 설정 방법, 주요 옵션 설명
categories: [devops]
tags: [webui, pipelines, n8n, docker, compose, 환경설정, 자동화, 워크플로우]
---

# Open WebUI, Pipelines, n8n 환경설정 통합 가이드

이 문서에서는 Open WebUI, Pipelines, n8n 워크플로우 자동화 툴을 Docker Compose를 이용해 한번에 손쉽게 구축하고 주요 설정을 관리하는 방법을 설명한다.
내 경우 Ollama는 Local 환경에 설치되어 있기에 필요에 따라 Ollama를 로컬 환경에 설치하거나 Docker에 추가하여 활용할 수 있다. (삭제하면 모델 가중치 파일도 다 날아가서 로컬에 설치하였음.)

---

## 1. 환경 개요

세 개의 서비스를 연동하여 운영된다:

- **Open WebUI:** 나의 경우 Ollama 백엔드와 연계하기 위한 Web UI 서비스  
- **Pipelines:** Open WebUI와 연결되는 파이프라인 자동화 서비스  
- **n8n:** 범용 워크플로우(워크플로 자동화) 오픈소스 플랫폼 (다양한 API 호출 지원)

모든 컨테이너는 최신 이미지로 유지하며 간단히 설정 변경 및 관리가 가능

---

## 2. docker-compose 구성 예시

아래 구성 예시를 `docker-compose.yaml`로 저장 후 사용:

```yaml
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: always
    network_mode: host
    environment:
      - PORT=3000
      - OLLAMA_BASE_URL=http://localhost:11434
    volumes:
      - open-webui:/app/backend/data

  pipelines:
    image: ghcr.io/open-webui/pipelines:main
    container_name: pipelines
    restart: always
    network_mode: host
    environment:
      - PORT=9099
      - PIPELINES_URL=http://localhost:9099
      - OLLAMA_BASE_URL=http://localhost:11434
    volumes:
      - ./pipelines:/app/pipelines

  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: always
    ports:
      - "5678:5678"
    volumes:
      - ~/.n8n:/home/{PATH}/.n8n
    environment:
      - N8N_HOST=0.0.0.0
      - N8N_PORT=5678
      - WEBHOOK_URL=http://localhost:5678/
    extra_hosts:
      - "host.docker.internal:host-gateway" 

volumes:
  open-webui:
```

---

## 3. 각 서비스별 주요 설정 설명

### Open WebUI

- **이미지:** `ghcr.io/open-webui/open-webui:main`
- **네트워크 모드:** `host` (로컬 네트워크와 동일하게 접근)
- **환경 변수:**
  - `PORT=3000`: WebUI 접속 포트
  - `OLLAMA_BASE_URL`: Ollama 등 모델 백엔드 주소
- **데이터 볼륨:** `open-webui:/app/backend/data`  
  (영구 데이터 저장)

### Pipelines

- **이미지:** `ghcr.io/open-webui/pipelines:main`
- **환경 변수:**
  - `PORT=9099`: 파이프라인 서비스 포트
  - `PIPELINES_URL`: 자기 자신 주소
  - `OLLAMA_BASE_URL`: Ollama 백엔드 주소(연동)
- **볼륨:** `./pipelines:/app/pipelines`  
  (파이프라인 구성 파일 등 영구 저장)

### n8n

- **이미지:** `n8nio/n8n:latest`
- **포트 매핑:** `5678:5678`
- **환경 변수:**
  - `N8N_HOST`, `N8N_PORT`, `WEBHOOK_URL` 등 워크플로/웹훅 구성
- **볼륨:** `~/.n8n:/home/{PATH}/.n8n`  
  (설정 및 워크플로 데이터 저장)
- **extra_hosts:**  
  - `"host.docker.internal:host-gateway"`  
    (Docker 컨테이너에서 호스트 네트워크 접근 시 필요)

---

## 4. 실행 방법

1. 위의 `docker-compose.yaml` 내용을 저장  
2. 터미널에서 해당 디렉토리로 이동  
3. 다음 명령으로 서비스 일괄 기동  
   ```bash
   docker compose up -d
   ```
4. 각 서비스는 아래 주소로 접근  
   - Open WebUI: <http://localhost:3000>
   - Pipelines: <http://localhost:9099>
   - n8n: <http://localhost:5678>

---

## 5. 참고 & 팁

- **최초 실행 시 볼륨/디렉토리가 자동 생성되며, 기존 데이터가 있으면 그대로 이어서 사용됨**
- **Ollama 등 AI 백엔드가 별도로 구동 중이어야 WebUI가 정상 동작**
- **포트/경로는 필요에 맞게 수정 가능**

---
