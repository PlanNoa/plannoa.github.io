---
layout: post
title: "KL-Divergence"
subtitle: "쿨백 라이블러 발산"
categories: data
tag: ml
comments: true
---

KL-Divergence는 두 확률 분포 간의 유사도를 판단하는 척도이다.

### 세 확률 분포![](C:/users/Develop/Desktop/Rplot1.png)

저 빨간색 확률 분포는 학생들의 전자기기 개수를 나타내는 확률 분포이다. 그리고 이 분포를 요약하기 위해 초록색의 정규 분포를 그렸다. 초록색은 N(3, 2²)를 따른다.

`lines(x, dnorm(x, mean = 3, sd = 2))`

어떤가? 정규 분포는 학생들의 전자기기 개수를 잘 표현하나? 아닌 것 같다. 그렇다면 얼마나 다른 걸까? 왜 다르다고 생각되는 걸까? 그래프가 비슷하지 않아서? 이를 수치로 표현할 수는 없는 걸까?

이런 이유에서 나온 것이 두 정규분포가 얼마나 다른 지를 표현하는 쿨백 라이블러 발산, 즉 KL-Divergence이다.

![](C:/users/Develop/Desktop/KL-Divergence.png)

초록과 빨강 확률분포의 KL-Divergence는 대략 1.2이다. 이번에는 더 정확한 확률 분포를 그려보자.

![](C:/users/Develop/Desktop/Rplot2.png)`lines(x, dnorm(x, mean = 6, sd = 1))`

파란색 확률분포는 붉은색 분포와 꽤 유사한 것 같다. 이 둘의 KL-Divergence는 0.4이다. 이로써 N(6, 1²)이 N(3, 2²)보다 실제 데이터를 더 잘 표현한다는 사실을 알 수 있다.

KL-Divergence는 두 확률 분포의 다름 정도를 설명한다.

머신러닝에서는 주로 확인되지 않은 모델을 확률분포에 근사시킬 때 KL-Divergence를 이용한다. 실제 데이터와 알맞는 모델을 찾고 싶으면 KL-Divergence가 최저인 정규분포를 찾으면 된다.

### KL-Divergence

KL-Divergence(쿨백 라이블러 발산)는 Relative Entropy(상대 엔트로피)라고도 불린다. KL-Divergence는 기본적으로 두 확률 분포를 비교하기 위해서 사용한다.

![](C:/users/Develop/Desktop/KL-Divergence.png)

KL-Divergence의 의미를 먼저 알아보자.

##### Bayesian Inference(베이지안 추론)에서의 KL-Divergence

Q는 사전확률 분포, P는 사후확률 분포이다. 이 때 Dkl(PIIQ)는 사전확률에서 사후확률로 변하는 도중 얻은 정보의 양이다.

##### P, Q

P는 일반적으로 True distribution(참 분포), Observations(관측치)같은 실제 데이터를 의미한다.

그리고 Q는 주로 Theory, Model, Appoximation(근사)로 사용된다.

##### 관측 데이터를 근사 확률 분포로 만들기 위한 목적함수

KL-Divergence는 몇 가지 특징이 있다.

1. 항상 0 이상의 값이다.

   두 분포가 동일할 때 0의 값을 가지며, Gibb's Inequality를 이용해 KL-Divergence가 항상 0 이상의 값을 가진다는 것을 유도할 수 있다.

2. 비대칭적이다.

   KL-Divergence는 비대칭함수로 Dkl(PIIQ)과 Dkl(QIIP)의 값이 다르다. KL-Divergence는 두 확률분포의 거리를 나타내지 않는다.