---

layout: post
title: "Quantitative Trading With R-14"
subtitle: "변동성 예측"
categories: data
tag: quant
comments: true
---

**변동성 예측**

변동성을 예측하는 HAR 모델과 HEAVY 모델은 조건부 모델링을 하지만 다른 접근법을 취한다. HAR 모델이 시장의 open-to-close 변동을 예측하는 반면, HEAVY 모델은 두 개의 식을 가진다. 하나는 HAR 모델처럼 종가 변동을, 다른 하나는 근접 변동을 예측한다.

HAR 모델의 주요 장점은 간단하고 추정하기 쉬움에도 불구하고 데이터의 특징을 재현할 수 있다는 것이고, HEAVY 모델의 주요 장점은 근접 조건부 분산과 종가의 조건부 기대치를 모두 모델링하는 것이다. 또 HEAVY 모델은 모멘텀과 평균 회귀 효과를 가지고, 변동성 수준에서 구조적 중단을 신속하게 진행한다..

1. HAR 모델

   HAR 모델은 Corsi 2009b가 제안한 변동성 예측 모델이다. 첫 단계로, 일일 수익률은 일일 변동성을 예측하는데 사용된다. 다음으로 실현된 변동성은 다른 지평에서 지연된 선형 함수의 파라미터로 사용된다. 일반적으로 실현 변동성을 예측하기 위해 일별, 주별, 월별 지평이 선택된다.

   Andersen et al.의 분석에 따르면 가격 수준의 점프에서 발생하는 변동성은 실현 변동의 연속 구성 요소에서 발생하는 변동에 비해 덜 지속적이며, 연속적인 이동에서 점프 이동을 분리하면 더 나은 예측 결과를 얻을 수 있다. 이들은 이 아이디어를 모델링하는 두 가지 방법을 제안했으며, 둘 모두 `highfrequency` 패키지로 구현할 수 있다. 

    `highfrequency` 패키지의 `harModel` 함수는 HARRV 모델을 구현한다. r(t)를 t일의 일일 수익률이고, 1부터 M(number of intraday returns per day)이면 실현 변동성은 

   ![](https://imgur.com/snYqdre.png)

   이다.

   1. HARRV 모델

      가장 간단한 모델이다. t가 1부터 T까지 증가하고, 일반적으로 h = 1이면 모델은 다음과 같다.

      ![](https://imgur.com/51Rmvzz.png)

   2. HARRVJ 모델

      일중 가격에 점프가 있는 경우 기존의 변동성 측정은 점프가 변동성에 끼친 영향에 따라 결정된다. 이 모델은 매끄러운 가격의 변동성만을 추정하기 위해 만들어졌다. 점프의 기여도를 J(t) = max[RV(t) − BPV(t), 0 ]이라 하면 모델은 다음과 같다.

      ![](https://imgur.com/71J4ps5.png)

   3. HARRVCJ 모델

      위에서 언급한 논문에서, 작은 점프들은 측정 오류 혹은 변동 프로세스의 일부로 생각하고 큰 값만 점프로 처리하는 것이 바람직할 수 있다고 말했다. 그러므로 Z(t)를 

      ![](https://imgur.com/eambqCl.png)

      로 가정하고,  TQ(t)를

      ![](https://imgur.com/8hrNfcG.png)

      로, µ(p) 를

      ![](https://imgur.com/L90hnqE.png)

      로, J(t)와 C(t)를 

      ![](https://imgur.com/TFSnI3D.png)

      ![](https://imgur.com/NsdhgWx.png)

      로 가정하면 모델은 다음과 같다.

      ![](https://imgur.com/JqtqjAr.png)

   4. 인자와 사용처

       `harModel` 함수는 일일 수익률을 포함한 xts 데이터를 입력받는다. `type` 인자로 사용할 함수의 종류를 지정할 수 있다. 일일 수익률을 기반으로 일일 실현 측정 값이 예측된다. `RVest` 인자로 일일 통합 분산과 연속 변동성을 추정하는 추정치를 설정할 수 있다. 이외에도 지평을 정할 때 사용하는 `periods` 와 `periodsJ` , `leverage`, 점프의 기여도를 계산하는 `jumptest` 와 `alpha`,  선형 모델의 변수를 조작하는 `transform` 같은 인자들이 있다.

   5. 예시

      먼저 Dow Jones Industrial 평균에 대한 일일 실현 변동성을 로드하고 2008 만 선택한다.

      ```R
      data(realized_library)
      DJI_RV = realized_library$Dow.Jones.Industrials.Realized.Variance
      DJI_RV = DJI_RV[!is.na(DJI_RV)]
      DJI_RV = DJI_RV['2008']
      ```

      두 번째로, HAR 모델을 계산한다. 모델의 출력은 S3 오브젝트다. HAR 모델은 결국 특별한 선형 모델이기 때문에, 출력은 선형 모델의 일반적인 클래스인 lm이다.

      ```R
      x = harModel(data=DJI_RV , periods = c(1,5,22), RVest = c("rCov"),                		       type="HARRV",h=1,transform=NULL)
      class(x)
      [1] "harModel" "lm"      
      x
      Model:
      RV1 = beta0  +  beta1 * RV1 +  beta2 * RV5 +  beta3 * RV22
      
      Coefficients:
          beta0      beta1      beta2      beta3  
      4.432e-05  1.586e-01  6.213e-01  8.721e-02  
      
      
          r.squared  adj.r.squared  
             0.4679         0.4608  
      
      summary(x)
      Call:
      "RV1 = beta0  +  beta1 * RV1 +  beta2 * RV5 +  beta3 * RV22"
      
      Residuals:
             Min         1Q     Median         3Q        Max 
      -0.0017683 -0.0000626 -0.0000427 -0.0000087  0.0044331 
      
      Coefficients:
             Estimate Std. Error t value Pr(>|t|)    
      beta0 4.432e-05  3.695e-05   1.200   0.2315    
      beta1 1.586e-01  8.089e-02   1.960   0.0512 .  
      beta2 6.213e-01  1.362e-01   4.560 8.36e-06 ***
      beta3 8.721e-02  1.217e-01   0.716   0.4745    
      ---
      Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
      
      Residual standard error: 0.0004344 on 227 degrees of freedom
      Multiple R-squared:  0.4679,	Adjusted R-squared:  0.4608 
      F-statistic: 66.53 on 3 and 227 DF,  p-value: < 2.2e-16
      
      plot(x)
      ```

      ![](https://imgur.com/BWeEXbl.png)

      `j=highfrequency` 를 사용해 복잡한 HAR 모델도 쉽게 사용할 수 있다. HARRVCJ 모델의 예시 코드다.

      ```R
      data("sample_5minprices_jumps"); 
      data = sample_5minprices_jumps[,1];
      data = makeReturns(data); #Get the high-frequency return data
      x      = harModel(data, periods = c(1,5,10), periodsJ=c(1,5,10), 
                        RVest = c("rCov","rBPCov"), type="HARRVCJ",
                        transform="sqrt");
      x
      
      Model:
      sqrt(RV1) = beta0  +  beta1 * sqrt(C1) +  beta2 * sqrt(C5) +  beta3 * sqrt(C10) 
             +  beta4 * sqrt(J1) +  beta5 * sqrt(J5) +  beta6 * sqrt(J10)
      
      Coefficients:
         beta0     beta1     beta2     beta3     beta4     beta5  
       -0.8835    1.1957  -25.1922   38.9909   -0.4483    0.8084  
         beta6  
       -6.8305  
      
          r.squared  adj.r.squared  
             0.9915         0.9661  
      ```

2. HEAVY 모델

   Shephard and Sheppard 2010은 고빈도 데이터에서의 변동성 모델링을 위해 HEAVY 모델을 도입했다. 위에서도 언급했지만 이 모델은 두 가지 방정식을 가지는데, 하나는 종가 변동의 조건부 기대치를 모델링하고, 다른 하나는 근접 조건부 분산을 모델링한다. F(t-1)HF로 고빈도 데이터(하루 종일 수익률)를 나타내고, F(t-1)LF로 저빈도 데이터(일일 수익률)을 나타낸다.

   일일 수익률을 다음과 같이 정의한다. ![](https://imgur.com/lBVHih7.png)

   일일 데이터를 기반으로 일일 실현 측정값을 계산할 수 있다. ![](https://imgur.com/0i1X881.png)

   T를 샘플의 총 일수라고 하면, HEAVY 모델은 다음과 같다. ![](https://imgur.com/Y6Pp6tU.png)

   위의 식이 근접 조건부 분산을, 아래의 식이 종가 변동 조건부 기대치를 모델링한다. 행렬 표기법에서 모델은 다음과 같다.

    ![](https://imgur.com/sY6yyIR.png)

   1. 인자와 사용처

      함수 `heavyModel` 은 (T x K) 데이터 행렬을 입력으로 받는다. T는 샘플의 총 일수이다. 일반적인 HEAVY 모델에서는 K = 2이고, 첫번째 열은 일일 수익률의 제곱이고, 두 번째 열은 일일 실현 측정값이다.

      위의 행렬 표기법 식을 보면 α와 β를 인자로 가지는 두 개의 행렬이 포함된다. 이 행렬은 일반적인 HEAVY 모델이 더 큰 행렬을 받을 수 있는 특별한 모델이라는 것을 보여준다. `heavyModel` 의 인자 `p` 와 `q` 로 이 두 행렬을 각각 지정할 수 있다.

      1. 어떤 파라미터가 non-integer 로서 추정되어야 하는지.
      2. 이 모델에 얼마만큼의 지연이 포함되어야 하는지.

      더 쉽게, `p`는 모델 혁신을 위한 지연 길이를 가지는 (K x K) 행렬이다. 행렬의 위치 (i, j)는 데이터 j열의 혁신에 대한 모델의 방정식 i의 지연 수를 나타낸다. 

      일반적인 HEAVY 행렬에서 `p`는 다음과 같이 나타난다.​	 ![](https://imgur.com/OJpn2bs.png)

      `p`와 비슷하게 `q`는 조건부 분산에 대한 지연 길이를 가지는 (K x K)의 행렬이다. 행렬의 위치 (i, j)는 j열에 해당하는 조건부 분산에 대한 모형의 방정식 i에서 지연 수를 나타낸다.

      일반적인 `q`는 다음과 같다. ![](https://imgur.com/F5QHrsI.png)

      최적화에 사용될 시작 값은 `startingvalues` 인수로 설정할 수 있다. `LB`와 `UB`는 파라미터의 범위를 설정한다. 기본 값으로 0과 Inf가 설정되어 있다. To specify how the estimation should be initialized you can use the `backcast` argument, which will be set to the unconditional estimates by default. 마지막으로 `compconst` 인자는 ω의 추정 방법을 나타내는 bool 값이다. TRUE면 ω는 최적화되고, FALSE 면 변동성이 추정되고 ω는 1에서 모든 α 및 β의 합계에 무조건 분산을 곱한 값을 뺀 값이 된다.

   2. 출력

      `heavyModel`의 출력값은 리스트다. 흥미로운 점은 리스트 내부의 `estparams`에는 매개 변수 추정치 및 이름이 포함된 행렬이 포함된다. 매개 변수는 ω 추정치, 최대 max (p)> 1 인 경우 가장 최근 지연이있는 0이 아닌 α의 추정치, 0이 아닌 경우에 대한 추정치 max (q)> 1 인 경우 가장 최근 지연이 발생한 β 추정치 순서로 나타난다. `loglikelihood` 리스트가 전체 로그 가능성을 나타내므로 사용자는 일일 로그 가능성을 평가하는 사람에게 조금 더 통찰력을 제시한다. 리스트 `convergence`는 최적화 수렴에 대한 정보를 나타낸다.

      리스트 `condvar`은 조건부 분산을 담는 (T x K) xts 오브젝트다.

   3. 예시

      먼저 데이터 realized_library를 로드한다. 데이터에서 일일 수익률과 일일 실현 측정값을 선택한다. `heavyModel`의 입력 행렬은 첫 번째 열에 일일 수익률을, 두 번째 열에서 일일 실현 측정값을 가진다. 추가로 일일 수익률의 분산과 평균 실현 커널에 역추론을 추가로 설정했다.

      ```R
      # Implementation of the heavy model on DJIA:
      data("realized_library");
      returns  =  realized_library$Dow.Jones.Industrials.Returns; 
      rk       =  realized_library$Dow.Jones.Industrials.Realized.Kernel; 
      returns  = returns[!is.na(rk)];  rk = rk[!is.na(rk)]; # Remove NA's 
      data     = cbind( returns^2, rk ); 
      backcast = matrix( c(var(returns),mean(rk)) ,ncol=1);  
      
      startvalues = c(0.004,0.02,0.44,0.41,0.74,0.56); # Initial values 
      output = heavyModel( data = as.matrix(data,ncol=2), compconst=FALSE, 
                           startingvalues = startvalues, backcast=backcast); 
      output$estparams
                   [,1]
      omega1 0.01750506
      omega2 0.06182249
      alpha1 0.45118753
      alpha2 0.41204541
      beta1  0.73834594
      beta2  0.56367558
      ```

      ![](https://imgur.com/9RoUBvA.png)

      2008년 경제 대공황 시기의 급등을 확인할 수 있다.

**유동성**

1. 가격과 거래 매칭

   거래와 가격은 종종 다른 파일로 제공된다. 많은 연구와 특정 질문들에서는 거래와 가격을 병합할 필요가 있다. 거래와 가격 보고는 지연에 따라 달라질 수 있어 결코 간단한 작업이 아니다. `matchTradesQuotes` 함수는 가격과 거래를 매칭하는데 사용할 수 있다. 함수를 이용하기 위해서는 가격이 거래보다 빨리 등록되는 시간을 입력해야 한다. 기본값은 [Vergote 2005](http://highfrequency.herokuapp.com/#vergote2005)의 연구를 기반으로 2초가 책정되었다.

2. 거래 방향 예측

   많은 거래와 가격 데이터베이스는 개별 거래가 시장 구매인지 시장 판매 주문인지를 나타내지 않는다. `highfrequency` 는 `getTradeDirection` Lee-Ready 규칙을 사용해 거래 및 시세에 따라 거래 방향을 추론한다.

3. 유동성 측정

   `tqLiquidity` 함수를 사용해서 거래와 가격 데이터로 수많은 유동성 측정이 가능하다. `tqLiquidity` 의 인자는 밑의 표에서 볼 수 있다.

   ![](https://imgur.com/5Nl4zWl.png)

   아래는 예시 코드이다.

   ```R
   #Load data samples
   data("sample_tdata")
   data("sample_qdata")
   
   #Match the trade and quote data
   tqdata = matchTradesQuotes(sample_tdata,sample_qdata)
   
   #Display information in tqdata
   colnames(tqdata)[1:6]
   [1] "SYMBOL" "EX"     "PRICE"  "SIZE"   "COND"   "CORR"  
   colnames(tqdata)[7:12]
   [1] "G127"   "BID"    "BIDSIZ" "OFR"    "OFRSIZ" "MODE"  
   
   #Get the inferred trade direction according to the Lee-Ready rule
   x = getTradeDirection(tqdata)
   
   #Calculate the proportional realized spread:
   prs = tqLiquidity(tqdata,sample_tdata,sample_qdata,type="prs")
   
   #Calculate the effective spread:
   es = tqLiquidity(tqdata,type="es")
   ```



이 글 Quantitative Trading with R 시리즈는 책 [Quantitative trading with R](https://www.amazon.com/Quantitative-Trading-Understanding-Mathematical-Computational/dp/1137354070)을 보고 공부한 내용들입니다.