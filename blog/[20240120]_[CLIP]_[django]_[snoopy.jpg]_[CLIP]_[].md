# Abstract

현재 가장 좋은 성능을 내는 컴퓨터 비전 모델은 [image, label]처럼 고정된 형태의 데이터를 통해 학습된다. 고정된 형태의 데이터를 이용하면 모델의 일반화 성능(generality)와 다른 task에서의 사용가능성(usablity)가 제한된다. 쉽게 말하여 입력으로 사용한 feature의 특징이 1000개라면 해당 모델의 출력은 1000개의 클래스에 대한 분류, 탐지만 가능할 것이다.

따라서, 해당 논문에서는 이미지와 이미지를 설명하는 raw text를 label, 즉 모델의 입력으로 사용하여 위의 문제를 해결하는 방안을 제안한다.

# Introduction and Motivating Work

NLP 분야에서는 BERT, GPT 등의 모델이 대규모 텍스트 데이터로 사전학습한 후, 각 task에 맞게 fine-tuning하는 방식으로 큰 성과를 이루었다. 

Vision 분야에서는 CNN 기반 모델에 라벨이 정해진 데이터셋을 기반으로 사전학습 후, 각 task에 맞게 fine-tuning하는 방식을 사용하였다. vision 분야에서 이 방식은 출력을 고정된 객체 범주 에 묶여 있고, 
새로운 task에 대해선 추가 라벨링을 해야하는 단점이 존재한다.

그래서 해당 논문에서는 이미지와 텍스트 쌍을 입력으로 사용하는 방법을 제안한다.

# Approach

### Natural Language Supervision
![cat](img/cat.jpg)
 - 이미지 학습은 지도학습으로 이뤄지는데 CLIP에서는 자연어 텍스트를 label 값으로 사용한다.
 - label로 숫자를 쓰는것보다 자연어를 사용함으로써 단순 분류를 넘어 개념적 표현 학습이 가능하다.

### Creating a Sufficiency Large Dataset
기존 데이터셋은 작고 정제되어 확장성이 부족하다.