---
layout: post
title: "Data augmentation"
subtitle: "이미지 부풀리기"
categories: data
tag: image
comments: true
---

**Image augmentation**

Image augmentation은 데이터를 확장시키는 것을 의미한다. 일반적으로 Deep Learning 에서는 데이터 분류를 위해 모델을 학습시킬 때, 매우 많은 양의 데이터가 필요하다. 그러나 데이터 수집은 개인이 하기에는 상당히 어려운 작업이다. 많은 시간이 드는 것은 당연하다. 이러한 문제를 해결하기 위한 기술이 Image augmentation 이다. Image augmentation은 적은 양의 데이터를 바탕으로 여러 조작을 통해 데이터의 양을 늘리는 작업으로, 관련 라이브러리는 albumentation, imgaug 등이 있다. 

**imgaug**

