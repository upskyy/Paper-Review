# SpecAugment: A Simple Data Augmentation Method for Automatic Speech Recognition
https://arxiv.org/pdf/1904.08779.pdf

## Abstract
---
- 음성인식을 위한 data augmentation 방법을 제안합니다.
- time warping 하는 것, frequency의 일부분을 가리는 것, time의 일부분을 가리는 것으로 augment 합니다.
- Listen, Attend and Spell 구조에 적용하였습니다.

## 1. Introduction
---
딥러닝 모델들은 쉽게 오버피팅 되고, 많은 학습 데이터를 요구합니다.
그래서 학습 데이터를 더 많이 만들기 위해 Data augmentation 방법이 나온 것 입니다.
논문에서 제안하고 있는 specaugment는 오디오, 그 자체를 augmentation 하는 것이 아니라
오디오의 log mel spectrogram을 augmentation 하는 것 입니다.    
3가지 변형 방법으로 SpecAugment를 합니다.  
첫 번째는 time warping,    
두 번째는 시간의 일부분을 마스킹하는 것,    
세 번째는 주파수의 일부분을 마스킹하는 것 입니다.    


## 2. Augmentation Policy
---
![a](https://user-images.githubusercontent.com/54731898/105640015-fdd00e00-5ebe-11eb-865a-698b3e1bffa7.PNG)  
원본 log mel spectrogram, time warping, frequency masking, time masking 한 것을 위에서 부터
차례대로 나타낸 사진입니다.  
### Time Warping
가로축을 시간축, 세로축을 주파수축으로 했을 때 임의의 한 점을 잡고 왼쪽이나 오른쪽으로 warping 하는 것 입니다.
조금 쉽게 말하면, log mel spectrogram을 이미지로 생각하고 약간 찌그러트리는 것 입니다.

### Frequency Masking
위의 세 번째 사진처럼 주파수의 일부분을 마스킹 하는 것 입니다.

### Time Masking
마찬가지 방법으로, 시간의 일부분을 마스킹 하는 것 입니다.

![b](https://user-images.githubusercontent.com/54731898/105640017-00326800-5ebf-11eb-91f3-74bdde4e3c14.PNG)  
- LB : LibriSpeech basic
- LD : LibriSpeech double
- SM : Switchboard mild
- SS : Switchboard strong
 
데이터 셋과 마스킹 정도에 따라 다음과 같이 정의하고 있습니다. 
위의 사진은 결과로 나온 파라미터들을 정리한 사진 입니다.

![c](https://user-images.githubusercontent.com/54731898/105640019-00cafe80-5ebf-11eb-97de-a0558826cd23.PNG)  
위의 사진은 앞에서 말씀드린 마스킹 기법들을 섞어서 여러 개의 주파수 마스킹과 시간 마스킹을 적용한 LB, LD의 예시 입니다.  


## 3. Model
---
### 3.1 LAS Network Architectures
log mel spectrogram 인풋 데이터가 stride가 2인 max-pooling을 포함하여 2-layer의 CNN을 거쳐 양방향 LSTM encoder의 인풋으로 들어갑니다. 
그리고 encoder의 아웃풋을 어텐션 기반의 2-layer decoder에 넣어 예측 시퀀스를 뽑아냅니다. 

### 3.2 Learning Rate Schedules
learning rate schedule은 성능을 결정하는데 매우 중요한 요소 라고 말하고 있습니다. 
크게 ramp-up, hold, exponential decay 이렇게 3개의 구간으로 나눌 수 있습니다.  

첫 번째는 learning rate 0부터 특정 값까지 급격하게 증가하는 ramp-up 구간이고 time step으로는 [0, Sr] 입니다.  
두 번째에서 [Sr, Snoise]구간에서는 learning rate에 표준 deviation 0.075인 noise를 더해서 진행하고 [Snoise, Si]구간에서는 기존 learning rate를 hold해줍니다.  
세 번째는 learning rate 최대 값의 1/100까지 급격하고 떨어뜨리는 exponential decay 구간이고, time step으로는 [Si, Sf] 입니다.  

![d](https://user-images.githubusercontent.com/54731898/105640020-01fc2b80-5ebf-11eb-9b5f-2d003ec1b2bb.PNG)  
논문에서 사용한 3개의 learning rate schedule 파라미터를 정리한 사진입니다.

### Label Smoothing
label smoothing이란 hard vector를 soft vector로 바꾸어주는 것입니다. 예를들어, 올바른 label이 1 이였다면 0.9로 낮추어 조금 더 불확실하게 학습을 하는 것입니다.
데이터 정규화(regularization) 테크닉 중에 하나로, 간단한 방법이면서도 모델의 일반화 성능을 높여줍니다.
더 자세한 내용은 [When Does Label Smoothing Help?](https://arxiv.org/pdf/1906.02629.pdf) 논문을 참고하시면 좋겠습니다.


### 3.3 Shallow Fusion with Language Models
![f](https://user-images.githubusercontent.com/54731898/105640034-20fabd80-5ebf-11eb-85a5-0bf4428cb13b.PNG)  
Augmentation으로도 State-Of-The-Art 결과를 얻었지만 성능을 높이기 위해 augmentation 뿐만 아니라, Language Model도 함께 사용합니다.
LAS 모델의 output과 language model의 output을 적절하게 고려해줍니다.  
본 논문에서는 λ 값을 0.35로 진행하였습니다.  

## 4. Experiments
---
![g](https://user-images.githubusercontent.com/54731898/105640036-22c48100-5ebf-11eb-946d-4cf1220416a5.PNG)  

![h](https://user-images.githubusercontent.com/54731898/105640037-23f5ae00-5ebf-11eb-8833-1f1ed279b8cf.PNG)  
SpecAugment를 적용한 모델이 압도적인 성능을 보여주었습니다.


## 5. Discussion
---
### Time warping contributes, but is not a major factor in improving performance.
time warping, time masking, frequency masking 중에서 time warping은 계산이 많은 것에 비해 성능 개선에는 영향력이 별로 없습니다. 따라서 모든 augmentation 방법을 할 수 없다면 time warping을 배제하는 것이 좋습니다. 
### Label smoothing introduces instability to training.
Label smoothing과 augmentation을 함께 사용했을 때 눈에 띄는 결과를 보여주었다고 합니다.

### Augmentation converts an over-fitting problem into an under-fitting problem.
![i](https://user-images.githubusercontent.com/54731898/105640038-2526db00-5ebf-11eb-89df-80f4e4707531.PNG)  
Augmentation을 가장 많이 적용한 LD를 보시면 Training set에서 오버피팅이 일어나지 않았고, Validation set에서는 WER이 가장 낮은 것을 확인할 수 있습니다. 즉, 여러가지 augmentation 방법을 통해 데이터셋을 늘려서 오버피팅 문제를 언더피팅 문제로 바꾸었습니다.   

### Common methods of addressing under-fitting yield improvements.
언더피팅을 해결하기 위해서는 더 깊고 더 넓은 모델로 긴 learning rate schedule과 함께 학습을 오래시키면 됩니다.

## 6. Conclusions
SpecAugment는 음성인식에서 오버피팅 문제를 언더피팅 문제로 바꾸었고, 
큰 네트워크와 긴 학습 시간으로 좋은 결과를 얻었다.


