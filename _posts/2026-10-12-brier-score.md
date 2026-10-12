---
title: "Brier Score: 확률 예측을 평가하는 방법"
date: 2026-10-12
categories: [ai]
tags: [brier-score, calibration, classification, probability, metric]
---

> Brier score는 예측 확률과 실제 결과의 평균 제곱 오차임. 분류의 맞힘 여부만 보는 accuracy와 달리, 확률이 얼마나 정확하고 보정되어 있는지를 함께 평가함.

---

## 개요

이진 분류에서 accuracy는 임계값(보통 0.5)을 넘겼는지만 봄. 같은 정답이라도 0.51로 맞힌 예측과 0.99로 맞힌 예측을 구분하지 못함.

확률 자체를 써야 하는 경우에는 이 차이가 중요함. 위험 점수, 클릭 확률, 기상 예보, 의료 사건 예측처럼 **얼마나 확신했는지**가 의사결정에 들어가는 문제임.

Brier score는 이런 확률 예측을 평가하기 위해 1950년 Glenn W. Brier가 제안한 지표임. 예측 확률과 실제 결과(0 또는 1)의 제곱 오차를 평균한 값이고, **낮을수록 좋음**.

---

## 정의

표본이 $N$개일 때 이진 Brier score는 다음과 같음.

$$
\mathrm{BS} = \frac{1}{N} \sum_{i=1}^{N} (p_i - y_i)^2
$$

- $p_i$: $i$번째 표본이 양성일 예측 확률. 범위는 0에서 1
- $y_i$: 실제 레이블. 양성이면 1, 음성이면 0

머신러닝에서 흔히 쓰는 정의는 위 식임. [scikit-learn의 `brier_score_loss`](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.brier_score_loss.html)도 이 값을 계산함.

원래 1950년 논문의 Brier score는 상호 배타적인 **모든 범주**에 대해 제곱 오차를 더함. 이진 문제에서는 양성 확률과 음성 확률의 오차가 같으므로, 원래 정의는 위 식의 2배가 됨. 논문 수치와 라이브러리 수치를 비교할 때 이 차이를 확인해야 함.

다중 클래스에서는 각 표본의 확률 벡터와 원-핫 정답 벡터의 제곱 오차를 모두 더한 뒤 표본 수로 나눔.

$$
\mathrm{BS} = \frac{1}{N} \sum_{i=1}^{N} \sum_{k=1}^{K} (p_{ik} - y_{ik})^2
$$

$K$가 클래스 수임. 클래스가 2개면 이 값은 이진 Brier score의 2배임.

---

## 값의 범위와 기준선

이진 정의 기준으로 Brier score는 0에서 1 사이임.

| 예측 | 의미 |
| --- | --- |
| 0 | 모든 표본에서 확률을 정확히 맞춤. 일어난 사건은 1, 일어나지 않은 사건은 0 |
| 0.25 | 모든 표본에 0.5를 예측한 경우. 양성 비율이 50%일 때의 단순한 기준선 |
| 1 | 확실한 예측이 전부 틀림. 0을 1로, 1을 0으로 예측 |

양성 비율이 $\bar{y}$인 데이터에서, 개별 특징을 보지 않고 항상 $\bar{y}$만 예측하면 Brier score는 $\bar{y}(1-\bar{y})$임. 양성 비율이 0.5면 0.25이고, 0.1처럼 불균형하면 0.09로 더 낮아짐.

그래서 절대값만 보고 모델을 비교하면 안 됨. 불균형 데이터에서는 쓸모없는 상수 예측도 낮은 점수를 받을 수 있음. 기준선 대비 개선을 보려면 Brier skill score를 같이 봄.

$$
\mathrm{BSS} = 1 - \frac{\mathrm{BS}}{\mathrm{BS}_{\text{ref}}}
$$

$\mathrm{BS}_{\text{ref}}$는 보통 항상 양성 비율을 예측하는 기준선임.

- 1에 가까우면 기준선보다 훨씬 나음
- 0이면 기준선과 같음
- 음수면 기준선보다 나쁨

---

## 계산 예

