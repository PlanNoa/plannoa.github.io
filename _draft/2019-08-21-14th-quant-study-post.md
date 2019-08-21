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

HAR 모델의 주요 장점은 간단하고 추정하기 쉬움에도 불구하고 데이터의 특징을 재현할 수 있다는 것이고, HEAVY 모델의 주요 장점은 근접 조건부 분산과 종가의 조건부 기대치를 모두 모델링하는 것이다. 또 HEAVY 모델은 모멘텀과 평균 회귀 효과를 가지고, 변동성 수준에서 구조적 중단에 신속히 조정된다.

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

   1. 인자와 사용처
   2. 출력
   3. 예시


**유동성**

1. Matching trades and quotes
2. Inferred trade direction
3. liquidity measures



