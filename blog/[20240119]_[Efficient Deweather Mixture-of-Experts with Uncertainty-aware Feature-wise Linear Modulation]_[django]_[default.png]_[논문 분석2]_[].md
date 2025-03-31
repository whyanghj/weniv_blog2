## Introduction

MoE(Mixture of Exprts)는 multi-task learning에서 높은 확장성을 보인다. 하지만 비효율적이고 많은 계산량이라는 단점을 갖고있다. 그렇기에 본 논문에서는 MoFME를 설명하고있다. MoFME는 expert의 개수를 줄이고 다양한 필터를 이용한다. 라우터는 UaR(uncertainty-aware Router)를 사용한다.

해당 논문에서 설명하는 MOE의 단점에 대해서 알아보겠다.

▶ 여러개의 FFN(Feed Forward Network) expert를 가져서 너무 크고 무겁다.

![MOE의 구조](img/컴퓨터비전/MOE.jpg)

위 MOE의 구조처럼 학습과 예측을 진행하는 expert가 병렬 형태로 여러개 존재하여 연산량이 늘어나고 메모리 사용량도 커진다. 그로인해 성능이 저하된다.
특히 자율주행자동차에 사용되는 작은 컴퓨터에는 MOE를 사용할 수 없다.  

▶ 기존에는 단순한 Linear Router를 사용하여 expert를 선정하였다. 

Linear Router 사용으로 라우터 가중치가 잘 조정되지 않았다. 쉽게 말해 예시2와 같이 입력이 복잡한 경우 전문가 선택에 있어서 선택 확률 분포가 왜곡되거나 편향되는 경우가 있다. 
이를 해결하기 위해 Multi-gate MoE(Ma et al. 2018) 라는 추가 게이팅 네트워크를 도입했다. 하지만 늘어난 게이트만큼 계산비용이 추가로 발생한다. 

< MOE와 Multi-gate MOE의 구조 >
>기존 MOE
>: 입력 -> Linear Layer -> sofrmax -> expert 선택
>
>Multi-gate MOE
>: 입력 -> gate 선택 -> 각 gate에 맞는 expert 실행

<복잡한 입력>
>- 비오는날 날씨 탐지
>- 안개낀 도로 차선 탐지
>- 눈 오늘 날 객체 탐지 

▶ 여러 expert들이 따로 학습되어 중복된 가중치를 학습하는 경우가 있다.                                                              
=> 자원 낭비가 발생하고 모델의 크기가 커진다.
이를 해결하기 위해 본 논문은 FM(Feature Modulation)기법을 활용하였다. 간단히 말하면 expert의 개수를 줄이고 expert 하나에 다양한 필터를 씌워 다양한 입력에 따른 다양한 가중치를 사용하게 만들어준다.

▶ Top-K 메커니즘으로 인한 모드 붕괴(mode collapse)
MOE의 라우터에서는 softmax의 결과값인 확률값에 Top-K를 적용한다. Top-K 적용시 비선형성이 추가되어 backpropagation 실행시 특정 expert의 가중치만 update되어 이외의 expert는 사용되지 않는 상황이 발생한다.

![t-SNE](img/컴퓨터비전/T_SNE.jpg)
t-ZNE는 high-dimential feature를 2D로 시각화해주는 도구다. 이를 통해 MoE에 비해 MoFME 사용시 라우터에 의한 입력의 분리와 군집이 잘 이루어졌음을 알 수 있다.

## Proposed Methods

#### Feature Modulated Expert(FME)

본 논문에서는 ViT(vision transformer) 아키텍처를 기반으로 한 MoE 설정을 설명한다. ViT 아키텍처에서 dense FFN은 MoE 계층으로 대체된다. FME는 모델 내부의 intermediate feature을 입력에 따라 선형적으로 modulate하는 기법이다.

![](img/컴퓨터비전/TOP_K.jpg)

