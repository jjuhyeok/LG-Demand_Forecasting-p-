# LG-Demand_Forecasting-P- [(Link)](https://dacon.io/competitions/official/236129/leaderboard)

## 🏆 Result
- **Public score 3rd** 0.60745 | **Private score 3rd** 0.58947




주최 : LG AI Research

주관 : DACON

규모 : 총 1400여명 참가

===========================================================================


===========================================================================
  
  

  

✏️
**대회 느낀점**
 - 이번 대회는 수요 예측에 대한 Task였습니다. 데이터는 15900여개의 제품의 2022년1월1일부터 2023년4월4일까지의 실제 일별 판매량과 실제 일별 총 판매금액, 브랜드의 연관키워드 언급량, 제품 특성의 메타 데이터로 구성되어 있었고  
   2023년 4월5일~4월25일까지의 총 21일(3주)간의 실제 판매량을 예측하는 대회였습니다.  
   실제 train 데이터에는 예측해야 하는 ID별로 제품 코드, 대분류, 중분류, 소분류, 브랜드 정보가 추가적으로 존재하였는데, 15900여개의 제품들을 대분류, 중분류, 소분류, 브랜드로 나누어서 추세, 주기성, 계절성과 같은 시계열성 정보를 확인해보았는데, 직관적으로 알기가 힘들었습니다.
   그래서 퓨리에 변환,Hierarchical Clustering, 유클리드 거리, cosine similarity 등으로 유사한 시계열성 정보를 가진 ID들을 분류하려 했지만, ID별로 워낙 판매량이 다 달라서 분류를 할 수 없었습니다.
   그래서 이러한 ID들을 각자 모델에 학습시켜 총 15900개의 모델을 만드는 것이, 정확한 예측을 할 수 있지 않을까? 라는 생각이 들었습니다.  
   하지만 15900개의 딥러닝 모델을 만드는 것은 시간이 오래걸리기 때문에, 최근 트랜스포머의 정확도에 의문을 가지며 등장한 LSTF Linear 모델들을 활용해보았습니다.(LSTF Linear, DLinear, NLinear)  
   하지만 ID들을 따로 따로 학습시켰을 때 정확도가 많이 올라가지 않았습니다.  
   그 이유가 무엇일까 하며 계속해서 EDA를 해 보았고, 판매량이 0일 때의 시점들에 주목하였습니다.  
   판매량이 0으로 라벨링이 되어있는 시점은, 실제로 팔리지 않았거나 수요가 있지만 공급이 되지 않을 때를 의미하기 때문에, 이러한 정보를 잘 살려야 하지 않을까 생각을 하였고, 0의 값을 impuation 하는 방법으로 접근을 하였습니다.  
   처음에는 모든 0의 값을 수요가 있지만 공급이 되지 않을 때로 가정하는 것은, 위험한 가정이라 생각을 하여 2022/01/01부터 2023/04/04 까지의 날짜 중 70%이하의 날이 0으로 구성되어야 수요가 있지만 공급이 되지 않을 때로 가정하였습니다.  
   그렇게 70%의 제품들을 필터링 후 0의 값들을 interpolate하여 선형 및 ffill, bfill로 값을 채워주었더니, 모델의 예측 정확도가 크게 상승하였습니다.  
   나의 가설이 맞아 떨어져서 정말 기분이 좋았던 순간이었고 팀장으로서의 역할을 제대로 하고 있는 것 같아 뿌듯했습니다.  
   이제 나의 imputation 방법을 develope 시키는 방법에 대해서 고민하였는데, 정확도를 평가하는 평가산식을 활용을 하였습니다.  
   이 TASK의 평가산식은 PSFA로 대분류 별 Pseudo 예측 정확도의 평균이었습니다.
   어떻게 활용했는지를 설명하기 전, PSFA에 대해서 간단히 설명을 하자면
   
   PSFA"라는 지표는 우리가 예측한 판매량이 얼마나 정확한지를 측정하기 위한 도구로 이 지표는 크게 두 부분으로 나뉘는데, 예시를 들어서 이해하기 쉽게 설명을 하면
1. 대분류별 Pseudo 예측 정확도 (PSFA Am) : 
 - 상품을 일정한 기준에 따라 여러 그룹으로 나눕니다. 예를 들어, 과일, 야채, 음료 등으로 나눌 수 있습니다.
 - 각 그룹 내에서 하루하루 얼마나 팔렸는지와 얼마나 팔릴 것이라고 예측했는지를 비교합니다.
 - 예를 들어, 사과가 하루에 100개 팔렸다고 해봅시다. 만약 우리 예측이 110개라면, 그 차이는 작은 편입니다. 하지만 예측이 200개라면, 그 차이는 큰 편이죠.
 - 이 차이를 각 상품과 날짜별로 계산해서, 그룹 전체의 평균 차이를 구합니다.

2. 전체 Pseudo 예측 정확도 (PSFA)
 - 이제 모든 그룹 (과일, 야채, 음료 등)에 대해 계산한 평균 차이를 하나로 합쳐 전체적인 예측의 정확도를 측정합니다.

