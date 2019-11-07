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

### GMM Classification

다음과 같이 세 개의 정규분포 A, B, C가 있다.

![img](https://t1.daumcdn.net/cfile/tistory/99F2D73359842CC912)

GMM Classification은 주어진 데이터 x가 다음 중 어떤 Gaussian distribution에서 만들어졌는지를 찾는 것이다. 이를 위해 responsibility(각 분포에서 x를 포함할 확률)를 다음과 같이 정의한다.

![img](https://t1.daumcdn.net/cfile/tistory/99E6C5455AC8AA8A01)

데이터 x가 주어졌을 때, K개의 responsibility를 계산해 가장 값이 높은 Gaussian distribution을 선택하면 된다. 학습을 통해 GMM의 모든 파라미터 π,μ,Σ의 값이 결정되었다면, 베이즈 정리(Bayes' theorem)을 이용하여 responsibility를 다음과 같이 계산할 수 있다.

![img](https://t1.daumcdn.net/cfile/tistory/99AD6E4A5AC8C7C639)



### EM algorithm을 이용한 GMM의 학습

GMM은 두 가지의 모수를 가진다. 첫 번째는 3가지 정규분포 중 확률적으로 어디에서 속해있는가를 나타내는 π(weight) 값이고, 두 번째, 각각의 정규분포의 모수 μ,Σ(평균, 분산)이다. 첫 번째 종류의 모수를 잠재변수라고 부르며, 잠재변수가 포함된 모델은(Mixture Model)에서의 모수 추정은 [MLE](<https://plannoa.github.io/data/2019/11/07/Probability-Theory/>)로 구할 수 없기 때문에 EM알고리즘을 통해 구하게 된다.

주어진 데이터 X에 대해 EM알고리즘을 적용하여 파라미터들을 추정한다. 파라미터를 추정하기 전에 log-likelihood를 정의한다.

![img](https://t1.daumcdn.net/cfile/tistory/99B62E4E5AC8A0CF24)

likelihood는 어떤 모델에서 데이터가 생성되었음을 나타낸다. log-likelihood도 같은 의미를 가지며 편의를 의해 log를 취한 형태이다. 따라서 log-likelihood를 최대화하는 것은 데이터 X를 가장 잘 추정하는 GMM을 구성하는 것과 같다.

이를 위해 각각의 π,μ,Σ에 대해 log-likelihood를 편미분한다. 

μ의 추정 과정은 아래와 같다.

![img](https://t1.daumcdn.net/cfile/tistory/99A074505AC8A8D50C)

Σ의 추정 과정은 다음과 같다.

![img](https://t1.daumcdn.net/cfile/tistory/99043F505AC8B19B01)

π는 앞서 말한 조건을 지키면서 추정되어야 한다. 따라서 π는 라그랑주 승수법을 이용해 추정하며, 라그랑지안은 다음과 같다.

![img](https://t1.daumcdn.net/cfile/tistory/9997D3495AC8C3E724)

라그랑지안을 π에 대해 편미분하면 다음과 같이 계산할 수 있다.

![img](https://t1.daumcdn.net/cfile/tistory/991622405AC8C24C30)

위의 결과값으로 다음과 같이 π를 추정할 수 있다.

![img](https://t1.daumcdn.net/cfile/tistory/99C26F445AC8C39D15)

GMM 파라미터 추정을 위한 EM 알고리즘의 E-step에서는 모든 데이터와 K개의 Gaussian distribute에 대한 responsibility를 계산한다. 그 다음 M-step에서는 위와 같이 모든 K개의 Gaussian distribution에 대한 π,μ,Σ를 추정한다. 이런 E, M-step을 수렴할 때까지 반복한다.