MoE 계층의 입력은 Multi-head Attention 계층에서 나온 N개의 토큰이며 x∈R^D로 표현된다. 각 토큰 x는 입력에 따라 동적으로 작동하는 라우터에 의해 E개의 전문가 집합 중 일부에 해당되며, 이때의 라우터 가중치는 r(x)로 표현된다. r(x)는 아래와 같이 표현된다. Wr은 학습 가능한 매개변수로, 입력 토큰을 E개의 전문가 선택을 위한 router logits으로 매핑하는 역할을 한다.
계산 비용을 줄이기 위해, 모델 내 expert들은 sparse하게 활성화되며, Top-K는 softmax의 확률값이 가장 큰 K개의 값만 남기고 나머지는 0으로 설정한다.

![](img/컴퓨터비전/MOE_.jpg)

명확한 표기를 위해 i번째 전문가의 라우터 가중치를 ri(x)로 나타낸다. 따라서 MoE 계층의 출력은 위와 같이 입력에 대한 expert들의 출력값을 라우터 가중치를 이용해 가중합하여 계산한다.
여기서 ei(x)는 i번째 expert의 기능을 의미하며, 일반적으로 ViT에서는 FFN으로 설계되는 부분이다.

![dddd](img/컴퓨터비전/FME.jpg)

위 식은 입력 x와 학습 파라미터 γ, β의 연산을 통해 MoFME의 연산과정을 나타낸 식이다. 

① : i번째 expert가 학습한 modulation parameter. affine transformation에서 사용된다. γ는 감쇠계수로 입력의 scale을 의미한다. β는 이동계수로 입력의 shift를 의미한다. 여기서 유의할 점은 γ, β 모두 작은 신경망이다. γ, β가 직접 학습되는게 아닌 γ, β를 계산하는 네트워크 파라미터가 학습된다.    

② : router가 판단한 i번째 expert의 중요도. softmasx와 Top-K를 통해 계산된 값이다.

③ : 입력 x를 i번째 expert에 맞춰 변형하는 식. γ와 x는 Hadamard Product(요소별 곱)을 한다.

④ : 각 expert의 중요도와 FM을 활용해 전처리한 입력을 곱해서 모두 더한 가중치 합.

#### Uncertainty-aware Router(UaR)

본 논문은 FME의 성능을 향상시키기 위해 UaR을 제시했다. UaR은 쉽게 말해 라우터의 결과에 대해서 불확실성을 따지는 것이다. softmax 또는 엔트로피 등 사용시 불확실한 확률값을 가진다면 별도의 조치를 시행한다. 취할수 있는 조치는 여러가지가 있다.
1. Fall lack expert를 따로 두어 불확실할 시 고정된 expert를 사용한다.
2. 확률 상위 3개의 expert만 실행후 평균 또는 가중합을 사용한다.(Top-K)
3. 확률을 기반으로 soft routing을 사용한다.
4. 입력을 거부한다.

MoFME의 라우터는 MC Dropout 기법을 기반으로, 라우터 가중치에 대한 암묵적인 불확실한 추정을 수행한다. Ovadia et al. (2019), Ashukha et al. (2020)의 연구에 따라 앙상블 기반 불확실성 추정 방법이 가장 우수한 calibration 및 예측 정확도를 가지지만 높은 계산 복잡도와 저장 비용이 요구된다. 그래서 MC Dropout기법을 선택하였다.

![ㅇㅇㅇ](img/컴퓨터비전/UaR_rx.jpg)

ri(x)는 위의 설명과 같이 특정 라우터의 출력이며 가우시안 분포로 간주하여 불확실성을 보정할 수 있다. 분포의 평균과 공분산을 "router ensemble"을 통해 추정할 수 있다. 즉, 입력 x를 MC Dropout을 적용한 동일한 라우터에 총 M번 통과시켜 위의 식과 같은 출력 집합을 얻는다.

![ㅇㅇㅇㅇ](img/컴퓨터비전/UaR.jpg)

위 수식은 라우터 출력 r(x)를 MC Dropout을 반복하여 알아낸 불확실성에 따라 정규화하는 과정이다. r(x)-μ로 현재 라우터 출력이 평균에서 얼마나 벗어나 있는지를 확인한다. 이 값들의 공분산을 구해서 작으면 중요도를 높아진다. 그 후 분모는 L2 norm을 사용하여 벡터의 크기를 1로 만들어 벡터의 방향 정보만 사용하였다.