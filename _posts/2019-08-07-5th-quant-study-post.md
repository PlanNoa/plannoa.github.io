---
layout: post
title: "Quantitative Trading With R-5"
subtitle: "선형 회귀 분석의 선형성, 변동성"
categories: data
tag: quant ts
comments: true
---

**선형 회귀 분석의 선형성**

선형 회귀 분석은 회귀자 간의 선형 관계를 파악하기 위한 수리적 모델이다. 어디까지나 '선형'으로, 밀접한 관계가 있으도 선형이 아니라면 파악하지 못할 수 있다.

이 점을 설명하는 좋은 예제는 포물선이다. 직접 보자.

```R
x <- seq(1:100)
y <- x^2

# 그래프 생성
plot(x, y)

# 회귀 분석 수행
reg_parabola <- lm(y ~ x)

# 가장 잘 설명하는 선을 그래프에 추가
abline(reg_parabola, lwd = 2)

# 결과보기
summary(reg_parabola)

## Call:
## lm(formula = y ~ x)

## Residuals:
##    Min     1Q Median     3Q    Max 
##   -833   -677   -208    573   1617 

## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -1717.000    151.683  -11.32   <2e-16 ***
## x             101.000      2.608   38.73   <2e-16 ***
## ---
## Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

## Residual standard error: 752.7 on 98 degrees of freedom
## Multiple R-squared:  0.9387,	Adjusted R-squared:  0.9381 
## F-statistic:  1500 on 1 and 98 DF,  p-value: < 2.2e-16
```

![](https://imgur.com/0JiHbwd.png)

이처럼 선형 회귀 분석은 비선형 관계를 파악하는 좋지 않다. 하지만 변수에 변형을 가하면 다시 lm함수를 사용할 수 있다.

``` R
plot(x, sqrt(y))
reg_transformed <- lm(sqrt(y) ~ x)
abline(reg_transformed)

summary(reg_transformed)

## Call:
## lm(formula = sqrt(y) ~ x)

## Residuals:
##        Min         1Q     Median         3Q        Max 
## -2.680e-13 -4.300e-16  2.850e-15  5.302e-15  3.575e-14 

## Coefficients:
##               Estimate Std. Error    t value Pr(>|t|)    
## (Intercept) -5.684e-14  5.598e-15 -1.015e+01   <2e-16 ***
## x            1.000e+00  9.624e-17  1.039e+16   <2e-16 ***
## ---
## Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

## Residual standard error: 2.778e-14 on 98 degrees of freedom
## Multiple R-squared:      1,	Adjusted R-squared:      1 
## F-statistic: 1.08e+32 on 1 and 98 DF,  p-value: < 2.2e-16
```

![](https://imgur.com/Q6ZUgp1.png)

변수에 수리적인 변환을 주는 기술은 선형 회귀 분석과 함께 잘 사용된다. 물론 알맞는 변환을 찾는 것은 쉬운 일이 아니다.



**변동성**

변동성은 숫자들이 평균으로부터 얼마나 퍼져있는지를 측정하는 지표다. 숫자들이 주어지면 분산을 추정할 수 있다. 확률 과정에서 변동성을 추정하는 데 널리 쓰이는 지표는 표준편차이다.

σ = √(Σ(N, i = 1)(Xi - μ)/N)

또 다른 지표는 평균으로부터 거리의 절대값을 모두 더하는 것이다. 

D = Σ(N, i = 1)lXi - μl

수익률의 변동성은 트레이더와 퀸트가 자세히 연구하는 중요한 지표다.

트레이딩 전략을 만들기 위해서는 금융 상품의 수익률이 얼마나 크고 자주 평균값을 진동하는지 신경 쓴다. 움직임의 방향 대신, 진폭에 신경을 쓴다. 방향을 예측하는 것보다 진폭을 다루는 것이 훨씬 쉽기 때문이다.

변동성 공식에는 수익률의 제곱 합이 있다. 일간이나 일중 수익률의 분포를 다룰 때 평균값은 0에 가깝다. 

이미 일 단위의 수익률 간 특별한 자기 상관이 없는 것을 확인했다. 수익률 분포의 더 높은 차수의 모멘토를 보자.

```R
# 1000개의 독립적이고 동일한 분포를 갖는 숫자를 정규분포로부터 생성
z <- rnorm(1000, 0, 1)

# 수익률과 제곱 수익률의 자기 상관
par(mfrow = c(2, 1))
acf(z, main = "returns", cex.main = 0.8,
   cex.lab = 0.8, cex.axis = 0.8)
grid()
acf(z, main = "returns squared",
   cex.lab = 0.8, cex.axis = 0.8)
grid()
```

![](https://imgur.com/B0Bm4Us.png)

설정상 임의로 설정된 수익률은 자기 상관이 없어야 한다. 제곱 수익률 역시 마찬가지이다.

다음은 실제 제곱 수익률의 자기 상관이다. 제곱 수익률은 분산의 대용치로 사용된다.

```R
par(mfrow = c(1, 1))
acf(sv[, 1]^2, main = "Actual returns squared",
   cex.main = 0.8, cex.lab = 0.8, cex.axis = 0.8)
grid()
```

![](https://imgur.com/Nrwfy6w.png)

통계적으로 유의한 자기 상관이 다양히 존재한다. 실제 분포의 높은 차수 모멘트도 자기 상관 효과를 보여주고, 수익률의 절대값도 그렇다.

```R
par(mfrow = c(1, 1))
acf(sv[, 1]^3)
acf(abs(sv[, 1]))
```

높은 차수의 모멘트에서 이런 자기 상관 구조를 설명하기 위해 다양한 차원의 계량 경제 모델이 개발되어왔다. 그 모델들의 주요 카테고리 3개의 이름이다.

- ARIMA
- GARCH
- Stochastic Volatility

이런 변동성의 움직임은 이분산성이라 불린다. 수익률을 시간에 따라 그리면 더 잘 볼 수 있다. 자기 상관이 없는 시계열은 군집 구간이 없어야 한다. 실제 금융 데이터는 군집 현상을 가지기 때문에, 고차원 모멘트에는 기억 효과가 있다.



이 글 Quantitative Trading with R 시리즈는 책 [Quantitative trading with R](https://www.amazon.com/Quantitative-Trading-Understanding-Mathematical-Computational/dp/1137354070)을 보고 공부한 내용들입니다.