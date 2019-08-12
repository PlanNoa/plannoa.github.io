---
layout: post
title: "Quantitative Trading With R-8"
subtitle: "스프레드 거래"
categories: data
comments: true
---

**스프레드 거래**

매수, 매도 신호가 계산되었으니 이제 얼마의 자본을 투입해야 하는지 정해야 한다. 모든 매수 타이밍에 계속 매수를 하고 모든 매도 타이밍에 계속 매도를 해도 될까? 매수 신호가 연이어 나온다면 같은 양으로 거래해야 할까? 아니면 늘려야 할까? 등 모두 가능한 질문이고, 답을 얻으려면 백테스팅 로직을 수정해 이런 시나리오들을 경험해봐야 한다.

거래 규모를 AAPL은 100주, SPY는 100 * β 주로 고정한다. 거래 수량은 ```data_out```  테이블에 기록될 것이다. 거래 수량 외에 실현 손익과 미실현 손익 역시 계산해야 한다. 이 모든 것들은 결국 전랙의 수익 곡선을 계산하는 데 쓰일 것이다.

수익 곡선은 트레이딩 전략의 수익, 손실 정보를 그래프로 나타낸 것이다. 수익 곡선은 트레이딩 전략에 내포된 리스크와 기대수익을 결정하는 데 매우 중요하다. 관심을 가질 성과 지표에는 수익 곡선의 변동선과 특정 기간 동안 수익 곡선의 최대 손실이 포함된다.

현재 ```data_out``` 테이블의 모습이다.

```R
##                  NA 108.18985 43.26082            NA     NA
##          0.26001417 108.97365 43.61122            NA     NA
##          0.17269804 109.15906 42.63133   14.24842818      1
##          0.19011041 108.08873 42.40482   23.73810741      1
##          0.18245381 107.94541 41.63390   21.11235400      1
##          0.11946020 108.18985 40.88802   21.14837219      1
##          0.37028397 108.80508 42.23086   29.23298055      1
##          0.41224121 108.86410 42.72519    2.41455875      0
##          0.45739679 109.28549 43.03181   -2.02017639      0
```

연속으로 매수 신호가 있다. 단순화를 위해 한 번에 하나의 매수나 매도 포지션을 가진다고 정한다. 다음은 주식 X와 Y의 거래 수량을 계산하는 방법을 보여준다.

```R
prev_x_qty <- 0
position <- 0
trade_size <- 100
signal <- as.numeric(data_out$signal)
signal[is.na(signal)] <- 0
beta <- as.numeric(data_out$beta_out_of_sample)

qty_x <- rep(0, length(signal))
qty_y <- rep(0, length(signal))

for(i in 1:length(signal)){
  if(signal[i] == 1 && position == 0){
    prev_x_qty <- round(beta[i] * trade_size)
    qty_x[i] <- -prev_x_qty
    qty_y[i] <- trade_size
    position <- 1
    }

  if(signal[i] == -1 && position == 0){
    prev_x_pty <- round(beta[i] * trade_size)
    qty_x[i] <- prev_x_qty
    qty_y[i] <- -trade_size
    position <- -1
    }

  if(signal[i] == 1 && position == -1){
    qty_x[i] <- -(round(beta[i] * trade_size) + prev_x_qty)
    prev_x_qty <- round(beta[i] * trade_size)
    qty_y[i] <- 2 * trade_size
    position <- 1
    }
    
    if(signal[i] == -1 && position == 1){
      qty_x[i] <- round(beta[i] * trade_size) + prev_x_qty
      prev_x_qty <- round(beta[i] * trade_size)
      qty_y[i] <- -2 * trade_size
      position <- -1
    }
}
```

샘플 밖의 마지막 날 두 주식의 수량이 남아있다. 이를 0으로 만들어 주고, ```data_out``` 테이블에 추가한다.

```R
qty_x[length(qty_x)] <- -sum(qty_x)
qty_y[length(qty_y)] <- -sum(qty_y)

data_out$qty_x <- qty_x
data_out$qty_y <- qty_y

data_out[1:3, ]
##            beta_out_of_sample        x        y   spread signal qty_x qty_y
## 2011-01-18          0.1726980 109.1591 42.63133 14.24843      1  -121   100
## 2011-01-19          0.1901104 108.0887 42.40482 23.73811      1  -121     0
## 2011-01-20          0.1824538 107.9454 41.63390 21.11235      1  -121     0

tail(data_out, 3)
##            beta_out_of_sample        x        y    spread signal qty_x qty_y
## 2012-12-26          0.9486305 124.6775 64.77053 -58.53931     -1  -121     0
## 2012-12-27          0.9426947 124.5104 65.03062 -53.08375     -1  -121     0
## 2012-12-28          0.8217917 123.1647 64.34000 -51.76672     -1  -121   100
```

위의 값들은 계산된 베타 값을을 고려하면 현실적으로 보인다. 각 스프레드에서 필요한 매수, 매도 수량을 정했으니 얼마만큼 실현, 미실현 손익이 존재하는지 알아보자.

100개의 수량을 구매하고 5일 뒤에 포지션을 종료했다면 실현 손익은 진입 가격과 매도 가격 차이에 총수량을 곱한 값이 된다. 하지만 2일, 3일, 4일에 포지션은 닫히지 않았지만 가능한 수익, 손실이 존재한다. 이를 미실현 부분이라 부른다. 미실현 손익을 계산하는 방법은 현재 시장 가격에 포지션을 정산하는 것이다.

비슷한 분석을 위의 스프레드에 적용해보자. 분석의 마지막 결과에는 두 주식의 수익 곡선에 포함될 두 개의 열이 포함될 것이다.

```R
# 수익 곡선을 계산할 함수
compute_equity_curve <- function(qty, price) {
    cash_buy <- ifelse(sign(qty) == 1, qty * price, 0)
    cash_sell <- ifelse(sign(qty) == -1, -qty * price, 0)
    position <- cumsum(cash_buy)
    cumulative_buy <- cumsum(cash_buy)
    cumulative_sell <- cumsum(cash_sell)
    
    equity <- cumulative_sell - cumulative_buy + position * price
    return(equity)
}

data_out$equity_curve_x <- compute_equity_curve(data_out$qty_x, data_out$x)
data_out$equity_curve_y <- compute_equity_curve(data_out$qty_y, data_out$y)

plot(data_out$equity_curve_x + 
    data_out$equity_curve_y, type = "l",
    main = "AAPL / SPY spread", ylab = "P&L",
    cex.main = 0.8,
    cex.axis = 0.8,
    cex.lab = 0.8)
```

![](https://imgur.com/jBGZSuW.png)

몇 가지 사실을 정리하자.

1. 전체적으로 우상향 곡선이다. 이 전략은 테스트 기간 동안 수익을 낸다.(생각했던 것보다 훨씬!)
2. 전략에 손실을 내거나 급격한 수익을 내는 구간은 없다. 안정적이다.
3. 전략의 리스크를 조금 더 늘려도 좋을 것 같다.

다음은 고려할 사항들이다.

1. 거래량 조절은 전략의 리스크 특성을 잘 반영했는가?
2. 거래 신호는 전략에 잘 맞는가?
3. 선견 편향이 존재하는가? 어떻게 해결할 것인가?
4. 거래 비용을 고려했는가?
5. 스프레드가 거래 가능한가?
6. 가격 차이 베타 대비 수익률 베타가 타당한가?