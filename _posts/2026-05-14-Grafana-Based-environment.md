---
title: "Prometheus + Grafana + NVIDIA DCGM 기반 GPU 통합 모니터링 환경 구축"
date: 2026-05-14
categories: [MLOps]
tags: [grafana, prometheus, gpu, dcgm, 모니터링, 환경구축, dashboard, exporter]
---


---

## 개요

Docker 기반 환경에서 Grafana를 활용하면 기존 sysstat/top 등 커맨드 방식보다 훨씬 손쉽게 시스템 리소스(GPU 포함)를 통합적으로 모니터링할 수 있다.  
이 구성은 다중 GPU 서버의 **CPU, 메모리, GPU 사용률/온도/전력/VRAM 사용량** 등 주요 자원 정보를 실시간으로 수집하고, Grafana 대시보드에서 시각화/분석하는 데 초점을 둔다.

**구성 요소**

| 구성 요소              | 역할                                         |
|----------------------|----------------------------------------------|
| Prometheus           | 메트릭 수집 및 저장                            |
| Grafana              | 대시보드 시각화                                |
| node_exporter        | CPU/메모리/디스크/네트워크 메트릭 수집         |
| NVIDIA DCGM Exporter | GPU 사용률/온도/전력/VRAM 메트릭 수집          |

---

## 전체 아키텍처

| 서버명      | 역할                   | 실행 서비스           | 포트         |
| ----------- | ---------------------- | --------------------- | ------------ |
| PC1         | 모니터링 서버          | Prometheus, Grafana   | 9090, 3000   |
| PC2         | 타겟 서버(모니터링 대상)| node_exporter         | 9100         |
|             |                        | dcgm-exporter         | 9400         |
| PC3         | 타겟 서버(모니터링 대상)| node_exporter         | 9100         |
|             |                        | dcgm-exporter         | 9400         |

---

## 포트 구성

> 아래 포트는 **기본(Default) 포트**이며, Docker 컨테이너 생성 시 `-p` 옵션 등으로 자유롭게 변경할 수 있음.

| 포트   | 역할                |
| ---- | -------------------- |
| 3000 | Grafana              |
| 9090 | Prometheus           |
| 9100 | node_exporter        |
| 9400 | NVIDIA DCGM Exporter |

---

## 방화벽(UFW) 및 내부망, 컨테이너 네트워크 보안 설정

> 실제 환경에서는 사내망(내부망) 내에서 수행하였기에, 외부 접근 차단 및 네트워크 경계 설정 등 보안 측면도 반드시 고려하였음.
> 모든 Prometheus, Exporter, Grafana 서비스에 대한 접근 제어를 위해 UFW 및 Docker 네트워크 정책을 별도로 강화하였고,  
> 아래 예시는 내부망에서만 접근 가능한 환경(예: 사내 192.168.x.x 대역)에서 사용한 설정 예시임.


### PC1 (Prometheus, Grafana 서버)

```bash
# Grafana(3000), Prometheus(9090) - 내부망만 허용
sudo ufw allow from 192.168.0.0/16 to any port 3000 proto tcp
sudo ufw allow from 192.168.0.0/16 to any port 9090 proto tcp
# 불필요한 외부(공인망) 접근 차단 권장
```

### PC2 / PC3 (Exporter 구동 서버)

```bash
sudo ufw allow from 192.168.0.0/16 to any port 9100 proto tcp
sudo ufw allow from 192.168.0.0/16 to any port 9400 proto tcp
```

#### Docker 컨테이너 포트 노출 제한 예시

```bash
# node-exporter를 내부망에서만 접근 가능하도록 실행
docker run -d \
  --name node-exporter \
  --restart unless-stopped \
  -p 192.168.0.100:9100:9100 \  # 특정 사내망 IP만 바인딩(예시)
  prom/node-exporter

```

#### 추가 보안 팁

- 사내망 VLAN 또는 방화벽 Level에서 외부 접속 원천 차단
- 꼭 필요한 포트만 허용, 불필요한 관리자 대시보드(port 3000 등)은 관리자 IP만 허용 가능
- Docker 컨테이너 --publish 옵션 사용 시, 반드시 바인드 IP 범위를 최소화할 것

---

## node_exporter 구성

### 실행

```bash
docker run -d \
  --name node-exporter \
  --restart unless-stopped \
  -p 9100:9100 \
  prom/node-exporter
```

