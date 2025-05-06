# Abstract

기존의 **Vision-Language Pre-training**은 아래 두가지의 문제점을 가진다.

1. understanding기반 task나 generation 기반 task 중 하나의 분야에서만 뛰어난 모습을 보였다.
    **ex)**
    - understanding 기반 task는 임의의 텍스트가 주어졌을 때 관련된 이미지를 찾는 것
    - generation 기반 task는 임의의 이미지가 주어졌을 때 그 이미지를 설명하는 문장을 생성하는 것

2. 학습에 사용한 데이터는 웹에서 수집한 노이즈가 있는 대규모 image-text 쌍이다. 하지만 이는 최적의 supervision 데이터가 아니다. 
    
이를 해결하기 위해 **BLIP**이라는 새로운 VLP 프레임워크를 제안한다. 장점은 아래와 같다.

1. 기존에는 2가지 task에 대해 다른 모델이 필요했지만, BLIP은 모델 하나로 understanding, generation 2가지 task 모두에 적용 가능하다. 
2. **caption bootstrapping**을 통해 위에서 말한 데이터속 노이즈를 제거한다.

# Introduction

Vision-language pre-training(VLP)는 다양한 multimodal downstream task에서 큰 성공을 거두었따. 그러나 아래의 두가지 주요 한계점이 존재한다.

1. model 관점에서의 한계

대부분의 방법들은 encoder-only 모델 또는 encoder-decoder 모델 중 하나를 채택한다. 하지만 **encoder-only**모델은 text generation에 바로 적용하기 어렵고, **encoder-decoder**모델은 이미지-텍스트 검색 task에 잘 쓰이지 않는다.

2. data 관점에서의 한계

CLIP, ALBEF 등의 multimodal model들은 대부분 웹에서 수집한 대규모 image-text pairs를 사용했다. 이 데이터의 text들은 노이즈가 많고, 실제 이미지와 매칭되지 않는 경우도 많다. 

#### => BLIP 제안

BLIP는 2가지 새로운 아이디어를 포함한다. 

1. Multimodal mixture of Encoder-Decoder(MED)

효과적인 멀티태스킹과 유연한 transfer를 위한 새로운 아키텍처이다. MED는 세가지 방식으로 작동한다.

 - Unimodal encoder
    - image-text의 contrastive learning(대조 학습)을 통해 비전 및 언어 표현을 정렬하도록 학습된다.
 - Image-grounded text encoder
    - 추가적인 cross-attention 층을 사용하여 비전-언어 상호작용을 모델링하고, image-text matching 손실로 학습된다. 즉, 주어진 이미지-텍스트 쌍이 맞는 쌍인지 아닌지 판단하는 인코더이다.(binary classification)
 - Image-grounded text decoder
    - 양방향 self-attention 층을 casual-attention층으로 교체하고, cross-attention과 feed-forward network는 인코더와 공유한다. 디코더는 이미지가 주어졌을때 그에 매칭되는 텍스트를 생성한다.

2. Captioning and Filtering
 - Captioner
    - 이미지가 주어졌을때 합성 캡션을 생성한다.
 - Filter
    - 원본 텍스트와 합성된 텍스트 모두에서 노이즈가 있는 캡션을 제거한다.
