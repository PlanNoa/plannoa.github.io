---
layout: post
title: "GMM, Gaussian Mixture Model"
subtitle: "가우시안 혼합 모델"
categories: data
tags: ml
comments: true
---

### Gaussian Mixture Model

Gaussian Mixture Model (GMM)은 이름 그대로 Gaussian 분포가 여러 개 혼합된 clustering 알고리즘이다. 현실에 존재하는 복잡한 형태의 확률 분포를 여러개의 Gaussian distribution을 혼합하여 표현하자는 것이 GMM의 기본 아이디어이다. 이때 Gaussian distribution의 개수는 데이터를 분석하고자 하는 사람이 직접 설정해야 한다.



![img](https://imgur.com/McwinS4.png)

주어진 데이터 x에 대해 GMM은 x가 발생할 확률을 아래 식과 같이 여러 Gaussian probability density function의 합으로 표현한다.



![](https://imgur.com/4VuqDHN.png)



위 식에서 mixing coefficient 라고 부르는 π는 k번째 Gaussian distribution이 선택될 확률을 나타낸다. 따라서 π는 아래의 조건을 만족한다.



![img](https://t1.daumcdn.net/cfile/tistory/999E08375AC7637F0C)

![img](https://t1.daumcdn.net/cfile/tistory/993FF4395AC763833C)



GMM을 학습시키는 작업은 데이터 x에 대해서 적절한 π, μ, ∑를 찾는 것과 같다.



#### GMM Classification

GMM Classification은 주어진 데이터 x가 어떤 Gaussian distribution에서 만들어졌는지를 찾는 것이다. 이를 위해 responsibility를 다음과 같이 정의한다.



![img](https://t1.daumcdn.net/cfile/tistory/99E6C5455AC8AA8A01)



Znk ∈ {0,1}는 Xn이 주어졌을 떄 k번째 Gaussian distribution이 선택되면 1, 아니면 0의 값을 가지는 binary variable이다. 즉 Znk == 1이라면 Xn이 k번째 Gaussian distribution에서 생성되었다는 것을 의미한다.

**GMM을 이용한 classification은 xnxn이 주어졌을 때, kk개의 γ(znk)γ(znk)를 계산하여 가장 값이 높은 Gaussian distribution을 선택하는 것이다.** 

학습을 통해 GMM의 모든 parameter π,μ,Σ의 값이 결정되었다면, 베이즈 정리(Bayes' theorem)을 이용하여 γ(znk)γ(znk)를 다음과 같이 계산할 수 있다.



