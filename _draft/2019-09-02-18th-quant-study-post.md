---
layout: post
title: "Quantitative Trading With R-18"
subtitle: "최적화"
categories: data
tag: quant
comments: true
---

**곡선 맞춤 예제**

x를 만기별 수익률, y를 이자율로 이뤄진 그래프를 생각하자. 목적은 그래프 위의 점들을 지나는 곡선을 찾는 것이다. 이 곡선이 무위험 차익 거래인지는 논의하지 않는다. 단순히 적당한 곡선을 찾는 예제다.

```R
rates = c(0.025, 0.03, 0.034, 0.039, 0.04,
  0.045, 0.05, 0.06, 0.07, 0.071,
  0.07, 0.069, 0.07, 0.071, 0.072,
  0.074, 0.076, 0.082, 0.088, 0.09)
maturities = 1:20

plot(maturities, rates, xlab = "years",
  main = "Yields",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
grid()
```

![](https://imgur.com/T6sCihL.png)

이 그래프를 간단한 5차 다항식으로 근사할 수 있다.

```R
poly_5 <- function(x, p) {
  f <- p[1] + p[2] * x + p[3] * x^2 +
    p[4] * x^3 + p[5] * x^4 + p[6] * x^5
  return(f)
}

obj_5 <- function(x, y, p) {
  error <- (y - poly_5(x, p)) ^ 2
  return(sum(error))
}

out_5 = optim(obj_5, par = c(0, 0, 0, 0, 0, 0),
  x = maturities, y = rates)

out_5
## $par
## [1]  2.4301235956099e-02  1.3138951147963e-03
## [3]  5.5229326602931e-04  7.5685740385076e-07
## [5] -4.2119475163787e-06  1.5330958809806e-07

## $value
## [1] 0.00017311660458207

## $counts
## function gradient
##    501       NA

## $convergence
## [1] 1

## $message
## NULL

lines(poly_5(maturities, out_5$par), lwd = 1.5, lty = 2)
```

![](https://imgur.com/O3m7QTh.png)

데이터에 근접한 선을 얻은 것처럼 보이지만 더 개선할 수 있다. 위 다항식은 그래프 중간에 튄 점을 반영할 수 없다. 더 높은 차수의 다항식을 이용해보자.

```R
poly_7 <- function(x, p) {
  f <- p[1] + p[2] * x + p[3] * x^2 +
    p[4] * x^3 + p[5] * x^4 +
    p[6] * x^5 + p[6] * x^6 +
    p[7] * x^7
  return(f)
}

obj_7 <- function(x, y, p) {
  error <- (y - poly_7(x, p)) ^ 2
  return(sum(error))
}

out_7 <- optim(obj_7, par = c(0, 0, 0, 0, 0, 0, 0, 0),
  x = maturities, y = rates)

lines(poly_7(maturities, out_7$par), lwd = 1.5, lty = 3)
```

![](https://imgur.com/qzLuBMO.png)

조금 개선되었지만 훨씬 나은 정도는 아니다. 다항식의 차수를 올려 거의 같게 만들 수 있지만, 그러면 과적합이 발생할 수 있다.

대안으로 그래프를 통해 10년 주변에서 어떤 국면 전환이 발생했을지 모른다는 점을 생각하는 것이다. 두 다항식을 붙여 연속된 곡선을 구한다면 연속된 문제가 줄어들 것을 기대할 수 있다.

```R
poly_5 <- function(x, a) {
  f <- a[1] + a[2] * x + a[3] * x ^ 2 +
    a[4] * x ^ 3 + a[5] * x ^ 4 +
    a[6] * x ^ 5
return(f)
}

poly_3 <- function(x, offset, intercept, b) {
  f <- intercept + b[1] * (x - offset) +
    b[2] * (x - offset) ^ 2 +
    b[3] * (x - offset) ^ 3
  return(f)
}

obj_3_5 <- function(x, y, offset, p) {

  fit <- rep(Inf, length(x))
  ind_5 <- x <= offset
  ind_3 <- x > offset

  fit[ind_5] <- poly_5(x[ind_5], p[1:6])
  fit[ind_3] <- poly_3(x[ind_3], offset,
    poly_5(offset, p[1:6]), p[7:9])

  error <- (y - fit) ^ 2
  return(sum(error))
}

```

만기가 9일때 두 다항식이 값을 갖게 한다. 3차 다항식의 x축에 보정 값을 사용해 이를 해결할 수 있다. 절편은 5차 다항식의 값과 일치해야 한다.

```R
offset <- 9
out_3_5 <- optim(obj_3_5, par = rep(0, 9),
  x = maturities, y = rates, offset = offset)

plot(maturities, rates, xlab = "years",
  main = "Yields",
  cex.main = 0.8,
  cex.lab = 0.8,
  cex.axis = 0.8)
grid()
lines(poly_5(maturities[maturities <= offset],
  out_3_5$par[1:6]), lwd = 2)
lines(c(rep(NA, offset),
  poly_3(maturities[maturities > offset], offset,
  poly_5(offset, out_3_5$par[1:6]),
  out_3_5$par[7:9])), lwd = 2)
abline(v = offset)
```

![](https://imgur.com/sDJssAI.png)

앞의 방법은 다소 억지이기 때문에, 더 좋은 방법을 많이 찾을 수 있다. smoothing splines을 이용하는 것도 하나의 방법이다. 데이터에 지역별로 가중치를 둔 회귀 분석 방법을 적용하는 것이다. `loess()` 함수로 수행할 수 있다.

```R
obj <- loess(rates ~ maturities, span = 0.5)

plot(maturities, rates, main = "Rates", cex.main = 0.8)
lines(predict(obj), lty = 2)
```

![](https://imgur.com/Fx3DPEd.png)

span 인자는 local fit의 유연한 정도를 조절한다. 



**포트폴리오 최적화**

최적화는 주어진 조건에서 투자 포트폴리오를 구성하는 문제에도 적용된다. 다음 최적화 예제는 가이 욜린의 작업을 기반을 둔다.

최적의 리스크-보상 특성을 가진 주식 포트폴리오를 찾는 일은 흥미로운 주제다. 논문에서 자주 언급되는 주제는 최소 분산 포트폴리오인데, 이는 이차 계획법을 계산해 얻어진다.

전체 포트폴리오의 수익을 조건으로 포트폴리오 리스크의 총합을 최소화해야 한다. 포트폴리오에 들어가는 각 주식의 가중치에도 제한을 둘 수 있다. 이런 이차 계획법은 매우 효율적인 방법으로 풀 수 있다는 것이 증명되었다. R의 quadprog 패키지 안에 일반적인 이차 계획법을 풀 수 있는 `solve.QP()` 가 있다.

여러 목적 함수의 최소화 문제는 명쾌한 해법이 없는 경우가 많다. 이런 경우 DEoptim 패키지에서 제공하는 더 복잡한 최적화 방법을 사용해야 한다. DE는 간단하지만 강력한 함수 최적화 툴로 다차원 다중 모달 함수의 최적화 문제를 푸는 데 이상적이다.

우리는 주식 리스트에서 각 주식의 최적 비율을 찾아 포트폴리오의 최대 손실을 최소화하는 일이다. 가중치의 합은 100%가 되어야 하며, 2%보다 낮은 가중치는 0으로 설정한다.

```R
install.packages("DEoptim")

require(DEoptim)

compute_drawdown <- function(x, returns_default = TRUE,
  geometric = TRUE) {
  if(returns_default) {
    if(geometric) {
      cumulative_return <- cumprod(1 + x)
    } else {
      cumulative_return <- 1 + cumsum(x)
    }
    max_cumulative_return <- cummax(c(1, cumulative_return))[-1]
    drawdown <- -(cumulative_return / max_cumulative_return - 1)
  } else {
    cumulative_pnl <- c(0, cumsum(x))
    drawdown <- cummax(cumulative_pnl) - cumulative_pnl
    drawdown <- drawdown[-1]
  }
  return(drawdown)
}

obj_max_drawdown <- function(w, r_matrix, small_weight) {

  portfolio_return <- r_matrix %*% w

  drawdown_penalty <- max(compute_drawdown(portfolio_return))

  weight_penalty <- 100 * (1 - sum(w)) ^ 2

  negative_penalty <- -sum(w[w < 0])

  small_weight_penalty <- 100 * sum(w[w < small_weight])

  obj <- drawdown_penalty + weight_penalty +
    negative_penalty + small_weight_penalty
  return(obj)
}

symbol_names <- c("AXP", "BA", "CAT", "CVX",
  "DD", "DIS", "GE", "HD", "IBM",
  "INTC", "KO", "MMM", "MRK",
  "PG", "T", "UTX", "VZ")

require(quantmod)
getSymbols(symbol_names)

price_matrix <- NULL
for(name in symbol_names) {
  price_matrix <- cbind(price_matrix, get(name)[, 6])
}

colnames(price_matrix) <- symbol_names

returns_matrix <- apply(price_matrix, 2, function(x) diff(log(x)))

small_weight_value <- 0.02

lower <- rep(0, ncol(returns_matrix))
upper <- rep(1, ncol(returns_matrix))

optim_result <- DEoptim(obj_max_drawdown, lower, upper,
  control = list(NP = 400, itermax = 1000, F = 0.25, CR = 0.75),
  returns_matrix, small_weight_value)
```

다음은 최적화가 끝나면 콘솔 화면에 보여주는 결과다.

```R
Iteration: 1000 bestvalit: 0.533454 bestmemit:    0.020017    0.020064    0.020025    0.057682    0.020002    0.162867    0.020002    0.064668    0.115124    0.162164    0.047037    0.022005    0.020069    0.108919    0.032042    0.047364    0.058079
```

DEoptim() 함수의 출력물은 리스트로 이뤄진 리스트다. optim 리스트에 관심 있는 매개변수들이 있다. 리스트 안의 bestmem 요소에 최적의 비중이 저장된다.

```R
weights <- optim_result$optim$bestmem
```

비중이 0 혹은 2 이상인 점이 흥미롭다. 이는 비중이 작을 때 벌을 주는 부분이 목적 함수에 추가되었기 때문이다. 모든 비중의 합이 0이라는 것도 증명할 수 있다.

```R
sum(weights)
## [1] 0.9981285
```

비중의 합이 정확히 1이 되도록 조절할 수 있다.

```R
weights <- weights/sum(weights)
```

푸펀된 비중과 동일 가중 포트폴리오의 누적 수익을 살펴보자. 의도한 대로 추천된 포트폴리오의 최대 손실이 실제로 작다면 DEoptim의 결과가 입증되는 셈이다.

```R
equal_weights <- rep(1 / 17, 17)
equal_portfolio <- returns_matrix %*% equal_weights
equal_portfolio_cumprod <- cumprod(1 + equal_portfolio)

optimized_portfolio <- returns_matrix %*% weights
drawdown_portfolio_cumprod <- cumprod(1 + optimized_portfolio)

main_title <- "Equal vs. Optimized Weights"
plot(drawdown_portfolio_cumprod, type = 'l', xaxt = 'n',
  main = main_title, xlab = "", ylab = "cumprod(1 + r)")
lines(equal_portfolio_cumprod, lty = 3)
grid(col = 'black')

label_location <- seq(1, length(drawdown_portfolio_cumprod),
  by = 90)
labels <- rownames(returns_matrix)[label_location]
axis(side = 1, at = label_location, labels = labels,
  las = 2, cex.axis= 0.8)
```

![](https://imgur.com/Dg7Im0u.png)

다음은 두 포트폴리오의 최대 손실이다.

```R
# 동일 가중
max(compute_drawdown(equal_portfolio))
## 0.615511

# 최적화 포트폴리오
max(compute_drawdown(optimized_portfolio))
## 0.5338488
```

최적화된 포트폴리오가 나은 누적 복리 수익을 보여준다. 이 분석은 샘플 내에서만 실행되었기 때문에, 최적화된 비중이 정말로 더 나은지는 같은 분석을 샘플 밖의 데이터에도 수행하야 하고, 견고함과 타당성을 검증하기 위해 세 번째 데이터도 필요하다.

차분 진화 알고리즘은 실행될 때마다 다른 결과를 보여준다는 점을 인식하는 것이 중요하다. 때문에 포트폴리오를 구성할 때는 특정 최적화 알고리즘의 결과를 그대로 받아들이면 안되고 추가적인 주의를 기울여 성과를 확인해야 한다.



이 글 Quantitative Trading with R 시리즈는 책 [Quantitative trading with R](https://www.amazon.com/Quantitative-Trading-Understanding-Mathematical-Computational/dp/1137354070)을 보고 공부한 내용들입니다.