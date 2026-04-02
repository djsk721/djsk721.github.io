---
title: "임상 데이터 Outlier 처리 및 변수별 기준값 정리"
date: 2026-04-02
description: 임상 바이탈사인, GCS, 검사값의 데이터 유효 범위 및 극단치(Outlier) 처리 기준 정리. 실제 코드 적용/분석 시 참고.
categories: [medical]
tags: [데이터전처리, 임상데이터, 바이탈, outlier, range, 기준값]
---

<aside>

임상 데이터 분석 및 모델링 과정에서 **존재할 수 없거나, 비현실적인 극단값(Outlier)** 은 반드시 적절히 처리해야 합니다.<br>
본 문서는 아래의 임상적 데이터 범위와 outlier 처리 권장 방식을 정리하였으며, 실무 적용 시 반드시 참고할 것.

- 상위 1% 초과, 하위 1% 미만의 극단치 값은 각각 `quantile(0.99)`, `quantile(0.01)` 로 Imputation 권고
- 변수가 허용하는 **임상적 최소~최대 값** 범위를 벗어나는 입력값도 "outlier"로 간주하여 동 방식으로 처리할 것 (범위 표 참고)
- 참고: [Biomedcentral - Table 3: Data range for vital signs and laboratory results](https://ccforum.biomedcentral.com/articles/10.1186/s13054-024-04866-7#Sec11)

</aside>

---

## Outlier 처리 로직 예시 (Python)

```python
def impute_outlier(series, min_v, max_v):
    q01, q99 = series.quantile([0.01, 0.99])
    # 임상적 min/max 범위 먼저 클리핑
    s = series.clip(lower=min_v, upper=max_v)
    # 허용 범위를 벗어난 outlier imputation(각각 quantile로 대체)
    s[s < q01] = q01
    s[s > q99] = q99
    return s
```

---

## Vital sign Data Range

| 변수   | 의미                | 데이터 범위       |
| ------ | ------------------- | ---------------- |
| HR     | Heart Rate          | 10 ~ 190         |
| RR     | Respiratory Rate    | 5 ~ 50           |
| SpO2   | O2 Saturation       | 68 ~ 100         |
| DBP    | Diastolic BP(mmHg)  | 20 ~ 130         |
| SBP    | Systolic BP(mmHg)   | 40 ~ 230         |
| BT     | Body Temperature(℃) | 32 ~ 41          |

---

## GCS (Glasgow Coma Scale) Score

| 변수         | 점수 범위  |
| ------------ | ---------- |
| GCS Total    |    3 ~ 15  |
| GCS Eye      |    1 ~ 4   |
| GCS Motor    |    1 ~ 6   |
| GCS Verbal   |    1 ~ 5   |

---

## Laboratory Results Data Range

![Lab Reference Range](/assets/images/posts/2026-04-02-Bio-Data-range/Untitled.png)

(테이블 출처: [Biomedcentral Table 3](https://ccforum.biomedcentral.com/articles/10.1186/s13054-024-04866-7#Sec11))

---

## 참고

- 임상 데이터 내 의미상 불가능한 값(ex. SpO2 > 100, HR < 10)은 측정 오류, 레코딩 실수 등으로 인식하여 **필히 outlier 처리/제거**  
- 최종 exception: 임상의/전문가가 별도로 인정하는 경우만 예외 허용

---
