## 1. introduction
#### <Need>
 - 자율 주행에서는 Visual perception이 중요하다. 2012년 AlexNet 이후, 딥러닝의 정확도는 매우 높아졌다. 하지만 자율주행처럼 실시간으로 움직이는 환경에서는 정확도 뿐만 아니라 '속도'도 매우 중요하다. 그래서 지금까지는 속도 향상을 위해 하드웨어 가속기나 모델 경량화와 같은 테크닉들이 쓰였다.
#### <Solution> 
 - 위 문제 해결을 위해 본 논문은 MultiNet을 제안한다. classification, detection, segmentation을 하나의 모델 안에서 한번에 수행한다. 각 task들이 인코더는 공유하고 디코더는 다르다. 이로 인해 속도 향상을 기대할 수 있다. 

## 2. Related Work
 MultiNet이 다루는 주요 task인 detection, classification, semantic segmentic에 대한 기존 방식들을 확인한다.

 >- Classificatio
 
 AlexNet 이후, 대부분의 classification 은 딥러닝을 활용해왔다. 특히 ResNet은 매우 깊은 신경망을 훈련할 수 있게 해주는 sort 방식이다. 여러 센서 정보를 결합하는 sensor fusion 방식도 사용된다. 본 논문에서는 classification이 detection, semantic segmentation의 task를 돕는 역할을 한다.

 >- Detection

 전통적인 딥러닝 기반 객체 탐지 방식은 보통 두 단계로 이어진다.
 1. region proposals 생성
 2. 각 제안의 CNN으로 분류

 성능 향상을 위해 proposal 생성 단계에도 CNN을 쓰거나, 3D 정보를 이용한 detection도 제안되었다.
 최근에는 proposal 없이 단일 딥러닝 네트워크로 end-to-end 학습을 통해 직접 탐지하는 방식도 등장했다.(YOLO, SSD 등)
 이러한 방식은 학습과 추론 속도 모두 우수하지만, 정확도는 기존방식보다 뒤처진다.

 본 논문에서는 이러한 성능 차이를 좁히는 end-to-end trainable detector를 제안하며, 제안 기반 탐지 방식의 장점 중 하나인 scale-adjustable features에 주목한다.
 이를 반영해 ROI pooling 기법을 구현한다.

 >- Segmentation

 딥러닝의 성공에 따라 CNN 기반 Classifiers를 semantic segmentation에 적용하는 연구들이 활발히 이루어졌다.
 초기 방식들은 CNN을 활용해 슬라이딩 윈도우 방식으로 segmentation을 시도했으며, 이후에는 FCN이 제안되어 end-to-end 방식으로 semantic segmentation을 수행하게 되었다. 

 저해상도 특징 맵을 업샘플링할 때는 Transposed Convolution을 사용하며, 이후 다양한 FCN 구조가 등장했다. 의미 분할 성능 향상을 위해 CRF와의 결합도 연구되었고, 이를 RNN처럼 학습 가능한 구조로 바꾸는 연구도 있었다. 

 >- Multi-Task Learning

 MTL은 여러 작업을 함께 작업하여 더 나은 결과를 얻는 방법이다. CNN 기반의 다양한 MTL 방식이 제안되어 왔고, 특히 얼굴 인식 같은 분야에 효과적으로 적용되었다.
 분할을 통해 탐지나 인스턴스 분할(instance segmentation)을 수행하는 연구도 있었지만, 이들은 대체로 인스턴스 단위 작업을 최종 목표로 하고, 의미 분할은 중간 결과로만 사용한다.

최근에는 분류/탐지/분할을 모두 가능한 하나의 시스템을 만들고, 각각의 작업마다 다른 파라미터 셋을 학습하는 방식도 등장했으나, 이 경우에는 공동 추론(joint inference)이 어렵다.

본 논문에서 제안한 시스템은 기존과 달리, 의미 분할에서 학습된 풍부한 특징을 다른 작업에도 직접 활용할 수 있는 최초의 접근 방식이라 할 수 있다.

## 3. MultiNet for Joint Semantic Reasoning
MultiNet은 feed-forward 구조이다. semantic segmentation, image classification, object detection 세가지 task를 공통 인코더와 각 task에 해당하는 decoder 3개를 이용하여 joint inference(공동 추론)을 할 수 있다. 

![MultiNet Architecture](img/컴퓨터비전/MultiNet.jpg)

MultiNet은 end-to-end로 학습이 가능하며, 세가지 작업을 모두 포함한 공동 추론이 45ms 이하의 시간 내에 수행 가능하다.

#### 3.1 Encoder
인코더는 ResNet-50과 VGG16을 활용한다. 그리고 ImageNet 분류 데이터로 사전 학습된 가중치를 초기값으로 사용한다. 

#### 3.2 Classification Decoder
첫번째 버전은 fully-connected layer와 sofrmax 활성함수를 사용한다. 이 버전은 입력 이미지 크기를 224x224로 고정해서 사용한다. 따라서 이 구조는 VGG나 ResNet 분류 네트워크와 동일한 형태가 된다. 이 구조는 기준선 성능을 제공하는 모델로 사용된다. 다만, 이 디코더는 분할이나 탐지가 공동 추론이 불가능하다. 두 작업은 더 큰 입력 크기를 필요로 하기 때문이다.

두번째 버전은 앞서 만든 인코더가 생성하는 고해상도 feature를 활용할 수 있도록 설계되었다. 일반적인 image classification은 이미지 중심에 하나의 큰 물체가 있는 경우가 많기 때문에 작은 입력 크기로도 충분하다. 하지만 도로 이미지의 경우에는 작고 다양한 물체들이 여기저기 흩어져 있기 때문에, 고해상도 입력이 매우 중요하다. 그래서 입력 크기를 1248x348로 늘려서, 이미지의 각 위치마다 특징을 추출하도록 했다. 이로인해 39x12의 특징 맵이 생성되며, 각 셀은 32x32 픽셀 영역을 대표한다. 이 특징들을 활용하기 위해 채널 수를 줄이는 1x1 컨볼루션(bottleneck layer)를 적용한다. 이 레이어는 채널 수를 30으로 줄여서 계산 효율을 높여준다.