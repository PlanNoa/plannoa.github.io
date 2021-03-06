---
layout: post
title: "Quantitative Trading With R-10"
subtitle: "수익 곡선에서 발견할 수 있는 추가 사항, 전략 특성"
categories: data
tag: ts
comments: true
---

**수익 곡선에서 발견할 수 있는 추가 사항**

최종 값이 같은 두 개의 주식 곡선을 생각해보자. 두 그래프 모두 우상향하고 있지만 두 번째 그래프의 변동성이 더 크다. 두 번째 그래프는 주기적으로 발생하는 큰 폭의 상승으로 좋은 성과를 내기 때문에 변동성이 크다. 이렇게 변동성이 좋은 경우도 있다.

또 다른 예시가 있다. 첫 번째 그래프는 최종 값이 낮지만 변동성이 적고, 두 번째 그래프는 최종 값이 높지만 변동성이 매우 크다. 이런 경우에는 두 번째 수익 곡선의 중간에서 일어나는 손실이 걱정스럽게 한다.

마지막 예시를 생각하자. 첫 번째 그래프는 거래가 3번, 두 번째 그래프에서는 11번 일어났다. 첫 번째 그래프는 거래 횟수가 적으므로, 아직 실제 성과를 보여주지 못했다고 생각할 수 있다. 작은 표본에서 만들어진 통계치를 가지고 판단하는 것은 결코 신중한 접근이 아니다.

샤프 지수는 1966년 월리엄 샤프가 개발한 지표로, 전략의 평균 수익과 무위험 이자율의 차이를 변동성으로 나눈다. 대부분의 트레이더는 샤프 지수가 높은 것을 선호하지만, 샤프 지수가 높다고 전략이 위험하지 않다는 뜻은 아니다.

s = (E(R) - rf) / σ

매주 외가격 콜옵션을 매도하는 트레이더를 생각해보자. 매도자는 대부분 잔존하는 프리미엄을 모두 얻을 수 있다. 이 전략은 일관된 수익을 내고 변동성도 작기 때문에 샤프 지수가 크다. 작은 분모는 샤프 지수를 증폭하는 경향이 있다. 하지만 이는 좋은 전략이 아니다. 기초 자산이 한 번 비정상적으로 움직이면 그 동안 벌었던 금액 이상의 손해를 볼 수도 있다. 무방비 콜옵션 매도는 무한대의 리스크를 가지기 때문이다.

그럼에도 불구하고 샤프 지수는 학계나 산업에서 널리 쓰인다. 트레이딩 전략을 개발하거나 성과를 보여줄 때 샤프 지수는 다른 리스크 지표와 함께 제시되야 한다. 다음은 R에서 샤프 지수와 손실 통계치를 계산하는 방법이다.

```R
sharpe_ratio <- function(x, rf){
    sharpe <- (mean(x, na.rm = TRUE) - rf) / sd(x, na.rm = TRUE)
    return(sharpe)
}

drawdown <- function(x){
    cummax(x) - x
}
```

이 둘을 예제에 적용하자.

```R
par(mfrow = c(2, 1))
equity_curve <- data_out$equity_curve_x + data_out$equity_curve_y

plot(equity_curve, main = "Equity Curve",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)

plot(drawdown(equity_curve), main = "Drawdown of equity curve",
 cex.main = 0.8,
 cex.lab = 0.8,
 cex.axis = 0.8)
```

![](https://imgur.com/A6wwIT9.png)

최대 손실이 약 1061$임을 확인할 수 있다. 평균 손실은 약 251$다. 전략의 샤프 지수는 다음과 같다.

```R
equity <- as.numeric(equity_curve[, 1])
equity_curve_returns <- equity/c(0, equity[-length(equity)])

invalid_values <- is.infinite(equity_curve_returns)|is.nan(equity_curve_returns)

sharpe_ratio(equity_curve_returns[!invalid_values], 0.03)
## [1] 0.3848998
```

2년간의 샤프 지수는 0.38이다. 

샤프 지수 외에도 다른 리스크 지표들은 많다. 그 중 고려할 만한 지표들이다.

1. 소티노 지수
2. 손실 기간
3. 오메가 지수
4. 수익 곡선의 직진도
5. 궤양 지표



**전략 특성**

특정 트레이딩 전략의 이상적인 성질을 나타내는 데 하나의 숫자는 충분하지 않다. 최대의 샤프 지수나 최소의 최대 손실을 가진 전략을 찾기보다는 확장성을 고려하는 게 좋다. 시장이 감당할 수 있는 것보다 거래 규모가 작을 대 전략은 훌륭한 리스크 특성을 가진다. 하지만 전략을 사용하는 비율이 높아지면 성과는 급격히 낮아진다. 확장성과 유동성 지표는 자본을 많이 사용할 가능성이 있는 전략을 설계할 때 고려해야 한다.

트레이더가 신경쓸 다른 전략의 특성은 시장에 주문이 나가는 빈도이다. 주문 빈도에 따른 리스크 수준은 트레이딩에 이용되는 기술 플래폼을 다루는 능숙도에 따라 다른다. 자본 배분도 영향을 미친다. 자본이 적을 땐 많은 처리를 요하는 전략을, 자본이 많을 때는 많은 처리를 피하려 한다.

당일로 끝나는 전략이 다음 날로 이어지는 전략보다 선호될 수 있다. 후자가 자본이 많이 들 뿐만 아니라 리스크도 따라서 커진다.



이 글 Quantitative Trading with R 시리즈는 책 [Quantitative trading with R](https://www.amazon.com/Quantitative-Trading-Understanding-Mathematical-Computational/dp/1137354070)을 보고 공부한 내용들입니다.