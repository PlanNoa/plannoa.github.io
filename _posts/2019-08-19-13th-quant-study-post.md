---
layout: post
title: "Quantitative Trading With R-13"
subtitle: "고빈도 데이터"
categories: data
tag: quant ts
comments: true
---

**고빈도 데이터**

지금까지 다룬 데이터는 일간 주식 데이터뿐이다. 이런 데이터는 시간 간격이 동일하다는 성질을 지닌다. 반면 틱 데이터는 본질적으로 시간이 균일하지 않다. 호가창의 거래와 갱신, 거래소 메시지는 임의의 시간 간격으로 도착한다.

고빈도 데이터는 많은 것을 제공한다 거래소에서 나오는 정보를 모두 분석하면 시장 미시 구조에 대한 많은 정보를 얻을 수 있다. 유동성이 큰 종목들은 일중 수십만 건의 시장 데이터와 거래 데이터가 생성괸다. SPY ETF가 대표적인 예시이다. 통상적인 거래일에 호가창은 천만에서 이천만 번 갱신된다. 하지만 이 중 일부에서만 거래가 일어난다. 고빈도 거래를 비판하는 사람들은 광대한 정보를 주고받지만 실제 거래는 매우 적게 일어난다는 점을 비판한다.

호가창 갱신이란 호가창의 어떤 수준에서 가격, 수량 등이 변하는 것을 말한다. 대부분의 시장 조성자나 트래이더들은 호가창의 최상단에서 벌어지는, 가치 있는 정보들에 관심을 가진다.

고빈도 데이터는 구하기 어렵다. 이런 데이터를 얻는 데는 복잡한 노력이 필요하다. 헤지펀드, 은행, 자기 자본 거래 회사는 거래소와 전용망으로 연결되어 있다. 이는 실시간으로 데이터를 얻고 정제해 저장하기가 용이하다.



**틱 데이터를 구할 수 없으므로 8장 뒤에 나오는 ```highfrequency```패키지를 사용해 보겠습니다.**

