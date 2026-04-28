---
title: "RAG 논문 리뷰: Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks"
date: 2026-04-28
description: RAG(Retrieval-Augmented Generation)의 핵심 아이디어, 수식, RAG-Sequence와 RAG-Token 차이, 실험 결과와 한계를 정리한 논문 리뷰
categories: [paperreview]
tags: [RAG, 검색증강, LLM, 자연어처리, 논문리뷰, knowledge-intensive, 답변생성]
---

## 논문 개요

이 글은 Patrick Lewis et al.의 **Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks** 논문 정리 리뷰로써, 본 논문은 Facebook AI Research와 University College London 연구진의 RAG(Retrieval-Augmented Generation) 모델 제안

RAG의 핵심 아이디어는 언어 모델이 파라미터 내 지식만으로 답변하지 않고, 외부 문서 집합에서 관련 문서 검색 후 해당 문서를 조건으로 답변 생성하는 방식. 이로써 모델은 더 사실적인 답변 생성 가능, 지식 변경 시 모델 전체 재학습 없이 문서 인덱스 교체만으로 최신 정보 반영 가능함.

---

## 1. 핵심 문제의식

기존 사전학습 언어 모델의 경우 많은 지식은 파라미터 내부에 암묵적 저장 방식이다. 하지만만 이 방식에서 발생하는 한계:

- 학습 이후 새 지식 반영 한계
- 답변 근거 문서 추적 한계
- 그럴듯하지만 사실과 다른 답변 생성(환각 hallucination 현상) 발생 가능성
- 지식 업데이트를 위한 대규모 재학습 또는 추가 학습 필요

RAG는 이를 **파라미터 지식(parametric memory)** 과 **비파라미터 지식(non-parametric memory)** 결합으로 해결하였다. 비파라미터 지식의 경우 위키피디아 등 외부 문서 집합과 검색 인덱스 의미함

---

## 2. RAG 모델 아키텍처 및 원리

