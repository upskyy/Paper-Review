# Conformer: Convolution-augmented Transformer for Speech Recognition 
https://arxiv.org/abs/2005.08100
 


## Abstract
---
- 최근에는 트랜스포머와 CNN을 기반으로 한 모델들이 음성인식에서 좋은 성능을 보이고 있습니다.
- 전체적인 의미를 잘 파악하는 트랜스포머와 부분적인 의미를 잘 파악하는 CNN을 결합한 **Conformer**를 제안합니다.
- Conformer encoder + RNN transducer 구조 입니다.

## 1. Introduction
---
트랜스포머와 convolution은 각각의 한계점을 가지고 있습니다. 트랜스포머는 넓게 넓게 정보를 잘 파악하지만, 좁은 범위의 특징을 뽑아내는 것을 할 수 없습니다. 반면에 convolution은 좁은 범위의 정보들을 잘 뽑아냅니다.
그래서 위, 아래로 feed forward 모듈을 갖는 샌드위치 형태로, 트랜스포머와 컨볼루션을 새롭게 결합한 Conformer를 소개합니다.  

또한 어텐션 헤드의 개수와 컨볼루션 커널 사이즈, activation 함수, feed-forward 레이어의 위치, 컨볼루션 모듈에 적용한 다양한 전략 등이 정확성 향상에 얼마나 영향을 미쳤는지 연구하였다고 본 논문은 말하고 있습니다.  
이 부분은 아래쪽에서 좀 더 자세히 설명하겠습니다!  


