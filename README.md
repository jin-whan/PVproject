# ☀️ LSTM-based MPPT Trigger Model for PV Systems
**태양광 발전 시스템의 부분 음영 조건(PSC) 한계 극복을 위한 LSTM 기반 센서리스 MPPT 트리거 모델**

![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)
![PyTorch](https://img.shields.io/badge/PyTorch-Red.svg)
![pvlib](https://img.shields.io/badge/pvlib-Simulation-yellow.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

## 📖 Overview
태양광 발전 시스템에서 **부분 음영 조건(Partial Shading Condition, PSC)**이 발생하면 P-V 출력 곡선에 다중 피크(Multi-peak)가 형성됩니다. 이로 인해 기존의 P&O(Perturb and Observe) 알고리즘은 진짜 최대 전력점인 **전역 최대 전력점(GMPP)**을 찾지 못하고 **국부 최대 전력점(LMPP)**에 갇히는 치명적인 한계를 가집니다. 

본 프로젝트는 외부 환경 센서(일사량계, 온도계) 없이, 인버터단에서 수집되는 과거 60초간의 시계열 전기적 데이터만으로 **새로운 GMPP 발생 여부를 즉각적으로 판별하는 초경량 딥러닝(LSTM) 트리거 모델**을 제안합니다.

## ✨ Key Features
- **🚫 무센서 제어 (Sensorless Control)**
  - 추가 센서 없이 인버터 내부의 과거 60초 데이터($V, I, P, dP/dV$)와 시간 메타데이터($Day, Hour$)만을 활용합니다.
- **⚡ 초경량 인공지능 (Ultra-lightweight AI)**
  - 주기성 인코딩(Sin/Cos)이 적용된 2-Layer LSTM 아키텍처를 사용하여 마이크로컨트롤러 환경에서도 가볍게 동작합니다.
- **🎯 압도적인 분류 정확도 및 전력 회수율**
  - LMPP 갇힘 현상 감지 정확도 **99.85%** 달성.
  - 불필요한 전역 스캐닝을 억제하고, 기존 P&O 대비 **+6.8% (1,501.65Wh)**의 전력을 추가 구출하여 **최종 MPPT 효율 99.65%**를 달성했습니다.
- **🌤️ 정밀한 물리 시뮬레이션 환경**
  - `pvlib`를 활용하여 모듈의 열관성(Thermal inertia) 및 동적 구름 이동(Moving Cloud) 현상을 정밀하게 모사했습니다.

## 🏗️ System Architecture
1. **Data Collection**: P&O 알고리즘이 추종하는 동작점 데이터를 1분 크기의 슬라이딩 윈도우(Sliding Window) 버퍼에 지속 저장합니다.
2. **Feature Engineering**: $V, I, P, dP/dV$ 물리량 스케일링 및 $Day, Hour$의 주기적 특성(Cyclical) 인코딩을 수행합니다.
3. **LSTM Trigger**: 1분 버퍼 데이터를 LSTM에 입력하여 GMPP 변동 여부를 이진 분류(0: 유지, 1: 탐색 필요)합니다.
4. **Action**: 출력이 1(Trigger ON)일 경우에만 Global MPPT(예: GWO)를 가동하여 새 GMPP로 갱신 후 P&O로 복귀합니다.

## 📂 Repository Structure
├── pvproject1_2.py        # 1. 태양광 시뮬레이션 환경 구축 및 AI 학습용 시계열 데이터 생성기 (pvlib)  
├── pvprojectML1_0.py      # 2. PyTorch 기반 2-Layer LSTM 모델 학습, 데이터 전처리 및 평가 루프  
├── pvproject_viz.py       # 3. P-V 곡선 변화 및 P&O 추종 실패(LMPP 갇힘) 애니메이션 시각화  
├── pvproject_test.py      # 4. LSTM 모델 작동 테스트 시각화  
├── dataset/               # 생성된 npz 형태의 시계열 훈련/검증 데이터 (예: V, I, P, dP/dV, Day, Hour)  
├── models/                # 학습 완료된 LSTM 가중치 (.pth)  
└── README.md              # 프로젝트 설명 (현재 문서)  

## 🚀 Quick Start
### 1. Requirements
pip install torch numpy pandas matplotlib tqdm pvlib scikit-learn

### 2. Data Generation
`pvlib` 물리 엔진을 구동하여 다양한 구름 이동(Moving/Uniform) 환경 하에서의 P&O 제어 데이터를 추출합니다.
python pvproject1_2.py

### 3. Model Training
추출된 데이터셋(`.npz`)을 바탕으로 LSTM 모델을 학습시킵니다.
python pvprojectml1_0.py

### 4. Visualization (Optional)
구름 이동에 따른 P-V 곡선 다중 피크 형성 및 LMPP 갇힘 현상을 시각화합니다.
python pvproject_viz.py

## 📊 Performance & Results
| Metric | Value | Description |
|---|---|---|
| **Accuracy** | `99.85%` | 테스트 데이터 샘플에 대한 LMPP 정체 판별 정확도 |
| **Energy Recovery** | `+1,501.65 Wh` | 14시간 시뮬레이션 기준, 기존 P&O 대비 추가 획득 발전량 |
| **Improvement** | `+6.8%` | 기존 P&O 알고리즘 대비 발전량 향상률 |
| **Total MPPT Efficiency**| `99.65%` | 이상적인 True GMPP 대비 달성한 최종 시스템 효율 |

## 🎓 Publication / Citation
본 연구 내용은 대한전기학회(KIEE) 하계학술대회에 제출되었습니다. 
> *"태양광 발전 시스템에서 부분 음영 조건(PSC) 발생 시 기존 P&O 알고리즘이 다중 피크로 인해 국부 최대 전력점(LMPP)에 정체되는 한계를 극복하고자, 시계열 전기적 데이터를 활용한 센서리스(Sensorless) 기반의 LSTM MPPT 트리거 모델을 제안하였다."*

## 🧑‍💻 Author
- **최진환** (광운대학교)
- **최성준** (광운대학교)
