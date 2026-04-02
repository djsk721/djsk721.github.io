---
title: "ulimit을 통한 Linux 리소스 제한 설정 및 관리"
date: 2026-04-02
description: ulimit 명령어로 Linux에서 프로세스별 리소스 한도를 조회, 설정, 관리하는 방법에 대한 실전 가이드
categories: [devops]
tags: [ulimit, linux, resource, limit, process, 관리, 제한, 운영]
---

## ulimit이란?

리눅스에서 각 사용자의 프로세스별 리소스(파일, 메모리, 프로세스 수 등) 최대 한도를 조회하거나 설정할 수 있는 기능입니다. 적절한 리소스 제한을 통해 시스템 안정성과 예측 가능한 운영 환경을 갖출 수 있습니다.

---

## 1. 현재 시스템의 ulimit 설정값 조회

```bash
# 모든 리소스의 현재 설정값(Soft Limit) 조회
ulimit -a

# 소프트(soft) 제한 값만 조회
ulimit -Sa

# 하드(hard) 제한 값만 조회
ulimit -Ha
```

---

## 2. ulimit 주요 옵션 및 설정 항목

아래는 ulimit으로 설정 가능한 주요 리소스 별 항목 및 옵션입니다.

1. core file size [core] (-c): 생성 가능한 core dump 파일의 최대 크기
2. data seg size [data] (-d): 하나의 프로세스가 사용할 수 있는 data segment 최대값
3. file size [fsize] (-f): 생성할 수 있는 파일의 최대 크기
4. pending signals [sigpending] (-i): 생성 가능한 pending signal의 최대 개수
5. max locked memory [memlock] (-l): locked memory의 최대값 (swap 방지 등)
6. max memory size [rss] (-m): resident set size(메모리) 최대값
7. open files [nofile] (-n): 최대 동시 열린 파일(파일 디스크립터) 개수
8. pipe size (-p): 파이프의 크기 (512바이트 블록 단위)
9. POSIX message queues [msgqueue] (-q): 메시지 큐의 최대 바이트 수
10. real-time priority [rtprio] (-r): 실시간 스케줄링 우선순위 최대값
11. stack size [stack] (-s): 스택의 최대 크기
12. cpu time [cpu] (-t): 한 프로세스가 사용할 수 있는 CPU 시간(초)
13. max user processes [nproc] (-u): 한 유저가 생성할 수 있는 프로세스 최댓값
14. virtual memory (-v): 최대 가상 메모리 크기
15. file locks [locks] (-x): 파일 잠금의 최대 개수

---

## 3. ulimit 값 변경하기

설정할 항목과 값을 명시해 아래와 같이 변경할 수 있습니다.  
**(셸에서 직접 적용 시 해당 셸, 하위 프로세스에만 유효, 임시적 효과)**

```bash
ulimit -n 4096      # 예시: 한 프로세스가 열 수 있는 파일 개수(soft/hard 둘 다 변경)
ulimit -s 8192      # 스택 크기 제한 설정(단위: KB)
ulimit -u 2048      # 최대 프로세스 수 제한

# (일부 설정값은 -S(soft), -H(hard)로 구분 적용 가능)
ulimit -Sn 1024     # 소프트 리미트만 변경
ulimit -Hn 4096     # 하드 리미트만 변경
```

---

## 4. ulimit의 소프트/하드 리밋 차이

- **소프트 리밋(Soft limit):** 프로세스 시작/실행 시 기본적으로 적용되는 리소스 한도
- **하드 리밋(Hard limit):** 해당 리소스의 절대 상한선, soft limit 값이 상향 조정 가능한 최대값
- Soft limit은 hard limit보다 항상 작거나 같으며, 일반 사용자는 소프트 리밋만 조절(하드 리밋 상향은 root 권한 필요)

> 실시간 운영에선 soft limit으로 먼저 제약 후, 필요시 일시적으로 hard limit까지 확장 가능

---

## 5. 영구적/시스템 전체 적용을 위한 설정

- 위 명령어들은 로그인 셸 단위(임시)로 적용되며, 시스템 전체/유저별 영구 설정을 위해선  
  `/etc/security/limits.conf` 파일을 편집하여 아래와 같이 추가합니다.

예시:

```
# [도메인] [타입] [항목] [값]
username   soft    nofile  1024
username   hard    nofile  4096
*          soft    nproc   4096
*          hard    nproc   16384
```

- 적용 후에는 로그아웃/시스템 재시작 또는 해당 사용자 재접속 필요

---

## 참고

- 자세한 옵션: `man ulimit` 또는 `help ulimit`, `ulimit --help` 참고
- 리소스 한도 미설정 시 시스템 전체나 여러 사용자가 동시에 사용하는 환경에서 장애나 성능 저하 위험 주의

---