---
title: "우분투 환경 및 대용량 SSD 마운트 · 디렉터리 구조 · 환경이전 가이드"
date: 2026-04-02
description: 대용량 SSD에 Ubuntu 데이터 환경을 마운트하고, 파티션/마운트/디렉터리/환경 이전까지의 실전 가이드
categories: [devops]
tags: [ubuntu, storage, mount, ssd, setup, tutorial]
---

## 디렉터리 구조 예시

대용량 SSD(예: 2TB 이상)에 환경 분할 시 아래와 같은 디렉터리 계층 구조를 권장합니다:

```
├── 루트 디렉토리 (ssd 별도 mount)
│   ├── usr/local/           # CUDA 등 시스템용 소프트웨어
│   ├── home
│   │   ├── account1         # 공용 또는 일반 계정(예시, 실제 계정명은 임의 지정)
│   │   ├── account2         # 공용 환경 (DATASET, ANACONDA), HDD(4TB) - 유저 파일 별도
│   │   ├── user1
│   │   ├── user2
│   │   ├── user3
```
※ 위 계정명은 예시입니다. 실제 환경에 맞는 사용자 계정으로 대체하세요.

---

## 1. 저장장치 파티션 및 마운트 과정

2TB 이상의 저장장치는 GPT 파티션으로 구성해야 전체 용량을 인식/사용할 수 있습니다.

### 1-1. 파티션 레이블 변경 (GPT)

```bash
# 디스크 파티션 툴 실행
sudo parted [저장장치명]    # 예: /dev/nvme0n1 또는 /dev/sdb

# (parted 프롬프트 진입)
(parted) mklabel gpt
# q 명령어로 종료
```

- parted를 이용해 저장장치의 파티션 레이블을 gpt로 변경 (ex. /dev/sdb)

### 1-2. 확인 및 파티션 생성

```bash
sudo fdisk -l     # 변경된 용량 및 파티션 레이블 확인
```

> 정상적으로 gpt가 적용되고 전체 용량이 보이면 다음 단계로 진행

### 1-3. 마운트 및 영구 마운트(fstab)

```bash
# 1. 물리적 디렉터리에 마운트
sudo mount [저장장치-파티션명] [마운트할디렉터리]
# 예시: sudo mount /dev/sdb1 /home

# 2. 마운트 정상 적용 확인
df -h

# 3. 부팅시마다 자동 마운트를 위한 UUID 확인
sudo blkid [저장장치-파티션명]
# 예시 출력: UUID="7612bf30-0ca6-4445-bc0f-314217c5a3b1" TYPE="ext4" ...

# 4. /etc/fstab 파일에 마운트 정보 등록
sudo vim /etc/fstab

# 파일 하단에 아래와 같이 추가 (UUID는 blkid 결과 사용)
UUID=7612bf30-0ca6-4445-bc0f-314217c5a3b1   /home   ext4    defaults    0   0
```

### 1-4. 재부팅 후 정상 마운트 확인

```bash
sudo reboot
# 부팅 후 다시 df -h 등으로 /home 등 원하는 위치에 mount 되었는지 확인.
```

---

## 2. 데이터 및 환경(Anaconda 등) 이동

- 기존 데이터셋, anaconda3 환경 등도 새로 마운트된 경로(`/home` 등)로 이동할 수 있음
- 각 유저 홈, 공용 데이터셋 폴더, anaconda3 설치 디렉터리 등 적절히 위치시키세요

> 아래 이미지는 파티션 분할·환경 이동 예시입니다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/8df4ea81-7c13-445e-9913-7e4a4f79c180/4c690a6a-ca56-42fe-8b9a-1b51d05f747f/Untitled.png)

---

## 결과

- 디스크 파티션 및 마운트 정상 완료
- 공용 데이터셋, anaconda3 환경 등 필요한 환경파일 모두 이동/구축
- 원활한 대용량 사용 환경, 멀티 유저/공용 환경 세팅 완료

---

## Reference

- [[Linux Storage] 리눅스 시스템 디스크 파티션 및 관련 개념 정리](https://itguava.tistory.com/100)

