# 합성곱 신경망 CNN이란?
이미지 학습 등에 널리 쓰이며 1998년에 얀 르쿤 연구팀이 개발한 대표적인 CNN 알고리즘 LeNet5가 제시되었지만 당시 하드웨어의 기술 부족으로 인해 최근에 들어서 빛을 발하고 있는 알고리즘이다.

### CNN이 필요한 이유
기존 DNN(Deep Neural Network)는 2차원 평면상의 이미지 학습을 어려워했음. 왜냐하면 DNN을 통해 학습시키기 위해선 1차원 데이터로 변환해야 했는데, 이 경우 이미지의 구조를 학습시키는 것이 힘들었기 때문임.
이때 등장한 것이 바로 CNN이고, 이는 일련의 과정을 거쳐 2차원 평면의 위치 관계를 반영한 Feature Map을 얻어 1차원 데이터로 변환해도 이미지의 특징을 보존하여 학습시킬 수 있다. 실제로 ILSVRC 대회에서 CNN을 이용한 AlexNet과 VGGNet이 굉장히 우수한 성과를 냈다.

아무튼 CNN 방식으로 학습시키기 위해선 아래의 과정이 요구된다.
Input -> Convolutional Layer(Filter를 이용) -> Pooling Layer -> 반복 -> Flatten Layer -> FC(Fully-Connected Layer)

위의 용어를 차근차근 정리해보겠다.
## Convolutional Layer?
CNN의 핵심이 되는 부분이다. 이 층에서는 이전 층의 값을 Filter를 이용해 곱하고, 모두 더한다. 이것이 "합성곱" 신경망인 이유이다.
## Filter?
Filter란 n X m 크기로 이루어지며 Convolutional Layer에서 값을 곱할 때의 기준이 된다. 이 FIlter를 이용해서 이미지의 패턴을 식별할 수 있다.
## Pooling Layer?
Pooling Layer는 이전 층의 값들의 크기를 축소하는 과정이다.
n X m 크기의 사각형을 이전 층의 값들 위에서 움직이며 다음 층의 값을 구성한다.