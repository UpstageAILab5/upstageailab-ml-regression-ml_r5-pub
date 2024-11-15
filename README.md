# 업스테이지 x 패스트캠퍼스 AI Lab 1차 경진대회: 서울 부동산 가격 예측
## 팀원
조성지, 안서인, 김태환, 조혜인

## 0. Overview

### Environment
- python3==3.10.13
- matplotlib==3.7.1
- numpy==1.23.5
- pandas==1.5.3
- scipy==1.11.3
- seaborn==0.12.2
- scikit-learn==1.2.2
- statsmodels==0.14.0
- tqdm==4.66.1

## 1. Competiton Info

### Overview

- 2007년부터 2023년도 6월까지의 서울특별시 내 아파트 데이터를 활용하여 2023년도 7~9월 약 3개월의 아파트 가격을 예측하는 경진대회
### Timeline
- 2024-11-11: 데이터 EDA 및 아이데이션 회의 진행(Model, Feature-Sellection, Feature-Engineering ...)
- 2024-11-12: 모델 학습 및 인사이트 공유
- 2024-11-13: 모델 개선 및 인사이트 공유
- 2024-11-14: 최종 결과물 제출

## 2. Components

### Directory

- _Insert your directory structure_

```
├── code
│   ├── Main
│   │   └── EDA.ipynb
│   │   └── LightGbm-v1.ipynb
│   │   └── LightGbm-v2.ipynb
│   │   └── RandomForest.ipynb
│   ├── train.py
│   └── Make_Output
│       └── FirstSubmmit.ipynb
│       └── SecondSubmmit.ipynb
│       └── ThirdSubmmit.ipynb
│       └── ForthSubmmit.ipynb
│
├── data
│   ├── data_outside
│   │   └── subway.csv
│   └──train.csv
│   └──test.csv
└── output
    └── FirstSubmmit.csv
    └── SecondSubmmit.csv
    └── ThirdSubmmit.csv
    └── ForthSubmmit.csv
```

## 3. Data descrption
### Dataset overview
- _Train Data: (1118822, 52)_
- _Test Data: (9272, 51)_
- _Test Data는 타겟값 미포함_
- _Train Data, Test Data의 시계열 데이터 겹치지 않음_
- _Null 데이터 상당수 포함되어 있어 전처리 필요_

### EDA
1. 51개의 Featrue 확인하며 결측치와 Featrue 각각의 의미를 파악
2. 전체 데이터 중 70% 이상의 결측치를 가진 컬럼 다수 존재
   ![image](https://github.com/user-attachments/assets/c1d82804-b401-40d5-aea3-4ed824a79276)
3. 23년 Target 예측을 해야하나 21~23년 데이터의 샘플 수 매우 적은 것 확인
   ![image](https://github.com/user-attachments/assets/e5c969c3-7e26-47f3-9ff0-07a333ce1b2c)
4. 계약년도와 Target 값의 시계열성 체크
   ![image](https://github.com/user-attachments/assets/cef5a739-c077-4793-b510-a77ed7b0e492)
5. 최근 년도 Target 값이 이전 기간 대비 높은 분산을 보이는 것 확인
   ![image](https://github.com/user-attachments/assets/ba35ebf6-0631-448f-aae5-7fa446e81220)



### Data Processing
1. 약 112만개의 샘플 중 결측치가 80만 개 이상 존재하는 37개의 Feature와 ' ', '-' 등 실질적인 결측치로 구성되어 있는 3개 Feature 삭제
2. 아파트명과 도로명 컬럼을 추가적으로 탐색하여 아파트명 결측치 2126개를 도로명으로 대체, 도로명 내 공백, 5개글자 미만 행은 Null값으로 대체
3. 계약년도와 Target값 사이의 시계열성을 보이는 것을 확인 -> 계약년도 + 계약일 컬럼을 합쳐 '계약날짜' 컬럼 생성
4. 좌표X, 좌표Y 컬럼 상당수 결측치가 있어 네이버지도 API를 사용하여 결측치 보간 진행
5. 지하철역 위치 정보를 가져와 X, Y 좌표와 매칭, 거리를 계산하여 역세권 여부 컬럼 생성 (500미터 미만 = 제1역세권, 1키로미터 미만 = 제2역세권)
6. 21~23년 데이터 증강 (약간의 노이즈 추가) 
7. 모델에 추가 시계열성 정보 제공을 위해 거래량의 7, 14, 30, 60, 90일 이동평균선 컬럼 생성
8. 거래량 변동성 지표 생성: 단위 기간별 거래량의 표준편차, 장기 추세 확인을 위해 직전년도 대비 거래량 변화율율

## 4. Modeling
### Model descrition

- 사용한 모델: Light Gbm
- 선택과정: 데이터 샘플수가 많고 특정 컬럼과 Target 변수간의 뚜렷한 상관관계가 보이지 않는 다차원 데이터이기 때문에 여러 패턴을 학습할 수 있는 앙상블 계열의 모델을 사용하는 것이 적절하다고 판단. 역할을 분담하여 Bagging 방식의 Random Forest모델과 Boosting 방식의 Light Gbm 모델을 학습하였고 Light-Gbm의 Validation, Test 성과 우수하여 최종적으로 Light Gbm 모델로 선정

### Modeling Process

1. EDA 진행
2. 데이터 전처리 및 파생변수 생성
3. 데이터 분할: train, test 세트 8:2,  random-state=10으로 통일
4. 모델학습
5. valid 세트, test 세트 성능 체크
6. 피팅: 하이퍼파라미터 튜닝, 변수조정, 파생변수 추가 등
7. 성능체크 및 추가피팅

## 5. Result
### Leader Board

![image](https://github.com/user-attachments/assets/3bd0d654-e8ce-4949-b976-ca9a2702bd89)


### Presentation



## etc

### Meeting Log
https://www.notion.so/5-be90a55ea6504d81b0aea6bc6a92b019
