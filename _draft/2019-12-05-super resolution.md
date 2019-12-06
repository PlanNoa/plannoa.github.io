---
layout: post
title: "Super Resolution"
subtitle: "초해상화"
categories: data
tag: dl image
comments: true
---

Super Resolution은 저해상도 이미지를 고해상도 이미지로 변환시키는 문제이다.

### 저해상도 이미지 문제

어느 분야의 딥 러닝이든 가장 중요한 것은 데이터의 품질이다. 아무리 좋은 모델을 만들더라도 '쓰레기를 넣으면 쓰레기가 나온다' 라는 법칙을 피하지 못한다. 이미지 분석이 목적인 모델은 특히 그런 면이 심한데, 저해상도 영상이나 이미지를 이용해 데이터셋을 만드는 것도 힘들 뿐더러 학습 후 정확도는 말도 하기 힘들 정도다.

이런 문제를 해결하기 위해 나온 방법이 Super Resolution(SR), 초해상화다.

SR은 고해상도 이미지를 얻기 위해 넣는 사진이 하나인지, 여러 장인지에 따라 Single Image Super Resolution(SISR), Multi Image Super Resolution(MISR)로 나뉜다. 이 중 더 간편하게 이용할 수 있는 것은 SISR이기 때문에 주로 SISR에 대한 연구가 주를 이루고 있다.

