---
layout: post
title: "Quantitative Trading With R-11"
subtitle: "백테스팅 방법"
categories: data
comments: true
---

백테스팅은 계량 분석과 퀸트 트레이딩에서 많은 시간을 차지하는 분야다. 이는 시장의 움직임에 대한 가설을 과거 데이터를 이용해 테스트하는 시스템적 방법론을 의미한다. 암묵적으로 과거의 패턴이 미래에도 반복된다 가정하는데, 다시 그런 패턴이 포착되면 활용할 수 있게 하는 것이 최종 목표이다. 대체로 백테스팅은 다음을 따른다.

1. 관측되거나 예측되는 시장 현상에 대해 질문을 던진다.
2. 던져진 질문에 충족하는 답을 찾기 위해 관련 연구를 진행한다.
3. 현상에 대한 가설을 세운다.
4. 가설을 작성해 실제 과거 시장 데이터에 돌려본다.
5. 결과를 보고 이론에 적용시켜본다.



**백테스팅 방법**

백테스팅의 첫 번째는 과거 데이터를 이용해 트레이딩 신호를 생성하는 것이다. 계량 경제학을 비롯한 복잡한 데이터 분석 작업이 이루어진다.

두 번째는 진입, 청산 규칙을 실행한다. 모든 신호에서 거래가 일어나야 하는 것은 아니며, 현재 포지션과 제약 사항, 현재 시장 데이터와 같이 트레이더가 중요하다 여기는 것들을 고려해 거래 여부가 결정된다.

세 번쨔는 거래 전반의 회계를 담당하는 시스템이다. 이는 트레이더가 전략의 전체적인 효율성을 판단하게 해주기 때문에 시스템의 중요한 부분이다. 여기서 포트폴리오의 손익과 두 번째 부분에 영향을 미칠 수 있는 관련 리스크 지표들이 계산된다. 시스템에는 포지션의 세부 사항이 모두 담겨야 한다.

네 번째는 주문 실행이다. 시스템이 실제로 외부와 연결되어야 하는데, 트레이더가 직접 거래소에 접속하는 방법도 있지만 이는 자동화 프로그램으로 해결이 가능하다.

```quantstrat``` 패키지는 퀸트 집단이 자기 자본 거래를 연구하기 위해 만들었다. 몇 줄의 코드로 신호 기반 트레이딩 시스템을 만드는 라이브러리이다. 사용자 지표나 신호, 규칙의 일부를 쉽게 사용할 수 있다.

두 가지 전략을 분석할 것이다. 첫 번째는 안드레아스 클레노의 추세 추종 전략, 두 번째는 로렌스 코너의 오실레이터 기반 전략이다.



**blotter와 PerformanceAnalytics**

```blotter``` 패키지는 관찰 중인 거래의 모든 면을 다루는 회계 분석 엔진이다. ```quantstrat```은 내부적으로 ```blotter```를 많이 활용한다. ```quantstrat```이 주문을 생성하면 ```blotter```가 전체적인 수익성을 추적한다. 이런 추적이 개별 거래 단위에서 상세한 분석을 가능하게 한다.

```PerformanceAnalytics```는 펀드와 금융 자산의 성과, 리스크 특성을 평가하는 데 쓰이는 라이브러리이다. 이 라이브러리의 목적은 일반적인 투자 의사 결정과정에서 성과와 리스크를 쉽게 묻고 답할 수 있게 하는 것이다.

```quantstrat```는 ```PerformanceAnalytics```의 그래픽과 통계 요약 기능을 사용한다. 이는 정규분포 수익률과 비정규분포 수익률을 모두 잘 처리하고 특정 전략의 위험 대비 수익을 고려할 때 주된 도구로 사용된다.



**초기 설치**

```quantstrat```는 트레이딩 전략을 수립하기 위해 필수 표준 코드를 작성해야 한다.

다음 코드는 2003년 이전의 모든 ETF를 다룬다. 이 코드를 demoData.R이라는 파일로 저장하고 필요할 때 불러온다.

