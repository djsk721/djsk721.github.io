---
title: "Qwen3.8 27B 서빙 툴별 추론 성능 비교"
date: 2026-08-19
categories: [ai]
tags: [ai]
---

> Qwen3.8 27B 모델이 RTX 3090 ×2 환경에서도 비교적 높은 추론 성능을 보임으로, Ollama·llama.cpp·vLLM 서빙 환경에서 MTP 적용 여부에 따른 생성 성능을 비교 측정함

---

## 1. 개요

Qwen3.8 27B 모델을 로컬 GPU 환경에서 테스트한 결과, 27B급 모델임에도 RTX 3090 ×2 환경에서 비교적 높은 생성 성능을 확인함.

특히 Qwen3.8 계열에서 지원되는 MTP(Multi-Token Prediction)를 활용할 경우 단일 요청 환경에서 생성 속도가 크게 향상됨을 확인하였음으로, 실제 서빙 툴에 따라 성능 차이가 어느 정도 발생하는지 비교하기 위해 추가 벤치마크를 수행함.

이번 실험에서는 다음 세 가지 서빙 환경을 비교함.

- Ollama
- llama.cpp
- vLLM

Ollama와 llama.cpp는 동일한 Q8_0 GGUF 모델을 사용하였으며, vLLM은 FP8 모델을 사용함.

따라서 Ollama와 llama.cpp 간 결과는 동일 모델 기준의 비교가 가능하지만, vLLM은 모델 정밀도 및 실행 백엔드가 다름으로 절대적인 런타임 우열보다는 **각 환경 내부에서 MTP 적용 전후의 성능 변화**를 중심으로 확인함.

---

## 2. 실험 환경

| 항목 | 설정 |
| --- | --- |
| Prompt | `Python으로 LRU Cache를 구현하고 동작 원리를 설명해줘.` |
| GPU | RTX 3090 ×2 |
| Concurrency | 1 |
| Context | 131,072 |
| Temperature | 0 |
| Max output | 2,048 tokens |
| Ollama / llama.cpp 모델 | `unsloth/Qwen3.8-27B-GGUF/Qwen3.8-27B-Q8_0.gguf` |
| vLLM 모델 | `unsloth/Qwen3.8-27B-FP8` |

이번 테스트는 동시 요청을 1개로 제한함으로 batching 성능보다는 **단일 요청의 순수 생성 성능과 MTP 적용 효과**를 확인하는 데 목적을 두었음.

---

## 3. 생성 성능 비교

| Runtime | Model | MTP | Output tokens | Generation throughput | MTP Speedup | Draft acceptance |
| --- | --- | --- | ---: | ---: | ---: | ---: |
| Ollama | Q8_0 GGUF | OFF | 2,048 | **26.69 tok/s** | 1.00× | - |
| Ollama | Q8_0 GGUF | ON | 2,048 | **50.57 tok/s** | **1.89×** | - |
| llama.cpp | Q8_0 GGUF | OFF | 1,760 | **28.13 tok/s** | 1.00× | - |
| llama.cpp | Q8_0 GGUF | ON | 1,765 | **66.18 tok/s** | **2.35×** | **74.8%** |
| vLLM | FP8 | OFF | - | **약 47.5~47.7 tok/s*** | 1.00× | - |
| vLLM | FP8 | ON | - | **최대 99.3 tok/s*** | **약 2.08×*** | **약 70.9%** |

> * vLLM의 throughput은 서버 로그에서 출력되는 10초 단위 `Avg generation throughput` 기준임. Ollama 및 llama.cpp처럼 요청 전체 generation 시간을 이용한 평균값과는 측정 방식이 다름으로 절대값을 직접 비교할 때 주의가 필요함.

---

## 4. Ollama

### 4.1 MTP OFF

```text
eval_count       = 2048
eval_duration    = 76.740572 s
```

Ollama의 생성 속도는 다음과 같이 계산함.

```text
2048 / 76.740572
= 26.69 tok/s
```

| 항목 | 결과 |
| --- | ---: |
| Output tokens | 2,048 |
| Generation time | 76.74 s |
| Generation throughput | **26.69 tok/s** |
| Total duration | 86.64 s |
| Model load | 9.65 s |

### 4.2 MTP ON

```text
eval_count       = 2048
eval_duration    = 40.498018 s
```

