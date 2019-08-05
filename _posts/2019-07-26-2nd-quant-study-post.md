---
layout: post
title: "Quantitative Trading With R-2"
subtitle: "무작위 과정, 주가 분포, 정상성, urca"
categories: data
comments: true
---

**무작위 과정**

데이터에 맞게 모델을 설정하려면, 먼저 모델을 생각해야 한다. 일단 주가가 무작위 과정을 통해 생성된다고 가정하자. 무작위 과정(확률 과정)은 시간에 따른 확률 변수들의 집합으로, 다음과 같은 것들이 있다.

- 매일 떨어지는 별똥별의 숫자
- 거래소 매칭 엔진에 도달하는 평균 분당 매수 주문량
- IWM의 매일 종가
- 삼성의 시간당 수익률

생성 과정을 알 수 없는 블랙박스라고 보면 될 것이다.

X1, X2, X3.. 같은 모음은 시간으로 인덱스되기 때문에 무작위 과정이다. 시간 간격이 정의될 필요는 없고, 시간 인덱스의 증가에 따라 생성되기만 하면 된다.

우리의 최종 목표 중 하나는 X1, X2, X3..같은 결과만이 주어졌을 때 블랙박스에 대해 의미 있는 이야기를 하는 것이다. 오늘의 주가를 생성하는 블랙박스를 볼 수 있다면 내일의 주가를 예측할 수 있을 것이다. 실제로 주가를 생성하는 단일 매커니즘은 없고, 실재하더라도 매우 복잡하고 유동적일 것이다.

하지만 수학과 프로그래밍 도구를 이용해 시장에 대한 질문을 계속해야 한다. 간단한 모델이더라도 보완을 계속하면 더 괜찮게 근사할 수 있을 것이기 때문이다. 

복잡한 시장 원리를 우리에게 맞추기 위해 몇 가지 가정을 한다.

1. 모델링할 수 있는 무작위 과정이 존재한다. 이 과정은 기록하고 분석할 수 있는 확률 변수를 생성한다.
2. 무작위 과정의 매개변수는 대부분 시간에 따라 변하지 않는다.



**주가 분포**

경험을 통해 우리는 주가가 0에서 무한대로 증가할 수 있다는 걸 알고 있다. 이론적인 확률 분포는 0에서 끝나고 오른쪽으로 x축과 닿지 않는 꼬리를 가져야 한다. 이에 관심이 있는 이유는 다음과 같은 질문을 하고 싶기 때문이다. "SPY ETF의 내일 종가가 $210 이상으로 끝날 확률은 무엇인가?"

다음 그래프는 이용 가능한 모든 과거 주가의 히스토그램이다.

```R
require(quantmod)
getSymbols("SPY")

# 주가를 뽑아 통계치를 계산
prices <- SPY$SPY.Adjusted
mean_prices <- round(mean(prices), 2)
sd_prices <- round(sd(prices), 2)

# 범례가 포함된 히스토그램 생성
hist(prices, breaks = 100, prob = TRUE, cex.main = 0.9)
abline(v = mean_prices, lwd = 2)
legend("topright", cex = 0.8, border = NULL, bty = "n", paste("mean=", mean_prices, 
                                                             "; sd=", sd_prices))
```

