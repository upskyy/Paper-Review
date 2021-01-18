# Attention Is All You Need
https://arxiv.org/pdf/1706.03762.pdf

## Abstract

- 복잡한 CNN과 RNN을 사용하지 않고 유일하게 attention mechanism만 사용하는 Transformer 구조를
제안한다. 

## 1. Introduction
---
Transformer는 recurrence를 아예 배제하고 전적으로 attention mechanism에만 의존한다.  
그래서 더 병렬화하여 계산할 수 있고 그 결과 8개의 GPU로 12시간 학습하여 State-Of-The-Art (SOTA) 정도의 성능을 낼 수 있었다.


## 2. Background
---
self-attention은 자기 문장 스스로에게 attention을 수행하여 문장의 표현들을 학습할 수 있도록 하는 것이다. intra-attention 이라고도 불린다.
최근에 나온 GPT나 BERT도 본 논문에서 제안한 transformer architecture를 따르고 있다.

## 3. Model Architecture
---
### 3.1 Encoder and Decoder Stacks
![a](https://user-images.githubusercontent.com/54731898/104893547-e83a8000-59b6-11eb-867f-9257e8d106ee.PNG)  
위의 사진은 transformer model의 전체적인 구조를 표현하고 있는 사진이다.

**Encoder** : RNN을 사용하지 않는 대신에 문장 내에 있는 단어들의 위치 정보를 encoding 해주기 위해서 input embedding size와 동일하게 하여 positional encoding 해주고 input embedding 결과값과 더해준다. 
Encoder는 동일한 6개의 레이어를 쌓은 형태이다. 각 레이어는 두 개의 서브 레이어로 이루어져 있는데 첫 번째는 multi-head self-attention이고, 두 번째는 feed-forward layer이다. 
앞에 더해주었던 embedding vector를 query, key, value로 각각 복사해주고 multi-head self-attention을 진행한 후에 residual connection을 한 후 layer normalization을 한다.
나온 결과 값을 두 번째 서브 레이어로 넣어주어 feed forward 계산을 하고 residual connection을 진행한 후 layer normalization을 한다.
multi-head attention은 인풋과 아웃풋 사이즈가 동일하다.  
하지만 논문에서는 residual connection을 쉽게 하기 위해 embedding layer 뿐만 아니라 모델의 모든 서브 레이어의 사이즈를 512로 진행하였기 때문에 서브 레이어의 인풋과 아웃풋 사이즈가 모두 동일하다.

**Decoder** : decoder에서도 encoder와 동일한 embedding 작업을 거친 후에 레이어의 넣어준다.
Decoder 또한 동일한 6개의 레이어를 쌓은 형태이고 각 레이어마다 세 개의 서브 레이어를 가지고 있다.
첫 번째 서브 레이어는 지금까지 출력한 단어만 어텐션 할 수 있도록 하기 위해 뒤의 단어들에 마스크를 
씌우고 multi-head self-attention, residual connection, layer normalization을 진행한다. 
두 번째 서브 레이어에서는 encoder output이 key, value로 들어오고 이전 서브 레이어의 결과값이 query로 들어와서 마찬가지 과정을 진행한다.
세 번째 서브 레이어에서는 feed-forward layer과정을 거친다.
이렇게 동일한 6개의 레이어를 거친 후에 linear layer를 거치고 softmax 계산을 하여 아웃풋 확률 값을 얻을 수 있다.

![f](https://user-images.githubusercontent.com/54731898/104893643-056f4e80-59b7-11eb-91ee-21e6fda9f8fb.PNG)
![g](https://user-images.githubusercontent.com/54731898/104893692-16b85b00-59b7-11eb-9725-93be09df1cd8.PNG)

Residual Connection을 하는 이유는 ?  
Residual Connection(다른 말로 skip connection)을 하면 y = x + F(x) 형태가 된다.
위의 사진 처럼 f2 레이어가 잘 작동하지 않아도 여러 가지 방법으로 그래디언트가 
잘 전달 될 수 있기 때문이다. 만약에 Residual Connection을 하지 않았다면 f2 레이어가 잘 작동하지
않았을 때 그래디언트가 잘 전달 될 수 없었을 것이다. 이 처럼 Residual Connection을 사용하면 조금 더
robust한 구조를 만들 수 있다.

더 자세한 내용은 Residual Networks Behave Like Ensembles of Relatively Shallow Networks 논문   
https://arxiv.org/pdf/1605.06431.pdf 을 참고하시면 좋을 것 같습니다.


![h](https://user-images.githubusercontent.com/54731898/104893694-17e98800-59b7-11eb-84d9-518cfc1f3bb8.PNG)    
Layer Normalization을 하는 이유는 ?
이미지와 다르게 자연어에서는 입력 시퀀스가 매번 다르기 때문에 Batch Normalization보다 
Layer Normalization을 해주는 것이 더 좋다.
둘을 비교 해보면 batch normalization보다 layer normalization이 파라미터 수가 더 적은 것을 알 수 있다.
또한 batch normalization을 사용했을 때, test sequence가 train sequence보다 길다면 문제가 발생할 수 있다.
하지만 layer normalization을 사용하면 문장마다 계산하기 때문에 길이가 더 긴 문장이 와도 문제가 발생하지 않는 것을 알 수 있다.

더 자세한 내용은 Layer Normalization 논문 https://arxiv.org/pdf/1607.06450.pdf 을 참고하시거나,  
잘 정리된 이 [링크](https://zhangtemplar.github.io/normalization/)를 참고하시면 좋을 것 같습니다.


### 3.2 Attention
![b](https://user-images.githubusercontent.com/54731898/104893552-e96bad00-59b6-11eb-858f-504dcbb82749.PNG)  
왼쪽은 Scaled Dot-Product Attention을 표현한 것이고, 오른쪽은 Multi-Head Attention을 표현한 것이다.
  
#### 3.2.1 Scaled Dot-Product Attention
![c](https://user-images.githubusercontent.com/54731898/104893570-ee306100-59b6-11eb-9b33-e8b0cc58cffa.PNG)  
위의 식은 scaled dot-product attention mechanism에서의 score 함수 식이다. 

우선 query와 key를 Dot Product하여 유사도를 구한다. 그 다음 key dimension의 루트값으로 나누어준다.
나누어주는 이유는 값이 너무 커서 그대로 softmax에 넣어주면 그레디언트 값이 너무 작아 학습이 잘 되지 않는다. 따라서 학습을 안정화 시키기 위해서 나누어준다.
그 다음으로 softmax를 취해서 확률 값으로 바꾸어주고 계산 결과를 value와 곱해준다.

흔하게 사용되는 dot-product attention과 additive attention은 시간 복잡도가 비슷하지만 dot-product attention이 더 빠르고 메모리 사용이 효율적이라서 dot-product attention을 사용하였다고 말하고 있다.
그리고 위의 설명한 이유 때문에 key dimension의 루트값으로 나누었다고 말하고 있다.
  
#### 3.2.2 Multi-Head Attention
![d](https://user-images.githubusercontent.com/54731898/104893634-02745e00-59b7-11eb-9cde-3e97334fd7e2.PNG)  
multi-head attention은 head를 여러개 사용하는 것이다. 예를 들어 query, key, value의 dimension이 512이고 head의 수가 8개라고 하면 각각의 head가 64 dimension을 가지고 병렬적으로 scaled dot-product attention을 진행하는 것이다.
나온 결과값을 합쳐주고 linear layer를 통과하여 아웃풋을 뽑아낸다.

multi-head attention을 사용하는 이유는 어텐션 맵을 head 수만큼 만들 수 있기 때문이다.
어텐션 맵을 여러개 만들어 보는 것이 좋은 이유는 모델이 다양한 경우의 수를 고려해볼 수 있기 때문이다.
일종의 앙상블 개념으로 이해할 수 있다.
  
#### 3.2.3 Applications of Attention in our Model
트랜스포머 구조에서 세 가지의 multi-head attention을 사용하고 있다.
첫 번째는 decoder의 두 번째 서브 레이어로, key와 value는 encoder의 아웃풋이 들어오고
query는 decoder의 이전 레이어 결과값이 들어온다.
두 번째는 encoder에서 사용하고 있는 self-attention이다. self-attention은 같은 인풋에 각각 다른 weight가 곱해진 결과값이 query, key, value에 들어간다.
세 번째는 decoder에서 mask를 씌우고 진행하는 multi-head attention이다. 

### 3.3 Position-wise Feed-Forward Networks
![e](https://user-images.githubusercontent.com/54731898/104893640-030cf480-59b7-11eb-8681-e7bbdc854aa2.PNG)  
linear layer를 거치고 ReLU activation 함수를 거친 후에 다시 linear layer를 거치는 것을 확인 할 수 있다.
dimension은 크게 input과 output으로 보면 모두 512이지만, 내부적으로 2048 dimension으로 깊게 맵핑된 후 나오는 것을 확인 할 수 있다.
  
### 3.4 Embeddings and Softmax
일반적인 Seq2Seq 구조처럼 단어를 embedding dimension으로 맵핑해주는 과정을 거친다.
  
### 3.5 Positional Encoding
RNN과 CNN을 사용하지 않기 때문에 위치에 대한 정보도 알려주어야 한다.
논문에서는 주파수가 다른 sin, cosine 함수를 사용했을 때와 
학습된 positional embedding을 사용했을 때 성능 차이는 거의 없었다고 말하고 있다.


## 4. Why Self-Attention
---
첫 번째 장점은 layer 마다 전체적인 계산 복잡도가 줄어든다.  
두 번째 장점은 recurrence를 아예 배제하여 병렬 처리가 가능하다.  
세 번째 장점은 긴 문장에 대해서도 잘 처리할 수 있다.  

![i](https://user-images.githubusercontent.com/54731898/104893699-191ab500-59b7-11eb-86a4-6c7a6c5b1661.PNG)  
n은 시퀀스 길이, d는 dimension, k는 convolution의 kernel size이다.
일반적으로 n이 d보다 작기 때문에 self-attention이 RNN보다 복잡도가 더 낮다.
또한 RNN은 순차적으로 하나하나씩 처리하기 때문에 시퀀스 길이만큼 필요하지만,
병렬적인 처리가 가능한 self-attention은 한번에 처리하는 것을 알 수 있다.


## 5. Training
---
### Residual Dropout
residual connection을 하기 전에 서브 레이어 아웃풋에 dropout ratio 0.1을 적용하였다.
뿐만 아니라 positional encoding과 input embedding을 더한 값에도 dropout을 적용하였다.

### Label Smoothing
label smoothing 값으로 0.1을 사용하였다.
즉, hard target을 soft target으로 바꾸는 것이다.
모델이 조금 더 불확실하게 학습하여 정확도를 향상시켰다. 

## 6. Results
![j](https://user-images.githubusercontent.com/54731898/104893706-1a4be200-59b7-11eb-85a2-56e078225c22.PNG)  
트랜스포머의 base model로도 State-Of-The-Art (SOTA) 정도의 성능을 낼 수 있었다.  
또한 학습 시간이 훨씬 짧은 것도 알 수 있다.


