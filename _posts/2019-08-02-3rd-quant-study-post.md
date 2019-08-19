---
layout: post
title: "Quantitative Trading With R-3"
subtitle: "정규성의 가정, 상관관계"
categories: data
tag: quant
comments: true
---

**정규성의 가정**

가격과 수익률 분포애 대해 단순화된 가정을 만들어 보자. 평균 μ와 표준편차 σ를 가지는 가우시안 분포의 공식은 다음과 같다.

f(x) = (1/√(2πσ)^2)*e^((x-μ)^2/2σ) 

이 정규분포를 실제 수익률 데이터에 대입해 어떻게 생겼는지 볼 수 있다.

```R
# 히스토그램과 밀도 그리기
mu <- mean(returns, na.rm = TRUE)
sigma <- sd(returns, na.rm = TRUE)
x <- seq(-5 * sigma, 5 * sigma, length = nrow(returns))

hist(returns, breaks = 100, 
    main = "Histogram of returns for SPY",
    cex.main = 0.8, prob = TRUE)
lines(x, dnorm(x, mu, sigma), col = "red", lwd = 2)
```

![](https://imgur.com/EtYadfR.png)dnorm() 함수는 주어진 x값의 범위,  평균, 표준편차로 정규분포를 생성한다.

lines()함수는 기존 히스토그램 위에 선을 그린다.

그래프를 보면 데이터가 정규분포를 따르고 있지 않다. 0% 부근에 값이 더 많이 존재한다. 이런 실증적 분포(empirical distribution)를 급첨 분포(leptokyrtic distribution)라고 부른다.

실증적 데이터와 이론적 정규분포 간의 차이를 시각화하는 다른 방법은 qqnorm()과 qqline()함수를 사용하는 것이다.

```R
# 그래프 창 설정
par(mfrow = c(1, 2))

# SPY 데이터
qqnorm(as.numeric(returns),
      main = "SPY empirical returns qqplot()",
      cex.main = 0.8)
qqline(as.numeric(returns), lwd = 2)
grid()

# 정규분포를 따르는 임의의 데이터
normal_data <- rnorm(nrow(returns), mean = mu, sd = sigma)
qqnorm(normal_data, main = "Normal returns", cex.main = 0.8)
qqline(normal_data, lwd = 2)
grid()
```

![](https://imgur.com/2wWjAVD.png)

선과 점들의 차이가 정규성과 멀어진 것을 표시해준다. 정규성과 실중 분포 간의 편차 값을 결정하는 다양한 통계 테스트가 오랫동안 발전되어 왔다. 이들 대부분은 R에서 새용 가능하다.

예를 들어 Shapiro 테스트는 원 수익률을 shapiro.test() 함수에 인자로 넘기면 실행된다. 결과값은 데이터가 정규분포에서 나올 가능성을 표시한 확률이다.

```R
answer <- shapiro.test(as.numeric(returns))

answer[[2]]
## [1] 9.709754e-46
```

이 결과에 따르면 이 데이터는 가우시안 분포에서 나왔을 확률이 거의 없다.

위의 분석은 약간의 의심을 가질 필요가 있다. 통계 테스트의 결과를 받아들이기 전에 분석 대상 데이터의 구조를 이해하는 것이 중요하기 때문에, 실무적인 관점에서 통계 분석을 수행하기 전에 시각적으로 데이터를 검사하는 것이 중요하다. 너무 많거나 적은 데이터는 결과 값을 극단적으로 치우치게 한다. 극단치가 어떻게 통계 테스트의 결과를 왜곡하는지 보자.

```R
set.seed(129)
normal_numbers <- rnorm(5000, 0, 1)
ans <- shapiro.test(normal_numbers)

ans[[2]]
## [1] 0.9963835
```

높은 p-value는 샘플 데이터가 정규분포에서 벗어났음을 의미한다. 데이터에 굉장히 큰 값 하나가 샘플에 들어간다면 어떨까?

```R
normal_numbers[50] <- 1000
ans <- shapiro.test(normal_numbers)

ans[[2]]
## [1] 1.775666e-95
```

이 경우 p-value는 0이라고 본다. Shapiro-Wilks 테스트는 극단치의 효과를 제거하지 못한다. 테스트를 수행하기 전 간단한 그래프를 그렸다면, 극단치를 확인할 수 있었을 것이다.

데이터를 시각화하는 것이 항상 실용적인 것은 아니다. 데이터 검사를 자동화해서 극단치를 제거하거나 견고한 통계 테스트를 사용할 수 있다.



**상관관계**

상관관계는 두 변수 간의 선형 관계를 측정한 통계치이다. 실무에서 가장 선호되는 상관관계는 피어슨 상관관계이다. 다른 유명한 측정치로는 순위 상관이 있다. 피어슨 상관계수의 공식은 다음과 같다.

Px, y = E[(X - μx)(Y - μy)]/σxσy

분자 E[(X - μx)(Y - μy)]는 금융에서 항상 볼 수 있는 수학적 구조이다. 이는 확률 변수 X와 Y의 공분산이라고 불린다. 둘 이상의 확률변수를 다룰 때 공분산을 행렬로 표현할 수 있다. 이를 공분산 행렬이라고 하는데, 현대 금융의 기본 중 하나이다. Px, y가 양의 값이라는 건 확률변수 X와 Y에 선형 관계가 있음을 의미한다. 

```R
getSymbols("VXX")
returns_matrix <- cbind(diff(log(SPY)), diff(log(VXX)))
sv <- as.xts(returns_matrix["2009-02-02::", c(4, 10)])

head(sv)
##               SPY.Close    VXX.Close
## 2009-02-02 -0.003022794 -0.003160468
## 2009-02-03  0.013949192 -0.047941603       
## 2009-02-04 -0.004908084 -0.003716543
## 2009-02-05  0.014770941 -0.006134680
```

R에서 상관관계는 시계열 수익률 행렬을 cor()함수를 이용해 얻을 수 있다.

```R
cor(sv)

##            SPY.Close  VXX.Close
## SPY.Close  1.0000000 -0.4603908
## VXX.Close -0.4603908  1.0000000
```

두 수익률은 음의 상관관계를 갖는 것처럼 보인다. VXX가 변동성을 쫓는 주식이라는 점을 생각하면 일리가 있다.
