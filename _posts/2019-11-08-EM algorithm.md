---
layout: post
title: "EM algorithm"
subtitle: "EM 알고리즘"
categories: data
tag: ml, optimization
comments: true
---

EM 알고리즘은 latent variable(잠재 변수)가 존재하는 probabilistic model(확률 모형)의 maximum likelihood(최대가능도) 혹은 maximum a posterior(최대사후확률) 문제를 풀기 위한 알고리즘 중 하나이다. [GMM](<https://plannoa.github.io/data/2019/11/06/gaussian-mixture-model/>), [HMM](<https://plannoa.github.io/data/2019/11/15/Hidden-Markov-Models/>), RBM 등의 문제를 해결하는 데 사용된다.

### 잠재 변수를 가진 확률 모형

Latent variable은 본래 가지고 있는 random variable이 아닌 우리가 임의로 설정한 hidden variable이다. 아래 그림같은 모델이 있다고 할 때, 우리가 관측 가능한 random variable은 θ로 parameterized 되어있는 X 하나이고, Z는 hidden variable이라고 해 보자.

![img](https://dfdazac.github.io/assets/img/01-vae_files/graph-latent.png)

위 모델에서 X의 maximum likelihood를 계산하고 싶다면 어떻게 해야 할까? 먼저 X의 maximum likelihood는 다음과 같이 표현된다.

![](https://imgur.com/0vAwUm8.png)

문제를 간단하게 하기 위해 Z를 discrete variable이라고 정의했다. 이 문제에서 가정할 것이 하나 있는데, marginal distribution(주변확률분포) p(XIθ)를 직접 계산하는 것이 매우 까다롭다는 것이다. 이때 Z는 마음대로 정할 수 있는 latent variable이기 때문에 joint distribution(결합확률분포) p(X, ZIθ)가 marginal distribution보다 쉬운 Z를 잡는 것이 가능하다.

### Decomposition of log-likelihood

만약 우리가 latent variable Z의 marginal distribution을 q(Z)로 정의한다면, log-likelihood를 다음과 같이 decompose 할 수 있다.

![](https://imgur.com/HCmaSND.png)

이때 L(q, θ)와 KL(qIIp)는 다음과 같이 정의된다.

![](https://imgur.com/9h4kQWQ.png)

위의 식에서 L(q, θ)는 Z의 marginal distribution q(Z)의 functional이고, KL(qIIp)는 q, p의 [KL divergence](<https://plannoa.github.io/data/2019/11/08/KL-Divergence/>)를 의미한다. 이렇게 log-likelihood를 decompose하면 한 쪽은 random variable X, Z의 joint distribution, 다른 한 쪽은 conditional distribution(조건부 분포)으로 표현된다는 것을 알 수 있다.

### EM algorithm

KL devergency의 특성 때문에 L(q, θ)의 값이 lower bound가 된다. 고로 lower bound가 최고값이 되는 θ와 q(Z)의 값을 찾고, 그에 해당하는 log-likelihood의 값을 찾는 알고리즘을 설계하는 것이 가능하다. 만약 θ와 q(Z)를 jointly optimize하기가 힘들다면 한 variable을 고정해두고 나머지를 optimize하고, 나머지 하나를 다시 optimize할 수 있다. 

이게 기본적인 EM 알고리즘의 아이디어인데, 말한 것처럼 E-step과 M-step을 번갈아가며 거쳐 θ와 q(Z)를 optimize한다. 이런 방식은 한 번에 수렴하기가 쉽지 않기 때문에 EM 알고리즘은 E, M step을 반복하는 iterative 알고리즘이 된다.

처음 가지고 있는 θ를 θ¹이라고 정의하자. EM 알고리즘의 E-step은 먼저 θ¹의 값을 고정해두고 L(q, θ)이 최대가 되는 q(Z)의 값을 찾는 과정이다. 이 과정은 log-likelihood in p(XIθ¹)은 q(Z) 값과는 전혀 관계가 없기 때문에 L(q, θ)를 최대로 만드는 조건은 KL divergence가 0이 되는 상황이기 때문이다. KL divergence는 q(Z) = p(ZIX, θ¹)인 상황에서 0이 되기 때문에 q(Z)에 posterior distribution p(ZIX, θ¹)을 대입해서 해결할 수 있다. 정리하면, E-step은 언제나 KL-divergence를 0으로 만들고 lower bound와 likelihood를 일치시키는 과정이 된다.

반면에 M-step에서는 q(Z)를 고정하고 log-likelihood를 가장 크게 만드는 새 θ²을 찾는 optimization 문제를 푸는 단계가 된다. E-step에서는 update하는 variable과 log-likelihood가 서로 무관했기 때문에 log-likelihood가 증가하지 않았지만, M-step에서는 θ가 log-likelihood에 직접 영향을 미치기 때문에 log-likelihood 자체가 증가하게 된다. 또한 M-step에서 θ¹가 θ²로 바뀌었기 때문에 E-step에서 구했던 p(Z)로는 더 이상 KL-divergence가 0이 되지 않는다. 따라서 다시 E-step을 진행시켜 KL-divergence를 0으로 만들고, log-likelihood의 값을 M-step을 통해 키우는 과정을 계속 반복해야 한다.

![img](http://sanghyukchun.github.io/images/post/70-2.png)

이는 각각 E-step과 M-step의 그림이다. E-step을 의미하는 왼쪽 그림에서 KL divergence는 0이 되고, lower bound인 functional과 log-likelihood의 값이 같아진다. 오른쪽 그림은 M-step을 표현하고 있으며, θ가 update되면서 log-likelihood의 값이 증가하게 되지만, 더 이상 KL divergence의 값이 0이 아니게 된다. 이 과정을 더 이상 값이 변화하지 않을 때 까지 충분히 많이 돌리게 되면 이 값은 log-likelihood의 어떤 값으로 수렴하게 될 것이다. 그리고 매 step마다 항상 optimal한 값으로 진행하기 때문에 이 값은 log-likelihood의 local optimum으로 수렴하게 된다는 사실까지 알 수 있다. EM algorithm은 아래와 같은 그림으로 표현할 수 있다.

![img](http://sanghyukchun.github.io/images/post/70-3.png)

각 curve는 θ 값이 고정이 되어있을 때 q(Z)에 대한 lower bound의 값을 의미한다. 매 E-step마다 고정된 θ에 대해 p(Z)를 풀게 되는데, 이는 곧 log-likelihood와 curve의 접점을 찾는 과정과 같다. 또한 M-step에서는 θ 값 자체를 현재 값보다 더 좋은 지점으로 update시켜서 curve 자체를 이동시키는 것이다. 이런 과정을 계속 반복하면 알고리즘은 언젠가 local optimum으로 수렴하게 될 것이다. Local optimum에 수렴한다는 성질은 얼핏보면 나빠보일 수도 있지만, log-likelihood를 계산하는 것이 불가능에 가깝다는 것을 생각하면 latent variable을 잘 잡기만 한다면 반드시 local optimum으로 수렴하는 EM 알고리즘은 매우 훌륭한 알고리즘이라는 사실을 알 수 있다.

EM 알고리즘이 항상 잘 동작하는 것은 아니다. E-step은 posterior를 계산하는 과정이므로 크게 문제가 되는 경우는 많지 않지만, M-step에서 문제가 발생하는 경우가 많다. 예를 들어 모든 θ를 한 번에 jointly optimize하는 것이 어려워 또 다른 alternative method를 사용해야할 수 도 있다. 그렇게 되면 iterative 알고리즘 안에 nested iterative 알고리즘이 발생하게 되어 전체 알고리즘의 수렴 속도가 매우 느려지게 된다. 가장 단순하게 이 문제를 해결하는 방법으로는 nested iterative 알고리즘을 완전히 푸는 것이 아니라, 수렴여부와 관계없이 iteration을 조금만 돌리고 다시 E-step을 구하고, 다시 M-step을 정확히 푸는 대신 iteration을 몇 번만 돌리는 등의 방식이 있다. 몇몇 경우에는 이런 방식이 local optimum에 수렴한다는 것이 증명되었다. 가장 대표적인 예가 RBM을 푸는 Contrastive Divergence이다.