[이곳](http://highfrequency.herokuapp.com/)의 설명을 참조했습니다.



**```highfrequency``` 패키지**

고빈도 데이터 분석의 경제적 가치는 이제 학계와 금융 세계에서 인정받고 있다. 고빈도 데이터는 일일 모니터링 및 예측, 포트폴리오 할당 프로세스와 고빈도 거래의 기초이다. ```highfrequency```패키지는 고빈도 데이터를 쉽게 다루기 위해 유동성, 상관 관계, 계산, 예측 등에 관한 기술을 제공한다.

고주파 데이터는 그 특성 때문에 다루는 것이 어려울 수 있다. ```highfrequency```패키지는 고빈도 데이터의 세 가지 과제를 해결한다.

1. 수백만 건의 광대한 관측치
2. 데이터 간 불규칙한 시간 간격
3. 데이터가 품고 있는 오류들

```highfrequency```패키지는 고빈도 거래 및 데이터를 관리하기 위해 인터페이스를 제공한다. ```highfrequency```의 도구는 대부분의 데이터 입력에서 작동한다. 이 패키지는 xts패키지가 제공하는 내용을 기반으로 하므로 xts객체를 다룰 때 가장 효율적으로 기능을 사용할 수 있다.

CRAN에 안정된 버전의 패키지가 있고, install.packages()명령어로 다운받을 수 있다.

```R
install.packages("highfrequency")
require(highfrequency)
```



**고빈도 데이터 구성**

고빈도 데이터를 xts로 저장할 시스템이 있다면 이 단계를 건너뛰어도 된다. ```highfrequency```패키지는 NYSE, TAQ, WRDS, Returns, Bloomberg 등의 다양한 입력을 받을 수 있다. 이 패키지는 여러 데이터 형식을 xts로 바꾸는 기능을 제공한다.

고빈도 데이터는 일반적으로 거래의 정보를 담고 있는 각각의 시세 파일 둘로 나뉘어져 있다. ```highfrequency```패키지는 NYSE TAQ 데이터베이스의 .txt, WRDS 데이터베이스의 .csv, Tickdata.inc의 .asc 세 가지 형식의 xts파일을 파싱할 수 있다.

```~/raw_data``` 폴더에 "2008-01-02", "2008-01-03" 두 개의 폴더가 있다고 해 보자. 또한 이 폴더들 안에는 NYSE에서 구매한 ```AAPL_trades.txt``` 와 ```AA_trades.txt``` 파일이 있다. 원시 데이터는 다음과 같이 변환할 수 있다.

```R
from = "2008-01-02";
to = "2008-01-03";
datasource = "~/raw_data";
datadestination = "~/xts_data";

convert(from, to, datasource, datadestination, trades=TRUE,
          quotes=FALSE,ticker=c("AA","AAPL"), dir=TRUE, extension="txt",
          header=FALSE,tradecolnames=NULL,quotecolnames=NULL,
          format="%Y%m%d %H:%M:%S");
```

여기서 ```~/raw_data``` 폴더에 일일 데이터를 가진 ```AAPL_trades.RData``` 와 ```AA_trades.RData``` 가 포함된 "2008-01-02", "2008-01-03" 폴더가 생길 것이다. 이 데이터들은 ```TAQLoad``` 함수로 불러올 수 있다. 

같은 폴더에 `IBM_quotes.csv` 와 `IBM_trades.csv`파일이 있다고 하자. 두 파일은 "2011-12-01" 부터 "2011-12-02"까지의 IBM 거래 데이터를 가지고 있다. 이 데이터는 다음과 같이 변환될 수 있다.

```R
from = "2011-12-01"; 
to = "2011-12-02"; 
datasource = "~/raw_data";
datadestination = "~/xts_data";
convert( from=from, to=to, datasource=datasource, 
              datadestination=datadestination, trades = T,  quotes = T, 
              ticker="IBM", dir = TRUE, extension = "csv", 
              header = TRUE, tradecolnames = NULL, quotecolnames = NULL, 
              format="%Y%m%d %H:%M:%S", onefile = TRUE )
```

이제 `~/xts_data` 에 "2011-12-01" ,"2011-12-02" 두 개의 폴더가 있다고 하자. 각각의 폴더는  `IBM_trades.RData` 이라는 파일을 가지고 있다. in which the trades for that day can be found and a file `IBM_quotes.RData`. 마지막으로 `TAQLoad` 함수는 데이터를 R 작업창으로 읽어올 수 있다.

```R
xts_data = TAQLoad( tickers="IBM", from="2011-12-01",
           to="2011-12-02",trades=F,
           quotes=TRUE, datasource=datadestination)
head(xts_data)

                    SYMBOL EX  BID      BIDSIZ OFR      OFRSIZ MODE
2011-12-01 04:00:00 "IBM"  "P" "176.85" "1"    "188.00" " 1"   "12"
2011-12-01 04:00:17 "IBM"  "P" "185.92" "1"    "187.74" " 1"   "12"
2011-12-01 04:00:18 "IBM"  "P" "176.85" "1"    "187.74" " 1"   "12"
2011-12-01 04:00:25 "IBM"  "P" "176.85" "1"    "187.73" " 1"   "12"
2011-12-01 04:00:26 "IBM"  "P" "176.85" "1"    "188.00" " 1"   "12"
2011-12-01 04:00:26 "IBM"  "P" "176.85" "1"    "187.74" " 1"   "12"
```

Tickdata.inc의 .asc 파일도 쉽게 변환할 수 있다. `~/raw_data`폴더에 `GLP_quotes.asc` 와`GLP_trades.asc` 파일이 있다고 하자.

```R
from = "2011-01-11"; 
to     = "2011-03-11"; 
datasource = "~/raw_data"; 
datadestination = "~/xts_data";
convert(from=from, to=to, datasource=datasource, 
             datadestination=datadestination, trades = TRUE, 
             quotes = TRUE, ticker="GLP", dir = TRUE, format = "%d/%m/%Y %H:%M:%OS",
             extension = "tickdatacom", header = TRUE,  onefile = TRUE );
```

이런 경우에 `~/xts_data` 폴더에는 `GLP_trades.RData` 과 `GLP_quotes.RData` 파일을 가진 "2011-01-11", "2011-02-11", "2011-03-11" 세 개의 폴더가 있다. 이도 ```TAQLoad``` 를 이용해 읽을 수 있다.

```R
options("digits.secs"=3); #Show milliseconds
xts_data = TAQLoad(tickers="GLP", from="2011-01-11", to="2011-01-11",
           trades=T, quotes=F, 
           datasource=datadestination)
head(xts_data)

                        SYMBOL EX  PRICE     SIZE   COND  CORR G127
2011-01-11 09:30:00.338 "GLP"  "T" "18.0700" " 500" "O X" "0"  ""  
2011-01-11 09:30:00.338 "GLP"  "T" "18.0700" " 500" "Q"   "0"  ""  
2011-01-11 09:33:49.342 "GLP"  "T" "18.5000" " 150" "F"   "0"  ""  
2011-01-11 09:39:29.280 "GLP"  "N" "19.2000" "4924" "O"   "0"  ""  
2011-01-11 09:39:29.348 "GLP"  "D" "19.2400" " 500" "@"   "0"  "T" 
2011-01-11 09:39:29.411 "GLP"  "N" "19.2400" " 200" "F"   "0"  "" 
```

패키지에서 제공하는 샘플 데이터를 이용할 수 있다.

```R
library(highfrequency);
data("sample_tdataraw");
head(sample_tdataraw);
data("sample_qdataraw");
head(sample_qdataraw);
```



**고빈도 데이터 다루기**

여러 이유로 원시 거래 데이터는 수치 에러를 가지고 있다. 이 상태의 데이터는 데이터 분석에 맞지 않고, 정제 작업은 필수적이다. `highfrequency` 패키지는 Barndorff-Nielsen의 단계별 정제를 구현했다. 밑의 표 1에는 제공하는  정제 함수의 목록이 있다. 사용자는 하나의 함수를 사용하거나 여러 정제 작업을 합쳐놓은 'wrapper' 함수를 사용할 수 있다. wrapper 함수는 원시 데이터를 불러오고 정제된 데이터를 하드 디스크에 다시 저장한다. 정제할 데이터가 여러 개인 경우 이 기능을 사용하기 좋다.

정제 과정에서 데이터에 대한 정보를 위해 남은 데이터의 관측량을 제공한다.

```R
data("sample_tdataraw");
dim(sample_tdataraw);
[1] 48484     7

tdata_afterfirstcleaning = tradesCleanup(tdataraw=sample_tdataraw,exchanges="N");
tdata_afterfirstcleaning$report;

      initial number       no zero prices      select exchange 
               48484                48479                20795 
     sales condition merge same timestamp 
               20135                 9105 

dim(tdata_afterfirstcleaning$tdata)
[1] 9105    7
```

![](https://imgur.com/Gj6isTP.png)



**고빈도 데이터의 집합**

고빈도 데이터의 가격은 일반적으로 같은 간격으로 기록되지 않는다. 또한 가격은 종종 다른 시점에 관찰되는 반면 대부분의 다변량 추정치는 동기화된 데이터에 의존한다. 이런 불규칙적이고 비동기적인 시간 간격을 동기적이고 정해진 간격을 가진 타임 그리드로 강제할 수 있는 방법이 있다.

가장 인기있는 방법인 previous tick aggregation은 각 그리드 이전의 마지막 가격을 가져와 등거리 그리드로 만든다. `highfrequency` 는 빠르고 쉬운 previous tick aggregation을 제공한다. 

```R
library("highfrequency")
# Load sample price data
data("sample_tdata")
ts = sample_tdata$PRICE

tsagg5min = aggregatets(ts,on="minutes",k=5)
head(tsagg5min)

                      PRICE
2008-01-04 09:35:00 193.920
2008-01-04 09:40:00 194.630
2008-01-04 09:45:00 193.520
2008-01-04 09:50:00 192.850
2008-01-04 09:55:00 190.795
2008-01-04 10:00:00 190.420

tsagg30sec = aggregatets(ts,on="seconds",k=30);
tail(tsagg30sec);

                      PRICE
2008-01-04 15:57:30 191.790
2008-01-04 15:58:00 191.740
2008-01-04 15:58:30 191.760
2008-01-04 15:59:00 191.470
2008-01-04 15:59:30 191.825
2008-01-04 16:00:00 191.670
```

예시에서 가격이 각각 5분과 30초의 규칙적인 시간 그리드로 설정되었다. 추가로 `aggregatets` 함수는 모든 측정 가능한 간격으로 사용할 수 있으며,  `align.by` 와 `align.period` 를 설정하여 호출할 수 있다. 이런 경우에 먼저 가격들에 정규 시간 그리드를 적용한 다음, 정규 기간 동안의 수익을 기반으로 측정 값을 계산한다.

다른 동기화 기능은 갱신 시간이다. 고빈도 데이터에서 함수 `refreshTime` 를 사용하면 시계열을 동기화할 수 있지만 반드시 같은 시간 그리드는 아니다. 갱신 시간은 마지막 갱신 뒤 최소 하나의 거래가 이루어진 시점이다. 

다음은 예제 코드이다.

```R
data("sample_tdata");
data("sample_qdata");

stock1 = sample_tdata$PRICE;
stock2 = sample_qdata$BID;

mPrice_1min = cbind(aggregatePrice(stock1),aggregatePrice(stock2));

mPrice_Refresh = refreshTime(list(stock1,stock2));

rbpcov1 = rBPCov(mPrice_1min,makeReturns=TRUE);
rbpcov2 = rBPCov(mPrice_Refresh,makeReturns=TRUE);

rtscov = rTSCov(list(stock1,stock2));
```



**변동성 측정**

고빈도 데이터의 유용성으로 연구자들은 수익률을 기반으로 사후 실현 변동성을 추정했다. 실제로 다변량 변동성 추정은 자산 관측의 비동시성, positive semidefnite covariance matrix estimator때문에 필요성이 제시되고 있다. `highfrequency` 패키지는 많은 변동성 및 공존성 측정을 구현했다.

 `highfrequency` 가 제공하는 일변량 및 다변량 추정의 개요다.

![](https://imgur.com/flU28Ju.png)

처음부터 두 번째 까지의 열은 기능이 일변량 또는 다변량 가격 시리즈에 적용될 수 있는지를 나타낸다. 그 다음 두 열은 추정값이 점프 및 미세 구조 노이즈에 적합한지를 나타낸다. 그 다음 열은 비동시 가격 시리즈를 입력할 수 있는지, 마지막 열은 추정값이 다변량 케이스에서 positive semidefinite한 행렬을 생성하는지 여부를 나타낸다.

다른 고빈도 데이터의 변동성 측정에서 흥미로운 것은 금융 시장의 개장, 점심과 폐쇄로 일어나는 고빈도 주식의 변동성의 주기성이다.  `highfrequency` 는 일반 주기성 추정과 Boudt et al. 2011b의  jump robust version을 제공한다. 이 추정 방법들은 일별 변동의 표준 편차가 일일 변동 요인과 일일 요소에서 분해 될 수 있다고 가정한다.

다음은 예시 코드이다.

```R
data("sample_real5minprices");

#하루 주기를 계산한다.
out = spotVol(sample_real5minprices,P1=6,P2=4,periodicvol="TML",k=5, dummies=FALSE);
head(out);

                          returns         vol    dailyvol periodicvol
2005-03-04 09:35:00 -0.0010966963 0.004081072 0.001896816    2.151539
2005-03-04 09:40:00 -0.0005614217 0.003695715 0.001896816    1.948379
2005-03-04 09:45:00 -0.0026443880 0.003417950 0.001896816    1.801941
```



이 글 Quantitative Trading with R 시리즈는 책 [Quantitative trading with R](https://www.amazon.com/Quantitative-Trading-Understanding-Mathematical-Computational/dp/1137354070)을 보고 공부한 내용들입니다.