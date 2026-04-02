---
title: "FHIR(의료정보 교환 표준) 실전 정리 및 활용 가이드"
date: 2026-04-03
description: FHIR(Fast Healthcare Interoperability Resources) 표준의 개념, 주요 리소스, 국내 적용시 참고사항, 예시 API 활용, 오픈테스트서버, 보안 이슈까지 실무 관점에서 정리
categories: [data]
tags: [fhir, 의료정보, hl7, interoperability, api, healthcare, 표준]
---

## FHIR이란?

**FHIR(Fast Healthcare Interoperability Resources)**는 HL7에서 정의한 차세대 의료정보 교환 표준입니다.
RESTful API, JSON/XML 등 현대적 웹 기술을 적용해 의료 데이터의 상호운용성과 표준화된 통신을 실현합니다.

---

### 주요 특징

- **경량 리소스(Resource) 단위**로 환자, 진료, 검사결과 등 다양한 정보를 관리
- **RESTful API**, **JSON/XML 지원** : 병원, 기관 간 실시간 데이터 연동 용이
- **확장성 및 프로파일링** : 각국/기관 상황에 맞춰 커스터마이즈 가능
- **광범위한 오픈 소스/테스트 환경**

---

## FHIR 구조 및 주요 Resource 예시

FHIR는 '리소스(Resource)'라는 기본 단위로 각종 의료 데이터를 표현합니다.

| 구분       | 예시 리소스   | 설명                                    |
|------------|:-------------|:-----------------------------------------|
| 환자       | `Patient`    | 기본 신상 및 식별 정보                   |
| 진료정보   | `Encounter`  | 진료/입원/외래 방문 등의 기록            |
| 처방/오더  | `MedicationRequest` | 의약품 처방 이력                      |
| 결과       | `Observation`| 혈압, 검사수치 등 각종 결과              |
| 의사정보   | `Practitioner`| 의료진(의사/간호사) 정보               |
| 문서       | `DocumentReference`| 검사보고서, 촬영이미지 등 파일 링크   |

---

## FHIR JSON 예제: 환자(Patient)

```json
{
  "resourceType": "Patient",
  "id": "12001",
  "name": [
    {
      "use": "official",
      "family": "Kim",
      "given": ["Jihun"]
    }
  ],
  "gender": "male",
  "birthDate": "1990-07-28"
}
```

---

## FHIR 기본 구조

- **Base URL**: `https://{fhir-server}/{ResourceType}/{id}`
  - 예: `https://hapi.fhir.org/baseR4/Patient/12001`
- 표준 HTTP 동작:  
    - GET (조회), POST (생성), PUT (수정), DELETE (삭제)

---

## 실습: 공개 FHIR 테스트 서버 활용

- [HAPI FHIR Public Test Server](https://hapi.fhir.org/)
- [Smart Health IT Sandbox](https://sandbox.smarthealthit.org/)
- [Inferno Tool (인증 테스트)](https://inferno.healthit.gov/)

예시: 환자(Patient) 정보 조회   
```bash
curl -X GET "https://hapi.fhir.org/baseR4/Patient/12001" -H "accept: application/fhir+json"
```

---

## 국내 적용시 유의사항

- 진료/환자 식별자(ID) 매핑, 한글 인코딩 등 **로컬 환경에 맞춰 커스터마이징 필요**
- 기관/사업 목적별 FHIR 프로파일 정의(최소항목, 필수값 표준화 등)
- 개인정보보호법, 의료법 등 **보안/프라이버시 고려 필수**
  - OAuth2, JWT 등 인증/인가 방법 도입 권고

---

## 오픈소스/참고 자료

- [공식 FHIR Documentation (HL7)](https://hl7.org/FHIR/)
- [HAPI FHIR Java Library](https://hapifhir.io/)
- [KoFhir (한국 FHIR 커뮤니티)](https://github.com/koFhir)
- [국내외 표준적용 사례 및 보고서(한국보건의료정보원 등)]
- [FHIR 한국어 자료(대한의료정보학회)](https://www.kosmi.org/)

---

> FHIR 적용 및 API 설계/운영시 반드시 각종 법적 규정 및 보안, 조직 내 가이드라인을 준수하세요.  
> 실서비스 전, 오픈테스트 서버/샌드박스 환경에서 테스트 적극 활용 권장