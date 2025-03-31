## introduction

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

본 논문에서는 ViT(vision transformer) 아키텍처를 기반으로 한 MoE 설정을 설명한다. ViT 아키텍처에서 dense FFN은 MoE 계층으로 대체된다. 

![](img/컴퓨터비전/TOP_K.jpg)

MoE 계층의 입력은 Multi-head Attention 계층에서 나온 N개의 토큰이며 x∈R^D로 표현된다. 각 토큰 x는 입력에 따라 동적으로 작동하는 라우터에 의해 E개의 전문가 집합 중 일부에 해당되며, 이때의 라우터 가중치는 r(x)로 표현된다. r(x)는 아래와 같이 표현된다. Wr은 학습 가능한 매개변수로, 입력 토큰을+ E개의 전문가 선택을 위한 router logits으로 매핑하는 역할을 한다.
계산 비용을 줄이기 위해, 모델 내 expert들은 sparse하게 활성화되며, Top-K는 softmax의 확률값이 가장 큰 K개의 값만 남기고 나머지는 0으로 설정한다.

![](img/컴퓨터비전/MOE_.jpg)

명확한 표기를 위해 i번째 전문가의 라우터 가중치를 ri(x)로 나타낸다. 따라서 MoE 계층의 출력은 위와 같이 입력에 대한 expert들의 출력값을 라우터 가중치를 이용해 가중합하여 계산한다.
여기서 ei(x)는 i번째 expert의 기능을 의미하며, 일반적으로 ViT에서는 FFN으로 설계되는 부분이다.

![dddd](img/컴퓨터비전/FME.jpg)

위 식은 입력 x와 학습 파라미터 γ, β의 연산을 통해 MoFME의 연산과정을 나타낸 식이다. 

① : i번째 expert가 학습한 modulation parameter. affine transformation에서 사용된다.aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa

② : router가 판단한 i번째 expert의 중요도. softmasx와 Top-K를 통해 계산된 값이다.

③ : 입력 x를 i번째 expert에 맞춰 변형하는 식. γ와 x는 Hadamard Product(요소별 곱)을 한다.

④ : 각 expert의 중요도와 FM을 활용해 전처리한 입력을 곱해서 모두 더한 가중치 합.

 

