# FinGuard: 실시간 이상거래 탐지 시스템

## 1. 프로젝트 개요
FinGuard는 카드 거래 데이터를 기반으로 이상 거래(Fraud)를 탐지하는 머신러닝 기반 시스템입니다.  
단순히 모델 성능을 높이는 것이 아니라, **실제 서비스 환경에서 동작 가능한 feature와 구조를 고려한 모델 설계**를 목표로 했습니다.

---

## 2. 문제 정의

금융 이상거래 탐지에서 중요한 것은 단순 정확도가 아닌 다음입니다:

- ❗ 사기 거래를 놓치지 않는 것 (Recall)
- ❗ 실제 운영 환경에서도 동일하게 동작하는 구조
- ❗ 특정 feature에 과도하게 의존하지 않는 모델

초기 모델에서는 높은 성능을 보였으나,  
일부 feature가 실제 서비스에서는 사용 불가능하거나 비현실적인 값임을 확인했습니다.

---

## 3. 데이터 개요

- 데이터 크기: 1,000,000건
- Feature 수:
  - Full: 7개
  - Service-aligned: 5개

### 사용 변수

```text
distance_from_home
distance_from_last_transaction
ratio_to_median_purchase_price
repeat_retailer
used_chip
used_pin_number (제거)
online_order (제거)
fraud (target)
```

## 4. 데이터 분석

### 클래스 분포

- 정상 거래: 약 91%
- 이상 거래: 약 9%

👉 심각한 클래스 불균형 존재  
👉 단순 Accuracy는 의미가 없으며 Recall 중심 평가 필요

---

## 5. Feature 의존성 분석

### Feature Importance (Full 모델 기준)

| Feature | Importance |
|--------|------------|
| ratio_to_median_purchase_price | 0.496 |
| distance_from_home | 0.154 |
| online_order | 0.151 |
| distance_from_last_transaction | 0.067 |
| used_pin_number | 0.051 |
| used_chip | 0.043 |
| repeat_retailer | 0.034 |

👉 특정 feature (ratio_to_median_purchase_price)에 대한 의존도가 매우 높음  
👉 모델이 일부 변수에 과도하게 의존할 가능성 확인

---

## 6. 실험 설계

본 프로젝트에서는 다음 두 가지 조건으로 실험을 수행했습니다.

### ① Full Feature 모델
- 모든 feature 사용
- 모델의 최대 성능 확인 목적

### ② Service-aligned 모델 (최종 모델)
- 실제 서비스 환경에서 사용 가능한 feature만 사용
- 제거된 변수:
  - used_pin_number (실제 환경에서 확보 불가)
  - online_order (서비스 특성상 의미 제한적)

👉 학습-추론 환경 일치 고려

---

## 7. 실험 결과

### 🔹 Full 모델

| Metric | Value |
|--------|------|
| Accuracy | 0.9985 |
| Precision | 0.9941 |
| Recall | 0.9883 |
| F1 | 0.9912 |

👉 매우 높은 성능  
👉 하지만 특정 feature 의존성 존재

---

### 🔹 Service-aligned 모델 (최종)

| Metric | Value |
|--------|------|
| Accuracy | 0.9456 |
| Precision | 0.6195 |
| Recall | 0.9795 |
| F1 | 0.7590 |

👉 실제 서비스 환경 반영  
👉 Recall 유지 + Precision 감소 (Trade-off 발생)

---

## 8. Threshold 최적화

### Threshold 변화에 따른 성능

| Threshold | Precision | Recall | F1 |
|----------|----------|--------|----|
| 0.5 | 0.6044 | 0.9983 | 0.7529 |
| 0.8 | 0.6195 | 0.9795 | 0.7590 |

👉 최종 선택: Threshold = 0.8  
👉 Recall 유지하면서 Precision 개선

---

## 9. Confusion Matrix (Threshold = 0.8)

- TN: 172003  
- FP: 10516  
- FN: 359  
- TP: 17122  

### 해석

- FN (사기 놓침): 매우 낮음 → 높은 Recall 유지
- FP (오탐): 존재 → Precision 감소

👉 금융 서비스에서는 FN 최소화가 더 중요

---

## 10. 핵심 인사이트

### 1️⃣ Feature 의존성

- 특정 변수에 모델이 강하게 의존
- 제거 시 성능 급락 확인

👉 모델 일반화 문제 존재

---

### 2️⃣ 현실 반영 시 성능 변화

- Full 모델: 이상적인 성능
- Service 모델: 현실적인 성능

👉 실제 환경에서는 성능이 달라짐

---

### 3️⃣ Precision vs Recall Trade-off

- Recall 유지 (~98%)
- Precision 감소

👉 사기 탐지에서는 Recall이 더 중요한 지표

---

## 11. 결론

- 모델 성능은 데이터 분포와 feature 구성에 크게 의존한다
- 특정 feature에 의존하는 모델은 실제 환경에서 일반화가 어렵다
- 서비스 환경을 반영한 feature 설계가 필수적이다
- 높은 성능보다 실제 운영 가능한 모델 설계가 중요하다

---

## 12. 기술 스택

- Python
- Pandas / NumPy
- Scikit-learn
- XGBoost
- Imbalanced-learn (SMOTE)
- Matplotlib / SHAP
