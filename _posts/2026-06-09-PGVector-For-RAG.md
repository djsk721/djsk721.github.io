---
title: "RAG를 위한 pgvector 실전 정리"
date: 2026-06-09
categories: [MLOps, VectorDB]
tags: [pgvector, postgres, rag, vector search, 벡터검색, 하이브리드, 인덱스, ANN, 실전구성]
---

---

## 개요

pgvector는 PostgreSQL에서 벡터 임베딩을 저장/검색할 수 있게 해주는 오픈소스 확장 툴
Postgres의 SQL, JOIN, 트랜잭션 등 강점은 그대로 활용하며, 벡터 유사도 검색을 매우 간단하게 구현할 수 있는 것이 특징이다
RAG, 의미 기반 검색, 추천, 지식 베이스 챗봇 등에 널리 사용되고 있으며, 벡터 DB 도입 없이 빠르게 POC 또는 규모 서비스를 구축하는 데 매우 실용적

---

## pgvector의 핵심 기능

| 기능                           | 설명                                                       |
| ----------------------------- | ---------------------------------------------------------- |
| 벡터 타입 지원                 | `vector`, `halfvec`, `bit`, `sparsevec` 등 다양한 벡터 타입 지원  |
| 벡터 유사도 검색               | 쿼리 벡터와 가장 가까운 데이터를 효율적으로 검색                 |
| 다양한 거리 연산자             | L2, cosine, inner product, L1, Hamming, Jaccard 등         |
| ANN 인덱스 (근사 최근접 탐색)  | HNSW, IVFFlat 인덱스 지원                                  |
| SQL 필터/조인/트랜잭션/백업 지원 | PostgreSQL의 표준 SQL, JOIN, ACL, 백업 등 전 기능 활용         |

- 공식 문서 및 설치: [pgvector 공식 GitHub][1]
- PostgreSQL 13 이상에서 지원, Docker/Homebrew/APT/conda-forge 등 다양한 설치 지원

---

## 기본 사용 예시

```sql
-- 확장 설치
CREATE EXTENSION vector;

-- 테이블 생성
CREATE TABLE documents (
  id bigserial PRIMARY KEY,
  content text,
  embedding vector(1536)
);
```

**데이터 삽입**
```sql
INSERT INTO documents (content, embedding)
VALUES (
  'pgvector는 PostgreSQL용 벡터 검색 확장입니다.',
  '[0.1, 0.2, 0.3, ...]'
);
```

**가장 유사한 문서 검색**
```sql
SELECT id, content
FROM documents
ORDER BY embedding <-> '[0.12, 0.18, 0.31, ...]'
LIMIT 5;
```
- `<->`는 L2 distance 연산자 (유클리드 거리)

---

## 주요 벡터 거리 연산자

| 연산자   | 의미                        | 사용 예                 |
|---------|---------------------------|-----------------------|
| `<->`   | L2 distance (유클리드 거리)  | 기본 벡터 유사도 검색      |
| `<#>`   | negative inner product      | 내적 기반 유사도          |
| `<=>`   | cosine distance             | 의미 유사도 (RAG에 권장)   |
| `<+>`   | L1 distance                 | Manhattan distance    |
| `<~>`   | Hamming distance            | binary vector         |
| `<%>`   | Jaccard distance            | binary/sparse 계열     |

**예시: cosine distance를 이용한 RAG 검색**
```sql
SELECT *
FROM documents
ORDER BY embedding <=> '[...]'
LIMIT 10;
```
- 텍스트 임베딩 기반 RAG에서는 보통 cosine distance (`<=>`)를 많이 사용

---

## ANN 인덱스: HNSW vs IVFFlat

pgvector는 "정확 검색"과 "근사 최근접(ANN) 검색" 모두 지원하며 대량 벡터 데이터셋에서는 다음 2가지 인덱스가 실무에서 많이 활용된다고 한다

### 1. HNSW (Hierarchical Navigable Small World)
```sql
CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops);
```
- 검색 속도·recall 모두 매우 우수
- 데이터가 없어도 인덱스 생성 가능(즉시 구축)
- 대체로 RAG, chat knowledge base에 권장
- 단, 빌드/업데이트 속도가 느릴 수 있고, 메모리 사용량은 큰 편

### 2. IVFFlat
```sql
CREATE INDEX ON documents
USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);
```
- 대량, 저메모리 환경에서 잠재적으로 효율
- `lists`, `probes` 값 조정 필요(튜닝 필수)
- 충분히 데이터가 쌓인 후 인덱스 생성 권장

자세한 인덱스·튜닝 정보는 [공식 GitHub][1] 도 참조.

