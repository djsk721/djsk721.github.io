---
title: "CUDA(CUDA Toolkit & Graphic Driver) 설치 및 관리 가이드"
date: 2026-04-02
description: CUDA Toolkit과 NVIDIA 그래픽 드라이버의 설치, 삭제, 환경 변수, 다중 사용자/서버 환경에서의 관리 및 주의사항 가이드
categories: [devops]
tags: [cuda, nvidia-driver, gpu, driver, multi-user, install, uninstall, 환경설정]
---

## CUDA Toolkit 설치 (runfile 방법)

- [CUDA Toolkit 12.1 Update 1 Downloads](https://developer.nvidia.com/cuda-12-1-1-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=runfile_local)  
- runfile(.run) 방식으로 CUDA만 설치하거나, 그래픽 드라이버 없이 CUDA만 설치 가능

설치 파일 다운로드 후 다음 명령어로 설치합니다.

```bash
# 다운로드 받은 runfile로 실행 (예: cuda_12.1.1_530.30.02_linux.run)
sudo sh cuda_12.1.1_530.30.02_linux.run

# 설치 옵션 중 'Driver 설치 여부' 선택 화면에서 그래픽 드라이버 설치 생략 가능
```

---

## Nvidia 그래픽 드라이버 설치

- **임의의 삭제, 업데이트, 업그레이드, 다운그레이드를 지양합니다.**
- 드라이버/패키지 관련 이슈 발생 시 반드시 팀원 논의 후 조치(자동 업데이트 주의)
- apt update, upgrade로 인한 드라이버 변경 주의! 반드시 팀 논의 후 진행

### 그래픽 카드 및 드라이버 상태 확인

```bash
# 장착된 그래픽카드와 드라이버 상태 확인 (driver=nouveau면 드라이버 미설치 상태)
sudo lshw -c display
```

### 드라이버 설치

```bash
# 설치 가능한 드라이버 목록 조회
sudo ubuntu-drivers devices

# 시스템에서 권장하는 드라이버 자동 설치
sudo ubuntu-drivers autoinstall

# (특정 버전 명시적 설치는 권장하지 않음, 예: sudo apt install nvidia-driver-470)
```

### 설치/적용 확인

```bash
# driver 상태가 nvidia인지 재확인
sudo lshw -c display

# prime-select로 활성화된 드라이버 확인
prime-select query

# nvidia가 아니라면 nvidia로 활성화
sudo prime-select nvidia

# 정상 할당 확인
nvidia-smi
```

### 설치 로그 확인

```bash
# CUDA 설치/에러 로그 확인
cat /var/log/cuda-installer.log
```

---

## CUDA & cuDNN 설치 참고

- 반드시 본인 그래픽카드와 OS, 드라이버 호환성을 확인 후 설치
- 지원 표:  
    - [CUDA](https://en.wikipedia.org/wiki/CUDA#GPUs_supported)
    - [NVIDIA CUDA Deep Neural Network (cuDNN)](https://developer.nvidia.com/cudnn)
    - [Installation Guide](https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html#download)

---

## CUDA 및 Nvidia 드라이버 삭제

```bash
# Nvidia 드라이버 삭제
sudo apt-get purge nvidia*
sudo apt-get autoremove
sudo apt-get autoclean

# CUDA 삭제
sudo rm -rf /usr/local/cuda*
sudo apt-get --purge remove 'cuda*'
sudo apt-get autoremove --purge 'cuda*'

# 삭제 대상 확인
sudo dpkg -l | grep nvidia
sudo dpkg -l | grep cuda

# 위 명령어에서 출력된 패키지명 개별 삭제 (예)
sudo apt-get remove --purge [패키지명]
```

---

## CUDA 환경 변수 설정 (필수, ~/.bashrc)

- **[CUDA_PATH]**에 설치된 CUDA 디렉터리(예: /usr/local/cuda-12.1)로 환경 변수 등록 필요
- 아래 설정을 각 사용자 `~/.bashrc` 하단에 추가

```bash
# CUDA 환경 변수 예시 (설치 경로에 따라 경로 수정)
export PATH=/usr/local/cuda-{Cuda-Version}/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-{Cuda-Version}/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

적용 후 터미널에서 재로그인하거나 아래 명령 실행

```bash
source ~/.bashrc
```

---

## 추가 참고 및 유의사항

- 외부에서 드라이버/라이브러리 조작 시에는 반드시 내부 논의 후 작업 진행
- 자동/의존성 업데이트에도 NVIDIA 드라이버가 변경될 수 있으므로 apt 사용시 유의
- CUDA 설치 후 동일 경로에 여러 버전이 공존할 경우, 환경변수 변경으로 사용 버전 지정 가능

---

