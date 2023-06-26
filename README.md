# 광고 지출 및 ROI 예측

[Snowflake Summit Opening Keynote](https://events.snowflake.com/summit/agenda/session/849836)에서 선보인 것과 동일한 데모입니다. Snowpark For Python 및 Streamlit을 사용하여 구축되었습니다. 질문 및 피드백은 <dash.desai@snowflake.com>으로 문의하십시오.

## 개요

이번 워크샵에서는 Snowpark for Python 및 scikit-learn을 사용하여 검색, 비디오, 소셜 미디어 및 이메일을 포함한 여러 채널에 걸쳐 가변 광고 지출 예산의 미래 ROI(투자 수익)를 예측하는 선형 회귀 모델을 교육합니다. 세션이 끝날 무렵에는 서로 다른 할당된 광고 지출 예산의 ROI를 시각화하는 대화형 웹 애플리케이션이 배포됩니다. 참고: 함께 제공되는 슬라이드는 [여기](https://github.com/Snowflake-Labs/snowpark-python-demos/blob/77f54633f850c66053dfa055c82a7fc6dec8deca/Advertising-Spend-ROI-Prediction/Snowpark%20for%20Python%20And%에서 찾을 수 있습니다. 20Streamlit%20ML%20Workshop.pdf).

워크숍 하이라이트:

* Snowpark 및 ML용으로 선호하는 IDE(예: Jupyter, VSCode) 설정
* Snowpark DataFrames를 사용하여 데이터를 분석하고 데이터 엔지니어링 작업을 수행합니다.
* 유지 관리 또는 오버헤드가 거의 없는 선별된 Snowflake Anaconda 채널에서 오픈 소스 Python 라이브러리 사용
* Python 저장 프로시저를 사용하여 Snowflake에 ML 모델 교육 코드 배포
* 추론을 위해 Scalar 및 Vectorized Python 사용자 정의 함수(UDF) 생성 및 등록
* Snowflake 작업을 생성하여 모델의 (재)학습을 자동화합니다.
* 사용자 입력을 기반으로 새로운 데이터 포인트에 대한 실시간 추론을 위해 Scalar UDF를 사용하는 Streamlit 웹 애플리케이션 생성

모두 잘 진행되면 브라우저 창에 다음 앱이 표시됩니다.

https://user-images.githubusercontent.com/1723932/175127637-9149b9f3-e12a-4acd-a271-4650c47d8e34.mp4

## 전제 조건

* [눈송이 계정](https://signup.snowflake.com/)
   * 하나의 브라우저 탭에서 계정으로 생성된 관리자 자격 증명(ORGADMIN 권한이 있는 역할)을 사용하여 [Snowflake 계정](https://app.snowflake.com/)에 로그인합니다. 워크숍 동안 이 탭을 열어 두십시오.
     * 왼쪽 패널에서 **청구**를 클릭합니다.
     * [약관 및 결제](https://app.snowflake.com/terms-and-billing)를 클릭하세요.
     * 워크숍을 계속하려면 약관을 읽고 동의하세요.
   * SYSADMIN 역할로
     * [웨어하우스](https://docs.snowflake.com/en/sql-reference/sql/create-warehouse.html), [데이터베이스](https://docs.snowflake.com/en/sql) 생성 -reference/sql/create-database.html) 및 [스키마](https://docs.snowflake.com/en/sql-reference/sql/create-schema.html)

## 설정

```sql
   역할 SYSADMIN을 사용하십시오.
```

### **1단계** -- 테이블 생성, 데이터 로드 및 설정 단계

(Snowpark_For_Python.ipynb) 노트북을 실행할 때 예제 데이터베이스, 테이블, 단계 및 파일 형식이 구성 및 생성됩니다.

이러한 단계는 공개적으로 액세스 가능한 S3 버킷에서 호스팅되는 데이터를 사용하도록 구성됩니다.

Streamlit 애플리케이션의 경우 지난 6개월의 예산 할당 및 ROI를 보유하는 BUDGET_ALLOCATIONS_AND_ROI 테이블을 생성해야 합니다.

```sql
   테이블 생성 또는 교체 SNOWPARK_ROI_DEMO.AD_DATA.BUDGET_ALLOCATIONS_AND_ROI(
     월 varchar(30),
     SEARCHENGINE 정수,
     소셜 미디어 정수,
     비디오 정수,
     이메일 정수,
     ROI 플로트
   );

   BUDGET_ALLOCATIONS_AND_ROI에 삽입(월, 검색 엔진, 소셜 미디어, 비디오, 이메일, ROI)
   가치
   ('1월',35,50,35,85,8.22),
   ('2월',75,50,35,85,13.90),
   ('3월',15,50,35,15,7.34),
   ('4월',25,80,40,90,13.23),
   ('5월',95,95,10,95,6.246),
   ('6월',35,50,35,85,8.22);
```

## 노트북 및 Streamlit 앱

### **1단계** -- 저장소 복제

* `git clone https://github.com/Snowflake-Labs/snowpark-python-demos` 또는 `git clone git@github.com:Snowflake-Labs/snowpark-python-demos.git`

* `cd 광고-비용-ROI-예측`

### **2단계** -- Conda 환경 생성 및 활성화

* 참고: miniconda 설치 프로그램은 다음에서 다운로드할 수 있습니다.
https://conda.io/miniconda.html. 또는 Python 3.8과 함께 다른 Python 환경을 사용할 수 있습니다.

* `conda create --name snowpark -c https://repo.anaconda.com/pkgs/snowflake python=3.8`

* `콘다 활성화 스노우파크`

### **3단계** -- Conda 환경에서 Snowpark for Python, Streamlit 및 기타 라이브러리 설치

* `conda 설치 -c https://repo.anaconda.com/pkgs/snowflake snowflake-snowpark-python pandas 노트북 scikit-learn cachetools streamlit`

### **4단계** -- Snowflake 계정 세부 정보 및 자격 증명으로 [connection.json](connection.json) 업데이트

* 참고: **account** 매개변수의 경우 [계정 식별자](https://docs.snowflake.com/en/user-guide/admin-account-identifier.html)를 지정하고 snowflakecomputing을 포함하지 마십시오. com 도메인 이름. Snowflake는 연결을 만들 때 이를 자동으로 추가합니다.

### **5단계** -- ML 모델 교육 및 배포

* 터미널 창에서 이 노트북을 다운로드한 폴더로 이동하고 명령줄에서 `jupyter notebook`을 실행합니다.
* [Jupyter 노트북](Snowpark_For_Python.ipynb)
   * 참고: Jupyter 노트북(Python) 커널이 ***snowpark***로 설정되어 있는지 확인하세요.

노트북은 다음을 수행합니다...

* 탐색적 데이터 분석(EDA) 수행
* 모델 학습을 위한 기능을 생성하고 Snowflake 테이블에 기록
* ML 모델 교육을 위한 Stored Proc 생성 및 모델을 Snowflake 단계에 업로드
* Stored Proc을 호출하여 모델 훈련
* 매개변수로 전달된 새 데이터 포인트에 대한 추론을 위해 모델을 사용하는 스칼라 및 벡터화된 사용자 정의 함수(UDF) 생성
   * 참고: Scalar UDF는 사용자 입력을 기반으로 새로운 예산 할당에 대한 실시간 추론을 위해 아래 Streamlit 앱에서 호출됩니다.
* 모델의 (재)학습을 자동화하는 Snowflake 작업 생성

### **6단계** -- Streamlit 앱 실행

* 터미널 창에서 이 파일을 다운로드한 폴더로 이동하고 `streamlit run Snowpark_Streamlit_Revenue_Prediction.py`를 실행하여 [Streamlit 앱](Snowpark_Streamlit_Revenue_Prediction.py)을 실행합니다.

* 모두 잘 진행되면 브라우저 창에 다음 앱이 표시되어야 합니다.

https://user-images.githubusercontent.com/1723932/175127637-9149b9f3-e12a-4acd-a271-4650c47d8e34.mp4