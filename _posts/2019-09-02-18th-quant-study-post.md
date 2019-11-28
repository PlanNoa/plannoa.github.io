---
layout: post
title: "Quantitative Trading With R-18"
subtitle: "실행 시간 개선"
categories: data
tag: quant, ts
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

비교 목적으로 만든 다음 c++ 코드는 실행이 훨씬 빠르다.

```c++
long add_cpp(long max_value) {
    long sum = 0;
    for (long i = 1; i <= max_value; i++) {
        sum = sum + 1;
    }
    return sum;
}
```

R 프레임워크는 포트란, C, C++로 빠르게 코드를 구현할 수 있게 해준다. 이렇게 구현된 코드는 R내부에서 호출된다. 하지만 이 방식은 간단하지 않다. 포트란이나 C에서 R extensions를 작성하는 방법은 다소 복잡하다.

대신 Rcpp 라이브러리를 사용한다. Rcpp는 C++ 코드를 명쾌하고 간결히 작성할 수있게 해준다. R과의 연동도 쉽게 된다.

다음은 .Internal()을 호출하는 함수의 예다.

```R
lapply()
function (X, FUN, ...)
{
  FUN <- match.fun(FUN)
  if (!is.vector(X) || is.object(X))
    X <- as.list(X)
    .Internal(lapply(X, FUN))
}
```

lapply()의 코드는 R문법과 다른 언어로 구현돼 내부적으로 컴파일된 것을 어떻게 결합할 수 있는지 보여준다. 다음은 C++로 컴파일된 함수를 R에서 활용하는 방법이다.

```R
library(Rcpp)

# C++ 함수 만들기
cppFunction('
  long add_cpp(long max_value) {
    long sum = 0;
    for(long i = 1; i <= max_value; ++i) {
     sum = sum + i;
    }
    return sum;
   }'
)

add_cpp
## function (max_value)
## .Primitive(".Call")(<pointer: 0x10f52fbb0>, max_value)

add_cpp(1e5)
## [1] 5000050000
```

microbenchmark 테스트로 C++ 구현의 실행 효율성을 알 수 있다.

```R
microbenchmark(loop = sum_with_loop_in_r(1e5),
  compiled = compiled_sum_with_loop_in_r(1e5),
  vectorized = sum_with_vectorization_in_r(1e5),
  compiled_cpp = add_cpp(1e5))

## Unit: nanoseconds
##          expr     min      lq       mean  median        uq     max neval
##          loop 1980146 2050315 2240366.79 2146958 2256194.5 5015136   100
##      compiled 1945191 2054171 2261880.08 2136163 2298861.0 3561384   100
##    vectorized       0     514    2282.41    1028    2313.5   23646   100
##  compiled_cpp   25702   26731   33979.14   28273   31614.5   84820   100
```

C++ 구현은 R루프보다 800배 빠르다. C++ 함수와 R 코드를 함께 작성해도 완벽히 작동한다. 하지만 C++ 코드를 별도의 파일로 나누면 더 명확해진다. sourceCpp()를 사용하면 .cpp 파일을 불러올 수 있다. .cpp 파일은 다음과 같다.

```c++
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
long add_2_cpp(long max_value) {
  long sum = 0;
  for(long i = 1; i <= max_value; ++i) {
    sum = sum + i;
  }
  return sum;
}
```

sourceCpp() 함수는 먼저 함수를 컴파일하고 같은 이름을 가진 R 함수를 만든 다음 R 환경으로 함수를 가져온다.

```R
sourceCpp('Chapter_11/add_2_file.cpp')

add_2_cpp(100)
```



**Rinside로 C++에서 R 호출**

때로는 R의 기능을 C++ 내부에서 호출할 필요가 있다. RInside는 R임베딩 API의 추상화 계층을 제공해 다른 어플리케이션에서 R인스턴스로 접근이 가능하게 만들어주는 패키지다. Rcpp의 클래스를 사용하면 R과 C++ 간 데이터 연동도 매우 간단하다.