![](https://imgur.com/9Sbh5r6.png)

이번에는 비슷하지만 다른 기간의 주가 분포를 살펴보자. 넘어가기 전, 앞의 코드를 함수로 만들어 다른 기간으로 여러 번 실행한다.

```R
plot_4_ranges <- function(data, start_date, end_date, title){
    
    # 그래프 창을 2행 2열로 설정
    par(mfrow = c(2,2))
    
    for(i in 1:4){
        # 적절한 날짜 기간을 갖는 문자열 생성
        range <- paste(start_date[i], "::", end_date[i], sep  = "")
        
        # 주가 벡터와 필요한 통계치 생성
        time_series <- data[range]
        
        mean_data <- round(mean(time_series, na.rm = TRUE), 3)
        sd_data <- round(sd(time_series, na.rm = TRUE), 3)
        
        # 히스토그램과 범례 그리기
        hist_title <- paste(title, range)
        hist(time_series, breaks = 100, prob = TRUE,
            xlab = "", main = hist_title, cex.main = 0.8)
        legend("topright", cex = 0.7, bty = 'n', 
              paste("mean=", mean_data, "; sd=", sd_data))
    }
    
    # 그리기 창 재설정
    par(mfrow = c(1, 1))
}
```

정의된 함수로 다음을 실행한다.

```R
# 시작 날짜와 종료 날짜 정의
begin_dates <- c("2007-01-01", "2008-06-06",
                "2009-10-10", "2011-03-03")
end_dates <- c("2008-06-05", "2009-09-09",
              "2010-12-30", "2013-01-06")

# 그래프 그리기
plot_4_ranges(prices, begin_dates, end_dates, "SPY prices for:")
```

![](https://imgur.com/Bt243Em.png)

그래프에서 볼 수 있는 것은 통계치가 시간에 따라 변한다는 점이다. 다른 기간은 매우 다른 결과를 보여준다. 시간에 따라 SPY의 가격 분포는 비정상성임이 나타난다.



**정상성**

무작위 과정은 시간에 따라 모양과 위치가 변하지 않는 확률 분포를 가질 때 완전한 정상이라 불린다. 무작위 과정 Xt를 생각해보자. 이 과정의 결과는 Xt1, Xt2, Xt3, ... 처럼 생긴 시계열이다. 완전 정상성은 다음을 따른다.

Xt1 + Xt2 + Xt3 + ... + Xtn = Xt(1+a) + Xt(2 + a) + Xt(3 + a) + ... + Xt(n + a)

사실 정상성은 현실에 존재하기 힘들다. 실무에서는 적어도 분포의 몇 가지 모멘트가 정상인지 살펴본다. 평균과 공분산이 이에 해당된다. 약한 정상성은 공분산 정상성, 약정상성 또는 이차 정상성이라 불린다.

다음을 만족하면 과정은 공분산 정상성을 가진다.

E(Xt) = E(X(t+r)) = u

cov(Xt, X(t+r)) = f(r)

두 번째 등식은 공분산이 r이라 불리는 하나의 매개변수의 함수로 표현된다는 것을 의미한다.

정상성에 관심을 가지는 이유는 미래를 예측하기 위해 미라는 과거처럼 움직인다는 가정이 필요하기 때문이다. 블랙박스가 최근 가격을 보지 못했다면 내일 주가가 얼마가 될지 결론을 내릴 수 없다. 하지만 학습 기간 중 새로운 입력 값이 이전과 비슷하다면 블랙박스는 미래에 대해 학습된 추측을 만들어낼 수 있을 것이다.

금융 데이터는 주로 비정상이다. 작업을 하기 위해서는 데이터가 정상성을 가지게 만들어야 한다. 하나는 원 가격보다 수익률을 보는 것이다. 다른 방법은 입력 값은 비정상이지만 조합이 정상일 때 공적분 스프레드를 만드는 것이다.

이전과 같은 시간 간격으로 SPY 주식을 살펴보자. 이번에는 주가 자체가 아니라 연속된 가격 차이의 자연 로그 값을 사용할 것이다. 이를 로그 수익률이라 부른다. 작은 수익률에 대해 로그 수익률은 좋은 근사치가 된다.

수학적으로는 다음을 따른다.

Rt = (Pt - P(t-1))/P(t-1) = Pt/P(t-1) - 1

log(1 + x)의 테일러 전개는 다음과 같다.(수치적으로 정확하게 다루기 어려운 함수들에 대해, 다항식으로 근사화)

log(1 + x) = x - x^2/2 + x^3/3 + O(x4)

위 식에서 x가 매우 작을 때를 생각해보자.

log(1 + x) ≒ x

x를 Rt로 대체하면 log(1+Rt) ≒ Rt가 된다.

= log(1 + Pt/P(t-1) - 1) ≒ Rt

= log(Pt/P(t-1)) = log(Pt) - log(P(t-1)) ≒ Rt

일반적인 수익률 대신 로그 가격 차이를 사용하는 이유는 로그 가격이 갖는 수리적 성질 때문이다. 가격이 로그-정규분포를 따른다면 %수익률도 정규분포를 따르는 것으로 쉽게 모델링할 수 있음을 살펴볼 것이다.

```R
# 로그 수익률 계산
returns <- diff(log(prices))

# 전과 같이 수익률 그래프를 그리는 함수 사용
plot_4_ranges(returns, begin_dates, end_dates, "SPY log prices for:")
```

![](https://imgur.com/5pcLm0F.png)

위 그래프에서 평균들은 0에 가깝고 표준편차도 값이 비슷한 것을 볼 수 있다. 전에 말했듯이 수익률 분포는 몇 가지 모멘트에 대해 정상이라는 것을 알 수 있다. 



**urca를 이용한 정상성 확인**

urca 패키지는 가설의 시계열이 정상인지 아닌지를 결정할 수 있게 해 준다. urca는 시계열 간 공적분 관계를 모델링하는 데도 쓰인다.

가격과 수익률의 움직임에 대한 논의를 통해 직관적으로 가격 분포는 비정상이고 수익률 분포는 정상임을 기대할 수 있다.

urca 패키지는 쿠엣코와스키 등의 단위근 테스트나 KPSS 같은 정상성 테스트를 제공한다. KPSS는 결정적 추세를 갖는 정상성을 귀무가설로 가정한다. 테스트 결과 값은 임계값과 비교해 귀무가설을 채택할지 말지를 결정한다.

```R
# SPY 데이터를 얻고 비정상성을 갖는지 테스트
require(quantmod)
getSymbols("SPY")
spy <- SPY$SPY.Adjusted

# 기본 설정 사용
require(urca)
test <- ur.kpss(as.numeric(spy))

# 결과물은 S4 객체
class(test)
## [1] "ur.kpss"
## attr(, "package")
## [1] "urca"

# 테스트 통계치 추출
test@teststat
## [1] 11.63543

# 임계값 살펴보기
test@cval
##                 10pct  5pct 2.5pct  1pct
## critical values 0.347 0.463  0.574 0.739
```

임계값이 시계열이 정상이라는 귀무가설을 기각할 만큼 크다.

```R
spy_returns <- diff(log(spy))

# 수익률에 대해 테스트
test_returns <- ur.kpss(as.numeric(spy_returns))
test_returns@teststat
## [1] 0.2310047

test_returns@cval
##                 10pct  5pct 2.5pct  1pct
## critical values 0.347 0.463  0.574 0.739
```

수익률의 경우 10%에서 겨우 귀무가설을 기각할 수 있었다. 더 안정되어 보이는 2013년 1월 이후 기간은 어떨까?

```R
test_post_2013 <- ur.kpss(as.numeric(spy_returns['2013::']))
test_post_2013@teststat
## [1] 0.06936672
```

여기서는 시계열이 정상성을 가진다는 귀무가설을 기각하지 못했다.
