---

layout: post
title: "Quantitative Trading With R-16"
subtitle: "최적화"
categories: data
tag: quant
comments: true
---

**최적화**

수천 개의 자산이 있을 때 최소의 위험으로 최대의 수익을 얻는 문제를 생각해보자. 이 문제의 풀이법은 주어진 조건에서 목적 함수를 최소화, 최대화시키는 매개변수를 찾는 것이다. 우리의 경우 정해진 기간의 총수익을 전략의 최대 손실로 나눈 값이 될 수 있겠다. 금융, 물리학, 전산 분야의 많은 문제들은 어떤 모델을 사용할지 선택하는 데 좌우된다. 최적화는 수학의 한 분야로, 제약 조건에서 매개변수들의 최적 값을 찾는 기술이다.



**동기를 유발하는 포물선**

함수 f(x) = (1 + x)^2를 최소로 하는 x값은 무엇인가?

미적분으로 답을 찾을 수 있다. x에 대해 1차 미분하고 이를 0으로 두면 답을 찾을 수 있다.

df(x)/dx = 2(1 + x) = 0, x = -1

다음 그래프는 함수 f(x)와 f'x(x)를 동시에 보여준다.

```R
f <- function(x) {
  return((1 + x) ^ 2)
}

fp <- function(x) {
  return(2 * (1 + x))
}
 
x = seq(-5, 5, 0.1)
plot(x, f(x), type = 'l', lwd = 2,
  main = "f(x) and f'(x)",
  cex.main = 0.8,
  cex.lab = 0.8,
cex.axis = 0.8)
grid()
lines(x, fp(x), lty = 3, lwd = 2)
abline(h = 0)
abline(v = 0)
```

![](https://imgur.com/0Y88pIx.png)

이 방법은 도함수의 형태를 미리 알고 있으며, 편미분 형태도 알고 있다면 다차원으로 확대할 수도 있다.



**뉴턴법**

뉴턴법은 함수가 0을 갖는 점을 수리적으로 계산하는 데 쓰이는 방법이다. 뉴턴법은 xn에서 f(x)의 테일러 전개를 활용해 xn 수열이 f'(x*) = 0을 만족시키는 x *로 수렴하게 구성한다. 임의의 초기값 x0로 시작해 반복해서 추정치를 개선시킨다.

R에서는 다음같이 나타난다.

```R
f <- function(x){
    return(x ^ 2 + 4 * x - 1)
}
```

![](https://imgur.com/JIt0Rox.png)

그래프를 보면 -4와 0주위에 해가 있는 것을 볼 수 있다.

```R
uniroot(f, c(-8, -1))
## $root
## [1] -4.236068

## $f.root
## [1] -2.568755e-07

## $iter
## [1] 8

## $init.it
## [1] NA

## $estim.prec
## [1] 6.103516e-05

uniroot(f, c(-1, 2))
## $root
## [1] 0.236044

## $f.root
## [1] -0.0001070205

## $iter
## [1] 6

## $init.it
## [1] NA

## $estim.prec
## [1] 6.103516e-05
```

r에 기본 설치된 uniroot 함수를 답을 찾을 수 있다. 이 숫자를 뉴턴법으로 나온 값과 비교하자.

```R
# 1차 근사 뉴턴법
newton <- function(f, tol = 1E-12, x0 = 1, N = 20){
    h <- 0.001
    i <- 1
    x1 <- x0
    p <- numeric(N)
    while(i <= N) {
        df_dx <- (f(x0 + h) - f(x0)) / h
        x1 <- (x0 - (f(x0) / df_dx))
        p[i] <- x1
        if (abs(x1 - x0) < tol) {
            break
        }
        x0 <- x1
    }
    return(p[1:(i-1)])
}
```

`newton()` 함수는 f(x)의 해에 근접한 벡터를 반환해야 한다. 해가 2개이니 두 번 호출한다.

```R
newton(f, x0 = -10)
## [1] -4.236068

newton(f, x0 = 10)
## [1] 0.236068
```

충분히 만족스러운 값이다.

R은 기호 연산도 수행한다. 수식이 있다면 `D()`함수는 어떤 변수에 대해 미분을 계산할 수 있다. 

```R
e <- expression(sin(x))

D(e, 'x')
## cos(x)
```

다음은 함수를 계산하고 도함수를 구하는 방법이다.

```R
f_expr <- expression(x ^ 2 + 4 * x - 1)
```

`eval()`함수로 수식을 계산할 수 있다.

```R
eval(f_expr, list(x = 2))
## [1] 11
```

`newton()`함수 내에서 `D()` 함수를 사용해 미분을 계산할 수 있다.

```R
newton_alternate <- function(f, tol = 1E-12, x0 = 1, N = 20){
    df_dx <- D(f, 'x')
    
    i <- 1
    x1 <- x0
    p <- numeric(N)
    while(i <= N){
        x1 <- (x0 - eval(f, list(x = x0)) /
              eval(df_dx, list(x = x0)))
        p[i] <- x1
        i <- i + 1
        if(abs(x1 - x0) < tol){
            break
        }
        x0 <- x1
    }
    return(p[1:(i-1)])
}

newton_alternate(f_expr, x0 = -10)
[1] -6.312500 -4.735960 -4.281736 -4.236525 -4.236068 -4.236068 -4.236068

newton_alternate(f_expr, x0 = 10)
[1] 4.2083333 1.5068512 0.4663159 0.2468156 0.2360937 0.2360680 0.2360680 0.2360680
```



**무작위 대입법**

이제 많은 점들을 지나는 최적의 선을 정하는 방법을 알아본다. 

```R
set.seed(123)
x <- rnorm(100, 0, 1)

y <- 3.2 + 2.9 * x + rnorm(100, 0, 0.1)
plot(x, y)
```





이 글 Quantitative Trading with R 시리즈는 책 [Quantitative trading with R](https://www.amazon.com/Quantitative-Trading-Understanding-Mathematical-Computational/dp/1137354070)을 보고 공부한 내용들입니다.