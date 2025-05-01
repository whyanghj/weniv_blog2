# Abstract

현재 가장 좋은 성능을 내는 컴퓨터 비전 모델은 [image, label]처럼 고정된 형태의 데이터를 통해 학습된다. 고정된 형태의 데이터를 이용하면 모델의 일반화 성능(generality)와 다른 task에서의 사용가능성(usablity)가 제한된다. 쉽게 말하여 입력으로 사용한 feature의 특징이 1000개라면 해당 모델의 출력은 1000개의 클래스에 대한 분류, 탐지만 가능할 것이다.

따라서, 해당 논문에서는 이미지와 이미지를 설명하는 자연어(raw text)를 label, 즉 모델의 입력으로 사용하여 위의 문제를 해결하는 방안을 제안한다.

# Introduction and Motivating Work

NLP 분야에서는 BERT, GPT 등의 모델이 대규모 텍스트 데이터로 사전학습한 후, 각 task에 맞게 fine-tuning하는 방식으로 큰 성과를 이루었다. 

Vision 분야에서는 CNN 기반 모델에 라벨이 정해진 데이터셋을 기반으로 사전학습 후, 각 task에 맞게 fine-tuning하는 방식을 사용하였다. vision 분야에서 이 방식은 출력을 고정된 객체 범주 에 묶여 있고, 
새로운 task에 대해선 추가 라벨링을 해야하는 단점이 존재한다.

그래서 해당 논문에서는 이미지와 텍스트 쌍을 입력으로 사용하는 방법을 제안한다.

# 2. Approach

### 2.1 Natural Language Supervision
![cat](img/cat.jpg)
 - 이미지 학습은 지도학습으로 이뤄지는데 CLIP에서는 자연어 텍스트를 label 값으로 사용한다.
 - label로 숫자를 쓰는것보다 자연어를 사용함으로써 단순 분류를 넘어 개념적 표현 학습이 가능하다.
 - 위 예시와 같이 원래는 label로 숫자(ID)를 사용했지만 CLIP에서는 'a photo of a cat'과 같이 자연어를 사용하였다.

### 2.2 Creating a Sufficiency Large Dataset
![sun_cat](img/컴퓨터비전/sun_cat.jpg)
 - CLIP은 기존의 COCO나 YFCC100M처럼 작은 규모의 이미지-텍스트 데이터셋으로는 부족하다고 판단하여 직접 데이터셋을 구축하였다.
 - 사람들이 검색할만한 자연어 문장 쿼리 50만개를 만들고 그에 맞는 이미지와 문장을 CLIP의 데이터셋으로 사용하였다.
 - 이로 인해 위 사진과 같이 원래의 비전 모델이었다면 1(cat)으로 예측했을 상황에서 "썬글라스를 쓴 고양이"와 같은 예측을 할 수 있게 되었다.

### 2.3. Selecting an Efficient Pre-Training Method
![Contrastive learning](img/컴퓨터비전/const.jpg)
 - 하지만 2.1, 2.2 방식대로 학습시 학습 속도와 성능이 떨어지는 모습을 보였다. 그래서 CLIP은 **Contrastive learning**을 사용한다. contrastive의 방식은 위 예시를 참고해보자.
 - 기존의 학습은 각 이미지에 매칭되는 하나의 label만 학습을 시켰다면(하늘색 화살표), 
Contrastive learning은 각 이미지에 매칭되지 않는 label과도 학습을 진행해 유사도를 떨어트린다(빨산색 화살표).
즉, 매칭되는 텍스트의 유사도는 높이고 그렇지 않은 텍스트의 유사도는 멀어지게 한다. 

### 2.4. Choosing and Scaling a Model
![model](img/컴퓨터비전/CLIP.jpg)
 - CLIP은 두가지 모델 구조를 사용하였다. ResNet과 Vision Transformer(ViT)
 - Text Encoder는 NLP의 대표적인 모델 Transformer를 사용하여 문장을 벡터로 변환하고, Image Encoder는 ResNet을 이용하여 이미지를 벡터로 인코딩한다. 
 - 다양한 크기(작은 ResNet부터 큰 ViT까지)의 모델을 실험하며, 모델 크기와 성능 간의 scaling law(모델의 크기나 데이터의 양을 늘릴수록 성능이 어떻게 개선되는지를 수학적, 경험적으로 정리한 법칙)를 분석했다. 이를 통해 CLIP은 단순한 구조에서도 확장 가능성과 일반화 성능이 뛰어남을 입증했다.

### 2.5. Training
 - 총 8개의 모델로 4억 쌍의 데이터를 학습했다.(ResNet-50, ResNet-101, RN50x4, RN50x16, RN50x64)
 - 모든 모델은 ImageNet을 통한 사전학습 없이 from scratch(모델의 가중치를 0 또는 무작위인 상태)로 학습을 진행했다.


=> 사전 학습 없이 from scratch로 학습한 이유는 **사전학습 없이 텍스트와 이미지만으로 충분한 성능을 낼 수 있다.** 는 것을 증명하기 위해서이다.



# 3. Experiments

### 3.1 Zero-Shot Transfer
Zero-Shot Transfer란 학습에 사용되지 않은 데이터셋에 대해 이미지 분류를 하는 task를 의미한다.

Zero-Shot Transfer를 처음 사용한 연구인 Visual N-Grams와 비교해보자.
![model](img/컴퓨터비전/n_grams.jpg)
 - 3가지 데이터셋에 대해 모두 CLIP의 성능이 뛰어남을 확인할 수 있다.

### 3.2 Representation Learning
![representation](img/컴퓨터비전/repre.jpg)
 - CLIP을 포함한 모든 모델에 Linear probe를 이용해 성능을 비교한 figure이다. 각각 12개, 27개의 데이터셋에 대한 점수를 평균낸 것이다.
 
 ※ Linear Probe 방식은 모델의 파라미터는 그대로 두고, 모델의 맨 마지막 출력 부분에 Classification을 위한 Linear layer만 추가해 그 부분만 학습시키는 방식이다.

 - **모든 크기에서 CLIP 모델이 다른 모델보다 좋은 성능을 보였음을 알 수 있다.**

 ### 3.3 Robustness to Natural Distribution Shift
 - deep learning 모델들은 간단한 task라도 실수를 하는 낮은 성능을 보여주었으며, 새로운 벤치마크 데이터셋에서는 인간보다도 낮은 정확도를 내기도 했다. 그 이유는 over-fitting이었다. 본 논문에서는 CLIP은 이 문제를 해결하여 학습에 사용된 데이터와 다른 분포의 데이터에 대해서도 좋은 성능을 보였음을 보이고있다.
![robust](img/컴퓨터비전/robust.jpg)
 - **위 figure를 보면 동일한 ImageNet score(ImageNet 데이터셋 분류 성능)을 가진 모델들 중 CLIP(빨간별)이 가장 좋은 Transfer Score을 기록했다.**

 ※ transfer score는 특정 데이터셋으로 학습한 내용을 다른 문제나 데이터셋에서도 잘 활용할 수 있는지를 나타내는 성능 지표이다.