---
layout: post
title: "Quantitative Trading With R-18"
subtitle: "실행 시간 개선"
categories: data
tag: quant
comments: true
---

**실행 시간 개선**

R은 효율적으로 알고리즘들이 구현돼 있지만 특정 분야에 적용하기에는 느리다. 일반적으로 모든 프로그래머는 언젠가는 효율성과 실행 속도 문제에 부딪힌다. 물론 기능이 정상적으로 작동하는 것이 개발자의 최우선 목표가 된다. 최적화는 그 다음이다.

이상적으로 지금부터 작성하는 모든 프로그램은 완벽한 프로그램이라고 가정한다. 

R에는 실행 속도 문제를 해결하는 몇 가지 방법이 있다. 첫 번째는 더 좋은 코드를 짜는 것이다. 대표적으로 for문이 있다. 다음 코드는 실행이 오래 걸린다.

```R
sum_with_loop_in_r <- function(max_value) {
    sum <- 0
    for(i in 1:max_value) {
        sum <- sum + i
    }
}
```

벡터화를 적용하면 속도 차이를 확인할 수 있다.

```R
sum_with_vectorization_in_r <- function(max_value) {
    numbers <- as.double(1:max_value)
    return(sum(numbers))
}
```



**R 코드 벤치마킹**

microbenchmark 라이브러리를 사용하면 두 가지 방법의 상대적인 시간 차이를 테스트할 수 있다.

```R
library(microbenchmark)
microbenchmark(loop = sum_with_loop_in_r(1e5),
  vectorized = sum_with_vectorization_in_r(1e5))

## Unit: microseconds
##        expr     min      lq       mean  median      uq     max neval
##        loop 1961640 2064708 2127829.45 2080901 2125624 2834508   100
##  vectorized       0       0   21497.92     514    2571 1961126   100
```

R코드 대부분은 스크립트 언어이다. compiler 패키지는 R에서 특정 기능을 컴파일해 더 빨리 실행할 수 있게 만들어준다.

```R
library(compiler)
compiled_sum_with_loop_in_r <- cmpfun(sum_with_loop_in_r)

microbenchmark(loop = sum_with_loop_in_r(1e5),
  compiled = compiled_sum_with_loop_in_r(1e5),
  vectorized = sum_with_vectorization_in_r(1e5))

## Unit: nanoseconds
##        expr     min      lq       mean  median      uq     max neval
##        loop 1945704 2007135 2175580.25 2077046 2208645 3540308   100
##    compiled 1914347 1984002 2099808.22 2062653 2108146 3000034   100
##  vectorized       0     514    4297.59    1542    3085  126972   100
```

컴파일을 하면 for문보다는 빠르지만 벡터화보다는 느리다. 이미 컴파일된 언어나 내부적으로 컴파일 언어로 구현된 함수에는 효능이 없다.



**Rcpp 솔루션**





이 글 Quantitative Trading with R 시리즈는 책 [Quantitative trading with R](https://www.amazon.com/Quantitative-Trading-Understanding-Mathematical-Computational/dp/1137354070)을 보고 공부한 내용들입니다.