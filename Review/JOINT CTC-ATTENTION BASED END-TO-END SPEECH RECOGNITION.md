## JOINT CTC-ATTENTION BASED END-TO-END SPEECH RECOGNITION 
## USING MULTI-TASK LEARNING
https://arxiv.org/pdf/1609.06773.pdf

### Abstract

- CTC와 attention을 합친 새로운 end-to-end 음성인식 방법을 제안한다. 

### 1. INTRODUCTION
---
End-to-End 음성인식 방법은 두 가지로 나눌 수 있다.  
첫 번째는 Connectionist Temporal Classification (CTC),  
두 번째는 Attention-based encoder-decoder 방법 이다.  

두 번째 방법으로 실시간 음성인식을 할 때, noise 때문에 alignment를 잘못 예측하기도 하고  
input sequence가 길 때, alignment를 잘못 예측하여 학습이 어려울 때도 있다. 
이러한 misalignment 문제를 해결하기 위해 joint CTC-attention model을 제안한다.
핵심은 CTC와 attention model이 encoder부분을 공유하고 있는 것이다.
forward-backward 알고리즘을 사용하는 CTC loss를 통해 alignment 문제를 해결하여   
성능을 높일 수 있다고 본 논문은 말하고 있다.

### 2. JOINT CTC-ATTENTION MECHANISM
---
#### 2.1 Connectionist temporal classification (CTC)
CTC의 핵심 아이디어는 blank와 반복되는 label을 통해 가능한 
모든 label sequence 조합을 구하는 것이다.
![a](https://user-images.githubusercontent.com/54731898/104768730-78a77380-57b1-11eb-94ee-be13ae75fef6.PNG)
P(y|x)가 최대가 되도록 모델을 학습한다.
더 자세한 내용은 [링크](https://github.com/hasangchun/Paper-Review/blob/main/Review/Connectionist%20Temporal%20Classification.pdf)를 참고하시면 좋을 것 같습니다.  


#### 2.2 Attention-based encoder-decoder
![b](https://user-images.githubusercontent.com/54731898/104768735-7a713700-57b1-11eb-9a2e-d4ab72d7257a.PNG)
전반적인 구조는 Encoder와 Attention-Decoder형태로,  
2개의 RNN으로 구성되어 있다.  

![c](https://user-images.githubusercontent.com/54731898/104768737-7ba26400-57b1-11eb-9d2a-a652243a2c62.PNG)
다음과 같은 과정을 통해 attention mechanism을 구현하고, decoder에서 output을 뽑아낼 때 사용한다.
본 논문에서는 location-based 방법이라고 적혀 있지만, location-aware 이라고 더 많이 불린다.
이 방법이 음성인식 attention mechanism으로 많이 사용된다.
더 자세한 내용은 [링크](https://github.com/hasangchun/Paper-Review/blob/main/Review/Attention-Based%20Models%20for%20Speech%20Recognition.md)를 참고하시면 좋을 것 같습니다.  

#### 2.3 Proposed model: Joint CTC-attention (MTL)
![d](https://user-images.githubusercontent.com/54731898/104768782-91b02480-57b1-11eb-9bbe-8f99e4e5e4db.PNG)
이 모델의 아이디어는 attention model encoder를 학습하는데 보조적으로 CTC loss 함수를 사용하는 것이다. 즉, 공유되는 encoder가 CTC와 attention model에 의해 학습이 되는 것이다.

CTC의 forward-backward 알고리즘이 단조로운 alignment를 강화함으로써 noisy한 상황에서 조금 더 robust 해진다고 말하고 있다. 또 다른 이점으로는 학습이 빠르게 진행된다는 것이다.

![e](https://user-images.githubusercontent.com/54731898/104768786-92e15180-57b1-11eb-9651-952dd8a8973f.PNG)
loss 함수는 CTC loss 값과 attention loss 값을 적절하게 더해주어서 사용한다.
λ는 하이퍼파라미터로 0 이상 1 이하의 값이다.

### EXPERIMENTS
---
![f](https://user-images.githubusercontent.com/54731898/104768794-94ab1500-57b1-11eb-8bb5-eb07f4910892.PNG)
본 논문 실험 결과에서는 λ값이 0.2 일 때 
CER(Character Error Rate)이 가장 낮은 것을 확인 할 수 있다.  

![g](https://user-images.githubusercontent.com/54731898/104768799-95dc4200-57b1-11eb-913f-68e62d1dfd6a.PNG)
CTC와 Attention 방법보다 MTL 방법이 빠르게 학습해나가는 것을 확인 할 수 있다.


