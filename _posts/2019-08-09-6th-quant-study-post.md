---
layout: post
title: "Quantitative Trading With R-6"
subtitle: "주식 스프레드 정의, 보통 최소 제곱법과 전체 최소 제곱법"
categories: data
tag: quant
comments: true
---

**주식 스프레드 정의**

두 주식 스프레드(쌍)는 하나의 주식을 사도 다른 주식을 파는 것으로 구성된다. 어떤 쌍은 두 개의 주식을 모두 팔거나 사기도 한다. 예를 들어 어떤 주식이 특정 지수와 반대로 움직이면(SDS ETF처럼) ETF 두 개를 사서 원하는 스프레드를 만들 수 있다.

두 회사가 비슷한 제품을 판매하면, 같은 업황에 영향을 받아야 한다. 경제가 흔들리면 두 회사의 매출도 같이 하락하고, A회사가 가격을 내린다면 B회사 역시 가격을 내려야 한다. 거시적인 관점에서 A와 B사는 연결되어 있다.

회사 A와 B의 주식 수익률을 일반적인 팩터 모델로 설명하는 대신 주식 스프레드의 움직임을 볼 것이다.

특정한 날에 코카콜라 주식과 펩시콜라 주식이 어떻게 움직일지는 알 수 없다. 하지만 공통된 내재 요인이 두 회사에 영향을 끼치고 주가에 반영될 것이다. 이것이 우리가 모델화하려는 관계이다.

코카콜라와 펩시 주식의 달러 수익률을 생각해보자. 달러 중립 포트폴리오를 구성하려면 코카콜라 주식 몇 주를 구매하고, 펩시 주식 몇 주를 팔아야 할까? 달러 중립 포트폴리오란 금전적 가치가 0에 가까운 포트폴리오를 의미한다.

스프레드 베타가 이의 답이다. 베타는 두 주식의 수익률 산점도를 가장 잘 설명하는 기울기이다. 또한 베타는 일반적으로 %수익률을 가지고 계산한다. 하지만 가격 변화는 트레이딩 관점에서 더 직관적인 답을 제공할 수 있다. 베타는 주가 자체나 로그 주가로도 계산할 수 있는데, 이런 베타들을 공적분 베타라고 부른다. 이런 회귀 분석이 유의미할 때는 오직 주가의 쌍이 공적분 관계를 가질 때다. 공적분 테스트는 결과로 나온 잔차에 대한 통계적 분석으로 이루어진다.

```R
pepsi <- getSymbols('PEP', from = '2013-01-01', 
                    to = '2014-01-01', adjusted = TRUE, auto.assign = FALSE)

coke <- getSymbols('COKE', from = '2013-01-01', 
                   to = '2014-01-01', adjusted = TRUE, auto.assign = FALSE)
Sys.setenv(TZ = "UTC")

prices <- cbind(pepsi[, 6], coke[, 6])
price_changes <- apply(prices, 2, diff)
plot(price_changes[, 1], price_changes[, 2],
	 xlab = "Coke price changes",
	 ylab = "Pepsi price changes",
	 main = "Pepsi vs. Coke",
	 cex.main = 0.8,
	 cex.lab = 0.8,
	 cex.axis = 0.8)
grid()

ans <- lm(price_changes[, 1] ~ price_changes[, 2])
beta <- ans$coefficients[2]
```

![](https://imgur.com/JWDGCGl.png)

해당 기간동안 코카콜라 주식이 ΔS만큼 움직였으면, 평균적으로 펩시콜라 주식은 BΔS만큼 움직인다. 코카콜라 주식 1000주를 샀으면 달려 평균을 유지하기 위해 1000/B만큼 주식을 팔아야 한다. 

코카콜라 주가 변화는 확률 변수가 아니고 펩시콜라의 주가 변수는 확률변수라고 가정했다. 두 시계열의 움직임이 오직 펩시콜라 때문이라는 것을 의미한다. 모든 변동이 코카콜라 때문이라면, 또 다른 베타 값을 얻는다.

```R
ans2 <- lm(price_changes[, 2] ~ price_changes[, 1])
beta2 <- ans2$coefficients[2]

beta
## [1] 0.2320238

beta2
## [1] 0.2855719 
```



**보통 최소 제곱법과 전체 최소 제곱법**

위의 두 베타는 직관적으로 생각할 수 있는 역수 관계가 아니다. 베타의 이런 모순 때문에 트레이더들은 보통 최소 제곱(OLS) 베타 대신 전체 최소 제곱(TLS) 베타를 사용한다.

전체 최소 제곱 회귀 분석은 시스템의 변동을 두 시계열 모두를 사용해 설명한다. 이는 산점도상의 모든 점과 선형 모델간의 수직 거리 제곱의 합을 구하면 얻을 수 있다. 반면 OLS는 점들과 선 사이의 수평 거리나 수직 거리를 최소화한다.

![](https://miro.medium.com/max/854/1*illoIj5LRD3NrQ69iV30kw.png)

TLS 베타는 주성분 분석을 활용해 유도될 수 있다. R에서 주성분 분석은 prcomp()함수로 이루어진다. PCA에 담긴 아이디어는 상관관계가 있는 관측치들을 선형 관계가 없는 관측치들로 바꿔주는 것이다.

이런 성분을 찾아내려면, 

1. 산점도에서 최대의 분산을 갖는 방향을 찾는다.
2. 첫 번째와 수직인 두 번째의 최적 방향을 찾는다.

이런 방향은 주성분이 된다. 이런 성분을 계산하는 코드다.

```R
# 데이터 구하기
SPY <- getSymbols('SPY', from = '2011-01-01',
                  to = '2012-12-31', adjusted = TRUE, auto.assign = FALSE)
AAPL <- getSymbols('AAPL', from = '2011-01-01',
                   to = '2012-12-31', adjusted = TRUE, auto.assign = FALSE)

# 가격 차이 계산
x <- diff(as.numeric(SPY[, 4]))
y <- diff(as.numeric(AAPL[, 4]))

plot(x, y, main = "Scatter plot of returns. SPY vs. AAPL",
     cex.main = 0.8, cex.lab = 0.8, cex.axis = 0.8)
abline(lm(y ~ x))
abline(lm(x ~ y), lty = 2)
grid()

# 전체 최소 제곱 회귀
r <- prcomp( ~x+y)
slope = r$rotation[2, 1] / r$rotation[1, 1]
intercept <- r$center[2] - slope * r$center[1]

# 그래프에 첫 번째 주성분 표시
abline(a = intercept, b = slope, lty = 3)
```

![](https://imgur.com/0J4zkoj.png)



이 글 Quantitative Trading with R 시리즈는 책 [Quantitative trading with R](https://www.amazon.com/Quantitative-Trading-Understanding-Mathematical-Computational/dp/1137354070)을 보고 공부한 내용들입니다.