---
title: "Docker 환경 구축 및 컨테이너 가이드"
date: 2026-04-02
description: Docker의 개념, 주요 사용 이유, 핵심 명령어, GPU 환경 및 실전 설치/배포법을 다루는 가이드
categories: [devops]
tags: [docker, container, gpu, setup, tutorial, ai, compose, kubernetes]
---

## Docker란?

컨테이너 기반 가상환경을 구성해서 SW를 패키징하고 배포하는 플랫폼입니다.


---

## 주요 구성 요소

- **이미지(Image)**  
  컨테이너를 실행하기 위한 파일과 설정이 포함된 패키지. Docker Hub 에 저장된 이미지 활용 또는 직접 Dockerfile을 빌드해 사용 가능.

- **컨테이너(Container)**  
  이미지를 기반으로 실행되는 격리된 환경. 모든 라이브러리, 패키지 및 종속성과 함께 앱을 실행. 독립된 프로세스 공간을 가짐.

- **도커 데몬(Docker Daemon)**  
  컨테이너 관리, 이미지 빌드/실행 등 핵심 작업을 백그라운드에서 담당.

- **도커 클라이언트(Docker Client)**  
  유저가 도커를 명령어 또는 API로 조작하는 도구.

---

## Docker 작동 원리

- 기존 VM(Virtual Machine)은 OS 위에 Guest OS를 올려 전체 자원을 가상화합니다. → 느리고 리소스 분할, 호환성 한계
- **Docker는 호스트 OS에서 커널을 공유하며 Guest OS 전체를 가상화하지 않고, 격리된 컨테이너 공간을 빠르게 실행**  
- 시스템 리소스 효율이 높고, VM 대비 훨씬 가볍고 빠름

<img src="/assets/images/posts/2026-04-02-Docker-Environment-guide/vm_vs_docker.png" alt="Docker vs VM" width="700"/>

---

## Docker를 사용하는 이유

- **일관성 있는 환경 제공**: 모든 종속성과 환경설정을 패키징, 어디서나 동일 환경 유지
- **경량화**: VM 대비 가볍고 빠른 실행
- **쉬운 배포 및 확장**: 여러 환경, 다수 컨테이너 배포·관리·확장 쉬움
- **리소스 효율성**: 여러 컨테이너가 하나의 호스트에서 리소스 공유
- **유연한 유지보수**: 환경 무관하게 쉽게 테스트·배포·롤백 가능
- **활성화된 커뮤니티**: 수많은 공개 이미지/문서/에러 대응 자료 존재
- **AI/ML, Data 솔루션 배포 필수**  
  - 최근엔 Docker Compose나 Kubernetes로 컨테이너 오케스트레이션: 여러 컨테이너의 배포/확장/네트워크를 자동화 관리함
  - Kubernetes 소개 및 도큐먼트: [Kubernetes Concepts](https://kubernetes.io/ko/docs/concepts/overview/)
  - Docker Compose: [Compose 소개](https://seosh817.tistory.com/387)

<img src="/assets/images/posts/2026-04-02-Docker-Environment-guide/meaning.png" alt="Docker Ecosystem" width="700"/>

---

## Docker 환경 세팅

### 1. Docker 설치

```bash
sudo wget -qO- http://get.docker.com/ | sh
```

### 2. Docker 권한 설정

```bash
# 현재 사용자에게 docker 권한 부여
sudo usermod -aG docker $USER

# (예시) 특정 사용자에게 권한 부여
sudo usermod -aG docker hjkim
```

### 3. NVIDIA CUDA Toolkit 설치 (GPU 도커 활용시 필수)

> **서버/환경에 맞는 CUDA Toolkit 버전 선택이 매우 중요합니다.**
>
> - 사용 중인 운영체제(Ubuntu 버전 등)와 목적에 맞는 CUDA Toolkit을 반드시 [공식 다운로드 페이지](https://developer.nvidia.com/cuda-downloads)에서 확인 후 설치하세요.
> - 도커에서 GPU를 사용할 경우, 해당 환경과 일치하는 버전을 설치해야 호환성 문제를 방지할 수 있습니다.
> - 예시 명령어는 Ubuntu 22.04 및 CUDA 12.3 기준이며, 실제 환경에 따라 경로·버전 번호를 반드시 확인해서 바꿔야 합니다.

```bash
# (아래는 Ubuntu 22.04 + CUDA 12.3 예시)
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-ubuntu2204.pin
sudo mv cuda-ubuntu2204.pin /etc/apt/preferences.d/cuda-repository-pin-600

wget https://developer.download.nvidia.com/compute/cuda/12.3.2/local_installers/cuda-repo-ubuntu2204-12-3-local_12.3.2-545.23.08-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu2204-12-3-local_12.3.2-545.23.08-1_amd64.deb
sudo cp /var/cuda-repo-ubuntu2204-12-3-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda-toolkit-12-3
```

> **꼭 본인의 Linux 배포판 및 원하는 CUDA 버전에 맞는 설치 파일을 선택하세요!**
> - 구버전 OS(예: Ubuntu 20.04 등)는 해당 OS용 CUDA를 받아야 문제가 발생하지 않습니다.
> - 자세한 설치법과 각 환경별 안내는 [NVIDIA 공식 CUDA 인스톨 가이드](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/)를 참조하세요.

### 4. Docker에서 GPU 사용 확인

```bash
docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```

---

## 5. Docker Image 사용 및 다운로드

- [도커 허브](https://hub.docker.com)에서 공개 이미지(예: Ubuntu, PyTorch 등) 검색·활용  
- 예시: Ubuntu 22.04 + CUDA 12.1 기반 PyTorch 이미지

```bash
docker pull pytorch/pytorch:2.1.2-cuda12.1-cudnn8-runtime
```
- [PyTorch Docker 공식 이미지 목록](https://hub.docker.com/r/pytorch/pytorch/tags) 참고  
- (runtime: 가벼운 모델링/실행용, devel: 개발/디버깅/빌드용, 용량↑)

### 6. Docker Image 확인

```bash
docker images
```

---

## 7. Docker Container 실행

- 사용자 로컬폴더를 마운트(mount), 포트 설정(예: 8888)  
- 분석용 환경/사용자별 독립적으로 컨테이너 분리해서 사용

```bash
sudo docker run -d -it --name [컨테이너이름] --gpus all --ipc=host -v [로컬디렉터리]:/workspace -p 8888:8888 [이미지명]

# 예시
sudo docker run -d -it --name pytorch --gpus all -v /Users/md/test:/workspace -p 8888:8888 pytorch/pytorch:2.1.2-cuda12.1-cudnn8-runtime
```

---

## 8. Docker 볼륨 생성/관리

```bash
docker volume create [볼륨이름]
docker volume ls
docker volume inspect [볼륨이름]
```

---

## 9. Docker Container 상태 및 접속

```bash
docker ps -a    # 전체 컨테이너 목록/상태

docker exec -it [컨테이너이름] bash # 컨테이너 내부(bash) 접속

# 접속 시 프롬프트가 root@xxxx:/workspace# 등으로 이동
```

---

## 10. GPU 사용 확인 (컨테이너 내부에서)

```bash
python

import torch
print(torch.cuda.is_available())
```

---

## 11. 기타 참고 및 기본 명령어

- https://dongle94.github.io/docker/docker-basic-use/

---

**Tip:**  
- Docker는 시스템 및 AI/ML 개발, 분석, 배포·운영 업무에 필수입니다.  
- 컨테이너 기반 워크플로에 익숙해질수록 환경 관리·테스트·배포가 편해집니다.

---