```R
# 경고 메시지 숨김
options("getSymbols.warning4.0" = FALSE)

# 기존 데이터 지우기
rm(list = ls(.blotter), envir = .blotter)

# 통화, 시간대 설정
currency('USD')
Sys.setenv(TZ = "UTC")

# 관심 있는 티커 정의
symbols <- c("XLB", #SPDR Materials sector
  "XLE", #SPDR Energy sector
  "XLF", #SPDR Financial sector
  "XLP", #SPDR Consumer staples sector
  "XLI", #SPDR Industrial sector
  "XLU", #SPDR Utilities sector
  "XLV", #SPDR Healthcare sector
  "XLK", #SPDR Tech sector
  "XLY", #SPDR Consumer discretionary sector
  "RWR", #SPDR Dow Jones REIT ETF
  "EWJ", #iShares Japan
  "EWG", #iShares Germany
  "EWU", #iShares UK
  "EWC", #iShares Canada
  "EWY", #iShares South Korea
  "EWA", #iShares Australia
  "EWH", #iShares Hong Kong
  "EWS", #iShares Singapore
  "IYZ", #iShares U.S. Telecom
  "EZU", #iShares MSCI EMU ETF
  "IYR", #iShares U.S. Real Estate
  "EWT", #iShares Taiwan
  "EWZ", #iShares Brazil
  "EFA", #iShares EAFE
  "IGE", #iShares North American Natural Resources
  "EPP", #iShares Pacific Ex Japan
  "LQD", #iShares Investment Grade Corporate Bonds
  "SHY", #iShares 1-3 year TBonds
  "IEF", #iShares 3-7 year TBonds
  "TLT" #iShares 20+ year Bonds
)

# SPDR ETF가 먼저, iShares ETF가 다음으로
if(!"XLB" %in% ls()) {
  # 데이터가 없으면 야후에서 불러옴
  suppressMessages(getSymbols(symbols, from = from, to = to))
}

# 금융 상품 유형 정의
stock(symbols, currency = "USD", multiplier = 1)
```



**첫 번째 전략: 단순한 추세 추종**

이제 첫 번째 전략을 백테스트해본다. 이 전략은 앞에서 만든 데모 데이터 파일을 이용하고 안드레아스 클레노의 단춘 추세 추종 전략에 기반을 둔다.

전략을 위해 일간 데이터를 사용한다. 신호는 R의 ```lag()```로 생성하고, 1년 전보다 가격이 높으면 매도, 가격이 낮으면 매수한다. 상품들 사이 위험을 동일하게 배분하기 위한 10일 ATR을 이용해 주문 주량을 조절하고, 거래당 2%의 위험을 가진다. ATR은 평균 실제 범위를 의미하며, TTR 패키지에서 사용 가능한 지표이다.

다음은 주문 수량을 계산하는 코드다.

```R
"lagATR" <- function(HLC, n = 14, maType, lag = 1, ...) {
  ATR <- ATR(HLC, n = n, maType = maType, ...)
  ATR <- lag(ATR, lag)
  out <- ATR$atr
  colnames(out) <- "atr"
  return(out)
}

"osDollarATR" <- function(orderside, tradeSize, pctATR,
  maxPctATR = pctATR,  data, timestamp,
  symbol, prefer = "Open", portfolio, integerQty = TRUE,
  atrMod = "", rebal = FALSE, ...) {
    if(tradeSize > 0 & orderside == "short"){
      tradeSize <- tradeSize * -1
    }

    pos <- getPosQty(portfolio, symbol, timestamp)
    atrString <- paste0("atr", atrMod)
    atrCol <- grep(atrString, colnames(mktdata))

    if(length(atrCol) == 0) {
      stop(paste("Term", atrString,
      "not found in mktdata column names."))
    }

    atrTimeStamp <- mktdata[timestamp, atrCol]
    if(is.na(atrTimeStamp) | atrTimeStamp == 0) {
      stop(paste("ATR corresponding to", atrString,
      "is invalid at this point in time.  Add a logical
      operator to account for this."))
    }
   
    dollarATR <- pos * atrTimeStamp

    desiredDollarATR <- pctATR * tradeSize
    remainingRiskCapacity <- tradeSize * maxPctATR - dollarATR

    if(orderside == "long"){
      qty <- min(tradeSize * pctATR / atrTimeStamp,
        remainingRiskCapacity / atrTimeStamp)
    } else {
      qty <- max(tradeSize * pctATR / atrTimeStamp,
        remainingRiskCapacity / atrTimeStamp)
    }

    if(integerQty) {
      qty <- trunc(qty)
    }
    if(!rebal) {
      if(orderside == "long" & qty < 0) {
        qty <- 0
      }
      if(orderside == "short" & qty > 0) {
        qty <- 0 
      }
    }
    if(rebal) {
      if(pos == 0) {
        qty <- 0
      } 
    }
  return(qty)
}
```

첫 번째 함수는 지연 ATR을 생성한다. 두 번째 함수는 첫 번쨰 결과를 찾아 거래 규모와 리스크에 맞춰 매수, 매도할 때 가장 작은 정수로 변환해 수량을 계산한다. 예를 들어 거래 규모가 10000이고 2%위험을 감수한다면 매수로 200ATR이 가능하고, 매도로 -200ATR이 가능하다.