### 메트릭 확인

```bash
curl http://localhost:9100/metrics
```

---

## NVIDIA Container Toolkit 설치

최신 설치 방법은 [NVIDIA 공식 설치 가이드](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)를 참고

GPU 메트릭 수집 또는 Docker 기반 GPU 컨테이너 실행을 위해 NVIDIA Container Toolkit 설치가 필요합니다.

### 패키지 저장소 및 GPG 키 등록

```bash
# 패키지 및 의존성 업데이트
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg

# GPG 키 등록
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  sudo gpg --dearmor -o /etc/apt/keyrings/nvidia-container-toolkit.gpg

# 저장소 등록 (예: Ubuntu 20.04, 필요시 codename 변경)
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
  sed 's#deb https://#deb [signed-by=/etc/apt/keyrings/nvidia-container-toolkit.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

### NVIDIA Container Toolkit 설치

```bash
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
```

자세한 내용 및 추가 옵션은 공식 문서의 [Install Guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)를 참고
```

#### Docker Runtime 설정

```bash
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

---

## NVIDIA DCGM Exporter 구성

### 실행

```bash
sudo docker run -d \
  --name dcgm-exporter \
  --restart unless-stopped \
  --runtime=nvidia \
  --cap-add SYS_ADMIN \
  -p 9400:9400 \
  nvcr.io/nvidia/k8s/dcgm-exporter:latest
```

### 메트릭 확인

```bash
curl http://localhost:9400/metrics
```

---

## Prometheus 구성

### prometheus.yml 예시

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "nodes"
    static_configs:
      - targets:
          - "<Target Server1>:9100"
          - "<Target Server2>:9100"

  - job_name: "gpu"
    static_configs:
      - targets:
          - "<Target Server1>:9400"
          - "<Target Server1>:9400"
```

### 실행

```bash
docker run -d \
  --name prometheus \
  --restart unless-stopped \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

### 상태 확인

- [http://<PROMETHEUS_IP>:9090/targets](http://<PROMETHEUS_IP>:9090/targets)
  - 상태: UP

---

## Grafana 구성

### 실행

```bash
docker run -d \
  --name grafana \
  --restart unless-stopped \
  -p 3000:3000 \
  grafana/grafana
```

### 접속

- 브라우저에서 접속: [http://<GRAFANA_IP>:3000](http://<GRAFANA_IP>:3000)
- 기본 계정(최초 설치 시 접속 계정): admin / admin

---

## Grafana Datasource 설정

1. 좌측 **Connections → Data Sources → Add Prometheus**
2. URL:  
   ```
   http://<PROMETHEUS_IP>:9090
   ```
3. Save & Test 후 "Successfully queried the Prometheus API" 확인

---

## 통합 GPU 대시보드 구성

### 주요 모니터링 항목

주요 모니터링 대상 메트릭들은 [http://<PROMETHEUS_IP>:9090](http://<PROMETHEUS_IP>:9090)에서 직접 확인할 수 있음

| 항목         | 메트릭                            |
| ------------ | --------------------------------- |
| GPU 사용률    | DCGM_FI_DEV_GPU_UTIL              |
| GPU 온도     | DCGM_FI_DEV_GPU_TEMP              |
| GPU 전력 사용량 | DCGM_FI_DEV_POWER_USAGE         |
| VRAM 사용량   | DCGM_FI_DEV_FB_USED               |
| CPU 사용률    | node_cpu_seconds_total            |
| 메모리 사용률 | node_memory_MemAvailable_bytes   |

---

## 최종 대시보드 구성

### 대시보드 구축 방법 요약

- 커스텀 대시보드 직접 제작 대신, Grafana 커뮤니티의 GPU/서버 모니터링용 **공개 대시보드 템플릿**을 가져와 환경에 맞게 수정하여 사용
    - 대표 대시보드 ID: **1860**, **12239** 등
    - 가져오기 경로: Grafana 메인 화면에서 **`+ Import`** → **Dashboard ID 입력** → **Prometheus 데이터소스 연결** → 필요 항목만 선택/수정

- 복잡한 JSON/YAML 포맷을 직접 수정할 필요 없이, 템플릿을 가져온 후 원하는 패널만 추가·삭제하면 신속하게 대시보드 구성 가능

---

### 패널 구성 예시

![Grafana 대시보드 예시](/assets/images/posts/2026-05-14-Grafana-Based-environment/grafana.png)


