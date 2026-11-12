# OCR: text detection and recognition

## 1. PP-OCRv5 학습 전후

공식 PP-OCRv5 mobile det와 Korean mobile rec를 초기값으로 사용한 강의 문서 도메인 실험입니다. 교정된 동일 평가 라벨을 사용한 비교만 포함합니다. GT 수정 전후의 baseline 상승은 모델 개선으로 계산하지 않습니다.

| 평가 / 비교 | 지표 | n / 유효 n | 초기·기준 | 학습·변경 후 | 변화 |
|---|---|---:|---:|---:|---:|
| clean-crops · pretrained-to-finetuned | exact-match | 3,493 / 3,493 | 75.87% | 86.89% | +11.02 pp |
| korean-crops · pretrained-to-finetuned | exact-match | 731 / 731 | 58.96% | 85.50% | +26.54 pp |
| out-of-domain · pretrained-to-finetuned | exact-match | 1,867 / 1,867 | 54.47% | 47.51% | -6.96 pp |
| clean-pages · pretrained-to-finetuned | Hmean | 139 / 139 | 92.76% | 95.75% | +2.99 pp |
| diagram-pages · pretrained-to-finetuned | Hmean | 219 / 219 | 73.81% | 80.56% | +6.75 pp |
| page-text · det-only-change | character-accuracy | 66 / 66 | 92.74% | 92.24% | -0.50 pp |
| page-text · rec-only-change | character-accuracy | 66 / 66 | 92.74% | 93.46% | +0.72 pp |

### Loss와 하이퍼파라미터

| 항목 | Detection | Recognition |
|---|---|---|
| Loss | DBLoss: shrink-map Dice, threshold-map, binary-map; alpha 5, beta 10, OHEM 3 | MultiLoss = CTC + NRTR |
| Optimizer | Adam, betas 0.9/0.999 | Adam, betas 0.9/0.999 |
| Learning rate | 1e-4, cosine, warmup 1 epoch | 5e-5, cosine, warmup 1 epoch |
| L2 regularization | 5e-5 | 3e-5 |
| Batch / GPU | 8 | nominal 32; multiscale sampler may vary actual batch |
| Training budget | 20 epochs configured; best epoch 2 | 6 epochs; best epoch 6 |
| Training size | 2,792 pages | 19,684 crops |
| Input | 640×640 training crops; 144-DPI source render | nominal 3×48×320; multiscale heights 32/48/64 |
| Augmentation | rotation ±3°, scale 0.6–1.4, appearance degradation p=0.6 | concatenation p=0.5, appearance degradation p=0.6, RecAug |

학습 시 불확실한 영역을 ignore mask로 제외했습니다. 합성 열화 성능을 실제 스캔 일반화로 확대하지 않습니다. Out-of-domain 정확도는 54.47% → 47.51%로 하락했습니다.

Detection Hmean 개선만으로 배포하지 않았습니다. 같은 66페이지 E2E에서 det만 교체하면 문자 정확도가 **92.74% → 92.24%**로 하락했습니다. rec만 교체한 비교는 **92.74% → 93.46%**였습니다.

## 2. OCR-v6: 같은 시작 모델에 대한 도메인 적응

| 평가 / 비교 | 지표 | n / 유효 n | 초기·기준 | 학습·변경 후 | 변화 |
|---|---|---:|---:|---:|---:|
| validation-all · same-model-domain-adaptation | exact-match | 9,484 / 9,484 | 85.28% | 85.54% | +0.26 pp |
| validation-all · same-model-domain-adaptation | CER | 9,484 / 9,484 | 5.01% | 5.09% | +0.08 pp |
| validation-documents · same-model-domain-adaptation | exact-match | 1,800 / 1,800 | 72.89% | 75.06% | +2.17 pp |
| validation-documents · same-model-domain-adaptation | CER | 1,800 / 1,800 | 3.00% | 2.83% | -0.17 pp |

개발셋은 전체 9,484개, 문서 부분집합은 1,800개입니다. Validation으로 고른 step 250의 결과이며 독립 테스트 수치가 아닙니다. 전체 정확도는 조금 상승했으나 전체 CER는 소폭 악화했습니다.

| 설정 | 값 |
|---|---|
| Objective | length-normalized CTC + 0.2 × direction CE + 3.0 × replay-only teacher KL |
| CTC | blank=0; per-sample loss / target length, then mean |
| Optimizer | Adam; L2 1e-5; global grad clip 5 |
| LR | 1e-5; warmup 200 steps; cosine to 10% |
| Layer LR multipliers | backbone 0.05; encoder 0.5 |
| Batch | 32 / GPU |
| Precision | FP32 |
| Input / labels | width 640; max label length 80; 25,924 dictionary classes |
| Budget / selection | max 4 epochs; 64,000 sampled examples/epoch; stopped at step 1,500; selected step 250 |
| Regularization | frozen direction parameters; checkpoint selected by guarded validation score |

### 별도의 v5 → v6 시스템 교체 비교

| 평가 / 비교 | 지표 | n / 유효 n | 초기·기준 | 학습·변경 후 | 변화 |
|---|---|---:|---:|---:|---:|
| fixed-multilingual · v5-to-v6-system-change | exact-match | 21,366 / 21,366 | 41.73% | 61.25% | +19.52 pp |
| fixed-multilingual · v5-to-v6-system-change | CER | 21,366 / 21,366 | 39.60% | 34.72% | -4.88 pp |
| fixed-documents · v5-to-v6-system-change | exact-match | 5,367 / 5,367 | 69.14% | 72.95% | +3.80 pp |
| fixed-documents · v5-to-v6-system-change | CER | 5,367 / 5,367 | 4.92% | 4.71% | -0.22 pp |

이 표는 같은 고정 평가셋에서 **공식 Korean v5와 다국어 v6 후보**를 비교합니다. 모델 세대와 문자 범위도 다르므로 전체 차이를 파인튜닝만의 효과로 해석하지 않습니다. 문서 성능과 다국어 전체 성능을 분리해 보세요. 이 후보를 운영 모델로 교체하지 않았습니다.

근거: `OCR-v2-metrics`, `OCR-v2-report`, `OCR-det_v2_train`, `OCR-rec_v2_train`, `OCR-v6-*` source IDs.