다음은 전략 설정이다.

```R
require(quantmod)
require(quantstrat)
require(PerformanceAnalytics)

initDate = "1990-01-01"
from = "2003-01-01"
to = "2013-12-31"
options(width = 70)

source("C:/Users/이희웅/Desktop/demoData.R")

tradeSize <- 10000
initEq <- tradeSize * length(symbols)

strategy.st <- "Clenow_Simple"
portfolio.st <- "Clenow_Simple"
account.st <- "Clenow_Simple"
rm.strat(portfolio.st)
rm.strat(strategy.st)

initPortf(portfolio.st, symbols = symbols, initDate = initDate, currency = 'USD')

initAcct(account.st, portfolios = portfolio.st, 
        initDate = initDate, currency = 'USD', initEq = initEq)

initOrders(portfolio.st, initDate = initDate)

strategy(strategy.st, store = TRUE)
```

먼저 quantstrat와 PerformanceAnalytics 패키지를 불러온다. 그리고 세 개의 날짜 initDate, from, to를 초기화한다. 초기 자본을 설정하고, 이름을 설정한 뒤 전략 환경을 초기화한다. 그 다음 포트폴리오, 계정, 주문을 정해진 순서대로 초기화한다. 마지막으로 앞으로의 사용을 위해 전략 객체를 저장한다.



**첫 번째 전략 백테스팅**

먼저 생성 지표에 사용할 함수와 매개변수를 설정한다.

```R
nLag = 252
pctATR = 0.02
period = 10

namedLag <- function(x, k = 1, na.pad = TRUE, ...) {
  out <- lag(x, k = k, na.pad = na.pad, ...)
  out[is.na(out)] <- x[is.na(out)]
  colnames(out) <- "namedLag"
  return(out)
}
```

252일동안 시간 지연을 사용하고, 2%의 리스크를 가지며 10일 ATR을 사용할 것이다.

```R
add.indicator(strategy.st, name = "namedLag",
  arguments = list(x = quote(Cl(mktdata)), k = nLag),
  label = "ind")

add.indicator(strategy.st, name = "lagATR",
  arguments = list(HLC = quote(HLC(mktdata)), n = period),
  label = "atrX")

test <- applyIndicators(strategy.st, mktdata = OHLC(XLB))
head(round(test, 2), 253)
```

마지막 두 열의 이름은 namedLag.ind와 atr.atrX이다. atrX는 전략에 쓰이는 주문 수량을 적절히 조절하는 데 사용할 열을 찾는 기능을 제공한다.

지표에 추가되는 인자들은 매우 직관적이다. 모든 개별 지표 추가는 네 가지 인자와 함께 ```add.indicator()``` 함수를 통해 호출된다.

1. 지표를 추가할 전략
2. 계산을 수행할 함수의 이름
3. 위 함수의 인자
4. 지표의 이름

신호는 매수 진입, 매수 청산, 매도 진입, 매도 청산이 있다. quantstrat에서는 청산이 진입보다 우선시되기 때문에 진입과 청산이 동시에 발생하면 청산 후 진입이 진행된다.. 매도에서는 청산 후 역매매 전략이 작동한다. 다음은 백테스트에 신호를 추가하는 방법이다.

```R
add.signal(strategy.st, name = "sigCrossover",
  arguments = list(columns = c("Close", "namedLag.ind"),
  relationship = "gt"),
  label = "coverOrBuy")

add.signal(strategy.st, name = "sigCrossover",
  arguments = list(columns = c("Close", "namedLag.ind"),
  relationship = "lt"),
  label = "sellOrShort")
```

신호는 지표와 동일한 형식을 지닌다. 함수 호출, 전략 이름, 필요한 함수의 이름, 인자, 이름으로 구성된다.

```sigCrossover``` 함수는 ```relationship``` 인자를 통해 주어진 첫 번째 열이 두 번째 열을 돌파할 때 알려준다. 여기서 gt는 초과, lt는 미만을 의미한다. 첫 번째 신호는 가격이 지연된 신호를 돌파할 때, 두 번째는 그 반대일 때이다.

트레이딩 시스템 트라이팩터의 마지막 요소는 ```add.rule``` 함수다. ```quantstrat```에서 규칙을 설정하는 방법을 보자.

