---
layout: post
title: "Quantitative Trading With R-12"
subtitle: "두 번째 전략: 누적 Connors RSI"
categories: data
comments: true
---

**두 번째 전략: 누적 Connors RSI**

교재의 단순한 전략이 아닌 좀 더 현실성 있는 전략을 테스트해보자. 사용할 지표는 Connors RSI(3, 2, 100)이다. 지표는 가격 움직임의 RSI-3, 연속된 가격 차이의 RSI-2(가격이 3일 연속 상승하면 1, 2, 3이 된다.), 가장 최근 수익률의 100일 % 순위를 인자로 가진다. 이 3가지 값들의 평균을 취한다. 이는 전통적인 RSI 지표의 최근 버전이다. 전략은 누적된 합이 Connors RSI의 특정 값을 넘어서면 매수한다.

전략은 2일 누적 Connors RSI가 40보다 작고 종가가 200일 평균 이동 선 위에 위치하면 매수한다. 1일 Connors RSI가 75를 하회하면 매도한다. 다음은 새로운 선수 함수다.

```R
connorsRSI <- function(price, nRSI = 3, nStreak = 2,
  nPercentLookBack = 100){
  priceRSI <- RSI(price, nRSI)
  streakRSI <- RSI(computeStreak(price), nStreak)
  percent <- round(runPercentRank(x = diff(log(price)),
    n = 100, cumulative = FALSE, exact.multiplier = 1) * 100)
  ret <- (priceRSI + streakRSI + percents) / 3
  colnames(ret) <- "connorsRSI"
  returns(ret)
}

# 진행 중인 연속된 플러스, 마이너스 수익 계산
computeStreak <- function(priceSeries) {
  signs <- sign(diff(priceSeries))
  posDiffs <- negDiffs <- rep(0,length(signs))
  posDiffs[signs == 1] <- 1
  negDiffs[signs == -1] <- -1

  # 누적 합계와 연속 기간 중 증가하지 않은 합계 벡터 계산
  # na.locf 이후 NA값은 0으로 처리
  posCum <- cumsum(posDiffs)
  posNAcum <- posCum
  posNAcum[posDiffs == 1] <- NA
  posNAcum <- na.locf(posNAcum, na.rm = FALSE)
  posNAcum[is.na(posNAcum)] <- 0
  posStreak <- posCum - posNAcum

  # 마이너스 누적 수익에 대해 반복  
  negCum <- cumsum(negDiffs)
  negNAcum <- negCum
  negNAcum[negDiffs == -1] <- NA
  negNAcum <- na.locf(negNAcum, na.rm = FALSE)
  negNAcum[is.na(negNAcum)] <- 0
  negStreak <- negCum - negNAcum

  streak <- posStreak + negStreak
  streak <- xts(streak, order.by = index(priceSeries))
  return (streak)
}

sigAND <- function(label, data=mktdata,
  columns,  cross = FALSE) {
  ret_sig = NULL
  colNums <- rep(0, length(columns))
  for(i in 1:length(columns)) {
    colNums[i] <- match.names(columns[i], colnames(data))
  }
  ret_sig <- data[, colNums[1]]
  for(i in 2:length(colNums)) {
    ret_sig <- ret_sig & data[, colNums[i]]
  }
  ret_sig <- ret_sig * 1
  if (isTRUE(cross))
    ret_sig <- diff(ret_sig) == 1
  colnames(ret_sig) <- label
  return(ret_sig)
}

cumCRSI <- function(price, nCum = 2, ...) {
  CRSI <- connorsRSI(price, ...)
  out <- runSum(CRSI, nCum)
  colnames(out) <- "cumCRSI"
  out
}
```

sigAND 함수는 사용자 정의 신호로 둘 이상의 신호의 교차로 만들어진다. 

다음은 전략의 전체 코드이다.