![img](https://imgur.com/c4mNJ1C.png)

Super Resolution은 저해상도 이미지를 고해상도 이미지로 복원해야 하는데, 복원해야 하는 타켓인 고해상도의 이미지는 정답이 여러 개가 존재할 수 있다. 다르게 말하면 유일한 정답이 존재하지 않는, 정의할 수 없는 문제를 의미한다. 이러한 경우를 Regular Inverse Problem 혹은 Ill-Posed Problem이라 부른다.

![img](https://imgur.com/ntiZNsZ.png)

이런 상황을 최대한 방지하기 위해 대부분 다음과 같이 고해상도 이미지와 고해상도 이미지에 noise를 일으켜 만든 저해상도 이미지를 짝으로 데이터셋을 만든다. 그 뒤 모델을 만들어 저해상도 이미지를 고해상도 이미지로 복원하도록 학습시킨다. 이 하나의 저해상도 이미지당 정답 데이터를 하나만 가지고 있다는 점이 SISR의 한계이며, noise를 일으킬 때 사용한 distortion, down sapling 기법이 무엇인지에 따라 SR의 성능이 달라진다.

일반적으로 논문에서는 bicubic down sampleing한 데이터셋을 사용하며, 실생활에서 Super Resolution을 사용하고자 하는 경우에는 unknown down sampling도 고려해야 한다.

###  SR 적용 사례

예전 인기있던 드라마 하얀거탑의 리마스터 버전이 나온 적 있다. 과거 TV에 맞던 저해상도 영상을 최신 TV에 맞게 변환해 더욱 선명하게 감상할 수 있어 좋은 반응을 받았었다. 여기에서 사용된 기술 중 하나가 SR이며, TV 외에도 많은 분야에서 SR기법이 적용되고 있다.

![](https://imgur.com/KtcDJhm.png)

SR기법이 가장 중요하게 쓰이는 분야 중 하나는 항공우주 분야인데, 우주에서 촬영한 이미지의 경우 초고해상도의 카메라로 촬영을 하여도 거리가 멀기 때문에 피사체의 크기가 작아서 분별하기가 어려운 경우가 많다. 그리고 이 때도 SR이 적용된다.

### SR 방법

SISR 문제에 접근하는 방법은 크게 세 가지가 존재한다.

1. **Interpolation-based method**
2. **Reconstruction-based method**
3. **Learning-based method**

이미지 관련 라이브러리(OpenCV, PIL)에는 image resize 함수가 있는데, 이게 대표적인 Interpolation-based method 이다. interpolation 옵션을 다르게 사용하는 경우 이미지의 품질이 달라진다. Bicubic, Bilinear, Nearest Neighbor 등 다양한 interpolation 옵션이 있으며, 이미지의 해상도를 키워주는 경우에는 보통 Bicubic**, **Bilinear, Lanczos interpolation을 사용한다.

![](https://imgur.com/c8RgC1a.png)

하지만 이런 방식은 이미지를 말 그대로 resize하는 것과 큰 변화가 없다. 이미지의 크기가 달라질 뿐 자세히 보면 열화된 부분을 찾을 수 있다. 다시 이를 해결하기 위해 Reconstruction-based, Learning-based method가 나왔으며 가장 중요한 부분은 Learning-based method이다.

### Learning based method

1. **SRCNN, first deep learning based super resolution**

   딥러닝을 처음으로 SR에 적용한 논문인  [**“Image Super-Resolution Using Deep Convolutional Networks, 2014 ECCV”**](https://arxiv.org/pdf/1501.00092.pdf)  은 **SRCNN**이라는 이름으로 불리고,  2014년 ECCV에 공개되었다.

   ![](https://imgur.com/O5UL137.png)

   논문이 나온 때가 딥 러닝이 주목받고 조금 뒤의 시점이라, 많은 layer을 쌓지는 않았고 3개의 conv layer만 사용했다. 재밌는 점은 모델의 세 layer가 각각 옛 SR 관점에서 patch extraction, nonlinear mapping, reconstruction을 담당하고 있다며 적혀있다.

2. **Improve SISR**

   다음 두 논문은 기존 SRCNN에서 input 이미지를 HR 이미지의 해상도로 늘린 뒤 convolution 연산을 하는 과정에서 비효율적인 연산이 발생하고 있음을 지적하며, 이를 개선하기 위한 방법을 제안한다.

   1. **FSRCNN**

       [“**Accelerating the Super-Resolution Convolutional Neural Network, 2016 ECCV**”](http://mmlab.ie.cuhk.edu.hk/projects/FSRCNN.html). SRCNN의 후속 논문이며 **FSRCNN** 이라는 모델 이름을 가지고 있다.

      ![](https://imgur.com/gby4sLp.png)



      Input 이미지를 그대로 convolution layer에 집어넣고 마지막에 feature map의 가로, 세로 크기를 키워주는 deconvolution 연산을 사용하여 HR 이미지로 만든다. 이렇게 convolution 연산을 하게 되면 마지막에 키워주는 배수에 제곱에 비례하여 연산량이 줄어들게 된다. 그 결과 SRCNN에 비해 굉장히 연산량이 줄어들었고 거의 실시간에 준하는 성능을 보일 수 있음을 강조한다. 또한 연산량이 줄어든 만큼 layer의 개수도 늘려주면서 정확도도 챙길 수 있음을 보여준다.

   2. **ESPCN**

      다음은  [“**Real-Time Single Image and Video Super-Resolution Using an Efficient Sub-Pixel Convolutional Neural Network, 2016 CVPR**” ](https://arxiv.org/pdf/1609.05158.pdf)로, **ESPCN** 이라는 이름이다. 마찬가지로 이미지를 그대로 convolution layer에 집어넣지만 마지막에 feature map의 가로, 세로 크기를 키워주는 연산이 FSRCNN과 다르다. 이 논문에서는 sub-pixel convolutional layer 라는 구조를 제안하였으며, 이 연산은 pixel shuffle 혹은 depth to space 라는 이름으로도 불리는 연산이다.

      ![](https://imgur.com/1jXQDvq.png)



      r배의 up scaling을 하고자 하는 경우, Convolutional layer를 거친 뒤 마지막 layer에서 feature map의 개수를 r 제곱개로 만들어 준 뒤, 각 feature map의 값들을 위의 그림처럼 순서대로 배치하여 1 채널의 HR 이미지로 만들어주는 방식이다. 이러한 방식을 통해 효율적인 연산이 가능하고, 하나의 layer를 통해 upscaling을 하는 대신, 여러 layer의 결과들을 종합하여 upscaling을 할 수 있어서 정확도 또한 좋아질 수 있다. 즉 ESPCN은 FSRCNN의 deconvolution layer 대비 여러 장점을 얻을 수 있으며, 

3. **Deeper SR**

   1. **VDSR**

      [**“Accurate Image Super-Resolution Using Very Deep Convolutional Networks, 2016 CVPR”** ](https://cv.snu.ac.kr/research/VDSR/)논문은 **VDSR** 이라는 이름으로 불리며 Very Deep한 ConvNet 구조를 사용하여 Super Resolution했고, Deep network를 사용함으로써 기존 대비 높은 정확도를 달성했다.

      ![](https://imgur.com/7kclJ3l.png)

      VGG 기반의 20-layer convolutional network와 input image를 output에 더해주는 residual learning 을 사용했다. 초기에 높은 learning rate를 사용해 수렴을 잘 되게 하기 위해 gradient clipping도 같이 수행한다. 결론적으로 deep network를 이용해 기존 대비 정확도도 높이고, 학습률도 잘 수렴함을 확인할 수 있다.

   2. **DRCN**

      ["**Deeply-Recursive Convolutional Network for Image Super-Resolution**"](https://arxiv.org/abs/1511.04491) 논문은 최초로 Recursive learning을 사용했으며, **DRCN** 이라는 이름을 가지고 있다. 전체 모델의 학습 파라미터 수를 줄이기 위해 한 모듈을 16번 반복하는 특징을 가지고 있다.

      ![](https://imgur.com/6Yudh5P.png)

      다만 태생적으로 gradient vanishing이나 exploding에 취약하기 때문에 residual leraning이나 multi-supervision과 같은 학습 방법을 동시에 사용한다. 반복 중간 중간의 feature output을 모아 마지막에 weighted sum을 해 output image 를 만든다.

   3. **SSResNet**, **SRDenseNet**

      VDSR 논문에서 deeper model을 성공적으로 학습시켰지만 사실 VGG network 형태는 이런 깊은 모델에 적합하지 않다. 더 나은 성능을 위해 가장 쉽게 시도할 수 있는 방법은 더 깊은 모델을 사용하는 것이다. 이런 면에서 기본 모델의 골격을 Resnet이나 Densenet같이 성능이 검증된 모델로 바꾸는것이 자연스러웠다. 

      가장 먼저 ResNet을 적용한 SR model은 [**Photo-Realistic Single Image Super-Resolution Using a Generative Adversarial Network**](https://arxiv.org/abs/1609.04802) 로, **SRResNet**이라는 이름을 가지고 있다.

      비슷하게 DenseNet을 사용한 모델로는 [**Residual Dense Network for Image Super-Resolution**](<https://arxiv.org/abs/1802.08797>), **SRDenseNet**이 있다.

   4. **EDSR**

      네트워크를 단순히 늘리는 것이 아니라 기존 ResNet 모델을 SISR에 더 적합하게 개선한 연구가 [**Enhanced Deep Residual Networks for Single Image Super-Resolution**](<https://arxiv.org/abs/1707.02921>) 이다. **EDSR**이라는 이름을 가진 이 모델은 성능이 매우 뛰어날 뿐만이 아니라 이후 SISR 모델들에 큰 영향을 미쳤다.

      ESDR 이전의 모델들은 모두 classification 문제에서 좋은 성능을 보였던 네트워크들이 SISR 문제도 잘 해결할 것이라는 가정을 바탕으로 네트워크를 디자인했다. 하지만 high-level abstraction에 중점을 두는 classification과 low-level vision 문제인 SISR은 서로 특성이 다르다.

      ![](https://imgur.com/dAueRP3.png)

   5. 