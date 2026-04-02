---
title: "우분투 환경 및 SSH 기본 세팅 가이드"
date: 2026-04-02
description: 우분투 개발 환경 세팅부터 SSH 보안 설정까지 실전 가이드
categories: [devops]
tags: [ubuntu, setup, ssh, security, tutorial]
---

## 개요

이 문서는 우분투(Ubuntu)에서 개발 환경을 빠르게 세팅하고, 계정 및 SSH 보안 연결을 설정하는 과정을 단계별로 안내합니다.

---

## 1. 우분투 기본 환경 세팅

### 1-1. 시스템 업데이트 및 필수 패키지 설치

우분투 시스템을 최신 상태로 유지하고, 개발에 필요한 기본 도구들을 설치합니다.

```bash
sudo apt update

# 개발 필수 도구 모음 패키지
sudo apt-get install build-essential

# 우분투 GNOME 환경 최적화 도구
sudo apt-get install -y gnome-tweak-tool

# 텍스트 에디터 설치 (vim)
sudo apt-get install vim
```

---

## 2. 계정 세팅 및 보안 정책 적용

### 2-1. Root 계정 비밀번호 설정

```bash
# root 계정 비밀번호 설정
sudo passwd root
```

### 2-2. 계정 보안 정책(암호 규칙, 만기 등) 편집

- 암호 규칙, 만료 정책 설정:  
  `/etc/login.defs` 파일에서 환경에 맞는 규칙 수정  
  예) 최소/최대 암호 길이, 만기 기간 등

### 2-3. 사용자 추가 및 권한 관리

- **사용자 추가**

1. `adduser` 방법

    ```bash
    adduser [username]
    ```

1. `useradd` 방법

    ```bash
    useradd [username]
    passwd [username]
    ```

※ 계정 삭제 

```bash
sudo userdel -r [username]
sudo rm -rf /home/[username]
```

- **권한 부여**

```bash
sudo usermod -aG sudo [username]
```

---

<aside>

💡 **참고** https://mungiyo.tistory.com/14

</aside>

---

## 3. SSH 연결 및 보안 설정

### 3-1. SSH 설치 및 기본 설정

```bash
# 1. SSH 서버 설치
sudo apt update
sudo apt install openssh-server

# 2. 방화벽(UFW)에서 SSH 포트(22) 허용
sudo ufw allow ssh
```

- SSH 서비스 설정 파일 위치: `/etc/ssh/sshd_config`
- 접속 포트, Root 로그인 허용 여부, 접속 제한 등 주요 옵션 조정
- 예시:  
  ```
  Port 22
  PermitRootLogin no
  PasswordAuthentication yes
  AllowUsers username
  AllowGroups sshusers
  ```
- 설정 변경 후, SSH 서비스 재시작:
  ```bash
  sudo systemctl restart sshd
  ```

### 3-2. SSH 접속 방법

```bash
# 외부 서버 접속
ssh username@ip_address [-i 인증키] [-p 포트번호]
```

### 3-3. hosts.allow, hosts.deny를 통한 접근 관리

- `/etc/hosts.allow`, `/etc/hosts.deny` 파일을 편집하여
  허용/차단할 네트워크 대역(IP, 호스트) 지정 가능

### 3-4. 방화벽(UFW, IPTables) 추가 설정

- UFW 기본 가이드:  
  [Ubuntu 방화벽(UFW) 설정 바로가기](https://webdir.tistory.com/206)

---

> 위의 절차를 따르면 우분투 개발 환경과 SSH 보안 연결을 신속하게 구축할 수 있습니다.  
> 필요에 따라 상세 설정(방화벽, 암호 정책, 그룹/권한 등)을 추가적으로 조정하세요.

