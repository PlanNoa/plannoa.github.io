---
layout: post
title: "Hidden Markov Models"
subtitle: "은닉 마르코프 모델"
categories: data
tag: ml time-series
comments: true
---

Hidden Markov model(HMM)은 날씨, 주식가격 등 현상의 변화를 나타내는 Markov model에 은닉 state와 observation을 추가해 확장한 모델이다.

### Markov chain

Hidden Markov Model(HMM, 은닉 마르코프 모델)은 Markov chain(마르코프 체인)을 전제로 하는 모델이다. Markov chain은 Markov Property(마코프 성질)을 지닌 discrete-time stochastic process(이산확률과정)으로, 1913년 러시아 수학자 마르코프가 제안한 개념이다.

한 상태의 확률은 그 이전 상태에 의해서만 변동된다는 것이 Markov chain의 핵심이다. 즉, 현재에서 미래 상태로의 transition(변화)는 그동안의 상태 로그 없이 현재 상태의 transition로 추정할 수 있다는 것이다. Markov chain은 다음과 같다.

![](https://imgur.com/EBgEDQL.png)

다음 그림은 날씨를 Markov chain으로 모델링한 예시이다. 

![](https://i.imgur.com/iCPKPWz.png)

각 노드는 상태, 선은 transition 확률을 나타낸다. 각 노드별로 transition 확률의 합은 1이다. 일반적인 날씨 상태와 다르게 시작과 끝 상태도 볼 수 있다.

### Hidden Markov Model

Hidden Markov Model(HMM, 은닉 마르코프 모델)은 각 상태가 Markov chain을 따르지만 은닉되어 있다고 가정한다. 풀어 말하자면, 우리가 50년 전 기후를 연구해야 하는데, 아는 정보는 커피 판매량뿐이다. 이 정보만으로 당시 날씨가 추웠는지, 더웠는지, 선선했는지, 따뜻했는지를 알고 싶다. 우리는 커피 소비 기록의 연쇄를 관찰할 수 있지만, 해당 날의 날씨가 무엇인지는 직접적으로 관측하기 어렵다. HMM은 이렇게 관측치 뒤에 은닉된 상태를 추정한다. 다음은 HMM을 도식화한 그림이다.

![](https://i.imgur.com/lEMDGBC.png)

위 그림에서 B₁은 날씨가 더울 때 소비할 커피의 수와 확률을 나타낸다. B₁은 날씨가 더울 때의 조건부확률이므로 HOT이라는 은닉상태와 연관이 있다. B를 방출확률이라고도 부르는데, 은닉 상태로부터 관측치가 방출될 확률이라는 뜻이다.

#### Likelihood

일단 likelihood부터 계산하자. likelihood는 모델 λ가 있을 때 observation(관측치) O가 나타날 확률 p(OIλ)를 가르킨다. 한 마디로 모델 λ이 관측치 하나를 뽑는데 O가 나올 확률이다. 이렇게 관측된 O가 커피 [3잔, 1잔, 3잔]이라고 치자. 그럼 λ가 위의 그림일 때 O가 뽑힐 확률은 얼마일까?

![](https://i.imgur.com/syZWL5E.png)

두 번째 날짜를 중심으로 보자. λ를 보면 날씨가 더울 때 커피를 마실 확률은 0.2이다. 그리고 두 번째 날이 전날에 이어서 더울 확률은 0.6이므로 둘을 곱해야 둘째 날의 상태확률을 계산할 수 있다. 여기서 Markov chain을 모른다고 가정하면 상태확률을 계산하기 위해 직진 상태만을 고려한다. 위 그림을 식으로 나타내면 다음과 같다.

![](https://imgur.com/pikPB2u.png)

각 날짜별로 날씨가 더울 수도, 추울 수도 있다. 따라서 2³가지의 경우가 존재한다. 

| 상태1 | 상태2 | 상태3 |
| ----- | ----- | ----- |
| cold  | cold  | cold  |
| cold  | cold  | hot   |
| cold  | hot   | cold  |
| hot   | cold  | cold  |
| hot   | hot   | cold  |
| cold  | hot   | hot   |
| hot   | cold  | hot   |
| hot   | hot   | hot   |

따라서 관측치 [3, 1, 3]에 대한 likelihood는 다음과 같다.

![img](https://i.imgur.com/3PorurT.png)

### Compute Likelihood : Forward Algorithm

HMM에는 문제가 하나 있다. 만약 T개의 상태가 있고, 관측치 길이가 N라면 likelihood 계산 시 Tⁿ개나 된다. 이런 비효율성을 완화하기 위해 dynamic programming을 사용한다. 간단하게 설명하자면, 중복되는 계산을 저장해 두었다가 푸는 것이 핵심 원리이다.

![img](https://i.imgur.com/UcXttLx.png)

아이스크림 3개(o1)와 1개(o2)가 연속으로 관측되었으며, 두 번째 타임스템프(t=2)의 날씨가 추웠을(q1)의 확률은 α2(1)이다. 또한 아이스크림 3개(o1)가 관측됐고 첫 번째 시점(t=1)의 날씨가 더웠을(q2) 확률은 α1(2)이다.

Forward Algorithm의 아이디어는 중복된 계산의 결과를 저장해놓고 필요할 때마다 사용하는 것이다. 위 그림에서 α2(1)을 구할 때 직전 단계의 계산인 α1(1), α1(2)을 활용한다. 예시에서는 계산량 감소가 두드러지지 않지만, 데이터가 커지면 효율성이 극대화된다. j번째 상태에서 o1, o2, ... , ot가 나타날 확률 α는 다음과 같이 정의된다.

![](https://imgur.com/tXWUikL.png)

### Decoding : Viterbi Algorithm

다음으로 관심을 가져야 하는 것은 모델 λ과 관측치 시퀀스 O가 주어졌을 때 가장 확률이 높은 은닉 상태 Q를 찾는 방법이다. 이를 decoding이라고 하는데, 포스태깅 문제로 예를 들면 단어의 연쇄를 가지고 품사 태그의 시퀸스를 찾는 것이다. HMM의 목적과도 같은 이 문제의 decoding 과정은 Viterbi Algorithm(비터비 알고리즘)이 주로 쓰인다.

Viterbi Algorithm이 계산하는 Viterbi Probability v는 다음과 같이 정의된다. vt(j)는 j번째 은닉상태의 Viterbi Probability을 가르킨다.

![img](https://i.imgur.com/6xVQRd4.png)

v와 Forward Algorithm의 전방확률 α를 계산하는 과정이 유사한 것을 볼 수 있다. Forward Algorithm이 각 상태에서의 α를 구하기 위해 모든 경우의 확률을 더했다면, decoding은 그 확률들 중 최대값을 구한다. 다음은 디코딩 과정을 설명한 그림이다.

![img](https://i.imgur.com/MXxxdo7.png)

각 상태에서의 v를 구하는 식은 다음과 같다. 전방확률을 계산하는 과정과 비교하면 공통점과 차이점을 분명히 알 수 있다. Viterbi Probability 역시 dynamic programming 기법을 사용한다.

![](https://imgur.com/ImqojPm.png)

Forward Algorithm과 Viterbi Algorithm 간에 가장 큰 차이점은 Viterbi의 backtracking 과정이다. decoding은 Viterbi Probability보다 최적 상태에 중점을 두기 때문에 당연한 것이다. 위 그림의 파란색 화살표가 역추적을 나타낸다.

예컨대 2번째 시점 2번째 상태 q2의 backtrace bt2(2)는 q2입니다. q2를 거쳐서 온 likelihood값(0.32×0.12)이 q1보다 크기 때문이다. 2번째 시점의 1번째 상태 q1의 backtrace bt2(1)는 q2입니다. 최적상태열은 이렇게 구한 backtrace들이 리스트에 저장된 결과이다.. 예컨대 위 그림에서 아이스크림 [3개, 1개]가 관측됐을 때 가장 확률이 높은 은닉상태의 시퀀스는 [HOT, COLD]가 되는 것이다..

t번째 시점 j번째 상태의 backtrace는 다음과 같이 정의됩니다.

![](https://imgur.com/Fq6hLvW.png)

### 전방확률과 후방확률

HMM 학습을 하기 위해서는 후방확률 β의 개념을 알아야 한다. 전방확률 α와 반대 방향으로 계산한 것이 후방확률이다.

![](https://imgur.com/dcPoClz.png)

먼저 전방확률부터 보자. α3(4)를 구하는 방법이다.

![img](https://i.imgur.com/mbBaTch.png)

후방확률 β3(4)는 다음과 같이 구한다.

![img](https://i.imgur.com/bP9BdJy.png)

따라서 α3(4) x β3(4)는 3번째 시점에서 4번째 상태를 가지는 모든 확률의 합을 가리킨다. 

![img](https://i.imgur.com/3SQDk3b.png)

![](https://imgur.com/ABZMKYE.png)

따라서 특정 시점의 전방확률과 후방확률을 곱한 값을 모든 상태에 더하면 앞서 계산한 likelihood와 동치가 된다. 후방확률을 관측치 매 끝부터 처음까지 계산하면 이 역시 앞에서 계산한 likelihood와 같다.

![](https://imgur.com/H8cNWLz.png)

### 베이즈 정리

[베이즈 정리](<https://plannoa.github.io/data/2019/11/07/Probability-Theory/>)에 의해 다음 식이 성립한다.

![](https://imgur.com/NVLKa5P.png)

### 방출확률 업데이트와 γ

방출확률 b를 업데이트하기 위해 γ개념을 보자. t시점에 j번째 상태일 확률 γt(j)는 베이즈 정리에 의해 다음과 같이 정의할 수 있다.

![](https://imgur.com/araOlhw.png)

j번째 상태에서 관측지 vk가 나타날 방출확률은 다음과 같이 정의된다.

![](https://imgur.com/IgmxIGD.png)

이를 해석하면.. 방출확률의 분모는 j번째 상태가 나타날 확률이다. 분자는 j번째 상태이면서 그 관측치가(ot)가 vk일 확률이다. ∑가 씌워진 이유는 방출확률은 시점과는 관계가 없는 값이기 때문이다. 어떤 시점이든 j번째 상태가 나타날 확률 y가 존재하므로 전체 관측치에 걸쳐 모든 시점에 대해 y를 더해주는 것이다.

### 전이확률 업데이트와 ξ

전이확률 a를 업데이트하기 위해 ξ를 보자. t번째 시점에 i번째 상태이고 t + 1번째 시점에 j번째 상태일 확률 ξ는 다음과 같이 정리된다. 이 역시 베이즈 정리가 적용된다. 

![](https://imgur.com/lai0x5e.png)

식을 이해하기 위한 그림이다. t시점에 i상태, t+1시점에 j상태인 경우를 도식화한 것이다.

![img](https://i.imgur.com/0QxMZTa.png)

이제까지 말했듯, αt(i)는 t시점 전의 모든 경우의 확률 합이다. βt+1(j)는 t+1시점 이후의 모든 경우의 확률 합이다. 하지만 이 둘로는 i에서 j로의 path 확률이 없다. 이를 연결하기 위해서 i번쨰 상태에서 j번째 상태로 transition 할 확률, j번째 상태에서 관측치 ot+1를 관측할 방출확률까지 곱해줘야 한다.

먼저 t시점에 i번째 상태이고 t+1시점에 j번째 상태일 확률 ξ은 다음과 같이 다시 쓸 수 있다.

![](https://imgur.com/S2okr8z.png)

i번째 상태에서 j번째 상태로 transition 할 확률은 다음과 같이 정의된다.

![](https://imgur.com/XuxzeQV.png)

식에서 분모는 i번째 상태에서 transition할 수 있는 모든 path들의 확률을 더한 값이며, 분자는 i번쨰 상태에서 j번째 상태로 transition할 확률이다. 전과 마찬가지로 방출확률은 시점 t와는 무관한 값이기 때문에 Σ이 적용된다. 어떤 시점이든 i번째 상태에서 다른 상태로 전이할 확률 ξt가 존재하므로 관측치 전체에 걸쳐 ξt를 더해주는 것이다.

위 식을 도식화한 그림이다. 분모는 굵은 적색 점선, 분자는 검은색 화살표다.

![img](https://i.imgur.com/G0FUlgo.png)

### Training : EM Algorithm

HMM의 파라미터는 전이확률 A와 방출확률 B이다. 하지만 이 둘을 동시에 추정하는 것은 어렵다. 날씨 예제를 기준으로 하면, 우리가 관측가능한 것은 커피의 판매량뿐이고 궁극적으로 알고 싶은 날씨는 숨겨져 있다. 고로 HOT에서 COLD로 전이할 확률은 물론 날씨가 더울 때 커피를 1잔 마실 방출확률 따위를 모두 한번에 알 수 없다. 이럴 때 주로 사용되는 것이 [EM 알고리즘](<https://plannoa.github.io/data/2019/11/08/EM-algorithm/>)이다.

#### E-step

모델 파라미터 A와 B를 고정시킨 상태에서 관측치 O를 바탕으로 전방확률과 후방확률을 추정한다. 이 둘을 바탕으로 각각 y와 ξ를 계산한다.

#### M-step

E-step에서 구한 y와  ξ를 바탕으로 모델 파라미터 A, B를 추정한다.