```R
# 매수 규칙
add.rule(strategy.st, name = "ruleSignal",
  arguments = list(sigcol = "coverOrBuy",
  sigval = TRUE, ordertype = "market",
  orderside = "long", replace = FALSE,
  prefer = "Open", osFUN = osDollarATR,
  tradeSize = tradeSize, pctATR = pctATR,
  atrMod = "X"), type = "enter", path.dep = TRUE)

add.rule(strategy.st, name = "ruleSignal",
  arguments = list(sigcol = "sellOrShort",
  sigval = TRUE, orderqty = "all",
  ordertype = "market", orderside = "long",
  replace = FALSE, prefer = "Open"),
  type = "exit", path.dep = TRUE)

# 매도 규칙
add.rule(strategy.st, name = "ruleSignal",
  arguments = list(sigcol = "sellOrShort",
  sigval = TRUE, ordertype = "market",
  orderside = "short", replace = FALSE,
  prefer = "Open", osFUN = osDollarATR,
  tradeSize = -tradeSize, pctATR = pctATR,
  atrMod = "X"), type = "enter", path.dep = TRUE)

add.rule(strategy.st, name = "ruleSignal",
  arguments = list(sigcol = "coverOrBuy",
  sigval = TRUE, orderqty = "all",
  ordertype = "market", orderside = "short",
  replace = FALSE, prefer = "Open"),
  type = "exit", path.dep = TRUE)
```

대다수의 ```quantstrat```의 규칙에는 ```ruleSignal```이 사용된다. 

중요한 인자들이다.

1. sigcol: 신호를 포함하는 열
2. ordertype: 주로 시장가 주문으로 사용한다. 지정가 주문이나 손절 주문에도 사용할 수 있다.
3. perfer: 월간 데이터까지 모든 시간 빈도에서 ```quantstrat```는 신호가 나온 뒤 다음 봉에 진입하는데, 기본은 종가 매수다. 전략이 일봉을 다룬다면 이를 시가로 바꾸는 것이 바람직하다.
4. orderside: 매수 혹은 매도로 주문 수량 조절 함수에서 사용된다.
5. replace: 항상 TRUE로 설정한다. FALSE면 해당 시간동안 다른 전략을 사용한다.
6. osFUN: ATR 주문 조절 함수가 호출되는 부분이다.
7. type: 주로 진입 혹은 청산.
8. path: ruleSignal에서 항상 TRUE로 설정한다.

이제 전략이 정의됐으니 실행한다.

```R
t1 <- Sys.time()
out <- applyStrategy(strategy = strategy.st,
  portfolios = portfolio.st)

t2 <- Sys.time()
print(t2 - t1)
```

정상적으로 작성되었다면 다음과 같이 나온다.

```R
## [1] "2007-10-22 00:00:00 XLY -655 @ 32.3578893111826"
## [1] "2007-10-22 00:00:00 XLY -393 @ 32.3578893111826"
## [1] "2007-10-23 00:00:00 XLY 393 @ 33.1349846702336"
## [1] "2007-10-23 00:00:00 XLY 358 @ 33.1349846702336"
## [1] "2007-10-25 00:00:00 XLY -358 @ 32.8639048938205"
## [1] "2007-10-25 00:00:00 XLY -333 @ 32.8639048938205"
## [1] "2009-09-30 00:00:00 XLY 333 @ 25.9947501843176"
## [1] "2009-09-30 00:00:00 XLY 449 @ 25.9947501843176"
## [1] "2009-10-02 00:00:00 XLY -449 @ 24.8800203565938"
```



**성과 평가**

이제 실행된 결과의 효율성을 분석할 것이다.

```R
updatePortf(portfolio.st)
dateRange <- time(getPortfolio(portfolio.st)$summary)[-1]
updateAcct(portfolio.st, dateRange)
updateEndEq(account.st)
```

위의 코드는 손익을 계산하고 거래 내역을 만들도록 호출한다. 거래 통계치는 수익의 %비율, 수익 팩터, 평균 손익비 등이 있다.

```R
tStats <- tradeStats(Portfolios = portfolio.st, use = "trades",
  inclZeroDays = FALSE)
tStats[, 4:ncol(tStats)] <- round(tStats[, 4:ncol(tStats)], 2)
  
print(data.frame(t(tStats[,-c(1,2)])))
aggPF <- sum(tStats$Gross.Profits) / -sum(tStats$Gross.Losses)
aggCorrect <- mean(tStats$Percent.Positive)
numTrades <- sum(tStats$Num.Trades)
meanAvgWLR <- mean(tStats$Avg.WinLoss.Ratio[
  tStats$Avg.WinLoss.Ratio < Inf], na.rm = TRUE)
```

tStats 테이블을 보면 XLP가 뛰어났다는 것을 볼 수 있고, 이는 트레이딩 전략이 이 금융 상품에 대하여 대부분의 손실을 회피할 수 있었다는 뜻이다.

다음은 전략의 트레이딩 통계치를 합산하는 방법이다.