---
layout: post
title: "Quantitative Trading With R-7"
subtitle: "스프레드 구성, 신호 생성과 검증"
categories: data
tag: quant
comments: true
---

**스프레드 구성**

주식 스프레드를 구하고 간단한 트레이딩 전략으로 구한 매수, 매도 신호를 살펴볼 것이다. 전략은 스프레드가 하단 임계치보다 낮으면 매수를, 특정 상단 임계치보다 높으면 매도한다. 앞으로 사용할 몇 개의 작업을 함수로 만들자.

```R
# 스프레드 계산 함수
calculate_spread <- function(x, y, beta) {
    return(y - beta * x)
}

# 주어진 시작, 종료 날짜에서 베타와 레벨을 계산할 함수
calculate_beta_and_level <- function(x, y, start_date, end_date) {
    require(xts)
    
    time_range <- paste(start_date, "::", end_date, sep = "")
    x <- x[time_range]
    y <- y[time_range]
    
    dx <- diff(x) 
    dy <- diff(y)
    r <- prcomp(~ dx + dy)
    
    beta <- r$rotation[2, 1] / r$rotation[1, 1]
    spread <- calculate_spread(x, y, beta)
    names(spread) <- "spread"
    level <- mean(spread, na.rm = TRUE)
    
    outL <- list()
    outL$spread <- spread
    outL$beta <- beta
    outL$level <- level
    
    return(outL)
}

# 임계치로 매수, 매도 신호를 계산하는 함수
calculate_buy_sell_signals <- function(spread, beta, level, 
                                       lower_threshold, upper_threshold) {
    
    buy_signals <- ifelse(spread <= level - lower_threshold, 1, 0)
    sell_signals <- ifelse(spread >= level + lower_threshold, 1, 0)
    
    output <- cbind(spread, buy_signals, sell_signals)
    colnames(output) <- c("spread", "buy_signals", "sell_signals")
    
    return(output )
}
```

이 함수들을 사용해보자.

```R
start_date <- "2009-01-01"
end_date <- "2011-12-31"
x <- SPY[, 6]
y <- AAPL[, 6]

results <- calculate_beta_and_level(x, y, start_date, end_date)
results$beta
## [1] 0.3916332

results$level
## [1] 3.331363

plot(results$spread, ylab = "Spread Value", 
     main = "AAPL - beta * SPY", 
     cex.main = 0.8, 
     cex.lab = 0.8, 
     cex.axis = 0.8)
```