다음은 그 에제이다.

```c++
#include <RInside.h>
int main(int argc, char *argv[]) {

 // create an embedded R instance
 RInside R(argc, argv);

 // assign a char* (string) to "txt"
 R["txt"] = "Hello, world!\n";

 // eval the init string, ignoring any returns
 R.parseEvalQ("cat(txt)");

 exit(0);
}
```



**testthat으로 단위 테스트 작성**

테스팅은 매우 중요하다. 코딩 작업에서 테스트 기반 접근을 시작하자 생산성이 증가했고, 코드의 버그도 감소했다. 테스트 기반 개발은 어던 형식적인 요구사항보다는 사고방식에 가깝다. 기본적인 아이디어는 다음을 따른다.

1. 완성할 프로그램의 기능을 개념적으로 작은 모듈 요소 여러 개로 나눈다.

   각 요소는 한 가지 일을 에러 없이 수행한다.

2.  각 기능의 테스트를 작성한다. 초기에는 테스트가 실패하게 만든다.

3. 모든 테스트를 수행한다. 테스트는 실패해야 한다.

4. 기능을 수행하는 코드를 작성한다.

5. 테스트를 다시 수행한다. 이번에는 모두 통과해야 한다.

6. 코드의 외부는 변경 없이 내부를 수정하고 항상 테스트가 통과하게 유의한다.

R에서 가장 많이 쓰이는 테스팅 패키지는 RUnit과 testthat이다. 

다음 예제에서 testthat을 작업에 적용해본다. 주어진 가격 벡터의 로그 수익률을 계산한다고 생각하자. 위의 순서대로, 기능의 뼈대를 만들고 테스트를 실패한다.

```R
convert_to_returns <- function(prices) {
  return(9)
}
```

실패할 초기 테스트를 작성한다. context함수로 관련된 기능을 함께 묶어준다. test_that()함수는 기대되는 모든 작동을 합쳐준다. 테스트는 expect_키워드로 시작한다.

```R
require(testthat)

context("Price to log-return conversion")

test_that("convert_to_returns produces the correct values", {

  input_prices <- c(100, 101, 102, 103, 99)

  expected_returns <- c(0.009950331,
    0.009852296, 0.009756175, -0.039609138)

  expect_equal(expected_returns,
    convert_to_returns(input_prices))
})
```

테스트를 수행하면 다음과 같은 결과를 얻는다.

```R
## Error: Test failed: 'convert_to_returns produces
## the correct values'
## Not expected: expected_returns not equal to
## convert_to_returns(input_prices)
## Numeric: lengths (1, 4) differ.
```

이제 올바르게 작동하도록 작업한다. 기능을 추가해 테스트가 통과하게 만든다.

```R
convert_to_returns <- function(prices) {
  return(diff(log(prices)))
}
```

테스트가 수행되면 에러가 발생하지 않는다. 여기서 입력된 가격의 수가 2보다 작은지 체크하고 에러  메시지를 생성하는 기능을 추가해본다.

```R
input_prices <- c(100)
msg <- "Not enough price entries."

expect_message(msg, convert_to_returns(input_prices))
## Error: expected_message no messages shown
```

함수를 다시 작성해 예외 상황을 처리한다.

```R
convert_to_returns <- function(prices) {
  if(length(prices) < 2) {
    message("Not enough price entries.")
  }
  return(diff(log(prices)))
}
```

다양한 시나리오와 작동을 테스트할 수 있음을 알 수 있다. 테스트는 별도의 소스 파일을 만들어 다음 명령어로 실행할 수도 있다.

```R
test_file("example_test_file.r")
```



이 글 Quantitative Trading with R 시리즈는 책 [Quantitative trading with R](https://www.amazon.com/Quantitative-Trading-Understanding-Mathematical-Computational/dp/1137354070)을 보고 공부한 내용들입니다.