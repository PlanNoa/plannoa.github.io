---
layout: post
title: "Quantitative Trading With R-16"
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





이 글 Quantitative Trading with R 시리즈는 책 [Quantitative trading with R](https://www.amazon.com/Quantitative-Trading-Understanding-Mathematical-Computational/dp/1137354070)을 보고 공부한 내용들입니다.