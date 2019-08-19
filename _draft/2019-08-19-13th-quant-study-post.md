---
layout: post
title: "Quantitative Trading With R-13"
subtitle: "고빈도 데이터"
categories: data
tag: quant
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