생성 속도는 다음과 같음.

```text
2048 / 40.498018
= 50.57 tok/s
```

| 항목 | 결과 |
| --- | ---: |
| Output tokens | 2,048 |
| Generation time | 40.50 s |
| Generation throughput | **50.57 tok/s** |
| Total duration | 41.55 s |
| Model load | 0.62 s |

MTP 활성화 후 생성 속도는 다음과 같이 증가함.

```text
50.57 / 26.69
= 약 1.89×
```

즉, Ollama에서는 MTP 적용 후 **약 89.5%의 생성 성능 향상**을 확인함.

`total_duration`은 모델 로딩 시간이 포함됨으로 MTP OFF와 ON에서 `load_duration` 차이가 크기 때문에, 순수 생성 성능 비교에는 `eval_duration`을 사용함.

---

## 5. llama.cpp

### 5.1 MTP OFF

```text
predicted_n             = 1760
predicted_ms            = 62539.035
predicted_per_second    = 28.1264
```

생성 속도는 **28.13 tok/s**로 측정됨.

### 5.2 MTP ON

```text
predicted_n             = 1765
predicted_ms            = 26655.997
predicted_per_second    = 66.1765

draft_n                 = 1632
draft_n_accepted        = 1220
```

생성 속도는 **66.18 tok/s**로 측정됨.

MTP 적용에 따른 성능 향상은 다음과 같음.

```text
66.18 / 28.13
= 약 2.35×
```

즉, llama.cpp에서는 MTP 적용 후 생성 성능이 **약 135% 증가**함.

Draft token acceptance rate는 다음과 같음.

```text
1220 / 1632
= 약 74.8%
```

전체 draft token 중 약 74.8%가 accepted되었음으로, 높은 acceptance rate가 생성 throughput 향상으로 연결된 것임.

---

## 6. Ollama와 llama.cpp 비교

Ollama와 llama.cpp는 동일한 Q8_0 GGUF 모델을 사용함으로 상대적으로 직접적인 비교가 가능함.

### 6.1 MTP OFF

```text
Ollama      26.69 tok/s
llama.cpp   28.13 tok/s
```

llama.cpp가 약 5.4% 높은 결과를 보였으나, 전체적으로는 두 환경의 기본 decoding 성능이 유사한 수준이었음.

따라서 이번 단일 요청 실험에서는 **Ollama와 llama.cpp 자체의 기본 추론 성능 차이는 크지 않은 것**임을 확인함.

### 6.2 MTP ON

```text
Ollama      50.57 tok/s
llama.cpp   66.18 tok/s
```

MTP 활성화 시에는 llama.cpp가 Ollama보다 약 31% 높은 생성 throughput을 보임.

즉, 기본 decoding에서는 두 환경의 성능이 유사했으나, MTP를 활성화한 이후에는 각 서빙 툴의 MTP 처리 방식에 따른 성능 차이가 더 크게 나타남.

---

## 7. vLLM FP8

### 7.1 MTP OFF

생성 중 안정 구간에서 다음과 같은 throughput이 관측됨.

```text
47.7 tok/s
47.6 tok/s
47.5 tok/s
```

따라서 steady-state generation 성능은 약 **47.6 tok/s** 수준으로 확인함.

초기 로그에서 확인된 낮은 throughput은 prompt 처리 및 측정 구간 시작 시점이 포함되어 있음으로 steady-state 생성 성능에서는 제외함.

### 7.2 MTP ON

MTP 활성화 후 안정적인 생성 구간에서 다음 결과가 관측됨.

```text
Avg generation throughput: 99.3 tokens/s
```

MTP OFF의 약 47.6 tok/s와 비교하면 다음과 같음.

```text
99.3 / 47.6
= 약 2.09×
```

즉, steady-state 구간 기준으로 약 2.1배 수준의 throughput 향상이 관찰됨.

다만 99.3 tok/s는 요청 전체 평균이 아니라 vLLM 서버가 출력하는 특정 10초 구간의 평균 throughput임.

---

## 8. vLLM MTP Acceptance

vLLM에서 관측된 MTP 관련 지표는 다음과 같음.

| 구간 | Mean acceptance length | Draft acceptance rate |
| --- | ---: | ---: |
| 초기 | 3.00 | 66.7% |
| 안정 구간 | 3.21 | 73.6% |
| 종료 구간 | 3.11 | 70.4% |

