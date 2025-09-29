객체 탐지 방식 중 Two-Stage 접근법이다.
RPN(Region Proposal Network)와 Fast R-CNN가 Convlution 층을 공유하여 계산 효율성을 극대화함.

## RPN이란?
RPN은 이미지의 Feature map을 직접 분석해 개체가 있을 위치를 딥러닝을 통해 예측하여(약 ~2k) 더 효율적이고 빠르게 객체 탐지를 할 수 있도록 한다.

> Pytorch에서 학습시켜볼 수 있다.