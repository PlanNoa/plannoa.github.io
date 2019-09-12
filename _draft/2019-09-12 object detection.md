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



먼저 One-stage Method 부터 알아보자.

**One-stage Method**

1. **YOLO(You Only Look Once)**

   YOLO는 이미지 내의 bounding box와 class probability를 하나의 regression problem으로 간주해 이미지를 한 번 보는 것으로 객체의 종류와 위치를 추측한다.

   기존의 method들과 비교했을 때 가지는 장단점이다.

   **장점**:

   - 처리과정이 간단해 매우 빠르다. 기존의 실시간 detection system과 비교해 2배정도 높은 mAP를 가진다.
   - 이미지를 한 번 보는 방식으로 class에 대한 이해도가 높다. 낮은 False-Possitive를 보인다.
   - 객체에 대해 보다 일반적인 특징을 학습한다.

   **단점**:

   - 작은 객체에 대해 낮은 정확도를 가진다.

   처리 과정은 다음과 같다.

   ![](https://imgur.com/FcbYf0A.png)

   1. 이미지를 S x S 그리드로 나눈다.
   2. 각 그리드 셀은 bounding box와 그에 대한 confidence score을 가진다. (객체가 없으면 0)
   3. 각 그리드 셀은 C개의 conditional class probability를 갖는다.
   4. 각 bounding box는 중심점 x, y, 이미지의 weight, height에 대한 상대값 w, h, confidence로 구성된다.

   **Network**

   [deepsystems.io](https://deepsystems.io)의 [슬라이드]( https://goo.gl/eFcsTv)를 보자.

   ![](https://imgur.com/vD1H9hg.png)

   **Inference Process**

   ![](https://imgur.com/O88dC9p.png)

   7x7은 그리드의 크기를 나타낸다. 그리고 각 그리드 셀은 B개의 box를 갖는데, 앞 5개는 셀의 첫 bounding box에 대한 정보가 채워진다.

   ![](https://imgur.com/6xbXC2a.png)

   10번째 값까지는 두 번째 bounding box의 정보가 채워진다.

   ![](https://imgur.com/ntOr1Jo.png)

   나머지 20개는 conditional class probability이다.

   ![](https://imgur.com/s8CsJXZ.png)

   각 bounding box의 confidence score와 각 conditional class probability를 합하면 각 bounding box의 class specific confidence score가 나온다.

   ![](https://imgur.com/91h6XGI.png)

   이렇게 98개의 class specific confidence score를 얻고, 이에 대해 20개의 클래스를 기준으로 non-maximum suprression을 해 객체 종류와 bounding box 위치를 결정한다.

   **Precondition**

   YOLO training에는 몇 가지 전제조건이 있다.

   1. 그리드 셀의 여러 bounding box들 중 ground-truth box와의 IOU가 가장높은 bounding box를 predictor로 설정한다.

   2. 다음 기호들을 사용한다.

      ![](https://imgur.com/uvVae43.png)

      (1) Object가 존재하는 grid cell i의 predictor bounding box j
      (2) Object가 존재하지 않는 grid cell i의 bounding box j
      (3) Object가 존재하는 grid cell i

   **Loss**

   ![](https://imgur.com/Tuqd823.png)

   (1) 객체가 있는 셀 i의 predictor bounding box j에 대한 x, y의 loss
   (2) 객체가 있는 셀 i의 predictor bounding box j에 대한 w, h의 loss(큰 박스에 대해 small deviation을 반영하기 위해 제곱근을 취한다.)
   (3) 객체가 있는 셀 i의 predictor bounding box j에 대한 confidence score의 loss(Ci = 1)
   (4) 객체가 없는 셀 i의 predictor bounding box j에 대한 confidence score의 loss(Ci = 1)
   (5) 객체가 있는 셀 i의 conditional class probability의 loss(Correct class c: pi(c)=1, otherwise: pi(c)=0)

   λcoord: coordinates(x,y,w,h)에 대한 loss와 다른 loss들과의 균형을 위한 balancing parameter.
   λnoobj: obj가 있는 box와 없는 box간에 균형을 위한 balancing parameter.(객체 있는 셀 < 객체 없는 셀)

   **Limitation**

   - 하나의 셀이 하나의 클래스를 예측하기 때문에 작은 객체 여러 개가 붙어 있으면 정확도가 떨어진다.
   - bounding box의 정확도가 상대적으로 떨어진다.

2. **SSD(Single Shot Detector)**

   SSD는 객체 검출 속도 및 정확도 사이의 균형이 있는 알고리즘이다. SSD는 입력 이미지에 대한 CNN을  한 번만 실행하고 형상 맵(feature map)을 계산한다. 경계 상자 및 객체 분류 확률을 예측하기 위해 이 형상 맵을 3 × 3 크기로 CNN을 수행한다. SSD는 CNN처리 후 경계 상자를 예측한다. 이 방법은 다양한 스케일의 물체를 검출 할 수 있다.