전체 관측값을 합산하면 다음과 같음.

```text
Accepted
= 418 + 737 + 76
= 1231

Drafted
= 627 + 1002 + 108
= 1737

1231 / 1737
= 약 70.9%
```

따라서 전체 관측 기준 draft acceptance rate는 약 **70.9%**임.

llama.cpp의 약 74.8%와 유사한 수준임으로, 두 환경 모두 상당수의 draft token을 실제 출력 token으로 수용함으로써 MTP에 따른 성능 향상을 얻음.

---

## 9. 결과 요약

이번 실험에서 확인된 주요 결과는 다음과 같음.

### 기본 Decoding 성능

동일 Q8_0 모델을 사용한 Ollama와 llama.cpp의 기본 생성 속도는 각각 다음과 같음.

```text
Ollama      26.69 tok/s
llama.cpp   28.13 tok/s
```

두 환경의 차이는 약 5% 수준으로 기본 decoding 성능은 유사하였음.

### MTP 적용 효과

```text
Ollama
26.69 → 50.57 tok/s
약 1.89×

llama.cpp
28.13 → 66.18 tok/s
약 2.35×

vLLM
약 47.6 → 최대 99.3 tok/s
약 2.1×
```

세 환경 모두 MTP 적용 시 생성 throughput이 크게 증가함.

이번 단일 요청 환경에서는 **서빙 툴 자체의 기본 decoding 성능 차이보다 MTP 적용 여부가 생성 성능에 더 큰 영향을 주었음.**

특히 동일 Q8_0 모델을 사용한 llama.cpp의 경우 MTP 적용 전 28.13 tok/s에서 적용 후 66.18 tok/s로 증가하여 가장 큰 상대 성능 향상을 확인함.

---

## 10. 해석 시 주의사항

이번 실험 결과를 해석할 때 다음 조건을 고려해야 함.

- Ollama와 llama.cpp는 동일한 Q8_0 GGUF 모델을 사용함.
- vLLM은 FP8 모델을 사용함으로 Q8_0과 직접적인 절대 성능 비교에는 모델 정밀도 및 실행 backend 차이가 포함됨.
- Ollama와 llama.cpp는 요청 전체 generation 시간을 기준으로 throughput을 계산함.
- vLLM은 서버 로그의 10초 단위 `Avg generation throughput`을 기준으로 함.
- Concurrency가 1임으로 vLLM의 주요 장점 중 하나인 continuous batching 및 다중 요청 처리 성능은 이번 테스트에 포함되지 않음.
- Context window는 131,072로 동일하게 설정했으나, 실제 테스트 prompt 자체는 짧음으로 long-context 입력 처리 성능을 비교한 실험은 아님.
- 단일 실행 결과임으로 최종적인 성능 비교를 위해서는 동일 조건에서 반복 측정 후 평균 및 편차를 추가로 확인할 필요가 있음.

---

## 11. 결론

Qwen3.8 27B는 RTX 3090 ×2 환경에서 27B급 모델임에도 비교적 높은 로컬 추론 성능을 확인할 수 있었음.

특히 MTP를 적용했을 때 Ollama, llama.cpp, vLLM 모두 생성 throughput이 크게 증가함으로, 이번 실험에서는 **MTP가 단일 요청의 decode 성능을 결정하는 주요 요소 중 하나임을 확인함.**

동일 Q8_0 모델 기준으로 MTP OFF에서는 Ollama와 llama.cpp가 각각 26.69 tok/s와 28.13 tok/s로 유사한 성능을 보였으나, MTP ON에서는 각각 50.57 tok/s와 66.18 tok/s로 차이가 확대됨.

따라서 Qwen3.8 27B를 단일 사용자 또는 낮은 동시성 환경에서 로컬 서빙할 경우, 단순히 서빙 툴 자체를 선택하는 것뿐 아니라 **MTP 지원 여부와 실제 MTP 처리 효율을 함께 고려하는 것이 중요함.**

vLLM FP8에서는 MTP ON 상태에서 최대 약 99.3 tok/s의 구간 throughput이 관측됨으로 높은 잠재 성능을 확인하였으나, 향후에는 동일 측정 방식으로 TTFT, TPOT, E2E latency 및 다중 동시 요청 환경까지 추가 비교할 필요가 있음.
