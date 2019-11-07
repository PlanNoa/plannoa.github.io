---

layout: post
title: "GMM, Gaussian Mixture Model"
subtitle: "가우시안 혼합 모델"
categories: data
tags: ml
comments: true
---

### GMM

Mixture model을 먼저 알아야 한다. Mixture model은 전체 분포에서 하위 분포가 존재한다고 보는 모델인데, 즉 데이터가 모수를 가지는 여러개의 분포로부터 생성되었다고 가정하는 모델이다. 

Gaussian Mixture Model (GMM)은 여러 개의 분포가 혼합되어 만들어지는 Mixture Model의 일종으로, Gaussian distribution가 여러 개 혼합된 clustering 알고리즘이다. 현실에 존재하는 복잡한 형태의 확률 분포를 여러개의 Gaussian distribution을 혼합하여 표현하자는 것이 GMM의 기본 아이디어이다. 이때 Gaussian distribution의 개수 K는 데이터를 분석하고자 하는 사람이 직접 설정해야 한다.

![img](https://imgur.com/McwinS4.png)

주어진 데이터 x에 대해 GMM은 x가 발생할 확률을 K개의 Gaussian distribution에서 Gaussian probability density function의 합으로 표현한다.

![](https://imgur.com/4VuqDHN.png)

위 식에서 π는 mixing coefficient로, k번째 Gaussian distribution이 선택될 확률을 나타낸다. π는 아래의 조건을 만족한다.

![img](https://t1.daumcdn.net/cfile/tistory/999E08375AC7637F0C)

![img](https://t1.daumcdn.net/cfile/tistory/993FF4395AC763833C)

GMM을 학습시키는 작업은 데이터 x에 대해서 적절한 π, μ, ∑를 찾는 것과 같다.



#### GMM Classification

다음과 같이 세 개의 정규분포 A, B, C가 있다.

![img](https://t1.daumcdn.net/cfile/tistory/99F2D73359842CC912)

GMM Classification은 주어진 데이터 x가 다음 중 어떤 Gaussian distribution에서 만들어졌는지를 찾는 것이다. 이를 위해 responsibility(각 분포에서 x를 포함할 확률)를 다음과 같이 정의한다.

![img](https://t1.daumcdn.net/cfile/tistory/99E6C5455AC8AA8A01)

데이터 x가 주어졌을 때, K개의 responsibility를 계산해 가장 값이 높은 Gaussian distribution을 선택하면 된다. 학습을 통해 GMM의 모든 파라미터 π,μ,Σ의 값이 결정되었다면, 베이즈 정리(Bayes' theorem)을 이용하여 responsibility를 다음과 같이 계산할 수 있다.

![img](https://t1.daumcdn.net/cfile/tistory/99AD6E4A5AC8C7C639)



### EM algorithm을 이용한 GMM의 학습



