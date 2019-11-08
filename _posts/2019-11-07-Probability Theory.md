---
layout: post
title: "Probability Theory"
subtitle: "확률 이론"
categories: data
tags: ml
comments: true
---

### Maximum Likelihood Estimation(MLE)

MLE는 파라미터를 추정하는 방법 중 하나인데, 오직 주어진 데이터만을 토대로 추정한다. 예를 들어 p의 확률로 앞면, 1-p의 확률로 뒷면이 나오는 동전을 던져 p를 예측한다고 하자. MLE로 p를 계산하려면 단순히 앞면이 나온 횟수를 전체 횟수로 나누기만 하면 된다.

더 복잡한 예시를 위해 모수 θ로 결정되는 확률변수의 모임 Dθ = (X0, X1, ... Xn)가 있고, Dθ의 pdf를 f라 하자. 그 확률변수에서 각각 값 x1, x2, ... xn을 얻었을 경우 likelihood는 다음과 같다.

 ![\mathcal{L}(\theta) = f_{\theta}(x_1, x_2, \cdots, x_n)](https://wikimedia.org/api/rest_v1/media/math/render/svg/a4e1f531eb40a44316ce85b1a50778199073d316)

여기서 likelihood를 최대로 만드는 θ는 이렇게 된다.

![\widehat{\theta} = \underset{\theta}{\operatorname{argmax}}\ \mathcal{L}(\theta)](https://wikimedia.org/api/rest_v1/media/math/render/svg/f090261c0399247a6f3303b2b161869f2a6d15fa)

만약 Dθ가 모두 독립적이며 같은 확률분포를 가지고 있다면 이렇게 된다.

 ![\mathcal{L}(\theta) = \prod_i f_{\theta}(x_i)](https://wikimedia.org/api/rest_v1/media/math/render/svg/e50fc202373688dbc613797075e166fa335fa0ed)

또한 로그를 씌워도 의미는 같으며, 이 경우 계산이 간단해진다.

![\mathcal{L}^*(\theta) = \log \mathcal{L}(\theta) = \sum_i \log f_{\theta}(x_i)](https://wikimedia.org/api/rest_v1/media/math/render/svg/1684f8814e202ed92340b13bac4e1e53f25f5a3e)

어쨌든 MLE는 관측치에 따라서 값이 민감하게 변한다는 단점이 있다. 동전 던지기에서도 n번을 던져 n번이 나올 수가 있는데, 이럴 경우 MLE는 앞면만 나오는 동전으로 판단한다.

### Maximum a Posteriori Estimation(MAP)

MLE의 단점을 해결하기 위해 이라는 방법을 사용하기도 한다. 이 방법은 θ가 주어지고, 그 θ에 대한 데이터들의 확률을 최대화하는 것이 아니라, 주어진 데이터에 대해 최대 확률을 가지는 θ를 찾는다. 수식으로 표현하면 다음과 같다.

![](https://imgur.com/qO0syWL.png)

MLE와 비교해 MAP는 보다 더 자연스러운 결과를 얻게 된다. MLE로 parameter estimation을 하게 되면 오직 지금 주어진 데이터만을 잘 설명하는 parameter 값을 찾게 된다. 그러나 앞서 예를 든 것처럼 만약 스팸필터를 만드는데 연속으로 스팸이 아닌 메일이 nn개가 들어왔다고해서 모든 메일이 스팸이 아니라고 할 수 있을까? 우리가 원하는 것은 지금까지 들어온 값에 대해서만 잘 설명하는 것이 아니라 보다 더 일반적인 모수를 원한다. 여러 paramter들 중에서 데이터가 주어졌을 때 가장 확률이 높은 θ를 고를 수 있다면 가장 좋은 결과를 얻을 수 있을 것이다.

하지만 안타깝게도 MAP를 계산하기 위해서는 f(θIX)가 필요하지만 우리가 관측할 수 있는 것은 오직 f(XIθ)뿐이다. f(θIX)를 구하기 위해서는 Bayes’ Theorem를 이용해야 한다.

### Bayes' Theorem

Bayes’ Theorem은 두 확률 변수의 사전 확률(p(XIθ))과 사후 확률(p(θIX)) 관계를 나타내는 정리다. 베이즈 학률론에 의하면 베이즈 정리는 사전확률로 사후확률을 구할 수 있다.

이때 f(XIθ)를 likelihood, f(θ)를 piror, f(θIX)를 posterior이라고 하며 각각은 observation(관측치), prior(사전정보), posterior(주어진 데이터에 대한 현상의 확률)을 의미한다.. 이 식을 통해서 관측치만을 사용하는 것보다 더 우수한 파라미터 추정을 할 수 있다.

Bayes's Theorem을 이용해 MAP와 MLE의 관계를 적을 수 있다.

![](https://imgur.com/8YHWRXT.png)

이때 f(X) term은 θ에 영향을 받는 term이 아니기 때문에 다음과 같이 적을 수 있다.

![](https://imgur.com/t5seOj9.png)

따라서 만약 사전정보를 알고 있으며 MLE가 아니라 MAP을 사용할 수 있다. 즉 θ에 대한 assumption를 사용해 결과를 더 향상시킬 수 있다. 시험 성적의 gaussian distribution을 추정하고 있을 때, 평균은 반드시 시험 점수 범위 안에 포함된다. 그리고 이전 시험 성적들이 대체로 60 ~ 80점 사이에 몰렸다는 정보가 있을 때, f(θ)를 가정하고 MAP을 사용할 수 있다. 데이터의 모집단에 대해 적절한 가정이 가능하다면 더 좋은 추론을 할 수 있는 것이다. 이를 prior이라고 하는데, 잘못된 prior을 선택하면 오히려 성능이 떨어질 수도 있다.