```R
rm(list = ls(.blotter), envir = .blotter)
  initDate = '1990-01-01'
  from = "2003-01-01"
  to = "2013-12-31"
  initEq = 10000

currency('USD')
Sys.setenv(TZ="UTC")
source("demoData.R")

strategy.st <- "CRSIcumStrat"
portfolio.st <- "CRSIcumStrat"
account.st <- "CRSIcumStrat"

rm.strat(portfolio.st)
rm.strat(strategy.st)

initPortf(portfolio.st, symbols = symbols,
  initDate = initDate, currency = 'USD')

initAcct(account.st, portfolios = portfolio.st,
  initDate = initDate, currency = 'USD',
  initEq = initEq)

initOrders(portfolio.st, initDate = initDate)
strategy(strategy.st, store = TRUE)

# 매개변수
cumThresh <- 40
exitThresh <- 75
nCum <- 2
nRSI <- 3
nStreak <- 2
nPercentLookBack <- 100
nSMA <- 200
pctATR <- .02
period <- 10

# 지표
add.indicator(strategy.st, name = "cumCRSI",
  arguments = list(price = quote(Cl(mktdata)), nCum = nCum,
  nRSI = nRSI, nStreak = nStreak,
  nPercentLookBack = nPercentLookBack),
  label = "CRSIcum")

add.indicator(strategy.st, name = "connorsRSI",
  arguments = list(price = quote(Cl(mktdata)), nRSI = nRSI,
  nStreak = nStreak,
  nPercentLookBack = nPercentLookBack),
  label = "CRSI")

add.indicator(strategy.st, name = "SMA",
  arguments = list(x = quote(Cl(mktdata)), n = nSMA),
  label = "sma")

add.indicator(strategy.st, name = "lagATR",
    arguments = list(HLC = quote(HLC(mktdata)), n = period),
    label = "atrX")

# 신호
add.signal(strategy.st, name = "sigThreshold",
  arguments = list(column = "cumCRSI.CRSIcum",
  threshold = cumThresh, relationship = "lt", cross = FALSE),
  label="cumCRSI.lt.thresh")

add.signal(strategy.st, name = "sigComparison",
  arguments = list(columns = c("Close", "SMA.sma"),
  relationship = "gt"), label = "Cl.gt.SMA")

add.signal(strategy.st, name = "sigAND",
  arguments = list(columns = c("cumCRSI.lt.thresh",
  "Cl.gt.SMA"), cross = TRUE), label = "longEntry")

add.signal(strategy.st, name = "sigThreshold",
  arguments = list(column = "connorsRSI.CRSI",
  threshold = exitThresh, relationship = "gt",
  cross = TRUE), label = "longExit")

# 규칙
add.rule(strategy.st, name = "ruleSignal",
  arguments = list(sigcol = "longEntry", sigval = TRUE,
  ordertype = "market", orderside  ="long", replace = FALSE,
  prefer = "Open", osFUN = osDollarATR, tradeSize = tradeSize,
  pctATR = pctATR, atrMod = "X"), type = "enter", path.dep = TRUE)

add.rule(strategy.st, name = "ruleSignal",
  arguments = list(sigcol = "longExit", sigval = TRUE,
  orderqty = "all", ordertype = "market", orderside = "long",
  replace = FALSE, prefer = "Open"), type = "exit", path.dep = TRUE)

# 전략 적용
t1 <- Sys.time()
out <- applyStrategy(strategy = strategy.st,
  portfolios = portfolio.st)
t2 <- Sys.time()
print(t2 - t1)

# 분석 적용
updatePortf(portfolio.st)
dateRange <- time(getPortfolio(portfolio.st)$summary)[-1]
updateAcct(portfolio.st,dateRange)
updateEndEq(account.st)
```

전략에서 sigThreshold 신호가 어떻게 작동하는지 알 수 있다. 한 번이 아니라 전체 기간 동안 발동한다는 점에서 sigCrossover, sigComparison도 사용한다. sigAND를 사용해 사용자 정의 신호를 만들고 앞에서 만든 ATR주문 로직을 다시 사용했다.



**평균 회귀 전략 평가**

이번 전략의 통합된 거래 통계치다.

```R
aggPF <- sum(tStats$Gross.Profits)/-sum(tStats$Gross.Losses)
[1] 3.792043

aggCorrect <- mean(tStats$Percent.Positive)
[1] 35.515

numTrades <- sum(tStats$Num.Trades)
[1] 1158
	
meanAvgWLR <- mean(tStats$Avg.WinLoss.Ratio[
    tStats$Avg.WinLoss.Ratio < Inf], na.rm = TRUE)
[1] 12.92867
```

조금 낮은 승률, 괜찮은 수익, 약 13의 손익비를 가진다. 다음으로 몇 가지 더 중요한 성과 측정 지표를 보자. 첫 번째 통계치들은 전략이 일간으로 어떤 성과를 냈는지 보여준다. 따라서 전략의 수익 팩터는 거래당이 아니라 일간으로 통합된다. 열 번의 플러스 거래에서 10개를 취하면 수익 팩터는 무한대가 된다. 하지만 그런 거래가 하락한 날보다 상승한 날이 많을수록 일간 수익 팩터는 1에 가까워진다.

다음은 일간 통계치 예제다.

```R
Stats <- dailyStats(Portfolios = portfolio.st, use = "Equity")
rownames(dStats) <- gsub(".DailyEndEq", "", rownames(dStats))
print(data.frame(t(dStats)))

                   XLV.DailyEqPL XLY.DailyEqPL
Total.Net.Profit         3844.99       4081.35
Total.Days                315.00        345.00
Winning.Days              169.00        192.00
Losing.Days               146.00        153.00
Avg.Day.PL                 12.21         11.83
Med.Day.PL                 11.98         18.31
Largest.Winner            690.23        632.56
Largest.Loser            -821.22       -782.98
Gross.Profits           21038.19      24586.07
Gross.Losses           -17193.20     -20504.73
Std.Dev.Daily.PL          166.09        177.79
Percent.Positive           53.65         55.65
Percent.Negative           46.35         44.35
Profit.Factor               1.22          1.20
Avg.Win.Day               124.49        128.05
Med.Win.Day                94.55        101.88
Avg.Losing.Day           -117.76       -134.02
Med.Losing.Day            -79.56        -85.41
Avg.Daily.PL               12.21         11.83
Med.Daily.PL               11.98         18.31
Std.Dev.Daily.PL.1        166.09        177.79
Skewness                   -1.01         -1.28
Kurtosis                   56.32         37.19
Ann.Sharpe                  1.17          1.06
Max.Drawdown            -2201.71      -2248.18
Profit.To.Max.Draw          1.75          1.82
Avg.WinLoss.Ratio           1.06          0.96
Med.WinLoss.Ratio           1.19          1.19
Max.Equity               3892.77       4081.35
Min.Equity              -1228.58       -631.80
End.Equity               3844.99       4081.35
```

다음 분석은 기간 측정을 다룬다. 사용자 함수를 정의한다.

```R

```

