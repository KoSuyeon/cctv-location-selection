# 🎥 김해시 시민안전 사각지대 해소를 위한 CCTV 설치위치 선정

<p align="left">
  <strong>발표자료</strong>&nbsp;
  <a href="./presentation/cctv_classification_presentation.pdf">
    <img align="center" src="https://img.shields.io/badge/PRESENTATION-PDF-red?style=for-the-badge&logo=adobeacrobatreader&logoColor=white">
  </a>
</p>

> LH 한국토지주택공사 「2022 COMPAS 공공데이터 활용 분석 공모전」참가 프로젝트  
> 112신고접수, 인구관련 데이터, 시설물 데이터 등 다양한 데이터를 활용하여 시민안전 사각지대 분석을 수행하고, 시민안전을 최대한 확보할 수 있도록 최적의 CCTV 설치위치를 선정

---
## 📌 프로젝트 진행기간: '22.07~'22.09
## 📌 프로젝트 개요

김해시는 경남 도내 5대 강력범죄 발생률이 가장 높은 지역 중 하나입니다.  
본 프로젝트는 **데이터 기반 회귀분석**을 통해 CCTV 설치 효과를 정량화하고,  
범죄 위험도와 CCTV 설치 효과를 종합하여 **신규 CCTV 설치 우선순위 50개 격자**를 선정합니다.

- **팀원**: 고수연, 김혜인, 이은지, 오연 (성신여자대학교 통계학과 19학번)


---

## 📁 디렉토리 구조

```
cctv-location-selection/
│
├── data/                        # 원본 데이터 (COMPAS 제공, 미포함)
│   ├── raw/                     # 원본 데이터
│   └── processed/               # 전처리된 데이터
│
├── notebooks/                   # Jupyter Notebook 분석 파일
│   ├── 01_EDA.ipynb             # 데이터 탐색 및 시각화
│   ├── 02_preprocessing.ipynb   # 데이터 전처리 (격자 매핑, 가중치 부여)
│   ├── 03_cctv_effect.ipynb     # CCTV 설치 전후 112신고 건수 비교
│   └── 04_regression.ipynb      # 회귀분석 및 최종 격자 선정
│
├── src/                         # Python 소스 코드
│   ├── preprocessing.py         # 전처리 함수
│   ├── feature_engineering.py   # 피처 엔지니어링 (버퍼, 가중치)
│   └── regression.py            # 회귀분석 모델
│
├── results/                     # 분석 결과
│   └── top50_cctv_locations.csv # 최종 선정 50개 격자
│
└── README.md
```

---

## 🗂️ 사용 데이터

COMPAS(공공데이터 분석 플랫폼)를 통해 제공된 김해시 공공데이터를 활용했습니다.

| 카테고리 | 데이터명 |
|---|---|
| CCTV | 김해시_CCTV설치현황 |
| 범죄 | 김해시_112신고이력(격자매핑) |
| 안전 | 김해시_보안등설치현황, 김해시_안전비상벨설치현황, 김해시_아동안전지킴이집현황 |
| 건물 | 김해시_어린이집현황, 김해시_유치원현황, 김해시_학교현황 |
| 지리정보 | 김해시_격자(100X100), 김해시_하천현황, 김해시_법정경계(읍면동) |
| 인구 | 김해시_성연령별_거주인구격자, 김해시_성연령별_요일별_유동인구 |

---

## 🔄 분석 프로세스

```
데이터 EDA → 데이터 전처리 → 회귀분석 모델링 → 결과 시각화
```

### 1️⃣ 데이터 탐색 (EDA)
- 원본 데이터 시각화 (Jupyter + QGIS 3.16)
- CCTV 설치 전후 112신고 건수 비교 분석
  - 2019→2020년: 격자당 평균 **0.12건 감소**
  - 2020→2021년: 격자당 평균 **0.14건 감소**
- CCTV 설치 효과 확인 → 회귀분석 방향 결정

### 2️⃣ 데이터 전처리

**QGIS 3.16**
- 변수별 격자 매핑 (100m × 100m 격자 기준)
- 학교 데이터 초/중/고 분리

**Python**
- 연도별 변수 개수 COUNT
- CCTV 인근 격자 버퍼 생성 및 가중치 부여
- 112신고 유형별 가중치 부여

