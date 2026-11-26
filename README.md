# Stock_Prediction

종목코드 036570의 일봉 데이터로 다음 날 종가 수익률을 예측하는 Bi-LSTM / CNN+Bi-LSTM 실험.

## 데이터

- `036570.xlsx` (레포 미포함): Date, Open, High, Low, Close, Volume
- 이동평균(3일) 적용 버전 기준 학습 4,147 / 검증 518 / 테스트 518행 (시계열 순서대로 80/10/10 분할)

## 방법

`main.ipynb` 하나로 구성.

1. 전처리: 각 컬럼 pct_change → 가격 4개 컬럼 공통 min-max, Volume 별도 min-max → 길이 32 윈도우로 시퀀스 생성 (타깃은 다음 행의 Close 수익률)
2. 모델 (Keras, loss=MSE, Adam, 200 epoch, val_loss 기준 체크포인트)
   - Bi-LSTM: Bidirectional LSTM(128, 128, 64) → GlobalAvg/MaxPool concat → Dense(64) → Dense(1)
   - CNN + Bi-LSTM: 1D Inception 블록(A, B, C 각 2개) 뒤에 위와 동일한 Bi-LSTM 헤드
   - 각각 원본 / 3일 이동평균 적용 데이터로 학습 (총 4개 실험)
3. 백테스트: 예측 결과를 엑셀로 내보내 외부에서 매매 조건(Condition) 컬럼을 만든 뒤, 조건 전환 시점의 시가로 매수/매도하여 누적 수익률 계산 (거래당 수수료 0.5% 반영)

## 결과

노트북에 저장된 평가 출력 (정규화된 수익률 스케일, 어느 실험의 출력인지 셀 출력만으로는 특정 불가):

| 구분 | Loss(MSE) | MAE | MAPE |
| --- | --- | --- | --- |
| Train | 0.0003 | 0.0139 | 1.7937 |
| Validation | 0.0002 | 0.0108 | 1.3838 |
| Test | 0.0002 | 0.0117 | 1.4912 |

백테스트 누적 수익률(ROR): 1.2158 (수수료 반영)

## 실행 방법

노트북 안의 `data_path`, `load_model` 경로를 로컬 경로로 바꾼 뒤 순서대로 실행. TensorFlow 2.5 기준.
