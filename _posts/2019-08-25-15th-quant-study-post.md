---

layout: post
title: "Quantitative Trading With R-15"
subtitle: "옵션"
categories: data
tag: quant
comments: true
---

**옵션**

옵션은 다른 자산의 가치로부터 파생되어 거래되는 상품이다. 위키에서는 `매수자가 기초 자산을 특정 날짜나 그 이전에 정해진 가격으로 사거나 팔 권리를 갖는 계약` 이라고 나와있다. 선물과도 비슷하지만, 옵션은 매매할 수 있는 권리를 제공한다. 이런 유연함, 선택성 때문에 옵션 계약은 선물과 다르게 프리미엄이 따른다. 옵션 매수자는 지불하고, 매도자는 받는다.



**옵션의 이론 가치**

옵션 계약의 프리미엄은 몇 가지 변수로 이루어진 함수다. 변수로 매도 가격 K, 현재 가격 S, 옵션 계약 만기까지의 기간 T, 현재 시점과 만기까지 기초 자산으로부터 배당의 형태로 나오는 현금 흐름 d, 무위험 이자율 r, 임시 요소 σ가 있다. 이 임시 요소는 옵션의 내재 변동성이라 불린다. 아래는 옵션의 이론 가치를 구하는 블랙 숄즈 공식이다.