## 2. Conformer Encoder
---
![a](https://user-images.githubusercontent.com/54731898/110105271-865d9a80-7deb-11eb-99cf-4e1eb4f1065f.PNG)  
Conformer block은 4개의 모듈로 구성되어 있습니다. 첫 번째 feed-forward 모듈, self-attention 모듈, convolution 모듈, 그리고 두 번째 feed-forward 모듈로 구성되어 있습니다.  



### 2.1. Multi-Headed Self-Attention Module 
---
![mhsa](https://user-images.githubusercontent.com/54731898/110105279-88bff480-7deb-11eb-837d-2a13700cef00.PNG)  

Multi-headed self-attention (MHSA)는 [Transformer-xl: Attentive language models beyond a fixed-length context](https://arxiv.org/abs/1901.02860) 논문에서 제안한 relative positional encoding을 사용 하였습니다.  
Relative positional encoding 이란 말 그대로 절대적인 위치가 아닌 상대적인 위치 정보를 통해 인코딩하는 방식입니다.  
또한, Relative positional encoding은 다양한 발화 길이를 좀 더 일반화 시켜줌으로서, 인코더를 robust 하게 만들어준다는 장점이 있습니다.  
Layernorm을 앞단에 주어 학습을 좀 더 원활하게 하도록 하였고, dropout을 통해 정규화를 해주었습니다.



### 2.2. Convolution Module
---
![b](https://user-images.githubusercontent.com/54731898/110105275-878ec780-7deb-11eb-8145-742562fd34d5.PNG)  

- #### Pointwise Convolution
![d](https://user-images.githubusercontent.com/54731898/110117323-0986ec80-7dfc-11eb-840b-8dbf9dd53fcd.PNG)  

다른 말로는 1 x 1 Convolution으로 불리며, output의 크기가 변하지 않기 때문에 channel의 수를 조절하고 싶을 때 사용합니다.  


- #### GLU Activation
![e](https://user-images.githubusercontent.com/54731898/110118051-1d7f1e00-7dfd-11eb-9856-c429d547d54d.PNG)  
이 함수는 입력의 절반에 시그모이드 함수를 취한 것과 나머지 입력의 절반을 가지고 pointwise 곱을 계산합니다.  
따라서 출력 값의 차원은 입력 값의 차원의 절반이 됩니다.  


- #### Swish Activation
![f](https://user-images.githubusercontent.com/54731898/110118516-d5acc680-7dfd-11eb-9bb7-be6a204181ce.PNG)  
![g](https://user-images.githubusercontent.com/54731898/110118690-14db1780-7dfe-11eb-932c-152440aaad24.PNG)  
시그모이드 함수에 x를 곱한 식 입니다.  


- #### Depthwise Convolution
![h](https://user-images.githubusercontent.com/54731898/110119173-c2e6c180-7dfe-11eb-8136-161f39feea7c.PNG)  
각 채널마다 필터가 존재하여, 각 채널 고유의 Spatial 정보만을 학습하게 됩니다.  
또한 결과적으로 입력 채널 수 만큼 그룹을 나눈 Grouped Convolution과 같습니다.  


## 2.3. Feed Forward Module
---
![ff](https://user-images.githubusercontent.com/54731898/110105282-89588b00-7deb-11eb-84d1-3670f99a55b1.PNG)  



## 2.4. Conformer Block
---
![z](https://user-images.githubusercontent.com/54731898/110121200-72249800-7e01-11eb-9e9d-6a4f112b9014.PNG)  

앞서 말씀드린 샌드위치 구조는 [Macaron-Net](https://arxiv.org/abs/1906.02762)에서 영감을 받았다고 합니다.  
절반의 residual connection과 함께 샌드위치 구조의 feed-forward 모듈이 single feed-forward 모듈에 비해 더 좋은 성능을 보인다고 말하고 있습니다.  


## 3. Experiments
---
![m](https://user-images.githubusercontent.com/54731898/110122656-3985be00-7e03-11eb-8eef-870a2d35843b.PNG)  
Decoder로는 single-LSTM을 사용하였고 small, medium 그리고 large 모델의 파라미터는 다음과 같습니다.  

![n](https://user-images.githubusercontent.com/54731898/110122647-37bbfa80-7e03-11eb-84b6-47d1b281cdce.PNG)  
현재 존재하는 모델 중에 word error rate(WER)가 가장 낮은 것을 확인 할 수 있습니다.  


## Ablation Studies
---
### Conformer Block vs. Transformer Block  

 ![X](https://user-images.githubusercontent.com/54731898/110124130-15c37780-7e05-11eb-9a70-3f4a98677cbd.PNG)  
 차이점들 중에, Convolution sub-block이 가장 중요한 차이라고 말합니다. 
또한 swish activation 함수를 사용하여 더 빠른 수렴을 이끌었다고 말합니다.  
표는 여러 가지 변화에 대한 영향을 보여줍니다.  


### Combinations of Convolution and Transformer Modules  
![Y](https://user-images.githubusercontent.com/54731898/110124133-16f4a480-7e05-11eb-83fd-8e877591023b.PNG)  
Depthwise convolution 대신에 lightweight convolution으로 바꾸어 보았습니다.  
또한 convolution 모듈을 MHSA 모듈 전으로 이동해보았습니다.  
마지막으로 인풋을 두 개로 나누어 하나는 MHSA 모듈로, 나머지 하나는 convolution 모듈로 보내고 결과값들을 합치는 방법으로 진행해보았습니다.  
표는 세 가지 경우에 대한 결과 값입니다.


### Macaron Feed Forward Modules  
![Q](https://user-images.githubusercontent.com/54731898/110124140-178d3b00-7e05-11eb-8ad1-b138d1fbf55a.PNG)  
Conformer와 single FFN 그리고 full-step residual의 결과 값들을 비교한 표 입니다.


### Number of Attention Heads  
![bb](https://user-images.githubusercontent.com/54731898/110126079-65a33e00-7e07-11eb-9f1d-1e89e068acbe.PNG)  
Attention 헤드 수를 증가시키면서 실험한 결과, 16 까지는 accuracy가 증가하는 것을 확인 할 수 있습니다.  


### Convolution Kernel Sizes  
![cc](https://user-images.githubusercontent.com/54731898/110126085-663bd480-7e07-11eb-82a7-560b9adab040.PNG)  

Large 모델에서 커널 사이즈를 3, 7, 17, 32, 65로 하여 실험한 결과, 32가 가장 적합하다고 말하고 있습니다.  


## Conclusion
---
Conformer 구조에서 convolution 모듈을 포함한 것이 성능에 중요한 영향을 미쳤고, 더 적은 파라미터 수로 더 좋은 성능을 얻었다고 합니다. 

## Reference
https://hichoe95.tistory.com/48
https://arxiv.org/abs/2005.08100
