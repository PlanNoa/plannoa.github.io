---

layout: post
title: "GMM, Gaussian Mixture Model"
subtitle: "가우시안 혼합 모델"
categories: data
tags: ml
comments: true
---

**Gaussian Mixture Model**

Gaussian Mixture Model (GMM)은 이름 그대로 Gaussian 분포가 여러 개 혼합된 clustering 알고리즘이다. 현실에 존재하는 복잡한 형태의 확률 분포를 여러개의 Gaussian distribution을 혼합하여 표현하자는 것이 GMM의 기본 아이디어이다. 이때 Gaussian distribution의 개수는 데이터를 분석하고자 하는 사람이 직접 설정해야 한다.



![img](https://imgur.com/McwinS4.png)

주어진 데이터 x에 대해 GMM은 x가 발생할 확률을 아래 식과 같이 여러 Gaussian probability density function의 합으로 표현한다.



![](https://imgur.com/4VuqDHN.png)



위 식에서 mixing coefficient 라고 부르는 π는 k번째 Gaussian distribution이 선택될 확률을 나타낸다. 따라서 π는 아래의 조건을 만족한다.



![img](https://t1.daumcdn.net/cfile/tistory/999E08375AC7637F0C)

![img](https://t1.daumcdn.net/cfile/tistory/993FF4395AC763833C)



GMM을 학습시키는 작업은 데이터 x에 대해서 적절한 π, μ, ∑를 찾는 것과 같다.

