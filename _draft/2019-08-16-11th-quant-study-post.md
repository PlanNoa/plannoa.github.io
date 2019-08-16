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
  suppressMessages(getSymbols(symbols, from = from,
    to = to,  src = "yahoo", adjust = TRUE))
}

# 금융 상품 유형 정의
stock(symbols, currency = "USD", multiplier = 1)
```