이 지표는 '어느 상품이나 날짜에서 예측이 얼마나 빗나갔는지'를 알려주는 평가지표이고, 더 나아가, 많이 팔린 상품에서의 예측 오차는 덜 팔린 상품에서의 예측 오차보다 더 중요하게 취급 됩니다. 왜냐하면 많이 팔린 상품이 비즈니스에 더 큰 영향을 미치기 때문이라고 생각할 수 있기 때문입니다.  
이 평가 지표에 따르면 만약 실제 판매량이 0이 되면 오차가 (∣0−p∣) / (max(0,p)) 되는데, 이때 예측값이 100이든 100000이든 평가지표에는 영향을 끼치지 않습니다.  
그렇기 때문에 이 전에 수행했던 imputation에서의 필터링이 과연 의미가 있을까? 라는 의문이 들기 시작하였고, train데이터 기간에 0이 엄청 많이 나온 ID들은 실제 판매량도 0이 될 가능성이 높고 모델도 실제로 0의 값으로 많이 예측을 할텐데,  
평가지표에 의해 수요가 있지만 공급이 없는 경우보다, 공급이 더 중요한 평가지표라 판단이 되어. 미리 수요에 대한 공급을 준비해 주기 위해 0이 아닌 이 값들을 train 기간의 0을 제외한 중앙값으로 대체를 해 주었습니다.  
중앙값으로 대체한 이유는 할인 기간같은 특별 세일 기간에 판매량이 엄청나게 증가하는 경우가 있어, 평균으로 하면 안될 것 같아 0을 제외한 중앙값을 사용하였습니다.

마지막으로 예측값들에 대한 강건함을 늘리기 위해, 예측값들의 평균으로 21일간의 값을 치환해주엇습니다.

이러한 모델링으로 강건한 결과를 가져왔습니다.

이러한 경험을 통해, 개개인에 대한 세부적인 데이터 분석이 불가능할 때, 어떻게 접근해야 할 지와 평가지표를 어떻게 이해해서 모델링에 적용시킬 지에 대해서 깨달았습니다.

===========================================================================

# 🚨 문제 발생과 해결 방안

### 문제
- 평소 LG AI Research에서 업로드하는 AI 관련 영상을 꾸준히 보는데, 현재 Task인 수요예측 관련한 영상도 본 경험이 존재하였다.   
  양진석 리더님이 진행해주신 영상에서는   
  ``` TFT, TCN, DeepAR, N-BEATS, TabNet, LightGBM ```과 같은 모델들을 ``` Base Model ```로 설정하셨다고 하셔서,  
  바로 나는 먼저 Temporal Fusion Transformer 모델을 현재 Task에 적용을 해 보았다.  
  하지만 정확도 성능이 정말 처참하였다.  
  분명 TFT 모델이 SOTA라고 하셨는데, 왜 정상적으로 작동을 하지 않는거지? 라는 의문점을 품의며 내가 하이퍼파라미터 튜닝을 너무 안한건가? 아니면 코드가 잘못된건가? 하며  
  며칠간 Optuna를 이용한 튜닝도 해보고, 내가 작성했던 코드도 수십번을 수정을 하며 결과를 보았다.  
  하지만 성능 향상은 존재하지 않았다.  
  그래서 TFT 모델 이외의 언급된 DeepAR, TCN, LGBM, XGB 등등의 모델을 엄청 사용해 보았지만 똑같이 ``` 성능이 나오지 않았다. ```   


### 해결
  나는 대회를 정말 많이 나가면서, 정말 다양한 데이터들에 대한 모델의 경험적 지식을 쌓았다.
  물론 완벽한 것은 아니지만, 이 데이터에는 이 모델이 좋겠다라고 하며 모델에 먼저 학습 후 예측을 시켜본다
  . ex) 전력 사용량 데이터는 시계열적 특성이 데이터에 존재하니, 시계열 데이터에 좋은 XGBoost를 먼저 사용해보자!
  그 후 이제 두가지로 나누어 지는데,
  1. 성능이 잘 나올 때
  2. 성능이 잘 나오지 않을 때
  현재 2번의 상황이다.
  1번은 나의 노하우니 여기서 밝히지는 않겠고 만약 궁금하신 분이 계신다면 따로 연락을 주시면 좋겠다.
  2번의 상황일 때는, EDA에 시간을 정말 많이 쏟는다.
  ``` 왜 이 모델이 이 데이터에 잘 적용이 안되는거지? 하며 문제를 찾으려고 노력한다. ```
  이번 상황에서는 크게 2개의 문제가 존재했다.
  첫 번째는 각 제품 ID가 15900여개지만 각자의 추세, 주기성, 계절성이 모두 다르다는 점이고
  두 번째는 0의 값이 너무 많다는 문제이다.
  그래서 나는 Custom Imputation 방법과 전체 및 개별의 제품에 대한 모델링 결과를 앙상블 하는 방법을 선택하여,
  이 ``` 문제를 해결하였고 총 1400여명이 참여한 대회에서 정확도 top3의 결과 ```를 가져왔더.


