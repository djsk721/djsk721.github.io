---
title: "지식 집약적 자연어 처리 태스크를 위한 검색 증강 생성(RAG) 논문 리뷰"
date: 2026-04-28
description: Facebook AI Research와 University College London의 RAG(Retrieval-Augmented Generation) 논문(2021.04.13) 심층 요약 및 주요 기술 해설
categories: [paperreview]
tags: [RAG, 검색증강, LLM, 자연어처리, 논문리뷰, knowledge-intensive, 답변생성]
---

## 논문 개요

이 논문은 Facebook AI Research와 University College London이 공동 연구한 ‘검색 증강 생성(Retrieval Augmented Generation, RAG)’ 모델을 다룹니다. 저자(주저자 Patrick Lewis 외)는 사전학습 기반 언어 생성기와 외부 지식 검색기를 결합해 다양한 지식 집약적 자연어 처리(NLP) 태스크에서 강력한 성능을 달성하는 범용 아키텍처를 제시합니다. 특히, 문서 인덱스 핫스왑만으로 지식 업데이트가 가능한 점과 향상된 사실적 정확성을 입증하였습니다.

---

## 1. 핵심 문제의식

기존 대규모 언어 모델은 파라미터 내 암묵적 지식만을 활용해 답변을 생성하는 한계가 있습니다. 이로 인해 실제적 지식 업데이트가 어렵고, 환각(hallucination)이나 해석 불가 등의 문제가 빈번히 발생합니다. 논문에서는 이러한 한계를 극복하기 위해 외부 지식 기반(비파라미터적) 검색 증강 접근법을 도입합니다.

---

## 2. RAG 모델 아키텍처 및 원리

![구조](https://paper-assets.alphaxiv.org/figures/2005.11401v4/img-0.jpeg)


### ● 하이브리드 구성
- **검색기**: Dense Passage Retrieval(DPR, 쿼리/문서별 BERT 인코더 사용) 기반, 대규모 지식코퍼스(2100만 위키피디아 단락)에서 관련 문서 검색
- **생성기**: BART-large와 같은 사전학습 시퀀스-투-시퀀스 생성 모델, 입력 쿼리와 검색된 문서를 통합해 최종 답변 생성
- 두 구성요소를 종단간(end-to-end) 방식으로 훈련

### ● 주요 수식
- **RAG-Sequence**: 시퀀스 전체가 단일 문서에 의해 영향 받음
  $$
  p_{RAG-Sequence}(y|x) \approx \sum_{z \in \text{top-k}(p(\cdot|x))} p_\eta(z|x)p_\theta(y|x,z)
  $$
- **RAG-Token**: 각 토큰이 서로 다른 문서에 영향 받아, 다수 소스 합성 가능
  $$
  p_{RAG-Token}(y|x) \approx \prod_i \sum_{z \in \text{top-k}(p(\cdot|x))} p_\eta(z|x)p_\theta(y_i|x,z,y_{1:i-1})
  $$
- 검색기는 FAISS 인덱스 기반 최대 내적 검색으로 빠른 문서 검색 지원
- 훈련 시 문서 인코더/인덱스는 고정, 쿼리 인코더만 생성기와 함께 미세조정

---

## 3. 훈련 및 최적화 방식

- 손실함수: 음의 주변 로그우도 최대화(End-to-end Joint Optimization)
- 슈퍼비전 없이 관련 구절을 암묵적으로 식별하는 비지도 검색 학습
- 하이퍼파라미터 k(검색 문서 수)는 일반적으로 5~50, 개발셋 기반
- Adam 기반 경사하강법, 생성기-검색기 쿼리 인코더 공동 최적화

---

## 4. 실험 결과 및 성능

- **오픈 도메인 QA**: Natural Questions, WebQuestions, CuratedTrec 등에서 T5-11B(매개변수 기반), REALM/DPR(추출형 모델) 대비 새로운 SOTA 달성
- **답변 합성**: 정답이 검색 문서에 명시적으로 없는 경우에도 옳은 답변 생성 (추출기 불가 상황에서 11.8% 정확도 확보)
- **MS-MARCO NLG**: RAG는 BART 대비 BLEU/Rouge-L 각각 +2.6p 향상
- **사실성/특이성 인간평가**: BART보다 사실 정확도(42.7% vs 7.1%), 특이성(37.4% vs 16.8%) 우위
- **지식 업데이트 실험**: 위키 문서 인덱스 변경만으로 모델의 최신성/정확도 즉시 향상

![result](https://paper-assets.alphaxiv.org/figures/2005.11401v4/img-2.jpeg)

---

## 5. 해석 가능성 및 지식 업데이트

- 외부 검색 메커니즘 도입으로 각 답변의 근거(문서/근거 URI 등) 추적 가능
- 위키피디아 인덱스를 교체(갱신)하는 것만으로 재학습 없는 지식 최신화 ('hot swapping' 실증)
- 투명성/설명가능성이 중요한 의료, 교육 분야 등에서 응용 유리

![result](https://paper-assets.alphaxiv.org/figures/2005.11401v4/img-1.jpeg)
---

## 6. 한계 및 시사점

- 외부 지식코퍼스의 편향·품질에 모델 성능이 직접적으로 의존(공정성/사실성 challenge)
- 잠재적으로 잘못된/오해 유발 정보 생성 가능성 역시 내재
- 저자들은 실제 응용 확대에 따른 책임 있는 개발·배포 필요성 강조

---

## 7. 결론 및 미래 전망

- RAG는 파라미터/비파라미터 지식 결합 기반의 새로운 NLP 패러다임을 제시
- 다양한 지식 태스크에 광범위한 적용 및 효율성(모델 크기 대비 성능 증가) 실증
- 향후 외부 코퍼스 관리/감독 및 Responsible AI 측면에서의 활용 기준 정립 필요

---

### 참고 논문:  
- [Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks, Patrick Lewis et al., 2021 (Facebook AI, UCL)](https://arxiv.org/abs/2005.11401)

---