![](https://imgur.com/FMfHPjz.png)

c가 콜옵션 방정식, p가 풋옵션 방정식이다.



**옵션의 역사**

옵션은 고대부터 존재했다. 고대 철학자 탈레스는 올리브 오일 프레스 사용 콜옵션을 거래했고, 훌륭한 수익을 냈다고 알려져 있다. 1848년 시카고 거래소가 문을 열었고, 이 거래소는 아직까지도 가장 크고 오래된 옵션 거래소다.

오랜 옵션의 역사가 있지만 1973년에 가장 큰 발전이 일어났다. 피셔 블랙과 마이런 숄즈, 로버트 머튼이 옵션의 이론 가격을 구하는 수학 공식을 증명했다. 이게 위에서 말한 블랙 숄즈 공식이며, 앞에서 언급한 변수 S, K, T, d, r, σ과 시장 가격 V를 변수로 취한다.

옵션에도 유럽식 옵션과 미국식 옵션이 있다. 유럽식 옵션은 만기일에만 행사가 가능하고 미국식은 만기일 전 언제든지 행사가 가능하다. 이는 가치 평가 측면에서 굉장한 차이를 만든다. 위의 모든 매개 변수가 주어져도 미국식 풋옵션의 가치는 산정하기 어렵다. 수리적으로 해를 계산할 수 있지만 완전한 공식은 없다.



**옵션의 가치**

R에는 유럽식, 미국식 옵션의 가치를 구하는 몇 가지 패키지들이 있다. 이 책에서는 `RQuantLib` 패키지를 사용한다. `RQuantLib` 은 실생활에서 모델링, 트레이딩, 리스크 관리를 하기 위한 라이브러리다.

```R
install.packages("RQuantLib", type="binary")
requirt(RQuantLib)

# 패키지 함수 보기
lsf.str("package:RQuantLib")
```

유럽식 옵션의 가격을 구하기 위해 `EuropeanOption()` 함수를 사용한다. 콜옵션과 풋옵션의 공식은 위에서 언급했다.

유럽식 콜옵션과 풋옵션의 그릭스는 비슷한 공식을 가진다. 실제 예시를 보자.

```R
call_value <- EuropeanOption(type = "call", underlying = 100,
	strike = 100, dividendYield = 0, riskFreeRate = 0.03,
	maturity = 1.0, volatility = 0.30)

Concise summary of valuation for EuropeanOption 
   value    delta    gamma     vega    theta      rho   divRho 
 13.2833   0.5987   0.0129  38.6668  -7.1976  46.5873 -59.8706 
```

call_value 객체의 클래스는 옵션의 가치와 그릭스를 동시에 가진다.

```R
class(call_value)
[1] "EuropeanOption" "Option"    
```

다음 코드는 유럽식 콜옵션의 손익 구조와 그릭스를 살펴볼 수 있게 해준다. 변동성, 만기 기간, 이자율, 배당금에 대해서도 민감도 분석을 수행할 수 있다.

```R
type <- "call"
underlying <- 20:180
strike <- 100
dividendYield <- 0
riskFreeRate <- 0.03
maturity <- 1.0
volatility <- 0.10

# 옵션 가치와 그릭스를 그리는 함수
option_values <- function(type, underlying, strike,
  dividendYield, riskFreeRate, maturity, volatility) {

  out <- list()
  for(i in seq_along(underlying)) {
      out[[i]] <- EuropeanOption(type = type, underlying = i,
        strike = strike, dividendYield = dividendYield,
        riskFreeRate = riskFreeRate, maturity = maturity,
        volatility = volatility)
  }

  par(mfrow = c(3, 2))
  names <- c("Value", "Delta", "Gamma",
    "Vega", "Theta", "Rho")

  for(i in 1:6) {
    plot(unlist(lapply(out, "[", i)) , type = "l",
      main = paste(names[i], "vs. Underlying"),
      xlab = "Underlying", ylab = names[i])
      grid()
      abline(v = strike, col = "red")
  }
  return(out)
}
```

함수를 호출하면 다음과 같이 나온다.

```R
option_values(type, underlying, strike, dividendYield, 
              riskFreeRate, maturity, volatility)

....
[[159]]
Concise summary of valuation for EuropeanOption 
    value     delta     gamma      vega     theta       rho    divRho 
  61.9554    1.0000    0.0000    0.0003   -2.9113   97.0445 -159.0000 

[[160]]
Concise summary of valuation for EuropeanOption 
    value     delta     gamma      vega     theta       rho    divRho 
  62.9554    1.0000    0.0000    0.0002   -2.9113   97.0445 -160.0000 

[[161]]
Concise summary of valuation for EuropeanOption 
    value     delta     gamma      vega     theta       rho    divRho 
  63.9554    1.0000    0.0000    0.0001   -2.9113   97.0445 -161.0000 
```

![](https://imgur.com/Dp4pw3C.png)

옵션의 만기가 감소하면 어떻게 되나 보자.

```R
option_values(type, underlying, strike, dividendYield, 
              riskFreeRate, maturity = 0.1, volatility)
```



![](https://imgur.com/d6ZbDYK.png)



**옵션 거래 데이터 살펴보기**

Tick Data Inc. 에서 데이터를 제공한다고 하는데.. 우린 데이터가 없으니 그냥 예제 코드나 보자.

```R
folder <- "path/Options/SPY_20130415_T/"
available_files <- list.files(folder)

temp <- read.csv(file = paste0(folder, available_files[1]),
  header = FALSE, stringsAsFactors = FALSE)
```

열 이름을 수정한다.

```R
column_names <- c("date", "time", "trade_indicator",
  "sequence_number", "option_exchange_code",
  "option_condition_code", "sale_price", "sale_size",
  "underlying_last_trade_price",
  "underling_last_trade_size",
  "stock_exchange_code", "stock_condition_code",
  "underlying_bid_price", "underlying_bid_size",
  "underlying_ask_price", "underlying_ask_size")
names(temp) <- column_names
```

이 데이터를 가지고 행사 가격과 만기별로 거래 수량을 시각화한다.

```R
output <- list()
for(i in 1:length(available_files)) {
  file_name <- available_files[i]

  type <- substr(file_name, 5, 5)
  date <- substr(file_name, 7, 14)
  date <- as.Date(date, format = "%Y%m%d")
  strike <- substr(file_name, 16, 26)
  strike <- strsplit(strike, "_XX")[[1]][1]

  temp <- read.csv(file = paste0(folder, file_name),
    header = FALSE, stringsAsFactors = FALSE)
  names(temp) <- column_names

  number_of_trades <- nrow(temp)
  avg_trade_price <- round(mean(temp$sale_price,
    na.rm = TRUE), 3)

  if(number_of_trades <= 1) {
    sd_trade_price <- 0
  } else {
    sd_trade_price <- round(sd(temp$sale_price,
      na.rm = TRUE), 3)
  }

  total_volume <- sum(temp$sale_size, na.rm = TRUE)
  avg_underlying_price <- round(mean(
    temp$underlying_bid_price, na.rm = TRUE), 2)
  underlying_range <- max(temp$underlying_ask_price) -
    min(temp$underlying_bid_price)

  output[[i]] <- data.frame(symbol = 'SPY', date = date,
    type = type, strike = strike,
    trades = number_of_trades,
    volume = total_volume,
    avg_price = avg_trade_price,
    sd_price = sd_trade_price,
    avg_stock_price = avg_underlying_price,
    stock_range = underlying_range,
    stringsAsFactors = FALSE)
}

# 테이블로 변환
results <- do.call(rbind, output)

head(results)

  symbol       date type strike trades volume 
 1   SPY 2013-04-20  C  120.00   12    33000
 2   SPY 2013-04-20  C  124.00    1       15
 3   SPY 2013-04-20  C  130.00    1        2
 4   SPY 2013-04-20  C  133.00    1        1
 5   SPY 2013-04-20  C  140.00    1       95
 6   SPY 2013-04-20  C  142.00    1        6

 avg_price sd_price avg_stock_price stock_range
 35.973    0.261      155.74      0.68
 31.380    0.000      155.44      0.01
 26.210    0.000      156.24      0.02
 24.600    0.000      157.58      0.01
 16.465    0.751      156.44      1.51
 13.920    0.000      155.87      0.01
```

작업 가능한 형식으로 행사 가격과 만기별 옵션 거래량의 분포를 살펴보자. 유형별로 나눌 수 있다.

```R
unique_maturities <- unique(results$date)

today <- as.Date("2013-04-15")
days_to_expiration <- as.Date(unique_maturities[1]) - today

# Extract only the relevant maturity range
single_maturity_table <- results[results$date ==
  unique_maturities[1], ]

# Look at the calls and puts separately
calls <- single_maturity_table[
  single_maturity_table$type == "C", ]
puts <- single_maturity_table[
  single_maturity_table$type == "P", ]

par(mfrow = c(2, 1))
plot(calls$strike, calls$volume,
  xlab = "Strike", ylab = "Volume",
  main = "Call volume", cex.main = 0.9)
abline(v = mean(calls$avg_stock_price), lty = 2)
grid()

plot(puts$strike, puts$volume,
  xlab = "Strike", ylab = "Volume",
  main = "Put volume", cex.main = 0.9)
abline(v = mean(puts$avg_stock_price), lty = 2)
grid()
```



**내재 변동성**

옵션 가격은 대부분의 트레이더들이 가장 중요시 여기는 변수가 아니다. 중요한 것은 내재 변동성이다.

블랙 숄즈 모델이 실제 옵션 가격의 움직임을 완벽히 포착하지는 못하지만, 전 세계 대부분의 옵션 트레이더가 사용한다. 정확하지는 않더라도, 유용한 기준이라는 뜻이다.

옵션 호가 데이터는 많은 정보를 가지고 있다. 상장된 옵션은 다양한 행사 가격으로 콜옵션과 풋옵션이 있다. 각 옵션 계약의 호가와 거래 정보가 장중에 기록된다.

예제는 다음 과정을 따른다.

1. 단일 만기에 대해 모든 매수, 매도 호가를 추출한다
2. 호가로 중앙 가격을 계산하고 내재 변동성을 구한다
3. 일일 내재 변동성 미소를 그래프로 그린다
4. 우린 데이터가 없다.

```R
folder <- "Chapter_09/SPY_20130410_QT/"

available_files <- list.files(folder)

temp <- read.csv(file = paste0(folder, available_files[1]),
header = FALSE, stringsAsFactors = FALSE)
head(temp)

column_names <- c("date", "time", "trade_indicator",
  "sequence_number", "option_exchange_code",
  "option_condition_code", "bid_price", "bid_size",
  "ask_price", "ask_size", "stock_exchange_code",
  "stock_condition_code", "underlying_bid_price",
  "underlying_bid_size", "underlying_ask_price",
  "underlying_ask_size")

# 만기가 7/20인 파일 찾기
files_to_use <- available_files[grep("20130720",
  available_files)]

length(files_to_use)
## [1] 142

strikes <- sapply(strsplit(files_to_use, "_"), "[", 4)
type <- sapply(strsplit(files_to_use, "_"), "[", 2)

# 관심 있는 데이터의 열 추출
quote_list <- list()

for(i in 1:length(files_to_use)) {
  temp <- read.csv(file = paste0(folder, files_to_use[i]),
    header = FALSE, stringsAsFactors = FALSE)
  names(temp) <- column_names

  # CBOE 호가만 추출
  filter <- temp$trade_indicator == "Q" &
    temp$option_exchange_code == "C"

  data <- temp[filter, ]

  # xts 객체 생성
  require(xts)
  time_index <- as.POSIXct(paste(data$date, data$time),
    format = "%m/%d/%Y %H:%M:%OS")
  data_filtered <- data[, c("bid_price", "ask_price",
    "underlying_bid_price", "underlying_ask_price")]
  data_filtered$type <- type[i]
  data_filtered$strike <- strikes[i]
  xts_prices <- xts(data_filtered, time_index)

  quote_list[[i]] <- xts_prices
}
```

옵션 호가의 장중 매수-매도 스프레드는 어떻게 생겼을까. 행사가가 158인 등가격 옵션을 보자.

```R
data <- quote_list[[49]]
spread <- as.numeric(data$ask_price) -
  as.numeric(data$bid_price)
plot(xts(spread, index(data)),
  main = "SPY | Expiry = July 20, 2013 | K = 158",
  cex.main = 0.8, ylab = "Quote bid-ask spread")
```

특정 시간에서 행사 가격별로 보자.

```R
time_of_interest <- "2013-04-10 10:30:00::
  2013-04-10 10:30:10"

strike_list <- list()
for(i in 1:length(quote_list)) {
  data <- quote_list[[i]][time_of_interest]
  if(nrow(data) > 0) {
    mid_quote <- (as.numeric(data$bid_price) +
      as.numeric(data$ask_price)) / 2
    mid_underlying <- (as.numeric(data$underlying_bid_price) +
      as.numeric(data$underlying_ask_price)) / 2
    strike_list[[i]] <- c(as.character(index(data[1])),
      data$type[1], data$strike[1], names(quote_list[i]),
      mid_quote[1], mid_underlying[1])
  }
}

# 열 합치기
df <- as.data.frame(do.call(rbind, strike_list),
  stringsAsFactors = FALSE)
names(df) <- c("time", "type", "strike",
  "mid_quote", "mid_underlying")

head(df)

plot(as.numeric(df$strike), as.numeric(df$mid_quote),
  main = "Option Price vs. Strike for Calls and Puts",
  ylab = "Premium",
  xlab = "Strike",
  cex.main = 0.8)
grid()
```

책의 그래프를 보면 콜옵션과 풋옵션의 프리미엄을 같이 나타냈다. 내재 변동성을 보기 위해 데이터를 필터링해 외가격 옵션 데이터만 유지한다. 옵션 가격에 맞는 내재 변동성을 구하는 일은 어렵다. 지금도 많은 연구가 있고 회사들은 이를 성공적인 변동성 트레이딩을 위한 핵심 단계로 여긴다.

```R
# 외가격 옵션 필터링
otm_calls <- df$type == "C" & df$mid_underlying <= df$strike
otm_puts <- df$type == "P" & df$mid_underlying > df$strike
otm <- df[otm_calls | otm_puts, ]

# 행사 가격으로 정렬
otm <- otm[order(otm[, "strike"]), ]
plot(otm$strike, otm$mid_quote,
  main = "OTM prices",
  xlab = "Strike",
  ylab = "Premium",
  cex.main = 0.8)
grid()
```

SPY 옵션은 미국식 옵션이기 때문에 변동성을 위해서는 몬테카를로 접근이나 트리, 유한 차분법을 사용해야 한다. `RQuantLib`의 `AmericanOption()` 이나 `ImpliedVolatility()` 함수가 이에 사용된다. 배당률과 무위험 이자율은 임의로 선택한다. 실제에서는 이런 매개변수 선택이 올바른 결과를 얻는 데 핵심적이다. 입력한 미래 가격이 적절하지 않으면 풋옵션과 콜옵션의 내재 변동성이 연속하지 않는다.

```R
# 외가격 옵션으로 내재 변동성 계싼
otm$iv <- NA
for(i in 1:nrow(otm)) {
  type <- ifelse(otm$type[i] == "C", "call", "put")
  value <- as.numeric(otm$mid_quote[i])
  underlying <- as.numeric(otm$mid_underlying[i])
  strike <- as.numeric(otm$strike[i])
  dividendYield <- 0.03
  riskFreeRate <- 0.02
  maturity <- 101/252
  volatility <- 0.15
  otm$iv[i] <- AmericanOptionImpliedVolatility(type,
    value, underlying, strike,dividendYield,
    riskFreeRate, maturity, volatility)$impliedVol
}

plot(otm$strike, otm$iv,
  main = "Implied Volatility skew for SPY on April 10, 2013 10:30 am",
  xlab = "Strike",
  ylab = "Implied Volatility",
  cex.main = 0.8)
grid()
```



이 글 Quantitative Trading with R 시리즈는 책 [Quantitative trading with R](https://www.amazon.com/Quantitative-Trading-Understanding-Mathematical-Computational/dp/1137354070)을 보고 공부한 내용들입니다.