네 개 표본의 예측 확률과 정답이 다음과 같다고 가정함.

| 예측 확률 $p$ | 정답 $y$ | $(p-y)^2$ |
| ---: | ---: | ---: |
| 0.90 | 1 | 0.01 |
| 0.70 | 1 | 0.09 |
| 0.20 | 0 | 0.04 |
| 0.10 | 0 | 0.01 |

$$
\mathrm{BS} = \frac{0.01 + 0.09 + 0.04 + 0.01}{4} = 0.0375
$$

양성 비율은 0.5이므로 기준선은 $0.5 \times 0.5 = 0.25$임.

$$
\mathrm{BSS} = 1 - \frac{0.0375}{0.25} = 0.85
$$

같은 네 개를 0.5 기준으로 자르면 accuracy는 1.0임. 정답 방향은 유지한 채 확신만 낮춘 예측 $(0.60, 0.55, 0.40, 0.45)$도 accuracy는 1.0이지만, Brier score는 0.18125로 나빠짐. 맞혔다는 사실과 별개로, 확률의 품질이 떨어진 것임.

반대로 $(0.01, 0.01, 0.99, 0.99)$처럼 정답을 자신 있게 틀리면 제곱 오차가 커져 Brier score는 0.9801까지 올라감.

---

## 세 가지로 나누어 보는 이유

Brier score는 보통 다음 세 항으로 분해해서 해석함. 부호는 점수에 들어가는 방향임.

$$
\mathrm{BS} = \text{Reliability} - \text{Resolution} + \text{Uncertainty}
$$

- **Reliability**: 예측 확률과 실제 발생 빈도의 차이. "80%라고 말한 집단에서 실제로 80%가 발생했는가"에 해당함. 보정(calibration)이 좋을수록 작아지고, 점수에는 더해지므로 나쁠수록 Brier score가 커짐
- **Resolution**: 예측이 평균 발생률과 얼마나 다르게 갈리는지. 위험 높은 표본과 낮은 표본을 구분할수록 커지고, 빼 주는 항이므로 점수는 낮아짐
- **Uncertainty**: 정답 자체의 불확실성 $\bar{y}(1-\bar{y})$. 모델과 무관함. 양성 비율이 0.5에 가까울수록 큼

같은 Brier score라도 원인이 다를 수 있음. 순위는 잘 나누지만 확률이 한쪽으로 치우친 모델은 resolution은 좋고 reliability가 나쁨. 반대로 확률은 평균에 잘 맞지만 표본을 거의 구분하지 못하면 reliability는 좋고 resolution이 나쁨.

보정만 따로 보고 싶으면 reliability diagram이나 ECE(Expected Calibration Error)를 같이 확인하는 편이 낫음. ECE는 보정 오차만 보고, 구분 능력은 반영하지 않음.

---

## 다른 지표와 비교

| 지표 | 보는 것 | 확률의 크기 | 특징 |
| --- | --- | --- | --- |
| Accuracy | 임계값 이후의 정답 여부 | 보지 않음 | 0.51과 0.99를 같게 취급 |
| ROC-AUC | 양성·음성의 순위 | 보정은 보지 않음 | 순위만 같으면 확률을 일괄 변환해도 동일 |
| Log loss | 예측 확률의 로그 손실 | 봄 | 자신 있게 틀린 예측을 매우 크게 벌점 |
| Brier score | 확률의 제곱 오차 | 봄 | 0–1로 해석하기 쉽고, 이상치에 log loss보다 덜 민감 |
| ECE | 구간별 보정 오차 | 보정만 봄 | 구분 능력은 평가하지 않음 |

Brier score와 log loss는 둘 다 **proper scoring rule**임. 기대 점수를 가장 좋게 만드는 전략이 실제 확률을 정직하게 보고하는 것임. Accuracy는 그렇지 않음. 실제 확률이 0.6이어도, 점수를 위해 0 또는 1로 잘라 말하는 편이 유리할 수 있음.

Log loss는 $\log(p)$ 때문에 0에 가까운 확률로 정답을 놓치면 손실이 무한히 커질 수 있음. Brier score는 오차가 최대 1로 막혀 있어서 비교와 평균이 안정적임. 대신 아주 드문 사건의 과신을 강하게 벌주고 싶다면 log loss가 더 민감함.

