---
layout: post
title: "Quantitative Trading With R-4"
subtitle: "데이터 필터링, R 구조식"
categories: data
tag: ts
comments: true
---

**데이터 필터링**

극단치를 다루는 데는 세 가지 방법이 있다. 극단치를 무시하거나, 제거하거나, 적절하게 조절하는 것이다. 먼저 잘못된 값을 제거하는 방법을 보자.

```R
# 극단치 찾기
outliers <- which(sv[, 2] > 1.0)

# 극단치가 존재하면 제거
if(length(outliers) > 0) {
    sv <- sv[-outliers, ]
}

cor(sv)
##            SPY.Close  VXX.Close 
## SPY.Close  1.0000000 -0.8066466
## VXX.Close -0.8066466  1.0000000
```

상관관계 추정치에 큰 변화가 있음을 볼 수 있다.

SPY와 VXX 간에는 선형 관계가 실제로 존재한다. 그 관계의 강도를 계량하는 것이 선형 회귀 분석의 R-squared값이다.(R-squared = SSR(*Sum of Square Regression*)/SST(*Sum of Square Total*) = 1 - SSE(*Sum of Square Error*)/SST) R-squared값에 제곱근을 취하면 피어슨 상관계수가 된다.

선형 회귀 분석의 결과는 2차원 공간의 산점도를 가장 잘 나타내는 직선이라 생각할 수 있다. 이런 선형 관계는 기울기와 절편을 구하면 알 수 있다. lm()함수가 이를 수행한다. 회귀 분석에 두 개 이상의 변수가 존재하면 모델은 고차원 동간에 존재하게 된다. 다음은 선형 회귀 분석으로 추정하려는 수리적 모델이다.

Y = α + β1X1 + β2X2 + ... + βnXn + ε

ε는 선형 모델로 설명되지 않는 변동을 설명한다. 이상적으로 이 항은 자기 상관이나 식별 가능한 구조를 가지지 않은 정규분포여야 한다.



**R 구조식**

R에서 lm(), glm(), nls()같은 함수들은 인자로 기호가 있는 표현은 받는다. R은 ~연산자로 그런 구조를 생성한다. y와 x 두 개의 변수가 주어졌을 때 기호 표현이나 구조식은 다음과 같이 만든다.

```R
# 구조식 맘ㄴ들기
my_formula <- as.formula("y ~ x") 

# 결과물은 무엇인가?
my_formula
## y ~ x

# my_formula의 클래스는 무엇인가?
class(my_formula)
## [1] "formula"
```

이는 R에게 종속 변수 y와 독립 변수 x간의 선형 관계를 구하고 싶다는 것을 알려준다. 그런 선형 관계의 기울기와 절편을 가지고 싶다는 점도 전달해준다. 우측은 y ~ x  - 1로 변형시키면 보델은 절편을 무시할 것이다. 독립 변수가 하나보다 많을 때 공식은 다음과 같다. y ~ x + z + w + z * w - 1. 이는 y, x, z, w간의 선형 관계와 z, w간의 상호 관계도 포함한다. I() 함수는 구조식 내의 수리적 연산을 수행한다. ex) y ~ x + z + I(x * w)는 x와 w의 곱과 일치하는 독립 변수를 생성한다.

```R
# 선형 회귀 객체 생성
reg <- lm(VXX.Close ~ SPY.Close, data = sv)

# 결과물
summary(reg)

## Call:
## lm(formula = VXX.Close ~ SPY.Close, data = sv)

## Residuals:
##     Min       1Q      Median     3Q       Max
## -0.085607 -0.012830 -0.000865 0.012188 0.116349

## Coefficients: 
##             Estimate Std. Error t   value Pr(>|t|)
## (Intercept) -0.0024365  0.0006641    -3.669  0.000254 ***
## SPY.Close   -2.5848492  0.0552193   -46.811   < 2e-16 ***
## ---
## Signif. codes: 0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1

## Residual standard error: 0.02287 on 1187 degrees of freedom
## Multiple R-squared:   0.6486,   Adjusted R-squared: 0.6483
## F-statistic:  2191 on 1 and 1187 DF, p-value: < 2.2e-16

```

가울기와 절편은 요약 결과의 회귀 계수 항목에 나와 있다. Pr(>|t|) 밑의 값은 추정 매개변수의 p-value다. p-value는 귀무가설이 사실일 때 추정 매개변수가 관측될 확률이다. 귀무가설은 매개변수가 모두 0이라는 주장이다. p-value가 매우 작다는 것은 귀무가설을 기각할 수 있음을 의미간다.

회귀 계수들은 다음과 같이 얻는다.

```R
b <- reg$coefficients[1]
a <- reg$coefficients[2]
```

선형 모델은 VXX와 SPY 수익률의 관계가 VXXt = aSPYt + b + εt이라고 말한다. εt는 회귀식의 잔차라고도 한다.

다음은 잔차가 어떻게 생겼는지를 보여준다.

