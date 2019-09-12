---
layout: post
title: "Object detection"
subtitle: "객체 검출"
categories: data
tag: dl
comments: true
---

**이미지 인식**

딥 러닝에서의 이미지 관련 분야는 크게 Classification, Localization, Object Detection, Instance Segmentation이 있다.

![](https://imgur.com/LoQbfGW.png)

**Classification**

![](https://imgur.com/gFRpQWh.png)

딥 러닝에서 classification은 이미지가 나타내는 객체의 종류를 분류하는 것이다. 가장 대표적인 것으로 mnist가 있다. 0부터 9까지의 숫자들로 이미지를 구별하며 딥 러닝 모델은 입력된 이미지가 어떤 숫자인지 분류하여 출력하게 한다. 위 사진의 경우 입력된 이미지가 cat을 나타낸다고 분류한 것이다.

**Localization**

![](https://imgur.com/6Jm8ctk.png)

Localization은 객체의 위치 정보를 출력한다. 보통 위 사진처럼 bounding box를 이용하며, left top, right bottom 좌표를 출력한다.

**Object Detection**

![](https://imgur.com/Uoful8j.png)

Object Detection은 Classification과 Localization이 합쳐진 형태이다. 목적에 따라 한 가지 객체만 학습시켜 객체의 유무를 확인할 수도 있고, 여러 개의 객체를 검출할 수도 있다. 위 영상처럼 실시간으로 인식하게 만들어 자율주행 자동차 등에 사용하기도 한다.

**Object Segmentation**

![](https://imgur.com/Opo2O5d.png)

Object Detection보다 훨씬 진보한 상태이다. bounding box가 아닌 픽셀 단위로 보여준다. 그러나 아직 연구가 충분히 진행되지 않은 상태이다. 데이터셋을 만드는 것도 힘들어 보인다.



**Localization 알고리즘**

일단 당장 필요한 지식은 Localization과 Detection이니, 그에 대해 먼저 알아보자. Cnn은 각 이미지를 sliding window 방식으로 스캔하는데, Localization에서는 모델이 필터처럼 1픽셀씩 움직이며 객체가 있는지 검사한다. 쭉 스캔하다가 객체의 형태가 보이면 확률을 올리는 것이다. 결국 아래처럼 작동한다.

![](https://imgur.com/9fqji1T.png)

이 알고리즘은 overfeat알고리즘이라 부르는데, 현재는 사용하지 않는다고 한다.  하지만 overfeat가 발전한 것이 Convolution Layer이다.

15년까지는 localization이 많이 쓰였다. 하지만 16년 이후  imagenet 대회에서 detection 알고리즘이 우승하며, detection 알고리즘이 가장 많이 쓰이게 되었다.

**Detection 알고리즘**

Localization과는 다르게 Detection은 한 이미지 내에 여러 객체가 있을 수 있고, bounding box도 여러 개일 수 있다.

Detection에는 두 가지 방법이 있다.

1. 속도가 빠른 One-stage Method(YOLO, SSD)

   ![](https://imgur.com/EZesWre.png)

2. 정확도가 좋은 Two-stage Method(R-Cnn, Fast R-Cnn, Faster R-Cnn)

   ![](https://imgur.com/X1qabUt.png)

Two-stage Method가 먼저 나왔다. One-stage Method가 깔끔하고, 쉽기 때문에 실무에서도 많이 쓴다고 한다. 특히 자율주행같은 부분에서 더 그럴 것 같다.



먼저 One-stage Method 부터 설명하겠다.