![](https://imgur.com/CW1ijdS.png)

현재 형태로 봐서는 트레이드 목적으로는 매력적이지 않다. 평균에 머무르는 구간과 급격히 위로 상승하는 구간이 섞인 것으로 보인다. 그럼에도 불구하고 특정 스프레드를 사용해 실제 트레이딩에 적용하면 생기는 문제들을 알아보자.

샘플의 베타와 스프레드의 평균값이 3.331363이란 것을 확인했다. 이 값으로 트레이딩 신호를 만들자.

```R
start_date_out_sample <- "2012-01-01"
end_date_out_sample <- "2012-12-31"
range <- paste(start_date_out_sample, "::", end_date_out_sample, sep = "")

spread_out_of_sample <- calculate_spread(x[range], y[range], results$beta)
plot(spread_out_of_sample, main = "AAPL - beta * SPY", 
     cex.main = 0.8, 
     cex.lab = 0.8, 
     cex.axis = 0.8)
```

![](https://imgur.com/DxHHDrE.png)



**신호 생성과 검증**

샘플 밖 그래프는 전체 스프레드가 샘플 수준보다 위에 위치함을 보여준다. 매수 신호는 없고, 매도 신호만 있다. 이는 스프레드를 계산한 방식이나 쌍을 이루는 주식을 선택한 방법에 문제가 있음을 의미한다. 사실 AAPL은 역동적인 주가를 가졌고, SPY는 그렇지 않다. AAPL을 같은 산업에 있는 기업과 쌍을 이루면 더 나은 결과를 얻을 것이다.

AAPL과 SPY를 개선된 시간 구간에서는 거래 가능한 쌍이라고 가정하자. 다음 코드는 10일짜리 변동 베타를 구하고 새로운 스프레드를 그린다. 이는 베타를 일간으로 업데이트하고, 새로운 베타를 다음 날에 이용할 수 있다.

이번에는 평균으로 회귀해 트레이딩이 가능해보이는 주식 스프레드를 계산하는 데 집중한다. 적절한 포지션 크기나 트레이딩 규칙은 무시한다. 베타가 변하면 포지션도 변해야 하지만, 지금은 신호를 생성하는 목적으로 변화된 스프레드의 변동에만 관심을 가진다.

```R
window_length <- 10

start_date <- "2011-01-01"
end_date <- "2011-12-31"
range <- paste(start_date, "::", end_date, sep = "")

x <- SPY[range, 6]
y <- AAPL[range, 6]

df <- cbind(x, y)
names(df) <- c("x", "y")

run_regression <- function(df){
    return(coef(lm(y ~ x - 1, data = as.data.frame(df))))
}

rolling_beta <- function(z, width){
    rollapply(z, width = width, FUN = run_regression,
             by.column = FALSE, align = "right")
}

betas <- rolling_beta(diff(df), 10)

data <- merge(betas, df)
data$spread <- data$y - lag(betas, 1) * data$y
```

동일한 스프레드가 수익률에서는 어떻게 생겼는지 보자. 

```R
returns <- diff(df)/df
return_beta <- rolling_beta(returns, 10)
data$spreadR <- diff(data$y)/data$y -
	return_beta * diff(data$x)/data$x

tail(data)
##                betas        x        y   spread      spreadR
## 2011-12-22 0.4010412 107.7922 49.87734 35.13656 -0.002342725
## 2011-12-23 0.4477577 108.7559 50.47554 30.23277  0.003315055
## 2011-12-27 0.4991907 108.8419 50.87602 28.09589  0.007022730
## 2011-12-28 0.4864968 107.4135 50.38919 25.23538  0.004251753
## 2011-12-29 0.4344114 108.5236 50.69957 26.03439 -0.003371086
## 2011-12-30 0.4346973 107.9901 50.68454 28.66660  0.004297836

plot(data$spreadR)
```

![](https://imgur.com/vvbv4u6.png)

시각적으로 스프레드는 전보다 더 평균 회귀의 성질을 가진 것처럼 보인다. 괜찮은 매수, 매도 시점을 정하기 위해 10일간 변동 구간을 사용하고 과거 기간 동안 스프레드의 표준편차를 계산한다. 스프레드의 변동성은 유지된다고 가정한다. 

```R
threshold <- sd(data$spread, na.rm = TRUE)

threshold
## [1] 7.739519
```

다음은 스프레드를 나타낸 그래프이다.

```R
plot(data$spread, main = "AAPL vs. SPY In-Sample",
    cex.main = 0.8,
    cex.lab = 0.8, 
    cex.axis = 0.8)
```

![](https://imgur.com/2P5PDBp.png)

스프레드의 수익성을 평가하는 방법은 가상의 매수, 매도 시점을 따라 백테스트를 수행하는 것이다. 한 번의 스프레드에 대해 오직 하나의 매수, 매도 포지션을 가진다고 가정한다. 스프레드 하나를 매수하고 있는데 매수 신호가 생기면 더 매수하지 않는다. 대신 포지션을 뒤바꿀 매수 신호를 기다린다. 단순하지만, 스프레드 트레이딩의 기본을 다루는 데 의미를 둔다. 더 나은 전략으로 포지션 조절 접근이 필요하다.

```R
window_length <- 10
start_date <- "2011-01-01"
end_date <- "2013-12-31"
range <- paste(start_date, "::", end_date, sep = "")

x <- SPY[range, 6]
y <- AAPL[range, 6]

df <- cbind(x, y)
names(df) <- c("x", "y")

beta_out_of_sample <- rolling_beta(diff(df), window_length)

data_out <- merge(beta_out_of_sample, df)
data_out$spread <- data_out
data_out$spread <- data_out$y - lag(beta_out_of_sample, 1) * data_out$x
```

트레이딩 로직은 다음과 같이 요약될 수 있다.

```R
buys <- ifelse(data_out$spread > threshold, 1, 0)
sells <- ifelse(data_out$spread < -threshold, -1, 0)
data_out$signal <- buys + sells
```

매수와 매도 신호를 보자.

```R
plot(data_out$spread, main = "AAPL vs. SPY out of sample", 
     cex.main = 0.8, 
     cex.lab = 0.8, 
     cex.axis = 0.8)

point_type <- rep(NA, nrow(data_out))
buy_index <- which(data_out$signal == 1)
sell_index <- which(data_out$signal == -1)

point_type[buy_index] <- 21
point_type[sell_index] <- 24
points(data_out$spread, pch = point_type)
```

![](https://imgur.com/hDwoNar.png)



매수와 매도 신호가 몇 번 발생했는가?

```R
num_buy_signals <- sum(na.omit(buys))
num_sell_signals <- abs(sum(na.omit(sells)))

num_buy_signals
## [1] 223
num_sell_signals
## [1] 160
```

