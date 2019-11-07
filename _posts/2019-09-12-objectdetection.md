---
layout: post
title: "Object detection"
subtitle: "객체 검출"
categories: data
tags: image ml
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

내가 사용할 method는 One-stage Method이다. Two-stage Method는 느리고 실용성이 떨어진다는 말이 많아 나중에 알아보도록 하자.

**One-stage Method**

1. **YOLO(You Only Look Once)**

   YOLO는 이미지 내의 bounding box와 class probability를 하나의 regression problem으로 간주해 이미지를 한 번 보는 것으로 객체의 종류와 위치를 추측한다.

   기존의 method들과 비교했을 때 가지는 장단점이다.

   **장점**

   - 처리과정이 간단해 매우 빠르다. 기존의 실시간 detection system과 비교해 2배정도 높은 mAP를 가진다.
   - 이미지를 한 번 보는 방식으로 class에 대한 이해도가 높다. 낮은 False-Possitive를 보인다.
   - 객체에 대해 보다 일반적인 특징을 학습한다.(사진으로 사람을 학습해도 그림에 그려진 사람을 맞춘다)

   **단점**

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
   (2) 객체가 있는 셀 i의 predictor bounding box j에 대한 w, h의 loss(큰 박스에는 small deviation을 반영하기 위해 제곱근을 취한다.)
   (3) 객체가 있는 셀 i의 predictor bounding box j에 대한 confidence score의 loss(Ci = 1)
   (4) 객체가 없는 셀 i의 predictor bounding box j에 대한 confidence score의 loss(Ci = 1)
   (5) 객체가 있는 셀 i의 conditional class probability의 loss(Correct class c: pi(c)=1, otherwise: pi(c)=0)

   λcoord: coordinates(x,y,w,h)에 대한 loss와 다른 loss들과의 균형을 위한 balancing parameter.
   λnoobj: obj가 있는 box와 없는 box간에 균형을 위한 balancing parameter.(객체 있는 셀 < 객체 없는 셀)

   **Limitation**

   - 하나의 셀이 하나의 클래스를 예측하기 때문에 작은 객체 여러 개가 붙어 있으면 정확도가 떨어진다.
   - bounding box의 정확도가 상대적으로 떨어진다.

2. **SSD(Single Shot Detector)**

   SSD는 먼저 아웃 풋을 만드는 공간을 나눈다(multi feature map). 각 피쳐맵에서 다른 비율과 스케일로 box를 생성하고 모델을 통해 계산된 좌표와 클래스값, box로 최종 bounding box를 생성한다.

   **장점**

   - 속도와 성능 간의 적당한 밸런스(YOLO보다 속도는 느리지만 더 정확하다.)

   **단점**

   - 비교적 소량의 데이터로 학습할 때, '비교적' 정확성이 낮다.(YOLO보다는 높다.)

   ![](https://imgur.com/biAYTSQ.png)

   **Network**

   ![](https://imgur.com/CYTfWlH.png)

   **Inference Process**

   ![](https://imgur.com/foS1j5b.png)

   이미지를 기본적으로 VGG-16 모델을 가져와 conv4_3까지 적용하는 것으로 처리하면 300x300x3 이 38x38x512로 바뀐다. 이 38x38, 19x19, 10x10, 5x5, 3x3, 1x1의 피쳐맵은 출력과 직결된다.

   ![](https://imgur.com/EYEQsRl.png)

   각 피쳐맵에서 적절한 연산을 통해 우리가 예측하고자하는 bounding box의 점수와 offset을 얻게된다. 이때 conv filter size는 3 x 3 x (박스 개수 x (class score + offset))이고 자세히 나와있진 않지만 stride=1, padding=1일 것으로 추정된다. 이 6개의 피쳐맵에서 예측된 bounding box의 총 합은 8732개이다.

   ![](https://imgur.com/DT8hy5y.png)

   하지만 바운딩박스의 아웃풋을 다 고려하지 않는다. 각 피쳐맵당 다른 스케일을 적용해 default 박스간의 IOU를 계산한다음 미리 0.5이상이 되는 box들만 고려하고 나머지는 없애버려 위와 같이 3개의 피쳐맵에서만 box가 감지될 수 있다.

   ![](https://imgur.com/1ELsqzP.png)

   이게 NMS를 통해 나온 최종 결과이다.

   **Train**

   ![](https://imgur.com/d2rzBlk.png)

   5x5 피쳐맵을 예시로 보자. 왼쪽 상단의 점을 하나의 셀로 본다. 각 셀 당 default box는 3개로 보인다. 이때 default box의 w, h는 feature map의 scale에 따라 서로 다른 s 값과 서로 다른 aspect ratio인 a 값을 이용해 도출된다. 또 default box의 cx와 cy는 feature map size와 index에 따라 결정된다.

   그러고 default box와 ground truth box(정답 box)간의 IOU를 계산해 0.5 이상인 값들만 매칭시켜 loss를 계산한다.

   ![](https://imgur.com/3wGIDQ0.png)

   빨간색 점선이 매칭된 default box라고 한다면, 거기에 해당하는 cell의 같은 순서의 predicted bounding box의 offset만 update 되고 최종적으로는 아래와 같이 예측된다.

   ![](https://imgur.com/83gDgZK.png)

   **Loss**

   ![](https://imgur.com/Vkfm2py.png)

   loss function은 class score loss와 bounding box loss로 나뉜다.

   ![](https://imgur.com/eerZGab.png)



   Lloc(*x, l, g*)(box score)

   (1) N 은 매치된 default boxes
   (2) l 은 predicted bounding box
   (3) g 는 ground truth box
   (4) d 는 default box, cx, cy는 그 박스의 x, y좌표, w, h는 그 박스의 width, heigth
   (5) α는 IOU가 0.5 인지 지표

   Lconf(*x, c*)(class score)

   (1) positive(매칭) class에 대해서는 softmax
   (2) negative(매칭 x) class를 예측하는 값은 c^0i이고 background이면 1, 아니면 0의 값을 가질 것
   (3) 최종 predicted class scores는 예측할 class + 배경 class 를 나타내는 지표

   **Choosing scales ansd aspect ratios for default boxes**

   여러 크기의 default box 생성을 위해 다음과 같은 식을 만든다.

   ![](https://imgur.com/gdhvknv.png)

   (1) Smin = 0.2, Smax = 0.9
   (2) 저 식에다 넣으면 각 feature map당 서로 다른 6개의 s 값들이 나옴

   **Limitation**

- 비교적 쉽게 예측되는 ratio 외에 특이한 ratio를 가진 물체는 예측할 수 없음.