| 신고 유형 | 가중치 |
|---|:---:|
| 중요범죄 (살인, 강도, 절도, 성폭력 등) | 5점 |
| 기타범죄 (폭력, 사기, 협박 등) | 4점 |
| 교통·질서유지 | 3점 |
| 기타경찰업무 | 2점 |
| 타기관·기타 | 1점 |

**제거 처리**
- 하천 격자 제외 (CCTV 설치 불가)
- 거주인구 0인 격자 제외

**최종 데이터셋 컬럼**

| 컬럼명 | 설명 |
|---|---|
| `gid` | 격자 ID |
| `year` | 연도 |
| `cctv_n_points` | 격자 내 CCTV 개수 |
| `cctv_buffer` | CCTV 버퍼 가중치 (인근 격자 포함 여부) |
| `security_light_n_points` | 보안등 개수 |
| `bell_n_points` | 안전 비상벨 개수 |
| `house_n_points` | 아동안전지킴이집 개수 |
| `어린이집` | 어린이집 개수 |
| `유치원` | 유치원 개수 |
| `pop_sum` | 유동인구 합계 |
| `police_weight` | 112신고 건수 × 가중치 (종속변수) |

### 3️⃣ 데이터 모델링 (회귀분석)

**Step 1. 데이터 격자 선택**
- 거주인구 0인 격자 제외
- 하천 격자 제외

**Step 2. 종속변수 처리**
- 112신고 가중치 데이터를 **로그 스케일 변환**
- IQR 기준 이상치 제거 (Q3 + 1.5×IQR 이상, Q1 - 1.5×IQR 이하)

**Step 3. 다중선형회귀**
- 독립변수: cctv_n_points, security_light_n_points, bell_n_points, cctv_buffer, 어린이집, 유치원, 초중고, 아동안전지킴이집
- 종속변수: police_weight

**Step 4. 다중공선성 확인 (VIF)**
- 초등학교, 중학교, 고등학교 → VIF = `inf` → **제거**

**Step 5. 단계적 변수 선택 (Stepwise Selection)**
- 최종 독립변수: `cctv_n_points`, `security_light_n_points`, `어린이집`, `아동안전지킴이집`

**Step 6. 최종 격자 선택**

```
CCTV 설치 필요도 = CCTV 설치 효과 점수 + 범죄 위험지역 점수

- CCTV 설치 효과 점수 = 감소한 112신고 건수
  (= 2021년 실제 112신고 건수 - 2021년 CCTV 설치 가정 후 112신고 예측 건수)

- 범죄 위험지역 점수 = 2022년 112신고 예측 건수
```

---

## 📊 분석 결과

- **최종 CCTV 설치 우선순위 50개 격자** 선정
- 기존 CCTV 공백 지역 + 범죄 위험도 높은 지역 중심으로 분포
- 상위 격자 대부분 **김해시 도심부 (삼계동, 봉황동 인근)** 집중

| 우선순위 | 격자번호 |
|:---:|---|
| 1 | 마라243939 |
| 2 | 마라278953 |
| 3 | 마라258934 |
| 4 | 마라243938 |
| 5 | 마라276943 |
| ... | ... |
| 50 | 마라286951 |

---

## 🛠️ 기술 스택

- **Python 3.8+**: pandas, geopandas, statsmodels, scikit-learn, matplotlib, folium
- **QGIS 3.16**: 격자 매핑, 공간 데이터 전처리
- **Jupyter Notebook**: 분석 및 시각화

---

## ⚙️ 실행 방법

```bash
# 1. 패키지 설치
pip install -r requirements.txt

# 2. COMPAS 데이터 다운로드 후 data/raw/ 에 배치

# 3. 순서대로 노트북 실행
jupyter notebook notebooks/01_EDA.ipynb
jupyter notebook notebooks/02_preprocessing.ipynb
jupyter notebook notebooks/03_cctv_effect.ipynb
jupyter notebook notebooks/04_regression.ipynb
```

---

## 📝 참고 문헌

- 선행연구 분석을 통한 CCTV 설치위치 평가 지표 개발 (2017)
- 유동인구 및 인구밀도를 활용한 안산시 방범용 CCTV의 입지모델링 연구
- 경찰청 112신고 접수코드(사건종별) 분류 기준