![구조](https://paper-assets.alphaxiv.org/figures/2005.11401v4/img-0.jpeg)

### 하이브리드 구성

RAG의 주요 구성 요소:

- **Retriever**: 입력 $x$를 쿼리로 사용해 외부 문서 집합에서 관련 문서 $z$ 검색하며, 논문 기준 DPR(Dense Passage Retrieval) 사용
- **Generator**: 입력 $x$와 검색 문서 $z$를 함께 조건으로 받아 출력 시퀀스 $y$ 생성하며, 논문 기준 BART 계열 seq2seq 모델 사용

검색기의 경우 FAISS 기반 최대 내적 검색(Maximum Inner Product Search)로 대규모 문서 인덱스에서 신속히 top-k 문서 검색 수행하며 학습 시 문서 인코더 및 문서 인덱스 고정, 쿼리 인코더와 생성기 동시 미세조정.

### 주요 수식

RAG는 검색된 문서를 잠재변수 $z$로 간주하고, 해당 문서에 대해 생성 확률 주변화(marginalization) 수행한다. 실제로는 retriever가 후보로 제공한 top-k 문서만 사용.

#### RAG-Sequence

RAG-Sequence는 출력 시퀀스 전체가 단일 문서 $z$에 의해 조건화된다고 가정.

$$
p_{\text{RAG-Sequence}}(y \mid x)
\approx
\sum_{z \in \text{top-k}(p_\eta(\cdot \mid x))}
\tilde{p}_\eta(z \mid x)
\prod_{i=1}^{N} p_\theta(y_i \mid x, z, y_{<i})
$$


여기서 $\tilde{p}_\eta(z \mid x)$는 top-k 문서 집합 안에서 확률합이 1이 되도록 재정규화한 검색 확률을 뜻한다. 아래는 해당 수식의 단순화된 형태이다.


$$
p_{\text{RAG-Sequence}}(y \mid x)
\approx
\sum_{z}
\tilde{p}_\eta(z \mid x)
p_\theta(y \mid x,z)
$$

즉, 하나의 후보 문서가 전체 답변 생성에 일관된 영향을 미침

#### RAG-Token

RAG-Token은 각 토큰 생성마다 문서에 대한 주변화 진행

$$
p_{\text{RAG-Token}}(y \mid x)
\approx
\prod_{i=1}^{N}
\sum_{z \in \text{top-k}(p_\eta(\cdot \mid x))}
\tilde{p}_\eta(z \mid x)
p_\theta(y_i \mid x, z, y_{<i})
$$

RAG-Token 경우, 토큰별로 서로 다른 문서 정보 활용 가능하기에 여러 문서의 근거 혼합으로 더욱 유연한 답변이 가능한 반면, 계산량과 구현 복잡도 측면에서 RAG-Sequence보다 증가함

---

## 3. 훈련 및 최적화 방식

RAG의 학습 목표: 정답 시퀀스 $y$의 주변 로그우도(marginal log-likelihood) 최대화. 실제 학습 과정에서는 음의 로그우도 최소화.

- 정답 문서 직접 supervision 없이도, 정답 생성에 도움이 되는 문서를 retriever가 더 높은 확률로 선택하도록 학습 진행
- 생성기와 retriever 쿼리 인코더 end-to-end 동시 미세조정
- 문서 인코더와 FAISS 인덱스 고정, 전체 문서 인덱스 재구축 불필요
- top-k 문서 수 $k$는 성능과 비용 사이 trade-off 초래하는 주요 하이퍼파라미터

---

## 4. 실험 결과 및 성능

논문상 RAG 평가 대상: 다양한 지식 집약형 NLP 태스크

- **Open-domain QA**: Natural Questions, WebQuestions, CuratedTREC 등에서 강한 성능 확인
- **Abstractive QA**: 정답이 특정 문장에 명시적 존재하지 않은 경우에도 검색 문서 기반 답변 생성 가능
- **MS MARCO NLG**: BART 기반 생성 모델 대비 더 나은 BLEU, ROUGE-L 점수 확인
- **사실성 평가**: 사람 평가 기준 RAG는 BART-only 모델 대비 더 구체적·사실적 답변 경향
- **지식 업데이트**: 모델 파라미터 재학습 없이 문서 인덱스 교체만으로 지식 갱신 가능성 확인

![result](https://paper-assets.alphaxiv.org/figures/2005.11401v4/img-2.jpeg)

---

## 5. 해석 가능성 및 지식 업데이트

RAG의 주요 장점: 답변 근거 추적 가능성 보유할 수 있으며 생성 결과가 어떤 검색 문서 조건에서 생성됐는지 확인 가능, 순수 생성 모델 대비 해석 가능성 강화

문서 인덱스 교체 방식으로 지식 갱신 가능하며, 논문에서는 **index hot-swapping**으로 용어화. 의료, 법률, 교육 등 최신성과 근거 확인 요구 영역에서 유용성 높음.

![result](https://paper-assets.alphaxiv.org/figures/2005.11401v4/img-1.jpeg)

---

## 6. 한계 및 시사점

RAG의 강점과 한계:

- 검색 대상 코퍼스 품질 저하 시 생성 결과 동반 저하 우려
- retriever가 부정확 문서 검색 시 generator가 해당 문서 근거로 그럴듯한 오답 생성 가능성
- 답변 근거 문서 존재만으로 항상 답변이 사실임을 의미하지 않음
- 검색 인덱스 구축과 유지보수 관련 비용 추가 발생
- 민감 도메인에서는 검색 문서 접근권한, 개인정보, 보안 등 이슈 중요

즉, RAG는 환각 완전 제거 기술이 아닌, 외부 근거 활용을 통한 사실성·업데이트 가능성 개선 프레임워크로 이해하는 것이 적절함

---

## 7. 결론 및 미래 전망

RAG: 파라미터 기반 언어 모델과 외부 검색 시스템 결합 대표 아키텍처로써 논문 발표 이후 QA, 챗봇, 문서 검색, 의료·법률 보조 시스템, 기업 지식베이스 응답 시스템 등 광범위 확장 진행됨

논문 의의: 단순한 검색 결과 프롬프트 첨부 방식을 넘어, 검색 문서를 잠재변수로 두고 생성 확률 주변화라는 모델링 관점 제시하였음. 실무 RAG 시스템은 논문 원형과 다소 차이 발생하나, “외부 지식 검색 + 조건부 생성” 기본 아이디어는 여전히 핵심 위치 유지

---

### 참고 논문:  

- [Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks, Patrick Lewis et al.](https://arxiv.org/abs/2005.11401)

---