```R
par(mfrow = c(2,2))
plot(reg$residuals, 
     main = "Residuals through time", 
     xlab = "Days", ylab = "Residuals")
hist(reg$residuals, breaks = 100, 
    main = "Distridution of residuals",
	xlab = "Residuals")
qqnorm(reg$residuals)
qqline(reg$residuals)
acf(reg$residuals, main = "Autocorrelation")
```

![](https://imgur.com/YKKMIbZ.png)

위의 네 가지 그래프는 잔차의 다른 성질을 보여준다. 처음에는 시간 그래프와 히스토그램이 잔차가 정규분포임을 나타내는 것 같다. 하지만 정규분포보다 더 두터운 꼬리를 가지고 있다. 이는 qqplot()이 그린 분위-분위 그래프에서도 볼 수 있다. acf() 함수는 특정 시계열의 자기 상관도를 계산한다. 이는 현재의 관측치와 과거의 관측치 간에 상관관계가 존재하는지를 묻는다. 이는 시계열을 한 단계 앞당겨서 원 시계열과 상관관계를 구하면 알 수 있다. 시간 지연이 0이라면 상관관계는 1이 되어야 한다. 그 외의 시간 지연은 다른 값을 나타낸다. 위의 모델을 보아서는 단순 선형 회귀 모델에 어떤 특별한 자기 상관 구조가 없음을 보여준다. 

자기 상관이 존재한다면 두 변수 간의 관계를 보기 위한 선형 회귀 분석의 가정을 다시 살펴보아야 한다. 이상적으로 잔차는 어떤 구조 없이 정규 분포를 따르면 좋다.

동시 수익률을 보는 것은 증권 간의 관계를 살피는 한 방법이다. 더 좋은 것은 어제의 수익률과 오늘의 수익률 간에 선형 관계가 존재하느냐이다. 만약 그렇다면, 선형 회귀 분석 모델이 종속 회귀자로 시장을 분석하는 좋은 시도가 될 것이다.

```R
vxx_lag_1 <- lag(VXX$VXX.Close, k = 1)
 
head(vxx_lag_1)
##            VXX.Close
## 2018-01-25        NA
## 2018-01-26     27.66
## 2018-01-29     27.66
## 2018-01-30     29.58
## 2018-01-31     30.55
## 2018-02-01     30.65
 
head(VXX$VXX.Close)
##            VXX.Close
## 2018-01-25    27.660
## 2018-01-26    27.660
## 2018-01-29    29.580
## 2018-01-30    30.550
## 2018-01-31    30.650
## 2018-02-01    29.107
```

lag 함수는 시계열을 하루 지연시킨다. 정제된 VXX 수익률 행렬에서 지연된 SPY와 VXX간의 선형 회귀를 살펴보자. 다음 예제는 암묵적으로 SPY 수익률이 VXX 수익률에 선행한다고 가정했다.

```R
# 수익률과 지연 수익률 합치기
sv <- merge(sv, lag(sv[, 1]), lag(sv[, 2]))

# 시간 지연된 SPY와 VXX간의 산점도
plot(as.numeric(sv[, 3]), as.numeric(sv[, 2]),
	main = "Scatter plot SPY lagged vs. VXX.",
	xlab = "SPY lagged",
	ylab = "VXX",
	cex.main = 0.8,
	cex.axis = 0.8,
	cex.lab = 0.8)
grid()
```

![](https://imgur.com/2LFEEO5.png)산점도는 SPY의 시간 지연 수익률과 VXX 수익률 간에 선형 관계가 없음을 보여준다. 선형 회귀 분석의 결과는 이를 확인시켜준다.

```R
reg2 <- lm(VXX.Close ~ SPY.Close.1, data = sv)

summary(reg2)

## Call:
## lm(formula = VXX.Close ~ SPY.Close.1, data = sv)

## Residuals:
##       Min        1Q    Median        3Q       Max 
## -0.074387 -0.020779 -0.003324  0.015179  0.160464 

## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)
## (Intercept) -0.004579   0.002925  -1.565    0.120
## SPY.Close.1  0.472353   0.380817   1.240    0.217

## Residual standard error: 0.03491 on 144 degrees of freedom
##   (1 observation deleted due to missingness)
## Multiple R-squared:  0.01057,	Adjusted R-squared:  0.0037 
## F-statistic: 1.539 on 1 and 144 DF,  p-value: 0.2169
```

ccf()함수는 독립적으로 두 개의 시계열을 지연시키고 상관관계를 계산함으로써 비슷한 분석을 수행한다. acf()와는 다르게 두 개의 시계열을 연산한다.

```R
ccf(as.numeric(sv[, 1]), as.numeric(sv[, 2]),
   main = "Cross correlation between SPY and VXX",
   ylab = "Cross correlation", xlab = "Lag", cex.main = 0.8, cex.lab = 0.8,
   cex.axis = 0.8)
```

![](https://imgur.com/KbzyhBb.png)





이 글 Quantitative Trading with R 시리즈는 책 [Quantitative trading with R](https://www.amazon.com/Quantitative-Trading-Understanding-Mathematical-Computational/dp/1137354070)을 보고 공부한 내용들입니다.