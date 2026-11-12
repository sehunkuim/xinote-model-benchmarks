# Evidence, comparisons, and disclosure boundary

## 비교 유형

1. **pretrained-to-finetuned**: 같은 모델 계열의 초기 공식 가중치와 도메인 학습 후를 비교합니다.
2. **previous-production-to-candidate / continued training**: 기존 운영 또는 이미 학습된 checkpoint를 기준으로 추가 개선을 비교합니다. 초기 공식 모델과의 차이로 바꾸어 쓰지 않습니다.
3. **system change**: tokenizer·문자 범위·모델 세대가 달라집니다. 전체 차이를 학습 하나의 인과 효과로 주장하지 않습니다.
4. **checkpoint-selection-dev**: 학습 및 모델 선택에 관찰된 개발셋입니다. 독립 test의 대체물이 아닙니다.

## 수치 처리

- 비율은 표에서 %로 변환합니다. 0.10 → 0.15는 +5pp입니다.
- CSV/JSON의 delta는 `after - before`; CER/WER 하락은 음수 delta가 정상입니다.
- relative improvement는 높은 것이 좋은 지표는 `(after-before)/before`, 낮은 것이 좋은 지표는 `(before-after)/before`입니다.
- 본문은 읽기 위해 반올림하며 데이터 파일은 원래 계산 정밀도를 보존합니다. 서로 다른 지표의 평균 개선율을 만들지 않습니다.
- Formula의 `n_evaluable`은 참조 렌더가 가능한 항목 수입니다. 이를 전체 표본 수와 함께 공개합니다.
- 단일 실행의 수치를 CI나 유의성 검정 결과로 표시하지 않습니다. GPU 부하가 다른 timing을 정확도 개선과 섞지 않습니다.
- 하락 항목은 삭제하지 않았습니다. 후보와 운영 가중치 일치 상태, 과거 기록을 구분합니다.

## 데이터 비공개

공개: 모델명, objective 설명, 하이퍼파라미터 요약, 평가 표본 수, 집계 지표, 학습·개발셋 숫자로 만든 그래프, 근거 파일 지문.

비공개: 데이터셋 파일과 원문, PDF·이미지·음성·전사문·라벨·개별 예측, 학습·평가 소스, 실행 가능한 전체 config, checkpoint·adapter·optimizer state, 서버 주소·로컬 경로·인증정보.

외부에 이미 공개된 학습 저장소의 내용을 이 저장소로 복사하거나, 기존 저장소의 공개 범위를 변경하지 않았습니다. 이번 저장소에서 학습 데이터와 코드를 새로 공개하지 않는다는 범위입니다.

## 버전 및 근거

기준일 2026-11-12에 보존된 결과를 읽고 집계했습니다. 새 평가를 수행하지 않았습니다. Source ID는 공개용 익명 식별자이며 SHA-256은 비공개 근거 파일의 내용 지문입니다. 지문은 원문 열람이나 완전 재현 수단이 아닙니다.

Folder names와 오래된 metadata보다 실제 weights 지문을 우선했습니다. Formula의 과거 epoch1 카드와 현재 모델을 구분했고, ASR v27도 학습 export와 실제 배포 파일을 비교했습니다. OCR 과거 v5 실험의 파일과 현재 캐시 파일은 동일하지 않아 현 운영 성능으로 연결하지 않았습니다.

Layout의 split 간 중복 0이라는 점은 이번 분할 내부의 성질입니다. 이전 실험에서 평가 문서를 관찰한 이력까지 없다는 뜻은 아닙니다. JEV 개발셋은 선택 편향이 있을 수 있으므로 후속 독립 test가 필요합니다.
