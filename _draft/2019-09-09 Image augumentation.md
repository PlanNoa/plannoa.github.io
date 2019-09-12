---
layout: post
title: "Image augumentation"
subtitle: "이미지 부풀리기"
categories: etc
tag: memo
comments: true
---

**imgaug를 이용하여 이미지 부풀리기**

Object detection을 포함한 많은 딥 러닝 이미지/영상 인식은 많은 수의 학습 이미지가 필요하다. 당장 확보할 수 있는 이미지의 수가 모델을 학습시키기에 적절하지 않을 때 원본 이미지를 변형해 새로운 이미지들을 만드는 부풀리기를 할 수 있다. 

Object detection을 하려 할 때, 이미지는 Annotation(keypoint, bounding box)을 만드는 수작업이 필요하다. 그럼 부풀리기를 통해 이미지를 더 많이 만들면, 각 이미지에 다시 작업을 해야 하는 것이 아닐까? 모양을 유지하고 필터를 추가하는 경우에는 괜찮지만, 이미지를 부풀리는 과정에서는 뒤집거나 모양을 꼬아버리는 경우도 많다. 

하지만 괜찮다. imgaug에서는 Annotation을 포함한 이미지 증폭이 가능하다. 이미지가 반전돠면, bounding box도 같이 반전된다.

![](https://imgur.com/DLByKLv.png)

- [imgaug](https://github.com/aleju/imgaug)	

imgaug은 학습 이미지를 bounding box, keypoint 등을 새 이미지에 적용한 채로 변환할 수 있다.



imgaug을 설치하다 shapely 에러가 났었다. shapely는 whl 파일로 다운받는 것을 추천한다.

**Examples**

여기부터는 [공식 페이지](<https://imgaug.readthedocs.io/en/latest/source/examples_basics.html>)를 참고한다.

