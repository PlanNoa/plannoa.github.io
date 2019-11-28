---
layout: post
title: "Quantitative Trading With R-1"
subtitle: "확률 변수, 확률, 확률 분포, 베이즈학파"
categories: data
tag: quant time-series
comments: true
---



**확률 변수**

표본 공간의 원소를 숫자로 매핑한 것.

일반적으로 대문자로 표기한다. 표본 공간은 Ω(빅 오메가), 원소는  ω(스몰 오메가).

ex) ***X(ω) = r***, 단 ***r ∈ R***.



**확률**

모든 가능한 확률 변수에 부여한 가중치 => 확률.

0은 절대 일어나지 않음을, 1은 항상 일어남을 의미.

***E***가 표본 공간 Ω의 사건일 때 ***E***의 확률은 0 이상의 실수 값을 가진다.

표본 공간의 모든 사건들의 확률의 총합은 1이다.

상호 배타적인 사건들의 확률 합은 개별 사건들의 확률 합과 같다.



**확률 분포**

확률 분포는 확률 변수 값에 확률을 지정한 함수이다.

이산 값을 가지는 확률 변수에 확률 질량 함수를 사용한다.

ex) 동전 던지기 실험.

​       ***P(Head) = P(X = 1) = 0.5***

​       ***P(Tail) = P(X = -1) = 0.5***

```R
plot(c(-1, 1), c(0.5, 0.5), type = "h", lwd = 3,
    xlim = c(-2, 2), main = "Probability mass function of coin toss",
    ylab = "Probability", xlab = "Random Variable", cex.main = 0.9)
```

 ![cointoss](https://imgur.com/7r8FGrM.png)

확률 질량 함수는 이산 분포에만 적용된다. 

연속 분포는 특정 값의 확률이 0이기 때문에, 확률을 정의하기가 불가능하다.

그래서 연속 분포는 값의 구간에 확률을 지정하는데, 

가장 중요한 연속 분포 중 하나인 가우시안 분포를 보자.

 ![gaussian](https://imgur.com/McMfBiq.png)

X가 0일때 정규분포가 최댓값을 가진다. ***P(X = 0) = 1/√2π***이다.

X는 -∞ 부터 ∞까지 어떤 값이든 될 수 있다. 이 범위에서 특정 X의 확률은 0이다.

하지만 ***P(X <= 0) = 0.5***같이 나타낼 수 있다.



**베이즈와 빈도학파 접근**

동전 던지기를 할 때, 앞면, 뒷면 외에도 옆으로 서거나, 공중에서 사라질 수도 있다.

이런 확률을 따지는 데에는 빈도학파와 베이즈학파가 있다.

빈도학파는 수많은 시도에서 나오는 상대적 빈도, 한계로 정의한다.

베이즈학파는 새로운 데이터가 나옴에 따라 변할 수 있는 구조로 다룬다.

베이즈 규칙은 특정 현상의 변수들의 분포를 내재적으로 만드는 매커니즘이다.

***P(EvidenceㅣData) = P(DataㅣEvidence)P(Evidence)/P(Data)***. 베이즈 규칙의 공식이다.

***P(EvidenceㅣData)*** 는 사후 분포, ***P(DataㅣEvidence)*** 는 우도 함수, ***P(Evidence)*** 는 사전 분포이다.



**동전 시뮬레이션**

아래 코드는 동전 던지기를 1000회 실시한다. 

```R
outcomes <- sample(c(0, 1), 1000, replace = TRUE)
```

다음은 결과물의 히스토그램이다.

  ![](https://imgur.com/IJejYlE.png)

대략 500번씩의 결과물이 나왔다. 빈도학파에 따르면 각각의 확률을 500/1000으로 지정한다.

이는 편향된 동전의 실험 코드이다.

```R
biased_outcomes <- sample(c(0, 1), 1000,
                          replace = TRUE, prob = c(0.4, 0.6))
```

데이터를 얻기 전에 빈도학파는 앞면의 확률을 정의할 수 없다고 말한다.

반면, 베이즈학파는 확률이 약 50%라는 가정에서 시작한다.

데이터가 들어오면 둘 다 윗면이 나올 확률에 대한 추정치를 업데이트한다.

이는 빈도학파의 결과물이다.

```R
prob_estimate <- sum(biased_outcomes) / length(biased_outcomes)

prob_estimate
## [1] 0.603
```

새로운 정보가 들어오면 베이즈학파의 분포는 수정된다. 

결국 평균은 실제 확률에 수렴하게 된다.



이 글 Quantitative Trading with R 시리즈는 책 [Quantitative trading with R](https://www.amazon.com/Quantitative-Trading-Understanding-Mathematical-Computational/dp/1137354070)을 보고 공부한 내용들입니다.

