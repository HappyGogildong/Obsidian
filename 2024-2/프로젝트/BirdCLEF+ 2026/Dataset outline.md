## 1) 데이터셋 요약

이 대회는 **브라질 판타날 지역의 1분짜리 환경음(soundscape)** 에서,  
각 **5초 구간마다 어떤 종이 울었는지**를 예측하는 문제다.

### 핵심 구조

- **train_audio/**
    - 개별 종의 짧은 녹음
    - birds + amphibians + mammals + reptiles + insects
    - 대부분 xeno-canto / iNaturalist 출처
    - 32kHz, ogg 포맷
- **test_soundscapes/**
    - 제출 시 숨겨진 테스트 음원 약 600개
    - 각 파일은 **1분 길이**
    - 파일당 여러 개의 **5초 row**로 평가
- **train_soundscapes/**
    - 테스트와 유사한 실제 환경음
    - 일부만 전문가 라벨이 있음
    - 이 데이터가 매우 중요함: **훈련 음원(train_audio)과 도메인이 다름**
- **train.csv**
    - 개별 녹음 메타데이터
    - `primary_label`, `secondary_labels`, 위경도, author, rating, collection 등 포함
- **taxonomy.csv**
    - 234개 클래스의 taxonomy 정보
    - 제출 컬럼과 1:1 대응
- **sample_submission.csv**
    - `row_id` + 234개 species probability 컬럼

### 문제 형태

- **multi-label classification**
- 각 row는 **하나의 5초 chunk**
- 출력은 각 클래스별 **presence probability**
- 평가는 **234개 클래스 확률의 ROC-AUC macro**

---

### 2) 이 데이터에서 중요한 해석 포인트

### (1) train_audio와 test_soundscapes는 성격이 다름

- `train_audio`는 깨끗한 개체 음성 중심
- `test_soundscapes`는 실제 현장 환경음
- 그래서 **단순히 개체 음성만 학습하면 도메인 갭**이 크다

### (2) train_soundscapes가 핵심 bridge 데이터

- 현장 잡음, 배경음, 동시출현 패턴을 학습하는 데 매우 중요
- labeled subset은 특히 유용함
- 이 데이터는 **실전 분포에 가까운 supervised data**로 봐야 한다

### (3) 일부 클래스는 test에 아예 없을 수 있음

- 그래서 클래스별 불균형이 심하고
- ROC-AUC에서는 **positive가 존재하는 클래스만 채점**한다

### (4) taxonomy가 그냥 부가정보가 아님

- bird / amphibian / mammal / reptile / insect 분포 파악 가능
- sonotype 클래스도 존재
- 클래스별 빈도와 계통적 유사성을 이해하는 데 도움 됨

---

### 3) 추천 데이터 파이프라인

아래 순서가 가장 실전적이다.

---

### A. EDA 단계

#### 1. 클래스 분포 확인

- `primary_label` 빈도
- `secondary_labels` 포함 시 종별 총 등장 빈도
- 희귀 클래스 비율
- taxonomy 상 class group별 비율  
    예: Aves / Amphibia / Mammalia / Reptilia / Insecta

#### 2. 메타데이터 분석

- `collection` 별 분포: XC vs iNat
- `rating` 분포
- `author` 별 녹음 편향
- `latitude / longitude` 분포
- 같은 종이 지역에 따라 얼마나 다르게 나타나는지

#### 3. soundscape 분석

- labeled `train_soundscapes`에서:
    - 어떤 종이 어느 시간대에 자주 나오는지
    - 5초 segment별 라벨 밀도
    - 동시 출현(co-occurrence) 패턴
- test_soundscapes와 시간 길이, 파일 수, segment 수 확인

#### 4. 오디오 품질 점검

- silent file, clipping, very low amplitude
- 길이 이상치
- 샘플레이트 일치 여부
- 채널 수 확인

---

### B. 전처리 단계

#### 1. 오디오 로딩 표준화

- 모두 **32kHz mono**로 통일
- waveform normalization
- 너무 긴 파일은 chunking
- 너무 짧은 파일은 padding

#### 2. chunking 전략

이 대회는 5초 단위 row가 기본이므로:

- 훈련 시에도 **5초 단위 segment 생성**
- 가능하면 overlap을 조금 주는 것도 고려
    - 예: 5초 window, 2.5초 hop
- soundscape에서는 boundary 걸림 현상이 많아서 overlap이 유리할 수 있음

#### 3. 라벨 생성

- 개별 녹음(train_audio):
    - primary_label = positive
    - secondary_labels도 positive 후보로 반영 가능
- soundscape 라벨:
    - 해당 5초 구간에 나타난 종들을 multi-hot vector로 변환

#### 4. feature 변환

CPU 제한을 생각하면 가장 현실적인 건:

- **log-mel spectrogram**
- STFT → mel filterbank → log compression
- 이미지처럼 CNN에 입력

권장 설정 예:

- n_mels: 128
- hop_length: 320 또는 512
- window length: 20~40ms 수준
- amplitude to dB 변환

---

### C. 학습용 샘플 구성

#### 1. 개체 음성 + 환경음 혼합 학습

가장 중요하다.

- `train_audio`만 쓰지 말고
- `train_soundscapes` labeled subset을 반드시 섞기

이유:

- 실제 test와 분포가 가까워짐
- 배경 잡음, 공존 종, 약한 신호에 강해짐

#### 2. balanced sampling

클래스 불균형이 심하므로:

- rare class oversampling
- class-aware sampling
- species group별 균형 샘플링 고려

#### 3. negative sample 확보

환경음 구간 중 라벨이 없는 segment를 negative로 활용

- 단, unlabeled = true negative라고 단정하면 안 됨
- 너무 공격적인 negative labeling은 위험

---

### 4) 추천 EDA 산출물

실제로 노트북에 넣으면 좋은 그림/표는 다음이다.

### 필수

- 클래스 빈도 bar chart
- taxonomy class별 비율
- `rating` 분포
- `collection` 분포
- 오디오 길이 분포
- mel spectrogram 샘플 시각화
- labeled soundscape segment 예시

### 있으면 좋은 것

- 클래스 co-occurrence heatmap
- 위경도 scatter plot
- 특정 종의 시간대별 등장 패턴
- train_audio vs train_soundscapes 스펙트로그램 비교

---

### 5) 학습 파이프라인 추천

### 입력

- 5초 waveform
- 또는 5초 mel spectrogram

### 모델

- CPU 제약이 있으므로:
    - EfficientNet-B2 / B3
    - EfficientNetV2-S는 성능은 좋지만 더 무거움
- 최종적으로는 **작고 빠른 모델 + ensemble**이 현실적

### loss

- BCEWithLogitsLoss
- class imbalance가 심하면 focal/asymmetric loss도 검토 가능

### validation split

매우 중요하다.

- 같은 soundscape 파일이 train/valid에 섞이면 안 됨
- 가능하면 **site / recording file 단위로 group split**
- train_audio와 train_soundscapes를 섞을 때도 leakage 방지 필요

---

### 6) 실전 파이프라인 한 줄 버전

**EDA → metadata / class imbalance / soundscape 패턴 분석 → 32kHz mono 표준화 → 5초 chunk 생성 → log-mel spectrogram 변환 → class-balanced 학습 샘플링 → 그룹 기반 validation split → BCE 학습 → chunk probability 출력**

---

### 7) 이 대회에서 특히 중요한 우선순위

1. **train_soundscapes 활용**
2. **5초 chunk 기반 학습과 추론 일치**
3. **class imbalance 대응**
4. **site/file 단위 validation**
5. **CPU 90분 안에 돌아가는 경량 모델**