---

## SQL 필터링 & 하이브리드 검색

pgvector의 가장 큰 강점 중 하나는 "일반 SQL과 벡터 검색을 결합"할 수 있다는 점이다.

**예시: 메타데이터 조건 + 벡터 검색**
```sql
SELECT id, content
FROM documents
WHERE category = 'database'
ORDER BY embedding <=> '[...]'
LIMIT 10;
```

**필터+벡터+join 통합 쿼리**
```sql
SELECT id, title, content
FROM document_chunks
WHERE tenant_id = 42
  AND created_at >= now() - interval '30 days'
ORDER BY embedding <=> $1
LIMIT 10;
```


---

## 장점 요약

- **Postgres만으로 벡터 검색**: 별도 DB없이 확장만 연결해서 사용
- **운영체계 단순**: 기존 백업/복제/권한/모니터링 등 PostgreSQL 체계 활용
- **SQL·JOIN·복잡 쿼리 가능**: 사용자/권한/조직/태그 등과 자유롭게 조인
- **RAG·유사 검색 실전 서비스에 적합**: 실험/중견 규모까지 신속히 적용
- **손쉬운 배포 및 유지보수**: 컨테이너, PaaS 등 환경에서 바로 사용

---

## 단점 및 주의점

- **초대형 전용 벡터DB(예: Milvus, Pinecone) 대비 한계**
    - 1억 개 이상, 초고QPS, 분산확장, 복잡 랭킹 등은 전용 제품이 더 적합
- **인덱스 튜닝 필수** - HNSW/IVFFlat 옵션에 따라 결과/속도 차이 큼
- **메타데이터 필터+ANN시 결과 변동성/성능 차이**
    - 쿼리 플래너/인덱스 선택에 따라 recall이 바뀔 수 있음 ([최근 arXiv 논문][3] 참고)

---

## 언제 pgvector을 쓰면 좋은가?

- 이미 PostgreSQL을 운영 중인 서비스
- RAG, semantic search, 챗봇 검색 등 기존 인프라에 빠르게 AI 검색 붙이고 싶을 때
- 수백만 미만/중견 규모 벡터 데이터
- 권한, 필터, 조인 등 복잡한 동적 조건이 중요할 때
- 별도의 "전용 벡터 DB" 운영 부담을 줄이고 싶은 곳

**다음과 같은 상황에서는 전용 벡터DB도 검토**
- 수천만~수억 단위 대규모 검색/서빙
- 매우 높은 동시 쿼리 QPS
- 분산검색, 고성능 랭킹이 필요
- "벡터 검색이 서비스 핵심 병목"인 시스템

---

## 실전 RAG 테이블 구성 예시

```sql
CREATE TABLE document_chunks (
  id bigserial PRIMARY KEY,
  document_id bigint NOT NULL,
  tenant_id bigint NOT NULL,
  content text NOT NULL,
  metadata jsonb,
  embedding vector(1536),
  created_at timestamptz DEFAULT now()
);

CREATE INDEX document_chunks_embedding_hnsw_idx
ON document_chunks USING hnsw (embedding vector_cosine_ops);

CREATE INDEX document_chunks_tenant_idx
ON document_chunks (tenant_id);
```

**RAG 검색 쿼리**
```sql
SELECT id, document_id, content
FROM document_chunks
WHERE tenant_id = 123
ORDER BY embedding <=> $1
LIMIT 10;
```
> 실제 운영에선 chunk 크기관리, embedding 모델버전, tenant/category/date 인덱스 추가, re-ranking, hybrid(키워드+벡터) 검색 등을 결합하게 됩니다.

---

## 한 줄 정리

**pgvector는 PostgreSQL만으로 임베딩 저장, 유사도 검색, ANN 인덱싱까지 모두 제공하는 확장으로, RAG·의미검색 등 실무 AI 검색 시스템을 빠르게 프로토타이핑·운영할 수 있게 해줌.
특대규모/초고성능 환경에서는 전용 벡터DB와 비교해 선택
실제로 실무에서는 Postgresql을 활용하고 있기에 Pgvector를 통한 RAG를 구현해보았는데 생각보다 직관적이라 앞으로도 활용을 많이 할 것 같음**

[1]: https://github.com/pgvector/pgvector "pgvector/pgvector Github Repository"
[2]: https://www.postgresql.org/about/news/pgvector-080-released-2952 "pgvector 0.8.0 Released!"
[3]: https://arxiv.org/abs/2602.11443 "Filtered Approximate Nearest Neighbor Search in Vector Databases: System Design and Performance Analysis"