```df_all.isnull().sum() ``` 을 통해 null 값이 있는 것을 확인했고, 다시 데이터 정제를 했다.  그리고 ```df_all.isin([-999.0]).sum() ``` 을 통해 -999 값도 있는지 확인 후, 있다면 해당 행을 삭제했다. 

<br>



## Data Info

* 각각의 칼럼에 대한 설명
* Features 
  * 아이디 / 날짜 / 요일 / 시간대 / 차로 수 / 도로 등급 / 중용구간 여부 / 연결로 코드 / 최고 속도 제한 / 통과 제한 하중 / 통과 제한 높이 / 도로 유형 / 시작 지점 위도 / 시작 지점 경도 / 도착 지점 위도 / 도착 지점 경도 / 시작 지점 회전 제한 여부 / 도착 지점 회전 제힌 여부 / 도로 명 / 시작 지점 명 / 도작 지점 명 / 통과 제한 차량
* Target
  * 평균 속도 (KM)
  
### Train Set

* ID : 실제 판매되고 있는 고유 ID(15900개) 
* 제품 : 제품 코드
* 대분류 : 제품의 대분류 코드
* 중분류 : 제품의 중분류 코드
* 소분류 : 제품의 소분류 코드
* 브랜드 : 제품의 브랜드 코드
* 2022-01-01 ~ 2023-04-04 : 실제 일별 판매량
단, 제품이 동일하여도 판매되고 있는 고유 ID 별로 기재한 분류 정보가 상이할 수 있음
즉 고유 ID가 다르다면, 제품이 같더라도 다른 판매 채널

  
---
🚨 문제 발생과 해결 방안
문제
딥러닝 모델을 학습시키는데 loss 값이 nan으로 나온다.
Epoch 1/10
  6800/10406 [=============>...........] - ETA:60s - loss: nan
해결
데이터에 null 값이 들어있는 것이 원인이었다. df_all.isnull().sum()  을 통해 null 값이 있는 것을 확인했고, 다시 데이터 정제를 했다. 그리고 df_all.isin([-999.0]).sum()  을 통해 -999 값도 있는지 확인 후, 있다면 해당 행을 삭제했다.


## Process

**1. EDA** 
  * 평균 도로 속도가 40인 도로의 평균 속도가 40을 초과하는 제일 높은 속도임을 확인, 데이터 불균형 확인
  * 2022년 7월 이전과 이후로 평균 target값의 차이가 심함을 파악
  * 각 도로의 특징 별로 target값의 차이가 심함을 파악
  * 각 도로마다 target의 이상값이 존재 -> 사고 or 폭주로 생각
  * 요일, 시간대별로 target 값 차이 존재
  
**2. Feature Engineering** 
  * 시간 feature들은 inherently cyclical 하다는 특징을 활용하기 위해 sin/cos 변환
  * 특정 피처들의 target 값의 평균으로 파생 변수 생성
  * 위,경도 값을 기준으로 clustering 하여 파생 변수 생성 -> 특정 도로는 인접한 도로읙 교통량에 영향을 미칠것이란 가설
  * 제주도 관광지 외부데이터를 활용해 위,경도 값을 기준으로 특정 거리 내의 관광지들을 count하는 파생 변수 생성 -> 관광지가 많은 곳은 교통량이 혼잡할 것이다 라는 가설
  * 공휴일 외부데이터를 활용해 공휴일 당일, 전,후날 파생변수 생성 -> 공휴일에는 교통량이 많을 것이라는 가설
  * 이외의 feature engineering을 통해 총 48개의 피처로 학습 진행

**3. Modeling**

  * XGBoostRegressor 모델 사용
  * Optuna를 활용하여 하이퍼파라미터 튜닝
  * 교차 검증 활용(stratified k-fold cross validation)
  * target 값은 정수형으로 이루어져있다는 사실에 기반하여 예측값을 round 

**추가 시도 사항** (최종 결과 도출에는 비사용)
  * IQR * 1.5 2.0 등등 값을 변화해가며 폭주 or 사고라고 생각하는 이상치들을 drop -> 정확도가 오히려 낮아짐
  * 3 sigma 법칙을 이용해 폭주 or 사고라고 생각하는 이상치들을 drop -> 정확도가 오히려 낮아짐
  * Stacking 기법 -> 마지막에 시도하였지만 시간이 부족해 base model의 충분한 하이퍼파라미터 튜닝을 하지 못해 성능이 약간 하락
    -> 대회 종료 후 다시 확인하였더니 Stacking 기법이 더 점수가 좋았음
  * 관광지 관련 파생변수 생성할 때 카카오맵 API를 사용하였는데 DataLeakage로 최종 제출에는 사용하지 않음 -> 제일 성능이 좋았음
  * 다양한 KFold K 변경


  