ROC-AUC는 보정되지 않은 점수도 순위만 유지되면 점수가 같음. 운영에서 확률을 위험도로 그대로 쓸 계획이면 AUC만으로 모델을 고르면 안 됨.

---

## Python 계산

아래는 앞의 예제를 직접 계산하고, 기준선과 skill score까지 확인하는 코드임.

```python
import numpy as np
from sklearn.metrics import brier_score_loss

y_true = np.array([1, 1, 0, 0])
y_prob = np.array([0.90, 0.70, 0.20, 0.10])

manual_score = np.mean((y_prob - y_true) ** 2)
sklearn_score = brier_score_loss(y_true, y_prob)

base_rate = float(y_true.mean())
baseline_score = base_rate * (1.0 - base_rate)
skill_score = 1.0 - manual_score / baseline_score

print(f"manual={manual_score:.4f}")
print(f"sklearn={sklearn_score:.4f}")
print(f"baseline={baseline_score:.4f}")
print(f"skill={skill_score:.4f}")
```

출력은 `manual=0.0375`, `sklearn=0.0375`, `baseline=0.2500`, `skill=0.8500`임.

다중 클래스 원-핫 확장은 클래스 축의 제곱 오차를 먼저 더한 뒤 표본 평균을 내면 됨.

```python
def multiclass_brier_score(y_true_onehot: np.ndarray, y_prob: np.ndarray) -> float:
    """Mean squared error between class probabilities and one-hot labels."""
    squared_error = (y_prob - y_true_onehot) ** 2
    return float(squared_error.sum(axis=1).mean())
```

`scikit-learn` 1.5 기준으로 `brier_score_loss`는 이진 분류용임. 다중 클래스는 위처럼 직접 계산하거나, 클래스별 이진 Brier score를 따로 보고 평균하는 방식을 목적에 맞게 선택하면 됨. 클래스별 평균은 원래의 다중 범주 Brier score와 수치가 다름.

---

## 쓸 때 확인할 것

- 학습 손실과 평가 지표를 구분해야 함. 모델을 log loss로 학습했어도, 보고용 지표로 Brier score를 쓸 수 있음. 다만 최적화 목표와 평가 목표가 다르면 최고 성능 지점이 어긋날 수 있음
- 검증 세트 하나로만 보지 않는 것이 좋음. 표본이 적거나 이벤트가 드물면 점수 분산이 큼
- 불균형 데이터에서는 절대 점수보다 skill score, 또는 동일 데이터의 기준선 대비 차이를 같이 적는 것이 안전함
- 확률 보정이 목표면 reliability diagram, ECE, Platt scaling, isotonic regression 전후의 Brier score 변화를 같이 보는 것이 좋음. Brier score가 조금 나아져도 순위 지표인 AUC는 거의 그대로일 수 있음
- 예측값이 0과 1 밖의 점수라면 그대로 넣으면 안 됨. 확률로 보정한 뒤 계산해야 함

---

## 요약

Brier score는 예측 확률과 실제 결과의 평균 제곱 오차임. 이진 문제에서 흔히 쓰는 정의는 0이 완벽하고 1이 최악이며, 아무것도 구분하지 않고 0.5만 예측하면 0.25임.

Accuracy가 놓치는 확신의 차이와, AUC가 놓치는 확률 보정을 함께 반영함. Log loss보다 이상치에 덜 민감해서 모델 비교에 쓰기 쉽지만, 데이터 양성 비율에 따라 기준선이 달라지므로 skill score나 기준선 점수를 함께 보는 것이 좋음.

---

## 참고

- Brier, G. W. (1950). [Verification of forecasts expressed in terms of probability](https://doi.org/10.1175/1520-0493%281950%29078%3C0001:VOFEIT%3E2.0.CO;2). *Monthly Weather Review*.
- Murphy, A. H. (1973). A new vector partition of the probability score. *Journal of Applied Meteorology*.
- [scikit-learn: Brier score loss](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.brier_score_loss.html)
