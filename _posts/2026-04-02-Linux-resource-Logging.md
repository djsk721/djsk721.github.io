---
title: "sysstat을 활용한 CPU 및 RAM 리소스 로깅 가이드"
date: 2026-04-02
description: sysstat 패키지로 Linux 시스템 CPU, 메모리 사용량을 기록하고 모니터링하는 방법
categories: [devops]
tags: [sysstat, linux, sar, cpu, memory, monitor, logging, resource]
---

## sysstat을 사용해서 CPU, RAM 사용량 변화 기록하기

Linux 시스템에서 `sysstat` 패키지를 설치하고, sar 명령어를 통해 CPU와 RAM 등의 자원 사용현황을 기록·확인할 수 있습니다.

---

### 1. sysstat 설치

```bash
sudo apt-get install sysstat
```

---

### 2. sysstat 설정

```bash
# /etc/default/sysstat 파일을 열어 활성화(ENABLED=true로 변경)
sudo vim /etc/default/sysstat

sudo service sysstat enable      # 서비스 활성화
sudo service sysstat restart     # 서비스 재시작

# 로그 주기 및 로그 항목은 cron설정으로 조절(필요시)
sudo vim /etc/cron.d/sysstat
```

---

### 3. 기본 sar 명령어 사용법

```bash
# CPU 사용량 2초마다 사람이 읽기 좋은 형태로 5회 기록
sar -p 2 5 --human

# RAM(메모리) 사용량 2초마다 사람이 읽기 좋은 형태로 5회 기록
sar -r 2 5 --human

# 전날(XX는 날짜) 기록된 로그 파일 확인
sar -f /var/log/sysstat/saXX
```

---

#### 주요 sar 옵션 요약

- `-u`: CPU 사용률 통계 표시
- `-r`: 메모리(RAM) 사용 통계 표시
- `-q`: 로드 평균 및 대기 큐 통계
- `-n`: 네트워크 통계 표시
- `-b`: 블록 디바이스 통계 표시
- `-d`: 디스크 I/O 통계 표시
- `-p`: 블록 디바이스와 파티션 자세히 보기
- `-W`: 스왑 공간 통계
- `-P`: 프로세서별 통계

---

### 참고

- sar의 자세한 사용법: `man sar` 또는 `sar --help` 참고
- 전체 리소스 모니터링 및 자동화 기록 환경 구축에 매우 유용

---